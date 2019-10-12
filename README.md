# cargo-audit-tags

A command line tool to run `cargo audit` on git tags.

This program can be run periodically on a Rust project to make sure
that tagged releases have no know security vulnerabilities in their
dependencies.

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
