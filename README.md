# rust-zdebuginfo

A demonstration of using compressed debug info with Rust. The linker is
instructed to compress the debug info via the flags in `.cargo/config`, which
Cargo reads automatically.

The contained binary prints a symbolicated backtrace and exits. By default there
are no symbols because none of the backtrace crate's symbolication features
have been activated:

```
benesch@marmoset$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/rust-zdebuginfo`
   0: <unknown>
   1: <unknown>
   2: <unknown>
   3: <unknown>
   4: <unknown>
   5: <unknown>
   6: <unknown>
   7: <unknown>
   8: <unknown>
```

With the `libbacktrace` feature activated, symbolication works perfectly:

```
benesch@marmoset$ cargo run --features=libbacktrace
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/rust-zdebuginfo`
   0: rust_zdebuginfo::main
             at src/main.rs:2
   1: std::rt::lang_start::{{closure}}
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/rt.rs:67
   2: std::rt::lang_start_internal::{{closure}}
             at src/libstd/rt.rs:52
      std::panicking::try::do_call
             at src/libstd/panicking.rs:305
   3: __rust_maybe_catch_panic
             at src/libpanic_unwind/lib.rs:86
   4: std::panicking::try
             at src/libstd/panicking.rs:281
      std::panic::catch_unwind
             at src/libstd/panic.rs:394
      std::rt::lang_start_internal
             at src/libstd/rt.rs:51
   5: std::rt::lang_start
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/rt.rs:67
   6: main
   7: __libc_start_main
   8: _start
```

With the `gimli` feature activated, symbolication almost entirely fails:

```
benesch@marmoset$ cargo run --features=gimli
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/rust-zdebuginfo`
   0: <unknown>
   1: <unknown>
   2: <unknown>
   3: <unknown>
   4: <unknown>
   5: <unknown>
   6: <unknown>
   7: __libc_start_main
   8: <unknown>
```

The `gimli` feature can be made to work by disabling compressed debug info.
(Setting `RUSTFLAGS` to the empty string overrides the flags in `.cargo/config.)

```
$ RUSTFLAGS= cargo run --features=gimli
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/rust-zdebuginfo`
   0: rust_zdebuginfo::main
             at src/main.rs:2
   1: std::rt::lang_start::{{closure}}
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/rt.rs:67
   2: std::rt::lang_start_internal::{{closure}}
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/rt.rs:52
      std::panicking::try::do_call
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/panicking.rs:305
   3: __rust_maybe_catch_panic
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libpanic_unwind/lib.rs:86
   4: std::panicking::try
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/panicking.rs:281
      std::panic::catch_unwind
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/panic.rs:394
      std::rt::lang_start_internal
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/rt.rs:51
   5: std::rt::lang_start
             at /rustc/b8cedc00407a4c56a3bda1ed605c6fc166655447/src/libstd/rt.rs:67
   6: main
   7: __libc_start_main
   8: _start
```
