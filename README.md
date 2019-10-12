# cargo-audit-tags

A command line tool to run `cargo audit` on git tags.

This program can be run periodically on a Rust project to make sure
that tagged releases have no know security vulnerabilities in their
dependencies.

## Installation

`cargo-audit-tags` is a self-contained Python script, requiring only a
Python 3 installation; it has been tested on Python 3.7, but older
versions of Python 3 *might* work as well. To install it, simply put
the script somewhere in your PATH, and make sure it has executable
permissions.

## Why Python (and not Rust)?

I originally submitted an [feature
request](https://github.com/RustSec/cargo-audit/issues/142) to
`cargo-audit` for a subset of the functionality that
`cargo-audit-tags` provides. After realizing that the functionality
can be implemented as an external tool, I closed the issue and went
ahead an implemented it the way I felt I would get to a working tool
the quickest; hence Python. If the `cargo-audit` maintainers decide
that this functionality makes sense as part of `cargo-audit` itself, I
might be tempted to re-implement it in Rust -- but for now, this is a
Python program.

The development of the initial version took a good afternoon; doing it
in Rust would have taken me probably a bit longer, even if not
much. However, now that the functionality is "done", re-implementing
it in Rust should be quite straightforward, should someone feel the
need. Maybe the functionality is even desired as part of `cargo-audit`
itself -- I'll keep an eye on the aforementioned issue.

## Description

The set of release tags can be given via `--tags=PATTERN`; the pattern
will be passed to `git tag -l`, by default all tags starting with "v"
will be considered release tags.

It is possible to check all tags matching the pattern, by using the
`--patches=all` command-line option, but the default is to only check
the newest tag in a release series.

The release series and patch components are extracted from a git tag
based on the `--series=REGEX` option; the supplied regular expression
must contain two capturing groups, which correspond to the series and
patch part. The supplied defaults (see `--help` output) should work
for many projects, assuming release tags have the format `vX.Y.Z`,
where `X` and `Y` are integers; a release series is then defined to be
all tags with equal `X` and `Y`, while `Z` indicates the patch
version, and is not restricted by the default `--series` regex. To
determine ordering of both release series and patch components, an
algorithm based on the [Debian package version comparison algorithm]
is used, which should work well enough for reasonable versioning
schemes.

As older release series are likely not maintained indefinitely, it is
possible to limit the checks to only the most recent `N` release
series by using the `--limit=N` option.

So, for a crate using the common `vX.Y.Z` tagging scheme, and where
the last two releases are maintained regarding security fixes, run:

```sh
cargo-audit-tags --limit=2
```

To check all release tags, use:

```sh
cargo-audit-tags --patches=all
```

Should you have a `Cargo.lock` file that is not located at the git
repository root, as might be the case for multi-language projects, you
can specify one or more lockfile locations as arguments, like this:

```sh
cargo-audit-tags rust/Cargo.lock
```

## Example output

Running `cargo-audit-tags` on the [`bat` crate] generates this output:

```
% cargo-audit-tags --limit 3
Running audit on v0.10.0 (newest patch 0 in series 0.10)...
    Fetching advisory database from `https://github.com/RustSec/advisory-db.git`
      Loaded 58 security advisories (from /home/rotty/.cargo/advisory-db)
    Scanning - for vulnerabilities (136 crate dependencies)
error: Vulnerable crates found!

[ ... details elided ... ]
Running audit on v0.11.0 (newest patch 0 in series 0.11)...
    Fetching advisory database from `https://github.com/RustSec/advisory-db.git`
      Loaded 58 security advisories (from /home/rotty/.cargo/advisory-db)
    Scanning - for vulnerabilities (144 crate dependencies)
error: Vulnerable crates found!

[ ... details elided ... ]
Running audit on v0.12.1 (newest patch 1 in series 0.12)...
    Fetching advisory database from `https://github.com/RustSec/advisory-db.git`
      Loaded 58 security advisories (from /home/rotty/.cargo/advisory-db)
    Scanning - for vulnerabilities (141 crate dependencies)
     Success No vulnerable packages found
Vulnerabilities found in v0.10.0, v0.11.0
% echo $?
1
```

So the most recent series, 0.12.x has no known vulnerabilities, while
the previous two (0.11.x and 0.12.x) do contain vulnerabilities in
their respective latest releases, which happen to be the only ones in
these series, so this is not an ideal example.

## License

The code and documentation in the `lexpr-rs` git repository is [Free
Software], dual-licensed under the [MIT](./LICENSE-MIT) or
[Apache-2.0](./LICENSE-APACHE) license, at your choosing.

[Free Software]: https://www.gnu.org/philosophy/free-sw.html
[Debian package version comparison algorithm]: https://www.debian.org/doc/debian-policy/ch-controlfields.html
[`bat` crate]: https://crates.io/crates/bat
