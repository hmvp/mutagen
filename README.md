# Breaking your Rust code for fun & profit

[![Build Status](https://travis-ci.org/llogiq/mutagen.svg?branch=master)](https://travis-ci.org/llogiq/mutagen)
[![Downloads](https://img.shields.io/crates/d/mutagen.svg?style=flat-square)](https://crates.io/crates/mutagen/)
[![Version](https://img.shields.io/crates/v/mutagen.svg?style=flat-square)](https://crates.io/crates/mutagen/)
[![License](https://img.shields.io/crates/l/mutagen.svg?style=flat-square)](https://crates.io/crates/mutagen/)

This is a work in progress mutation testing framework. Not all components are there, those that are there aren't finished, but you can see the broad direction it's going to take.

### Mutation Testing

The idea behind mutation testing is to insert changes into your code to see if they make your tests fail. If not, your tests obviously fail to test the changed code.
The difference to line or branch coverage is that those measure if the code under test was *executed*, but that says nothing about whether the tests would have caught any error.

This repo has two components at the moment: A helper library and a procedural macro that mutates your code.

### How mutagen Works

Mutagen works as a procedural macro. This means two things:

1. You'll need a nightly rust toolchain to compile the plugin.
2. it only gets to see the code you mark up with the `#[mutate]` annotation, nothing more.

It also will only see the bare AST, no inferred types, no control flow or data flow, unless we analyse them ourselves. But not only that, we want to be *fast*.  This means we want to avoid doing one compile run per mutation, so we try to bake in all mutations into the code once and select them at runtime via a mutation count. This means we must avoid mutations that break the code so it no longer compiles.

This project is basically an experiment to see what mutations we can still apply under those constraints.

### Running mutagen

Again, remember you need a nightly `rustc` to compile the plugin. Add the plugin and helper library as a dev-dependency to your `Cargo.toml`:

```
[dev-dependencies]
mutagen = "0.1.0"
mutagen-plugin = "0.1.0"
```

Now, you can add the plugin to your crate by prepending the following:

```
#![cfg_attr(test, feature(plugin))]
#![cfg_attr(test, plugin(mutagen_plugin))]
#![feature(custom_attribute)]

#[cfg(test)]
extern crate mutagen;
```

Now you can advise mutagen to mutate any function, method, impl, trait impl or whole module (but *not* the whole crate, this is a restriction of procedural macros for now) by prepending:

```
#[cfg_attr(test, mutate)]
```

This ensures the mutation will only be active in test mode. Now you can run `cargo test` as always, which will mutate your code and write a list of mutations in `target/mutagen/mutations.txt`. For every mutation, counting from one, you can run the test binary with the environment variable `MUTATION_COUNT=1 target/debug/myproj-123456`, `MUTATION_COUNT=2 ..`, etc.

To automate this install the runner. Run `cargo install` in the runner dir. Compile the test for the project under test (`cargo +nightly test --no-run`). Run `cargo mutagen`.

### Contributing

issues and PRs welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) on how to help.
