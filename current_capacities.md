---
order: 7
---

# Current capacities

## Types

* Primitives - `bytes`, `bstr`, `tstr`, `text`, `uint`, `nint`
* Fixed values - `null`, `nil`, `true`, `false`
* Array values - `[uint]`
* Table types as members - `foo = ( x: { * a => b } )`
* Inline groups at root level - `foo = ( a: uint, b: uint)`
* Array groups - `foo = [uint, tstr, 0, bytes]`
* Map groups (both struct-type and table-type) - `foo = { a: uint, b: tstr }` or `bar = { * uint => tstr }`
* Embedding groups in other groups - `foo = (0, bstr) bar = [uint, foo, foo]`
* Group choices - `foo = [ 0, uint // 1, tstr, uint // tstr }`
* Tagged major types - `rational =  #6.30([ numerator : uint, denominator : uint])`
* Optional fields - `foo = { ? 0 : bytes }`
* Type aliases - `foo = bar`
* Type choices - `foo = uint / tstr`
* Serialization for all supported types.
* Deserialization for almost all supported types (see limitations section).
* CDDL Generics - `foo<T> = [T]`, `bar = foo<uint>`
* Length bounds - `foo = bytes .size (0..32)`
* cbor in bytes - `foo_bytes = bytes .cbor foo`
* Support for the CDDL standard prelude (using raw CDDL from the RFC) - `biguint`, etc
* default values - `? key : uint .default 0`

We generate getters for all fields, and setters for optional fields. Mandatory fields are set via the generated constructor. All wasm-facing functions are set to take references for non-primitives and clone when needed. Returns are also cloned. This helps make usage from wasm more memory safe.

Identifiers and fields are also changed to rust style. ie `foo_bar = { Field-Name: text }` gets converted into `struct FooBar { field_name: String }`

## Group choices

Group choices are handled as an enum with each choice being a variant. This enum is then wrapped around a wasm-exposed struct as `wasm_bindgen` does not support rust enums with members/values.
Group choices that have only a single non-fixed-value field use just that field as the enum variant, otherwise we create a `GroupN` for the `Nth` variant enum with the fields of that group choice. Any fixed values are resolved purely in serialization code, so `0, "hello", uint` puts the `uint` in the enum variant directly instead of creating a new struct.
## Type choices

Type choices are handled via enums as well with the name defaulting to `AOrBOrC` for `A / B / C` when inlined as a field/etc, and will take on the type identifier if provided ie `foo = A / B / C` would be `Foo`.
Any field that is `T / null` is transformed as a special case into `Option<T>` rather than creating a `TOrNull` enum.

A special case for this is when all types are fixed values e.g. `foo = 0 / 1 / "hello"`, in which case we generate a special c-style enum in the rust. This will have wasm_bindgen tags so it can be directly used in the wasm crate. Encoding variables (for `--preserve-encodings=true`) are stored where the enum is used like with other primitives.
