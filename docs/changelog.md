# Changelog

## v0.6.1 (unreleased)

**Breaking changes**

- The number of workers initialized (for non-distributed pools) is now set to, in order of preference, 1. the environment variable `COTENGRA_NUM_WORKERS`, 2. the environment variable `OMP_NUM_THREADS`, or 3. `os.cpu_count()`.

**Enhancements**

- add [RandomGreedyOptimizer](cotengra.pathfinders.path_basic.RandomGreedyOptimizer) which is a lightweight and performant randomized greedy optimizer, eschewing both hyper parameter tuning and full contraction tree construction, making it suitable for very large contractions (10,000s of tensors+).
- add [optimize_random_greedy_track_flops](cotengra.pathfinders.path_basic.optimize_random_greedy_track_flops) which runs N trials of (random) greedy path optimization, whilst computing the FLOP count simultaneously. This or its accelerated rust counterpart in `cotengrust` is the driver for the above optimizer.
- add `parallel="threads"` backend, and make it the default for `RandomGreedyOptimizer` when `cotengrust` is present, since its version of `optimize_random_greedy_track_flops` releases the GIL.
- significantly improve both the speed and memory usage of [`SliceFinder`](cotengra.slicer.SliceFinder)
- alias `tree.total_cost()` to `tree.combo_cost()`


## v0.6.0 (2024-04-10)

**Bug fixes**

- all input node legs and pre-processing steps are now calculated lazily,
  allowing slicing of indices including those 'simplified' away {issue}`31`.
- make [`tree.peak_size`](cotengra.ContractionTree.peak_size) more accurate,
  by taking max assuming left, right and parent intermediate tensors are all
  present at the same time.

**Enhancements**

- add simulated annealing tree refinement (in `path_simulated_annealing.py`),
  based on "Multi-Tensor Contraction for XEB Verification of
  Quantum Circuits" by Gleb Kalachev, Pavel Panteleev, Man-Hong Yung
  (arXiv:2108.05665), and the "treesa" implementation in
  OMEinsumContractionOrders.jl by Jin-Guo Liu and Pan Zhang. This can be
  accessed most easily by supplying
  `opt = HyperOptimizer(simulated_annealing_opts={})`.
- add [`ContractionTree.plot_flat`](cotengra.plot.plot_tree_flat): a new method
  for plotting the contraction tree as a flat diagram showing all indices on
  every intermediate (without requiring any graph layouts), which is useful for
  visualizing and understanding small contractions.
- [`HyperGraph.plot`](cotengra.plot.plot_hypergraph): support showing hyper
  outer indices, multi-edges, and automatic unique coloring of nodes and
  indices (to match `plot_flat`).
- add [`ContractionTree.plot_circuit](cotengra.plot.plot_tree_circuit) for
  plotting the contraction tree as a circuit diagram, which is fast and useful
  for visualizing the traversal ordering for larger trees.
- add [`ContractionTree.restore_ind`](cotengra.ContractionTree.restore_ind)
  for 'unslicing' or 'unprojecting' previously removed indices.
- [`ContractionTree.from_path`](cotengra.ContractionTree.from_path): add option
  `complete` to automatically complete the tree given an incomplete path
  (usually disconnected subgraphs - {issue}`29`).
- add [`ContractionTree.get_incomplete_nodes`](cotengra.ContractionTree.get_incomplete_nodes)
  for finding all uncontracted childless-parentless node groups.
- add [`ContractionTree.autocomplete`](cotengra.ContractionTree.autocomplete)
  for automatically completing a contraction tree, using above method.
- [`tree.plot_flat`](cotengra.plot.plot_tree_flat): show any preprocessing
  steps and optionally list sliced indices
- add [get_rng](cotengra.utils.get_rng) as a single entry point for getting or
  propagating a random number generator, to help determinism.
- set ``autojit="auto"`` for contractions, which by default turns on jit for
  `backend="jax"` only.
- add [`tree.describe`](cotengra.ContractionTree.describe) for a various levels
  of information about a tree, e.g. `tree.describe("full")` and
  `tree.describe("concise")`.
- add [ctg.GreedyOptimizer](cotengra.pathfinders.path_basic.GreedyOptimizer)
  and [ctg.OptimalOptimizer](cotengra.pathfinders.path_basic.OptimalOptimizer)
  to the top namespace.
- add [ContractionTree.benchmark](cotengra.ContractionTree.benchmark) for
  for automatically assessing hardware performance vs theoretical cost.
- contraction trees now have a `get_default_objective` method to return the
  objective function they were optimized with, for simpler further refinement
  or scoring, where it is now picked up automatically.
- change the default 'sub' optimizer on divisive partition building algorithms
  to be `'greedy'` rather than `'auto'`. This might make individual trials
  slightly worse but makes each cheaper, see discussion: ({issue}`27`).


## v0.5.6 (2023-12-07)

**Bug fixes**

- fix a very rare but very infuriating bug related somehow to
  [ReusableHyperOptimizer](cotengra.ReusableHyperOptimizer) not being
  thread-safe and returning the wrong tree on github actions


## v0.5.5 (2023-11-15)

**Enhancements**

- [`HyperOptimizer`](cotengra.HyperOptimizer): by default simply warn if an
  individual trial fails, rather than raising an exception. This is to ensure
  rare failures do not spoil an entire optimization run. The behavior can
  be controlled with the `on_trial_error` argument.

**Bug fixes**

- fixed bug in greedy optimizer that produced negative scores and otherwise
  inaccurate scores.
- fixed bug for contraction with many inputs and also preprocessing steps


## v0.5.4 (2023-10-17)

**Bug fixes**

- the `auto` and `auto-hq` optimizers are now safe to run under multi-threading.

## v0.5.3 (2023-10-16)

- [``einsum``](cotengra.einsum), [`einsum_tree`](cotengra.einsum_tree)
  and [`einsum_expression`](cotengra.einsum_expression): add support for all
  numpy input formats, including interleaved indices and ellipses.
- remove some hidden `opt_einsum` dependence (via a `PathOptimizer` method)


## v0.5.2 (2023-10-13)

- add [``ctg.array_contract_path``](cotengra.array_contract_path) for returning
  the raw contraction path only, with caching. Add caching to
  [``array_contract_expression``](cotengra.array_contract_expression) and
  related functions too.
- fix [`tree.get_eq()`](cotengra.ContractionTree.get_eq) when the ``inputs``
  are a tuple of `Sequence[str]` rather than a `str`.


## v0.5.1 (2023-10-3)

- add [tree.contraction_scaling](cotengra.ContractionTree.contraction_scaling)
- add [get_symbol_map](cotengra.get_symbol_map)


## v0.5.0 (2023-09-26)

- add [`einsum`](cotengra.einsum)
- add [`einsum_tree`](cotengra.einsum_tree)
- add [`einsum_expression`](cotengra.einsum_expression)
- add [`array_contract`](cotengra.array_contract)
- add [`array_contract_tree`](cotengra.array_contract_tree)
- add [`array_contract_expression`](cotengra.array_contract_expression)
- add [`AutoOptimizer`](cotengra.AutoOptimizer)
- add [`AutoHQOptimizer`](cotengra.AutoHQOptimizer)
- remove most hard dependencies (`numpy`, `opt_einsum`)
- update [`tree.plot_contractions`](cotengra.plot.plot_contractions)
