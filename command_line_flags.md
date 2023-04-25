---
order: 8
---


# Command line flags

:::info `--input`
Specifies the input CDDL file(s).

For a single file:
```bash
cddl-codegen --input examples/test.cddl --output export
```


If a directory is specified e.g. `--input=some_dir` then it will read all files in this directory (non-recursively). 
The output format changes here. If there's a `lib.cddl` the types contained there are the standard [output](output_format.mdx) , and any other file e.g. `foo.cddl` will have its own module `foo/mod.rs` with its own `foo/serialization.rs`, etc.

```bash
cddl-codegen --input examples --output export
```
:::

<br/><br/>


:::info `--output`
Specifies the output directory.

```bash
cddl-codegen --input examples --output export
```


:::

<br/><br/>


:::info `--lib-name`
Specify the rust crate name for the output library. The wasm crate will have `-wasm` appended.

  ```bash
  cddl-codegen --input=example --output=export --lib-name some-crate-name
  ```
:::

<br/><br/>


:::info `--to-from-bytes-methods`
Generates `to_cbor_bytes()` / `from_cbor_bytes()` methods on all WASM objects. On by default.

(The rust code doesn't need this as you can directly use the `Serialize`/`Deserialize` traits on them.)  
      
Possible values: true, false
```bash
cddl-codegen --input=example --output=export --to-from-bytes-methods true
```
:::

<br/><br/>


:::info `--wasm`
Whether to output a wasm crate. On by default.  

Possible values: true, false
```bash
cddl-codegen --input=example --output=export --wasm false
```
:::

<br/><br/>

:::info `--preserve-encodings` 

Preserves CBOR encoding upon deserialization e.g. definite vs indefinite, map ordering. For each module this will also create a `cbor_encodings.rs` file to potentially store any structs for storing these encodings. This option is useful if you need to preserve the deserialized format for round-tripping (e.g. hashes) or if you want to modify the format to conincide with a specific tool for hashing.

Possible values: true, false
```bash
cddl-codegen --input=example --output=export --preserve-encodings true
```
:::

<br/><br/>


:::info `--canonical-form` 
Used primarily with `--preserve-encodings` to provide a way to override the specific deserialization format and to instead output canonical CBOR. This will have `Serialize`'s trait have an extra `to_canonical_cbor_bytes()` method. Likewise the wasm wrappers (with `--to-from-bytes-methods`) will contain one too.

Possible values: true, false
```bash
cddl-codegen --input=example --output=export --canonical-form true
```
:::

<br/><br/>

:::info `--json-serde-derives` 
Derives serde::Serialize/serde::Deserialize for types to allow to/from JSON

Possible values: true, false
```bash
cddl-codegen --input=example --output=export --json-serde-derives true
```
:::

<br/><br/>

:::info `--json-schema-export`
Tags types with sonSchema derives and generates a crate (in wasm/json-gen) to export them. This requires `--json-serde-derives`.

**Possible values:**  true, false<br></br>
**Default:** true

**Example:**
```bash
cddl-codegen --input=example --output=export --json-schema-export true
```
:::

<br/><br/>


:::info `--package-json`
Generates a npm package.json along with build scripts (some of these scripts require `--json-serde-derives`/`--json-schema-export` to work).

**Possible values:** true, false<br></br>
**Default:** false
```bash
cddl-codegen --input=example --output=export --package-json true --json-schema-export true
```
:::