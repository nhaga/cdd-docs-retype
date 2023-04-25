---
sidebar_position: 7
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


# Examples



## Aliases


Type alias
```
hash = bytes
```


==- See generated output
#### input 

```rust 
hash = bytes
```

#### output

```rust export/rust/src/lib.rs
pub type Hash = Vec<u8>;
```
===



Create a newtype around another type instead of an alias
```
special_hash = bytes ; @newtype
```

==- See generated output

#### input 

```rust
special_hash = bytes ; @newtype
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct SpecialHash(pub Vec<u8>);

impl SpecialHash {
    pub fn get(&self) -> &Vec<u8> {
        &self.0
    }

    pub fn new(inner: Vec<u8>) -> Self {
        Self(inner)
    }
}

impl From<Vec<u8>> for SpecialHash {
    fn from(inner: Vec<u8>) -> Self {
        SpecialHash::new(inner)
    }
}

impl From<SpecialHash> for Vec<u8> {
    fn from(wrapper: SpecialHash) -> Self {
        wrapper.0
    }
}
```

===





or don't generate either and directly use the aliased type instead
```
hidden_hash = bytes ; @no_alias

hashes = [
  hash,
  special_hash,
  hidden_hash,
]
```


==- See generated output

#### input 

```
hash = bytes
special_hash = bytes ; @newtype
hidden_hash = bytes ; @no_alias

hashes = [
  hash,
  special_hash,
  hidden_hash,
]
```



#### output

```rust export/rust/src/lib.rs
pub type Hash = Vec<u8>;

#[derive(Clone, Debug)]
pub struct Hashes {
    pub hash: Hash,
    pub special_hash: SpecialHash,
    pub hidden_hash: Vec<u8>,
}

impl Hashes {
    pub fn new(hash: Hash, special_hash: SpecialHash, hidden_hash: Vec<u8>) -> Self {
        Self {
            hash,
            special_hash,
            hidden_hash,
        }
    }
}

#[derive(Clone, Debug)]
pub struct SpecialHash(pub Vec<u8>);

impl SpecialHash {
    pub fn get(&self) -> &Vec<u8> {
        &self.0
    }

    pub fn new(inner: Vec<u8>) -> Self {
        Self(inner)
    }
}

impl From<Vec<u8>> for SpecialHash {
    fn from(inner: Vec<u8>) -> Self {
        SpecialHash::new(inner)
    }
}

impl From<SpecialHash> for Vec<u8> {
    fn from(wrapper: SpecialHash) -> Self {
        wrapper.0
    }
}
```

===

:::info
pay attention to the @name comment placement as it can be finicky
:::



## Size/length requirements on primitives

```
limitations = [
	u_8: uint .size 1,
	u_16: uint .le 65535,
	u_32: 0..4294967295,
	u_64: uint .size 8,
	i_8: -128..127,
	i_64: int .size 8,
    hash32: bytes .size 32,
    bounded: text .size (10..20),
]

```


Integer restrictions that map to rust types are directly translated

u8 in rust

```
u_8: uint .size 1
```

u16 in rust

```
u_16: uint .le 65535
```


u32, etc...

```
u_32: 0..4294967295
u_64: uint .size 8
i_8: -128..127
i_64: int .size 8

```

One can also limit strings (text or bytes) to a specific length

```
hash32: bytes .size 32
```

or to a range e.g. between 10 and 20 bytes
```
bounded: text .size (10..20)
```

==- See generated output

#### input 

```rust 
limitations = [
    u_8: uint .size 1,
    u_16: uint .le 65535,
    u_32: 0..4294967295,
    u_64: uint .size 8,
    i_8: -128..127,
    i_64: int .size 8,
    hash32: bytes .size 32,
    bounded: text .size (10..20),
]
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct Limitations {
    pub u_8: u8,
    pub u_16: u16,
    pub u_32: u32,
    pub u_64: u64,
    pub i_8: i8,
    pub i_64: i64,
    pub hash32: Vec<u8>,
    pub bounded: String,
}

impl Limitations {
    pub fn new(
        u_8: u8,
        u_16: u16,
        u_32: u32,
        u_64: u64,
        i_8: i8,
        i_64: i64,
        hash32: Vec<u8>,
        bounded: String,
    ) -> Self {
        Self {
            u_8,
            u_16,
            u_32,
            u_64,
            i_8,
            i_64,
            hash32,
            bounded,
        }
    }
}
```

===




## Array struct


```
foo = [
  int,
  name: text,
  fp: float64,
]

```

All primitives are supported, e.g. uint, nint and int supported. int generates special code as no rust equivalent. 
Unnamed array fields try to derive name from type if possible:

```
  int
```

Text/bytes is also supported, or one can give them an explicit name:

```
  name: text
```

As well as floats (without --preserve-encodings=true)

```
  fp: float64
```

==- See generated output

#### input 

```rust 
foo = [
  int,
  name: text,
  fp: float64,
]
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct Foo {
    pub index_0: Int,
    pub name: String,
    pub fp: f64,
}

impl Foo {
    pub fn new(index_0: Int, name: String, fp: f64) -> Self {
        Self { index_0, name, fp }
    }
}

#[derive(Clone, Debug)]
pub enum Int {
    Uint(u64),
    Nint(u64),
}

impl Int {
    pub fn new_uint(value: u64) -> Self {
        Self::Uint(value)
    }

    /// * `value` - Value as encoded in CBOR - note: a negative `x` here would be `|x + 1|` due to CBOR's `nint` encoding e.g. to represent -5, pass in 4.
    pub fn new_nint(value: u64) -> Self {
        Self::Nint(value)
    }
}

impl std::fmt::Display for Int {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            Self::Uint(x) => write!(f, "{}", x),
            Self::Nint(x) => write!(f, "-{}", x + 1),
        }
    }
}

impl std::str::FromStr for Int {
    type Err = IntError;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let x = i128::from_str(s).map_err(IntError::Parsing)?;
        Self::try_from(x).map_err(IntError::Bounds)
    }
}

impl TryFrom<i128> for Int {
    type Error = std::num::TryFromIntError;

    fn try_from(x: i128) -> Result<Self, Self::Error> {
        if x >= 0 {
            u64::try_from(x).map(Self::Uint)
        } else {
            u64::try_from((x + 1).abs()).map(Self::Nint)
        }
    }
}

#[derive(Clone, Debug)]
pub enum IntError {
    Bounds(std::num::TryFromIntError),
    Parsing(std::num::ParseIntError),
}
```

===


## Mark as externally defined. 

user has to insert/import code for this type after generation

```
extern_foo = _CDDL_CODEGEN_EXTERN_TYPE_
```

==- See generated output

#### input 

```rust 
foo = _CDDL_CODEGEN_EXTERN_TYPE_
bar = [
    x: uint,
    y: foo,
]
```

#### output

```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct Bar {
    pub x: u64,
    pub y: Foo,
}

impl Bar {
    pub fn new(x: u64, y: Foo) -> Self {
        Self { x, y }
    }
}
```

===


## Map struct 

Map struct + tagged fields + .cbor + optional fields + constants + .default


```
bar = {
	foo: #6.1337(foo),
  extern_foo: bytes .cbor extern_foo
	? derp: uint,
	1 : uint / null, ; @name explicitly_named_1
  ? 5: "five",
	five: 5,
  ? 100: uint .default 0,
}
```

Fields can be tagged and this remains a serialization detail (hidden from API)

```
  foo: #6.1337(foo),
```

They can also be encoded as CBOR bytes which remains a serialization detail (hidden from API). This can be combined with tags as well i.e. #6.42(bytes .cbor extern_foo)

```
  extern_foo: bytes .cbor extern_foo
```

Optional field (generates as `Option<T>`)

```
	? derp: uint,
```

Type choice with null will result in `Option<T>` too for the API. Also, you can give explicit names that differ from the key value for maps like this:
	
```
  1 : uint / null, ; @name explicitly_named_1
```

Optional string constant (no field generated)

```
  ? 5: "five",
```

Integer constant (no field generated)

```
	five: 5,
```

This will not be an optional field in rust, as when it is not present, it will be set to 0

```
  ? 100: uint .default 0,
```


==- See generated output

#### input 

```rust 
bar = {
    foo: #6.1337(foo),
    extern_foo: bytes .cbor extern_foo
    ? derp: uint,
    1 : uint / null, ; @name explicitly_named_1
    ? 5: "five",
    five: 5,
    ? 100: uint .default 0,
}
```

#### output

```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct Bar {
    pub foo: Foo,
    pub extern_foo: ExternFoo,
    pub derp: Option<u64>,
    pub explicitly_named_1: Option<u64>,
    pub key_100: u64,
}

impl Bar {
    pub fn new(foo: Foo, extern_foo: ExternFoo, explicitly_named_1: Option<u64>) -> Self {
        Self {
            foo,
            extern_foo,
            derp: None,
            explicitly_named_1,
            key_100: 0,
        }
    }
}
```

===


## Basic groups 


Basic groups are supported and have their own type

```
basic = (
  b: #6.23(uint),
  c: text,
)
```


Basic groups are fully supported in array groups

They can be put into an array struct directly i.e. embed their fields into outer, which is only a serialization detail. this field will be of type Basic;
or one can embed them into a repeatable homogeneous array


```
outer = [
  a: uint,
  embedded: basic,
  homogeneous_array: [* basic],
]

other_basic = (
  b: uint,
  c: uint,
)
```




Basic groups can be embeded in maps, BUT deserialization will not be generated due to technical limitations

:::warning limitation
A single basic group cannot be put into both a map and an array group for serialization which is
why we had to define a separate one `other_basic` instead of just using `basic`

:::


```
outer_map = {
  a: uint,
  embedded: other_basic,
}
```

==- See generated output

#### input 

```rust 
basic = (
  b: #6.23(uint),
  c: text,
)

outer = [
  a: uint,
  embedded: basic,
  homogeneous_array: [* basic],
]

other_basic = (
  b: uint,
  c: uint,
)

outer_map = {
  a: uint,
  embedded: other_basic,
}
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct Basic {
    pub b: u64,
    pub c: String,
}

impl Basic {
    pub fn new(b: u64, c: String) -> Self {
        Self { b, c }
    }
}

#[derive(Clone, Debug)]
pub struct OtherBasic {
    pub b: u64,
    pub c: u64,
}

impl OtherBasic {
    pub fn new(b: u64, c: u64) -> Self {
        Self { b, c }
    }
}

#[derive(Clone, Debug)]
pub struct Outer {
    pub a: u64,
    pub embedded: Basic,
    pub homogeneous_array: Vec<Basic>,
}

impl Outer {
    pub fn new(a: u64, embedded: Basic, homogeneous_array: Vec<Basic>) -> Self {
        Self {
            a,
            embedded,
            homogeneous_array,
        }
    }
}

#[derive(Clone, Debug)]
pub struct OuterMap {
    pub a: u64,
    pub embedded: OtherBasic,
}

impl OuterMap {
    pub fn new(a: u64, embedded: OtherBasic) -> Self {
        Self { a, embedded }
    }
}
```

===

One can directly define homogeneous maps as fields (or define them at top-level).
Also define homogenous arrays as fields (or define them at top-level)

```
table_arr_members = {
	tab: { * text => text },
	arr: [ * uint ],
}

type_choice = 
    0              ; @name you
  / "hello world"  ; @name can
  / uint           ; @name name
  / text           ; @name variants
  / bytes          ; @name like
  / #6.64([*uint]) ; @name this
```


==- See generated output

#### input 

```rust
table_arr_members = {
    tab: { * text => text },
	arr: [ * uint ],
}

type_choice = 
    0              ; @name you
  / "hello world"  ; @name can
  / uint           ; @name name
  / text           ; @name variants
  / bytes          ; @name like
  / #6.64([*uint]) ; @name this
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct TableArrMembers {
    pub tab: BTreeMap<String, String>,
    pub arr: Vec<u64>,
}

impl TableArrMembers {
    pub fn new(tab: BTreeMap<String, String>, arr: Vec<u64>) -> Self {
        Self { tab, arr }
    }
}

#[derive(Clone, Debug)]
pub enum TypeChoice {
    You,
    Can,
    Name(u64),
    Variants(String),
    Like(Vec<u8>),
    This(Vec<u64>),
}

impl TypeChoice {
    pub fn new_you() -> Self {
        Self::You
    }

    pub fn new_can() -> Self {
        Self::Can
    }

    pub fn new_name(name: u64) -> Self {
        Self::Name(name)
    }

    pub fn new_variants(variants: String) -> Self {
        Self::Variants(variants)
    }

    pub fn new_like(like: Vec<u8>) -> Self {
        Self::Like(like)
    }

    pub fn new_this(this: Vec<u64>) -> Self {
        Self::This(this)
    }
}
```

===

## Type Choices

If a type choice only has constants it will generate as a c-style enum (directly wasm-exposable)

```
c_style_enum =
    0 ; @name foo
  / 1 ; @name bar
  / 2 ; @name baz
```

==- See generated output

#### input 

```rust
c_style_enum =
    0 ; @name foo
  / 1 ; @name bar
  / 2 ; @name baz
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub enum CStyleEnum {
    Foo,
    Bar,
    Baz,
}

impl CStyleEnum {
    pub fn new_foo() -> Self {
        Self::Foo
    }

    pub fn new_bar() -> Self {
        Self::Bar
    }

    pub fn new_baz() -> Self {
        Self::Baz
    }
}

```

===

If there is only one non-constant field in the inlined group then that will be inlined in the enum, but if there are multiple then a new struct will be generated from this variant

```
group_choice = [
  foo //                  ; @name these
  0, x: uint //           ; @name are
  1, x: uint, y: text //  ; @name also
  basic                   ; @name nameable
]
choices = [
  type_choice,
  c_style_enum,
  group_choice,
]
```


==- See generated output

#### input 

```rust
foo = [
  int,
  name: text,
  fp: float64,
]

basic = (
  b: #6.23(uint),
  c: text,
)

type_choice = 
    0              ; @name you
  / "hello world"  ; @name can
  / uint           ; @name name
  / text           ; @name variants
  / bytes          ; @name like
  / #6.64([*uint]) ; @name this

c_style_enum =
    0 ; @name foo
  / 1 ; @name bar
  / 2 ; @name baz

group_choice = [
  foo //                  ; @name these
  0, x: uint //           ; @name are
  1, x: uint, y: text //  ; @name also
  basic                   ; @name nameable
]
choices = [
  type_choice,
  c_style_enum,
  group_choice,
]
```

#### output


```rust export/rust/src/lib.rs
#[derive(Clone, Debug)]
pub struct Are {
    pub x: u64,
    pub y: String,
}

impl Are {
    pub fn new(x: u64, y: String) -> Self {
        Self { x, y }
    }
}

#[derive(Clone, Debug)]
pub struct Basic {
    pub b: u64,
    pub c: String,
}

impl Basic {
    pub fn new(b: u64, c: String) -> Self {
        Self { b, c }
    }
}

#[derive(Clone, Debug)]
pub enum CStyleEnum {
    Foo,
    Bar,
    Baz,
}

impl CStyleEnum {
    pub fn new_foo() -> Self {
        Self::Foo
    }

    pub fn new_bar() -> Self {
        Self::Bar
    }

    pub fn new_baz() -> Self {
        Self::Baz
    }
}

#[derive(Clone, Debug)]
pub struct Choices {
    pub type_choice: TypeChoice,
    pub c_style_enum: CStyleEnum,
    pub group_choice: GroupChoice,
}

impl Choices {
    pub fn new(
        type_choice: TypeChoice,
        c_style_enum: CStyleEnum,
        group_choice: GroupChoice,
    ) -> Self {
        Self {
            type_choice,
            c_style_enum,
            group_choice,
        }
    }
}

#[derive(Clone, Debug)]
pub struct Foo {
    pub index_0: Int,
    pub name: String,
    pub fp: f64,
}

impl Foo {
    pub fn new(index_0: Int, name: String, fp: f64) -> Self {
        Self { index_0, name, fp }
    }
}

#[derive(Clone, Debug)]
pub enum GroupChoice {
    Foo(Foo),
    These(These),
    Are(Are),
    Basic(Basic),
}

impl GroupChoice {
    pub fn new_foo(index_0: Int, name: String, fp: f64) -> Self {
        Self::Foo(Foo::new(index_0, name, fp))
    }

    pub fn new_these(x: u64) -> Self {
        Self::These(These::new(x))
    }

    pub fn new_are(x: u64, y: String) -> Self {
        Self::Are(Are::new(x, y))
    }

    pub fn new_basic(b: u64, c: String) -> Self {
        Self::Basic(Basic::new(b, c))
    }
}

#[derive(Clone, Debug)]
pub enum Int {
    Uint(u64),
    Nint(u64),
}

impl Int {
    pub fn new_uint(value: u64) -> Self {
        Self::Uint(value)
    }

    /// * `value` - Value as encoded in CBOR - note: a negative `x` here would be `|x + 1|` due to CBOR's `nint` encoding e.g. to represent -5, pass in 4.
    pub fn new_nint(value: u64) -> Self {
        Self::Nint(value)
    }
}

impl std::fmt::Display for Int {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            Self::Uint(x) => write!(f, "{}", x),
            Self::Nint(x) => write!(f, "-{}", x + 1),
        }
    }
}

impl std::str::FromStr for Int {
    type Err = IntError;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let x = i128::from_str(s).map_err(IntError::Parsing)?;
        Self::try_from(x).map_err(IntError::Bounds)
    }
}

impl TryFrom<i128> for Int {
    type Error = std::num::TryFromIntError;

    fn try_from(x: i128) -> Result<Self, Self::Error> {
        if x >= 0 {
            u64::try_from(x).map(Self::Uint)
        } else {
            u64::try_from((x + 1).abs()).map(Self::Nint)
        }
    }
}

#[derive(Clone, Debug)]
pub enum IntError {
    Bounds(std::num::TryFromIntError),
    Parsing(std::num::ParseIntError),
}

#[derive(Clone, Debug)]
pub struct These {
    pub x: u64,
}

impl These {
    pub fn new(x: u64) -> Self {
        Self { x }
    }
}

#[derive(Clone, Debug)]
pub enum TypeChoice {
    You,
    Can,
    Name(u64),
    Variants(String),
    Like(Vec<u8>),
    This(Vec<u64>),
}

impl TypeChoice {
    pub fn new_you() -> Self {
        Self::You
    }

    pub fn new_can() -> Self {
        Self::Can
    }

    pub fn new_name(name: u64) -> Self {
        Self::Name(name)
    }

    pub fn new_variants(variants: String) -> Self {
        Self::Variants(variants)
    }

    pub fn new_like(like: Vec<u8>) -> Self {
        Self::Like(like)
    }

    pub fn new_this(this: Vec<u64>) -> Self {
        Self::This(this)
    }
}
```

===


