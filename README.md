# rsgen-avro &emsp; [![latest]][crates.io] [![doc]][docs.rs]

[latest]: https://img.shields.io/crates/v/rsgen-avro.svg
[crates.io]: https://crates.io/crates/rsgen-avro
[doc]: https://docs.rs/rsgen-avro/badge.svg
[docs.rs]: https://docs.rs/rsgen-avro

A command line tool and library for generating [serde][]-compatible Rust types from
[Avro schemas][schemas]. The [apache-avro][] crate, which is re-exported, provides a way to
read and write Avro data with such types.

## Command line usage

Download the latest [release](https://github.com/lerouxrgd/rsgen-avro/releases) or install with:

```sh
cargo install rsgen-avro
```

Available options:

```
Usage:
  rsgen-avro [options] <schema-files-glob-pattern> <output-file>
  rsgen-avro (-h | --help)
  rsgen-avro (-V | --version)

Options:
  --fmt              Run rustfmt on the resulting <output-file>
  --nullable         Replace null fields with their default value when deserializing.
  --precision=P      Precision for f32/f64 default values that aren't round numbers [default: 3].
  --union-deser      Custom deserialization for apache-avro multi-valued union types.
  --derive-builders  Derive builders for generated record structs.
  --derive-schemas   Derive AvroSchema for generated record structs.
  -V, --version      Show version.
  -h, --help         Show this screen.
```

## Library usage

As a libray, the basic usage is:

```rust
use rsgen_avro::{Source, Generator};

let raw_schema = r#"
{
    "type": "record",
    "name": "test",
    "fields": [
        {"name": "a", "type": "long", "default": 42},
        {"name": "b", "type": "string"}
    ]
}
"#;

let source = Source::SchemaStr(&raw_schema);

let mut out = std::io::stdout();

let g = Generator::new().unwrap();
g.gen(&source, &mut out).unwrap();
```

This will generate the following output:

```text
#[derive(Debug, PartialEq, Eq, Clone, serde::Deserialize, serde::Serialize)]
pub struct Test {
    #[serde(default = "default_test_a")]
    pub a: i64,
    pub b: String,
}

#[inline(always)]
fn default_test_a() -> i64 { 42 }
```

Various `Schema` sources can be used with `Generator`'s `.gen(..)` method:

```rust
pub enum Source<'a> {
    Schema(&'a rsgen_avro::Schema),    // Avro schema enum re-exported from `apache-avro`
    Schemas(&'a [rsgen_avro::Schema]), // A slice of Avro schema enums
    SchemaStr(&'a str),                // Schema as a json string
    GlobPattern(&'a str),              // Glob pattern to select schema files
}
```

Note also that the `Generator` can be customized with a builder:

```rust
let g = Generator::builder().precision(2).build().unwrap();
```

The `derive_builders` option will use the [derive-builder][] crate to derive builders for the generated structs.
The builders will only be derived for those structs that are generated from Avro records.

## Limitations

* Avro schema `namespace` fields are ignored, therefore names from a single schema must
  not conflict.

[schemas]: https://avro.apache.org/docs/current/spec.html
[apache-avro]: https://github.com/apache/avro/tree/master/lang/rust
[serde]: https://serde.rs
[derive-builder]: https://github.com/colin-kiegel/rust-derive-builder
