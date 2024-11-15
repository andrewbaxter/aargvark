<table align="right" margin="1em"><tr>
<td><a href="https://crates.io/crates/aargvark"><img alt="crates.io" src="https://img.shields.io/crates/v/aargvark"></a></td>
<td><a href="https://docs.rs/aargvark"><img alt="docs.rs" src="https://img.shields.io/docsrs/aargvark"></td></a>
</tr></table>

A simple and consistent derive-based command line argument parsing, in the same genre as Clap-derive. It currently supports

- Command line parsing
- Help

Generally speaking this is intended to make CLI parsing simple by virtue of being simple and consistent, rather than poweruser-optimized keypress-minimizing parsing.

This attempts to support parsing arbitrarily complex command line arguments. Like with Serde, you can combine structs, vecs, enums in any way you want. Just because you can doesn't mean you should.

```
$ ; This is an example help output, sans light ansi styling
$ spagh -h
Usage: spagh COMMAND [ ...FLAGS]

    A small CLI for querying, publishing, and administrating spaghettinuum.

    COMMAND: COMMAND
    [--debug]

COMMAND: ping | get | http | ssh | identity | publish | admin

    ping ...      Simple liveness check
    get ...       Request values associated with provided identity and keys
                  from a resolver
    http ...
    ssh ...
    identity ...  Commands for managing identities
    publish ...   Commands for publishing data
    admin ...     Commands for node administration

```

```
$ spagh publish set -h
Usage: spagh publish set IDENTITY DATA

    IDENTITY: IDENTITY-SECRET-ARG  Identity to publish as
    DATA: <PATH> | -               Data to publish.  Must be json in the
                                   structure `{KEY: {"ttl": MINUTES, "value":
                                   DATA}, ...}`. `KEY` is a string that's a
                                   dotted list of key segments, with `/` to
                                   escape dots and escape characters.

IDENTITY-SECRET-ARG: local

    An identity with its associated secret.

    local <PATH>  A file containing a generated key

```

# Why or why not

Why this and not Clap?

- It has a super-simple interface (just `#[derive(Aargvark)]` on any enum/structure)
- This parses more complex data types, like vectors of sub-structures, or enums
- It's more consistent

Why not this?

- It's newer, with fewer features and limited community ecosystem, extensions
- Some command line parsing conventions were discarded in order to simplify and maintain self-similarity. A lot of command line conventions are inconsistent or break down as you nest things, after all.
- Quirky CLI parsing generally isn't supported: Some tricks (like `-v` `-vv` `-vvv`) break patterns and probably won't ever be implemented. (Other things just haven't been implemented yet due to lack of time)

# Conventions and usage

To add it to your project, run

```sh
cargo add aargvark
```

To parse command line arguments

1. Define the data type you want to parse them into, like

   ```rust
   /// General description for the command.
   #[derive(Aargvark)]
   struct MyArgs {
     /// Field documentation.
     velociraptor: String,
     #[vark(flag = "-d", flag = "--deadly")]
     deadly: bool,
     color_pattern: Option<ColorPattern>,
   }
   ```

2. Vark it
   ```rust
   let args = vark::<MyArgs>();
   ```

Non-optional fields become positional arguments unless you give them a flag with `#[vark(flag = "--flag")]`. Optional fields become optional (`--long`) arguments. If you want a `bool` flag that's enabled if the flag is specified (i.e. doesn't take a value), use `Option<()>`.

You can derive structs, enums, and tuples, and there are implementations for `Vec`, `HashSet`, `Map` with `FromString` keys and values as `K=V` arguments, most `Ip` and `SocketAddr` types, and `PathBuf` built in.

Some additional wrappers are provided for automatically loading (and parsing) files:

- `AargvarkFile<T>`
- `AargvarkJson<T>` requires feature `serde_json`
- `AargvarkYaml<T>` requires feature `serde_yaml`

To parse your own types, implement `AargvarkTrait`, or if your type takes a single string argument you can implement `AargvarkFromStr` which is slightly simpler.

# Advanced usage

- Sequences, plural fields, Vecs

  Sequence elements are space separated. The way sequence parsing works is it attempts to parse as many elements as possible. When parsing one element fails, it rewinds to after it parsed the last successful element and proceeds from the next field after the sequence.

- Use flags, replace flags, and add additional flags

  Add ex: `#[vark(flag="--target-machine", flag="-tm")]` to a _field_.

  If the field was optional, this will replace the default flag. If the field was non-optional, this will make it require a flag instead of being positional.

- Rename enum variant keys

  Add ex: `#[vark(name="my-variant")]` to a _variant_.

  This changes the command line key used to select a variant.

- Prevent recursion in help

  Add `#[vark(break_help)]` to a _type_, _field_, or _variant_ to prevent recursing into any of the children when displaying help. This is useful for subcommand enums - attach this to the enum and it will list the variants but not the variants' arguments (unless you do `-h` after specifying one on the command line).

- Change the help placeholder string

  Add `#[vark(placeholder="TARGET-MACHINE")]` to a _type_, _field_, or _variant_.

  This is the capitalized text (like XYZ) after an option that basically means "see XYZ section for more details"
