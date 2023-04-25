---
order: 5
---


# Wasm Differences

In the wasm crate we can't always go one to one with the rust crate. Here are some differences/extra types in the WASM create. `AsRef` `From` and `Into` are implemented to go between the rust and wasm crate types to help.

## Heterogeneous Arrays

`wasm_bindgen` cannot expose doubly-nested types like `Vec<Vec<T>` which can be a limitation if `T` was a non-byte primtive.
Any array of non-primitives such as `[foo]` will generate another type called `FooList` which supports all basic array operations.
This lets us get around the `wasm_bindgen` limitation (without implementing cross-boundary traits which could be inefficient/tedious/complicated).
This array wrapper implements `len() -> self`, `get(usize) -> T` and `add(T)`.

## Tables

Map literals also generate a type for them with `len() -> usize` and `insert(K, V) -> Option<V>`. The table type will have a `MapKeyToValue` name for whichever `Key` and `Value` types it's exposed as if it's anonymously inlined a as a member, or will take on the identifier if it's a named one.

## Enums

Both type/group choices generate rust-style enums. On the wasm side we can't do that so we directly wrap the rust type, and then provide a `FooKind` c-style enum for each rust enum `Foo` just for checking which variant it is.
