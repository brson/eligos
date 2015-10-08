# eligos
Services framework for Rust!

Codecs serialize requests and deserialize responses.  Workers are threads that run their own MIO event loop.  Receivers are the code that workers run.

```
extern crate bytes;
extern crate eligos;

use self::bytes::{Buf, ByteBuf};
use self::eligos::{Service, Codec, Receive};

struct CountCodec;

impl Codec<ByteBuf, usize> for CountCodec {
    fn decode(&mut self, buf: &mut ByteBuf) -> Vec<usize> {
        println!("got some bytes in codec!");
        vec![buf.bytes().len()]
    }

    fn encode(&self, us: usize) -> ByteBuf {
        let byte_str = format!("{}", us);
        let bytes = byte_str.as_bytes();
        ByteBuf::from_slice(bytes)
    }
}

fn build_codec() -> Box<Codec<ByteBuf, usize>> {
    Box::new(CountCodec)
}

struct ByteAddReceiver {
    counter: usize,
}

// take requests of usize, return how much we have so far
impl Receive<usize, usize> for ByteAddReceiver {
    fn receive(&self, req_bytes: &usize) -> Option<usize> {
        self.counter += *req_bytes;
        println!("in handler for srv! bytes so far: {}", self.counter);
        Some(count)
    }
}

fn main() {
    let receiver = Box::new(ByteAddReceiver {
        counter: 0,
    });

    let mut service = Service::new(
        6666,
        build_codec, // request codec
        build_codec, // response codec
    ).unwrap();

    let n_workers = 4;
    service.run(vec![receiver]);
}
```