# Finite Function Family Explorer

## Purpose

Finite Function Family Explorer is a single-file browser app for visually exploring **finite families of complex-valued functions**. Its core question is:

> If a finite rule system generates many related functions or transformations, what geometric structure appears when all of them are evaluated at the same complex input?

The app is intended primarily for mathematical curiosity, visual discovery, generative-art experimentation, and rapid comparison of related finite constructions. It is not designed as a proof assistant or as a numerically certified scientific-computing package.

## Core model

The app separates two ideas:

1. **Generator** — the finite set of mathematical objects to enumerate.
2. **Evaluation rule** — how each generated object, together with the complex input `z`, produces one or more complex outputs.

The resulting outputs are plotted as a point cloud in the complex plane. Moving `z` lets the user watch the entire family deform.

This organization is deliberately broader than a polynomial explorer. Ordinary restricted-coefficient polynomials are one case, but the same interface also supports sparse families, constrained integer coefficients, permutations, integer partitions, rational pairs, recurrences, Möbius maps, and matrix products.

## Family generators

The current app includes:

- coefficient words over an explicit complex alphabet;
- sparse coefficient families;
- constrained nonnegative or signed integer coefficient families;
- distinct permutations of a coefficient multiset;
- integer partitions encoded through coefficient multiplicities;
- independent polynomial pairs `(P, Q)`;
- words over a finite alphabet of 2×2 transformations or matrices;
- optional adjacency constraints on coefficient or transformation words.

Several generators have exact combinatorial counting and can be sampled deterministically without first materializing the entire family.

## Evaluation rules

Generated objects can be interpreted through several kinds of complex-valued constructions, including:

- generalized linear combinations / polynomials;
- product families;
- rational functions `P/Q`;
- finite continued fractions;
- nested radicals;
- quadratic compositions;
- a user-defined recurrence;
- Möbius-map compositions;
- observations of matrix products, such as trace, determinant, entries, eigenvalues, or spectral radius.

For generalized polynomial-like families, the basis can also vary. Built-in examples include ordinary powers, square/cubic/triangular exponents, Fibonacci-like and prime exponents, Chebyshev and Legendre polynomials, Fourier-style exponentials, sine terms, and a custom basis expression.

## Interaction and visualization

The app is built for interactive exploration rather than one-off static plotting. Major capabilities include:

- combined, split, and output-only views;
- draggable or hover-controlled complex input;
- path animation and degree sweeps;
- realtime-preview, complete-frame, and fixed-subset animation modes;
- deterministic seeded sampling;
- point rendering and density rendering;
- many coloring modes based on coefficient, word, permutation, partition, or descriptor properties;
- configurable complex-plane axis orientation;
- optional axes, ticks, unit circle, path display, and input trail;
- pan, zoom, fit, and locked-camera workflows;
- output transforms such as `F(z)`, `1/F(z)`, `log F(z)`, and `F(z)/|F(z)|`;
- bookmarks, comparison pinning, and PNG export;
- a small mathematical expression language for paths, custom bases, recurrences, and matrix entries.

## Default example

The shipped default is a visually interesting rational-function family:

- independent polynomial pairs `(P, Q)`;
- coefficients in `{-1, +1}`;
- degrees up to 5;
- triangular-number exponents `z^(k(k+1)/2)`;
- output `P(z)/Q(z)`;
- word-symbol-change coloring;
- Lissajous 3:4 input-path animation;
- a 15,876-member fixed subset, which in this default happens to be the entire family.

The path itself is hidden by default while the input trail remains visible.

## Precision and performance

The app has two broad execution paths:

- a GPU-accelerated path for the unrestricted ordinary polynomial case where the browser can support it;
- a general CPU double-precision path for the broader family/evaluator combinations.

The UI reports the active backend. Sampling and fixed-subset modes are provided because many finite families grow exponentially or combinatorially, so exhaustive rendering can quickly become impractical even when individual function evaluations are cheap.

## Expression language

Custom expressions use complex arithmetic and familiar mathematical names such as `sin`, `cos`, `exp`, `log`, `sqrt`, `abs`, `arg`, and `conj`. The language recognizes constants such as `i`, `pi` / `π`, `tau` / `τ`, and `e`, plus context-specific variables such as `z`, `t`, `k`, `N`, `w`, `a`, and `x`.

The parser is intentionally limited to mathematical expressions rather than arbitrary JavaScript.

## Scope and philosophy

The app is meant to make it cheap to ask questions such as:

- What changes when the coefficient set changes?
- What changes when the same combinatorial family is evaluated in a different basis?
- Which restrictions create qualitatively new geometry rather than another familiar affine-IFS pattern?
- How does a cloud deform as the complex input moves along a path?
- Which combinatorial statistics make useful visual color coordinates?
- Which constructions produce visually distinctive families worth studying more deeply?

The emphasis is on **exploration of families**, especially combinations of discrete combinatorial rules with complex-valued evaluation.

## Running the app

Open the HTML file directly in a modern desktop browser. The app is self-contained and does not require a server or external JavaScript dependencies.
