---
order: 9
---

# Installation & Getting Started


## Install


```bash

git clone https://github.com/dcSpark/cddl-codegen && cd cddl-codegen
```


## Run Example


To run execute `cargo run -- --input=input.cddl --output=EXPORT_DIR` to read `./input.cddl` and produce output code in `./EXPORT_DIR/`.


```bash
cargo run -- --input=example/test.cddl --output=export
```


## Build

```bash
cargo build --release
target/release/cddl-codegen --input example/test.cddl --output export
```
