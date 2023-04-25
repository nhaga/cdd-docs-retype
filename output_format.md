---
order: 6
---

# Output format

- Inside of the output directly the tool always produces a `rust/` directory (including Cargo.toml, etc). 
- Unless we pass in `--wasm=false` the tool also generates a corresponding `wasm/` directory.
- The default format for `rust/` is to have a `lib.rs` containing the structs and `serialization.rs` containing their (de)serialization implementations/corresponding types.
- The `wasm/` directory is full of wasm_bindgen-annotated wrappers all in `lib.rs` for the corresponding rust-use-only structs in `rust/` and can be compiled for WASM builds by running `wasm-pack build` on it.

**Example Output**

![](/img/output.png)

:::note 
The output format can change slightly depending on certain command line flags:
:::

`--wasm=false`

![](/img/output2.png)


`--preserve-encodings=true`

![](/img/output3.png)


`--json-schema-export true`

![](/img/output4.png)


`--package-json true --json-schema-export true`

![](/img/output5.png)
