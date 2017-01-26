# The FastCGI Rust implementation.

[![Build Status](https://travis-ci.org/Gedweb/rust-gfcgi.svg)](https://travis-ci.org/Gedweb/rust-gfcgi) [![docs.rs](https://docs.rs/gfcgi/badge.svg)](https://docs.rs/gfcgi) [![Cargo](https://img.shields.io/crates/v/gfcgi.svg)](https://crates.io/crates/gfcgi) [![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

#### Description
*gfcgi* a native Rust library for FastCGI.  
Library is supporting multithreaded socket listener and HTTP-instances multiplexed onto a single connection.

#### About FastCGI
FastCGI it's great solutions to handling HTTP-requests without overhead. Completely supporting HTTP or HTTPS by any popular web-servers. 

[Specification](doc/fcgi-spec.md) 

#### Example
Import the library within your code.
```rust
    extern crate gfcgi;
    
    use std::io::{Read, Write}; 
    use std::thread;
```
Some your router struct
```rust
    #[derive(Clone)]
    struct Router;
        
    impl Router
    {
        fn new() -> Self
        { 
            Router{}
        }
    }
```
Implement [`gfcgi::Handler`](https://docs.rs/gfcgi/0.4.3/gfcgi/trait.Handler.html) trait for your router, all code in `process` method is optional
```rust
    impl gfcgi::Handler for Router
    {
        fn process(&self, fcgi: &mut gfcgi::HttpPair)
        {
            // get a header
            let h = fcgi.request().header_utf8(b"HTTP_X_TEST");
            println!("{:?}", h);
    
            // read content
            let mut buf = Vec::new();
            fcgi.request().read_to_end(&mut buf).unwrap();
            println!("{:?}", String::from_utf8(buf));
    
            // set header
            fcgi.response().header_utf8("Content-type", "text/plain");
    
            // send content
            fcgi.response().write(b"hello world!").expect("send body");
        }
    }
```
Now run [`listener`](https://docs.rs/gfcgi/0.4.3/gfcgi/struct.Client.html), you can spawn thread if set `spawn` feature in `Cargo.toml`
```rust
    fn main() 
    {
        let client = gfcgi::Client::new("127.0.0.1:4128");
    
        // run listener
        client.run(Router::new());
    
        if cfg!(feature = "spawn") {
            client.run(Router::new()); // spawn one more
            thread::park(); // keep main process
        }
    }
```
#### Planned
- [x] Role
  - [x] responder
  - [ ] filter
  - [ ] authorizer
- [x] Header
  - [ ] get_values
  - [ ] get_values_result
  - [x] unknown_type
  - [x] begin_request
  - [x] abort_request
  - [x] end_request
  - [x] params
  - [x] stdin
  - [ ] data
  - [x] stdout
  - [ ] stderr

#### Trace
    socket
        stream
            connection
            handler (pair)
                request
                | → read headers
                | → [read body]
                response
                | ← write headers
                | ← [write body]