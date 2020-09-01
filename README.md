**(WORK IN PROGRESS)**

# Crepe

Crepe is a library that allows you to write declarative logic programs in
Rust, with a [Datalog](https://en.wikipedia.org/wiki/Datalog)-like syntax.
It provides a procedural macro that generates efficient, safe code and
interoperates seamlessly with Rust programs.

## Example

The program below computes the transitive closure of a directed graph. Note
the use of the `crepe!` macro.

```rust
use crepe::crepe;

crepe! {
    @input
    struct Edge(i32, i32);

    @output
    struct Reachable(i32, i32);

    Reachable(x, y) <- Edge(x, y);
    Reachable(x, z) <- Edge(x, y), Reachable(y, z);
}

fn main() {
    let mut runtime = Crepe::new();
    runtime.extend(&[Edge(1, 2), Edge(2, 3), Edge(3, 4), Edge(2, 5)]);

    let (reachable,) = runtime.run();
    for Reachable(x, y) in reachable {
        println!("node {} can reach node {}", x, y);
    }
}
```

You can do much more with Crepe. The next example shows how you can use
stratified negation, Rust expression syntax, and semi-naive evaluation to find
all paths in a weighted graph with length at most `MAX_PATH_LEN`.

```rust
use crepe::crepe;

const MAX_PATH_LEN: u32 = 20;

crepe! {
    @input
    struct Edge(i32, i32, u32);

    @output
    struct Walk(i32, i32, u32);

    @output
    struct NoWalk(i32, i32);

    struct Node(i32);

    Node(x) <- Edge(x, _, _);
    Node(x) <- Edge(_, x, _);

    Walk(x, x, 0) <- Node(x);
    Walk(x, z, len1 + len2) <-
        Edge(x, y, len1),
        Walk(y, z, len2),
        (len1 + len2 <= MAX_PATH_LEN);

    NoWalk(x, y) <- Node(x), Node(y), !Walk(x, y, _);
}
```

## Features

- Semi-naive evaluation
- Stratified negation
- Automatic generation of indices for relations
- Arbitrary Rust expression syntax allowed in rules
- Typesafe way to initialize `@input` relations
- Very fast, compiled directly with the rest of your Rust code

## Acknowledgements

This work was heavily inspired by [Souffle](https://souffle-lang.github.io/)
and [Formulog](https://github.com/HarvardPL/formulog), which both use similar
models of Datalog compilation for static analysis.
