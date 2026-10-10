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
- point rendering, density rendering, and screen-space Voronoi rendering;
- many coloring modes based on coefficient, word, permutation, partition, or descriptor properties;
- Voronoi layers with independently controllable sites, cell fills, and walls;
- configurable complex-plane axis orientation;
- optional axes, ticks, unit circle, path display, and input trail;
- pan, zoom, fit, and locked-camera workflows;
- output transforms such as `F(z)`, `1/F(z)`, `log F(z)`, and `F(z)/|F(z)|`;
- bookmarks, comparison pinning, and PNG export;
- a small mathematical expression language for paths, custom bases, recurrences, and matrix entries;
- built-in path presets, including several Lissajous curves and smoothly looping in-out spirals;
- transformation/matrix presets aimed at visually rich Möbius and matrix-word experiments.


## Voronoi visualization

Voronoi mode treats the currently rendered complex outputs as sites and partitions the visible output plane according to the nearest rendered site. It is intended as a geometric reinterpretation of the same finite family rather than as a separate generator. Sampling therefore matters: a Voronoi diagram made from a subset is the diagram of that subset.

The mode can independently show or hide the sites, cell fills, and cell walls. Cell colors can inherit the ordinary point-color mapping, use deterministic site-identity colors, use output geometry such as angle or radius, or use a proximity-aware palette that attempts to give neighboring cells visibly different colors. Wall rendering has separate thickness, opacity, and color strategies, including fixed colors, alternating colors, automatic contrast, and colors derived from the adjoining cells.

The cell field is produced in screen space so it follows the current camera and is suitable for animation. The general family evaluator still determines the site locations; the Voronoi rasterization is a downstream visualization step. PNG export preserves the current Voronoi raster; custom export dimensions scale that raster rather than recomputing a new Voronoi field at the export resolution.

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

Points mode uses **batched WebGL2 drawing** across the general evaluators. Möbius compositions, rational functions, and linear combinations have cached double-precision evaluation:

- **Möbius words:** decode the selected words once and build a prefix tree. Each frame evaluates the matrix alphabet once and each distinct word prefix once, sharing intermediate results across descendants. This supports constant and `z`-dependent matrix entries, adjacency constraints, exact/up-to lengths, and sampled families with BigInt identifiers. An intermediate pole still invalidates the word and its descendants.
- **Independent P/Q pairs:** decode each distinct polynomial once, store shared references, and evaluate each used polynomial once per frame. The default 15,876 pairs contain just 126 distinct polynomials. Triangular powers and other basis values are shared across those evaluations. Ordinary powers retain Horner evaluation.
- **Other rational and linear families:** cache coefficient arrays and share basis values across the selected family. Other evaluators also benefit from cached point colors and GPU drawing.
- **Initial preparation:** compute only the statistics required by the selected coloring. Independent P/Q families share decoded coefficient arrays across all pairs, avoiding repeated polynomial decoding during selection preparation.
- **Stable frames:** retain the family, selected-word structure, colors, and typed work buffers across animation frames. Camera and appearance changes reuse evaluated values when `z` is unchanged. Mathematical settings rebuild the relevant caches; changing a preview or fixed-subset budget immediately updates the selection.
- **GPU resources:** upload a packed buffer of visible point positions and colors, then draw it in one batch per chunk. Density and Voronoi render targets are allocated only when needed.

`Auto: GPU drawing / fast evaluation` also permits the existing f32 GPU evaluator for supported ordinary polynomial families. `f64 evaluation + GPU drawing` keeps mathematical evaluation in JavaScript double precision while still using the GPU to draw Points. In the general Points path, output transforms and camera projection happen in double precision before screen coordinates are converted to f32. Colors and rasterization use GPU precision. Circular GPU point edges can differ slightly from Canvas 2D rasterization. If WebGL2 is unavailable, cached evaluation still works and Points fall back to Canvas 2D; unusually large points exceeding the GPU's supported size also use that fallback.

For smooth animation, use **Fixed Sample Subset** or **Realtime Preview**, then increase the corresponding point budget. The HUD reports main-thread frame work in milliseconds for directly rendered animation frames; it measures JavaScript preparation, any CPU evaluation and projection, and GPU draw submission. GPU evaluation and rasterization run asynchronously, so this timing does not measure GPU completion or displayed FPS. At a 30 FPS target the whole frame has about 33 ms, so leave room for browser drawing and compositing. The first frame after changing a family, subset, or coloring may take longer while caches are built. Very large initial selections can briefly block the UI; retained caches also trade memory for speed. Sampling remains important because family sizes grow exponentially.

Complete-frame Points animation uses the same direct rendering and playback timer as Fixed Sample Subset, with a retained selection containing every family member in sequential order. Each frame evaluates, projects, and submits the entire selection before updating the HUD. Evaluation plans share prefixes and distinct polynomials across the whole selection. As frame work grows beyond the target frame interval, the refresh rate decreases; the HUD keeps its completed count, progress bar, and main-thread frame-work timing between frames. The first frame prepares the selection and its plans, and preparation time and retained memory grow with the family size.

**Complete Each Frame evaluates the entire family**, independent of the preview and fixed-subset budgets. Its Points HUD reports the full count and main-thread frame work using the same format as the sampled modes. Mathematical settings rebuild the selection and evaluation plans. Density and Voronoi retain their existing complete-render paths. The app remains a self-contained HTML file: no server, worker files, or external dependencies are required.

Density rendering keeps its existing evaluation path. Voronoi mode evaluates sites on the CPU and performs its screen-space nearest-site stage on the GPU when WebGL2 floating-point render targets are available.


### Hover settling and drag release

In Points mode, stopping a hover or releasing the input continues the seeded preview toward the selected **Render amount**. If the preview already contains that many members, its existing raster and evaluation caches are kept. If the normal render requests more members, only the missing members are appended in one direct batch, without clearing the preview or waiting for a display refresh between small batches. **Render amount → All** can still take additional time when the full family is larger than the interaction budget.

The remaining selection retains its IDs and evaluation plan between gestures for all target sizes. A smaller normal target uses a cached shorter prefix. Completion cancels any pending preview callback so it cannot replace the final render. As with direct animation frames, initial preparation can block input while a large selection's plan is built; retained plans trade memory for faster subsequent renders. Optional automatic fitting uses already evaluated Points values and redraws them at the fitted camera.

To keep exactly the moving subset after a hover, select **Hover behavior → Fixed subset only**. Density and Voronoi use their existing normal-render completion path.

### Validation and measured frame costs

In headless Chromium at a 950×800 output size with software WebGL, the optimized Points path produced these measurements. Warm figures are medians of seven changing-input frames and include CPU evaluation, projection, clearing, HUD updates, and draw submission; they exclude the input overlay and are not displayed-FPS measurements. Hardware GPU results and browser scheduling will differ.

| Family | Members | First frame, including preparation | Warm main-thread frame work |
| --- | ---: | ---: | ---: |
| Default triangular-basis rational pairs | 15,876 | 127 ms | 2.8 ms |
| Four-symbol z-dependent Möbius words, up to D=7 | 21,845 | 114 ms | 6.6 ms |
| Same, up to D=8 | 87,381 | 437 ms | 11.6 ms |
| Same, up to D=9 | 349,525 | 1,677 ms | 43.0 ms |

Numerical regression checks compared 15,972 outputs across 54 configurations with the original descriptor evaluator, with no observed numerical differences. Coverage included all matrix presets, constrained words, exact and up-to lengths, the empty word, intermediate poles, BigInt samples, and rational functions from six generators over five bases. Hover-settle regression checks verified zero draw/clear/decode calls when the preview already met the final target (87,381 points; about 0.4 ms completion), append-only completion without duplicate IDs, cached completion on repeated gestures, cancellation by new movement, smaller targets, and fitting without decoding. Browser checks also covered Points/density/Voronoi switching, ordinary polynomial GPU evaluation, matrix eigenvalue pairs, budget changes, cache invalidation, Canvas fallback, and composite snapshots.

Complete-frame regression checks additionally verified mode switching from an exhaustive preview without decoding, exact once-only coverage of all 1,398,101 members of the four-symbol Möbius family up to D=10, persistent batch identities, cancellation into a 200-member fixed subset, and pause/resume with no decoding. In the same headless software-WebGL setup, this large complete family took about 5.5 seconds to prepare initially and 0.2 seconds per warmed complete frame (about 171 ms active CPU work). Switching from an exhaustive 21,845-member preview took about 3 ms. These timings illustrate preparation versus reuse, not a hardware FPS guarantee.

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
