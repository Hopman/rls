[![Build Status](https://travis-ci.org/rust-lang-nursery/rls.svg?branch=master)](https://travis-ci.org/rust-lang-nursery/rls) [![Build status](https://ci.appveyor.com/api/projects/status/cxfejvsqnnc1oygs?svg=true)](https://ci.appveyor.com/project/jonathandturner/rls-x6grn)



# Rust Language Server (RLS)

[impl period](https://blog.rust-lang.org/2017/09/18/impl-future-for-rust.html) has been started! Join us at [Gitter.im](https://gitter.im/rust-impl-period/WG-dev-tools-rls).

**This project is in the alpha stage of development. It is likely to be buggy in
some situations; proceed with caution.**

The RLS provides a server that runs in the background, providing IDEs,
editors, and other tools with information about Rust programs. It supports
functionality such as 'goto definition', symbol search, reformatting, and code
completion, and enables renaming and refactorings.

The RLS gets its source data from the compiler and from
[Racer](https://github.com/phildawes/racer). Where possible it uses data from
the compiler which is precise and complete. Where its not possible, (for example
for code completion and where building is too slow), it uses Racer.

Since the Rust compiler does not yet support end-to-end incremental compilation,
we can't offer a perfect experience. However, by optimising our use of the
compiler and falling back to Racer, we can offer a pretty good experience for
small to medium sized crates. As the RLS and compiler evolve, we'll offer a
better experience for larger and larger crates.

The RLS is designed to be frontend-independent. We hope it will be widely
adopted by different editors and IDEs. To seed development, we provide a
[reference implementation of an RLS frontend](https://github.com/rust-lang-nursery/rls-vscode)
for [Visual Studio Code](https://code.visualstudio.com/).


## Setup

### Step 1: Install rustup

You can install [rustup](http://rustup.rs/) on many platforms. This will help us quickly install the
rls and its dependencies.

If you already have rustup installed, be sure to run self update to ensure you have the latest rustup:

```
rustup self update
```

If you're going to use the VSCode extension, you can skip steps 2 and 3.

### Step 2: Update stable

Update the stable compiler. If you require features from nightly you can also update nightly too:

```
rustup update
```

### Step 3: Install the RLS

Once you have rustup installed, run the following commands:

```
rustup component add rls-preview
rustup component add rust-analysis
rustup component add rust-src
```

#### Note (nightly only)
Sometimes the `rls-preview` component is not included in a nightly build due to certain issues. To see if the component is included in a particular build and what to do if it's not, check [#641](https://github.com/rust-lang-nursery/rls/issues/641).

## Running

Though the RLS is built to work with many IDEs and editors, we currently use
VSCode to test the RLS.

To run with VSCode, you'll need a 
[recent VSCode version](https://code.visualstudio.com/download) installed.

Next, you'll need to run the VSCode extension (for this step, you'll need a
recent [node](https://nodejs.org/en/) installed:

```
git clone https://github.com/rust-lang-nursery/rls-vscode
cd rls-vscode
npm install
code .
```

VSCode will open into the `rls-vscode` project.  From here, click the Debug
button on the left-hand side (a bug with a line through it). Next, click the
green triangle at the top.  This will launch a new instance of VSCode with the
`rls-vscode` plugin enabled. VSCode setting `"window.openFoldersInNewWindow"`
cannot be set to `"on"`. From there, you can open your Rust projects using
the RLS.

You'll know it's working when you see this in the status bar at the bottom, with
a spinning indicator:

`RLS analysis: working /`

Once you see:

`RLS analysis: done`

Then you have the full set of capabilities available to you.  You can goto def,
find all refs, rename, goto type, etc.  Completions are also available using the
heuristics that Racer provides.  As you type, your code will be checked and
error squiggles will be reported when errors occur.  You can hover these
squiggles to see the text of the error.

## Configuration

The RLS can be configured on a per-project basis, using the official Visual
Studio Code extension this will be done via the workspace settings file
`settings.json`.

Other editors will have their own way of sending the
[workspace/DidChangeConfiguration](https://github.com/Microsoft/language-server-protocol/blob/master/protocol.md#workspace_didChangeConfiguration)
method.

Entries in this file will affect how the RLS operates and how it builds your
project.

Currently we accept the following options:

* `build_lib` (`bool`, defaults to `false`) checks the project as if you passed
  the `--lib` argument to cargo. Mutually exclusive with, and preferred over
  `build_bin`.
* `build_bin` (`String`, defaults to `""`) checks the project as if you passed
  `-- bin <build_bin>` argument to cargo. Mutually exclusive with `build_lib`.
* `cfg_test` (`bool`, defaults to `true`) checks the project as if you were
  running `cargo test` rather than `cargo build`. I.e., compiles (but does not
  run) test code.
* `unstable_features` (`bool`, defaults to `false`) enables unstable features.
  Currently, this includes range formatting and `workspace_mode`,
  `analyze_package` options.
* `sysroot` (`String`, defaults to `""`) if the given string is not empty, use
  the given path as the sysroot for all rustc invocations instead of trying to
  detect the sysroot automatically
* `target` (`String`, defaults to `""`) if the given string is not empty, use
  the given target triple for all rustc invocations
* `wait_to_build` (`u64`, defaults to `500`) time in milliseconds between
  receiving a change notification and starting build
* `workspace_mode` (`bool`, defaults to `false`) Experimental mode, requires
  `unstable_features` turned on. When turned on, RLS will try to scan current
  workspace and analyze every package in it.
* `analyze_package` (`String`, defaults to `""`) When `workspace_mode` is
  enabled, analysis will be only provided for the specified package (runs as
  if `-p <analyze_package>` was passed).

## Troubleshooting

For tips on debugging and troubleshooting, see [debugging.md](debugging.md).

## Contributing

You can look in the [contributing.md](https://github.com/rust-lang-nursery/rls/blob/master/contributing.md)
in this repo to learn more about contributing to this project.

If you want to implement RLS support in an editor, see [clients.md](clients.md).
