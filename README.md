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

## License

The code and documentation in the `lexpr-rs` git repository is [Free
Software], dual-licensed under the [MIT](./LICENSE-MIT) or
[Apache-2.0](./LICENSE-APACHE) license, at your choosing.

[Free Software]: https://www.gnu.org/philosophy/free-sw.html
[Debian package version comparison algorithm]: https://www.debian.org/doc/debian-policy/ch-controlfields.html
