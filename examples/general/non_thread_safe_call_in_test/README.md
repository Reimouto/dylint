# non_thread_safe_call_in_test

### What it does

Checks for calls to non-thread-safe functions in code attributed with
`#[test]`. For this lint to be effective, `--tests` must be passed to `cargo check`.

### Why is this bad?

"When you run multiple tests, by default they run in parallel using
threads" ([reference]). Calling a non-thread-safe function in one test could affect the
outcome of another.

### Known problems

- Synchronization is not considered, so false positives could result.
- Tries to flag uses of `std::process::Command::new("cargo").arg("run")`, but does not track
  values. So false negatives will result if the `Command::new("cargo")` is not
  `Command::arg("run")`'s receiver.

### Example

```rust
#[test]
fn set_var() {
    std::env::set_var("KEY", "SOME_VALUE");
    std::process::Command::new("env").status().unwrap();
}
```

Use instead:

```rust
#[test]
fn set_var() {
    std::process::Command::new("env")
        .env("KEY", "SOME_VALUE")
        .status()
        .unwrap();
}
```

[reference]: https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-tests-in-parallel-or-consecutively
