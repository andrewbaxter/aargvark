Self-similar derive-based command line argument parsing, in the same genre as Clap-derive. It supports

- Command line parsing
- Help

This attempts to support parsing arbitrarily complex command line arguments. Like with Serde, you can combine structs, vecs, enums in any way you want. Just because you can doesn't mean you should.

```
$ echo This is an example help output, sans light ansi styling
$ ./target/debug/spagh-cli publish -h
Usage: ./target/debug/spagh-cli publish PUBLISH

Create or replace existing publish data for an identity on a publisher server


PUBLISH: SERVER IDENTITY DATA

 SERVER: <URI>                          URL of a server with publishing set up
 IDENTITY: IDENTITY                     Identity to publish as
 DATA: <PATH>|-                         Data to publish.  Must be json in the structure `{KEY: {"ttl": SECONDS, "value": "DATA"}, ...}`

IDENTITY: local | card

 local <PATH>|-
 card PCSC-ID PIN
```

# Why or why not

Why this and not Clap?

- This parses more complex data types, like vectors of sub-structures, or enums
- It's more consistent
- It has a super-simple interface (just `#[derive(Aargvark)]`)

Why not this?

- Some command line parsing conventions were discarded in order to simplify and maintain self-similarity. A lot of command line conventions are inconsistent or break down as you nest things, after all.
- There's less customizability. Some tricks (like `-v` `-vv` `-vvv`) break patterns and probably won't ever be implemented. Other things just haven't been implemented yet due to lack of time.
- Alpha

# Conventions and usage

To add it to your project, run

```sh
cargo add aargvark
```

To parse command line arguments

1. Define the data type you want to parse them into, like

   ```rust
   #[derive(Aargvark)]
   struct MyArgs {
     velociraptor: String,
     deadly: bool,
     color_pattern: Option<ColorPattern>,
   }
   ```

2. Vark it
   ```
   let args = aargvark::vark::<MyArgs>();
   ```

Optional fields in structs become optional (`--long`) arguments. If you want a `bool` long option that's enabled if the flag is specified (i.e. doesn't take a value), use `Option<()>`.

You can derive structs, enums, and tuples, and there are implementations for `Vec`, `HashSet`, most `Ip` and `SocketAddr` types, and `PathBuf` provided.

Some additional wrappers are provided for automatically loading (and parsing) files:

- `AargvarkFile<T>`
- `AargvarkJson<T>` requires feature `serde_json`
- `AargvarkYaml<T>` requires feature `serde_yaml`

To parse your own types, implement `AargvarkTrait`, or if your type takes a single string argument you can implement `AargvarkFromStr`.
