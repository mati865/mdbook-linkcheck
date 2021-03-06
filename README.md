# MDBook LinkCheck

[![Build Status](https://travis-ci.org/Michael-F-Bryan/mdbook-linkcheck.svg?branch=master)](https://travis-ci.org/Michael-F-Bryan/mdbook-linkcheck)
[![Build status](https://ci.appveyor.com/api/projects/status/5ysqtugw3205omc1?svg=true)](https://ci.appveyor.com/project/Michael-F-Bryan/mdbook-linkcheck)
[![Crates.io](https://img.shields.io/crates/v/mdbook-linkcheck.svg)](https://crates.io/crates/mdbook-linkcheck)
[![Docs.rs](https://docs.rs/mdbook-linkcheck/badge.svg)](https://docs.rs/mdbook-linkcheck/)
[![license](https://img.shields.io/github/license/michael-f-bryan/mdbook-linkcheck.svg)](https://github.com/Michael-F-Bryan/mdbook-linkcheck/blob/master/LICENSE)

A backend for `mdbook` which will check your links for you. For use alongside
the built-in HTML renderer.

## Getting Started

First you'll need to install `mdbook-linkcheck`.

```
cargo install mdbook-linkcheck
```

If you don't want to install from source (which often takes a while) you can
grab an executable from [GitHub Releases][releases] or use this line of
`curl`:

```console
curl -LSfs https://japaric.github.io/trust/install.sh | \
    sh -s -- --git Michael-F-Bryan/mdbook-linkcheck
```

Next you'll need to update your `book.toml` to let `mdbook` know it needs to
use the `mdbook-linkcheck` backend.

```toml
[book]
title = "My Awesome Book"
authors = ["Michael-F-Bryan"]

[output.html]

[output.linkcheck]
```

And finally you should be able to run `mdbook build` like normal and everything
should *Just Work*.

```
$ mdbook build
```

> **Note:** When multiple `[output]` items are specified, `mdbook` tries to
> ensure that each `[output]` gets its own sub-directory within the `build-dir`
> (`book/` by default).
>
> That means if you go from only having the HTML renderer enabled to enabling
> both HTML and the linkchecker, your HTML will be placed in `book/html/`
> instead of just `book/` like before.

## Configuration

The link checker's behaviour can be configured by setting options under the
`output.linkcheck` table in your `book.toml`.

```toml
...

[output.linkcheck]
# Should we check links on the internet? Enabling this option adds a
# non-negligible performance impact
follow-web-links = false

# Are we allowed to link to files outside of the book's root directory? This
# may help prevent linking to sensitive files (e.g. "../../../../etc/shadow")
traverse-parent-directories = false

# If necessary, you can exclude one or more web links from being checked with
# a list of regular expressions
exclude = [ "google\\.com" ]

# The User-Agent to use when sending web requests
user-agent = "mdbook-linkcheck-0.4.0"

# The number of seconds a cached result is valid for (12 hrs by default)
cache-timeout = 43200

# How should warnings be treated?
#
# - "warn" will emit warning messages
# - "error" treats all warnings as errors, failing the linkcheck
# - "ignore" will ignore warnings, suppressing diagnostic messages and allowing
#   the linkcheck to continuing
warning-policy = "warn"

# Extra HTTP headers that must be send to certain web sites
# in order to link check to succeed
#
# This is a dictionary (map), with keys being regexes
# matching a set of web sites, and values being an array of
# the headers.
[http-headers]
# Any hyperlink that contains this regexp will be sent
# the "Accept: text/html" header
"crates\.io" = ["Accept: text/html"]

# mdbook-linkcheck will interpolate environment variables
# into your header via $IDENT.
#
# If this is not what you want
# you must escape the `$` symbol, like `\$TOKEN`. `\` itself can also be escaped
# via `\\`.
"website\.com" = ["Authorization: Basic $TOKEN"]
```

## Continuous Integration

Incorporating `mdbook-linkcheck` into your CI system should be straightforward
if you are already [using `mdbook` to generate documentation][mdbook-ci].

For those using GitLab's built-in CI:

```yaml
book:
  stage: build
  image : rust:latest
  variables:
    # makes sure the `~/.cargo` directory gets cached too
    CARGO_HOME: $CI_PROJECT_DIR/_cargo
  before_script:
    - rustc --version --verbose && cargo --version --verbose
    - export PATH=$CARGO_HOME/bin:$PATH
    # Install mdbook and mdbook-linkcheck without optimisations, if not
    # already installed
    - command -v mdbook >/dev/null 2>&1 || cargo install --debug mdbook
    - command -v mdbook-linkcheck >/dev/null 2>&1 || cargo install --debug mdbook-linkcheck
  script:
    - mdbook build
  artifacts:
    paths:
      - book

pages:
  image: busybox:latest
  stage: deploy
  dependencies:
    - book
  script:
    - mkdir -p public
    - cp -r book public
  artifacts:
    paths:
    - public
  only:
    - master
```

[@danieltrautmann][danieltrautmann] has also created [a docker image][docker]
that comes with `mdbook` and `mdbook-linkcheck` pre-installed.

[releases]: https://github.com/Michael-F-Bryan/mdbook-linkcheck/releases
[mdbook-ci]: https://rust-lang.github.io/mdBook/continuous-integration.html
[danieltrautmann]: https://github.com/danieltrautmann
[docker]: https://gitlab.com/danieltrautmann/docker-mdbook/container_registry
