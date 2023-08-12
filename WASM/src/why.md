# Why Rust and WebAssembly?

JS is ran using a JIT compiler. This has advantages as well as disadvantages    
### Advantages
- fast startup time, an application can be fetched in packets.
- optimization according to real-time runtime conditions
- Many others...

### Disadvantages
- unreliable performance : "Just-In-Time (JIT) compilers heavily rely on assumptions about the code's behavior to perform optimizations. These assumptions form what's commonly referred to as the "happy path." When the code deviates from this happy path, it can lead to performance regressions and unexpected behavior." -- I do not know the details that back this statement up


## Wasm modules are small
Code size is incredibly important since the .wasm must be downloaded over the network.  
- stack based binary is smaller than register based binary
- Using Rust to wasm results in binaries that have functions only. No Garbage collection or other bloat

Rust - wasm integrates well with other standard technologies 

