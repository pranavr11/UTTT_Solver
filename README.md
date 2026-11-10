# Towards Solving Ultimate Tic Tac Toe

**URL:** https://pranavr11.github.io/UTTT_Solver/

## Summary

We aim to reproduce Nelson Elhage's high-performance parallel Ultimate Tic Tac Toe (UTTT) solver and improve its efficiency through further optimizations. If time permits, we will also extend it to a distributed solver and explore neural-network-guided position analysis.

## Background

UTTT is played on a 3x3 grid of ordinary tic tac toe boards. A move's position within its small board determines which small board the opponent must play next. If that board is already won or full, the opponent may play anywhere. Winning a small board claims the corresponding cell of the large board, and the first player to claim three large-board cells in a row, column, or diagonal wins.

Unlike traditional tic tac toe, the game is not solved due to the added computational complexity introduced in the game modifications, making it a prime candidate to apply parallelization and a series of optimizations to improve existing solving efforts. Our eventual goal is a "weak" solution: proving whether the first player has a winning strategy, without storing the value of every reachable position.

Elhage's 2020 project ([minimax.dev](https://minimax.dev/docs/ultimate/)) is the current SOTA. His solver (ultimattt) is a hybrid of Proof-Number Search and Depth-First Proof-Number Search (PN-DFPN):

- A small explicit proof-number tree near the root hands bound DFPN searches for worker threads.
- Virtual proof numbers keep workers from all pursuing the same most-promising node.
- All workers share a concurrent transposition table (TT) with fine-grained per-entry locking.
- Game state is bit-packed, win detection uses SIMD, and "critical square" analysis prunes parts of the tree without search.

Even so, he estimated that a full solution would cost on the order of 2–10M of cloud CPU time (at the time).

## The Challenge

The existing solver already uses most standard optimizations, but still does not solve the game. There is still plenty of work on the table to be optimized/parallelized, so we hope to dive deeper into the problem in the term of our project and determine what is feasible and will result in incremental performance improvements.

We plan to use profiling and other techniques from the class to improve the SOTA and further progress to a working and complete solver.

We hypothesize the final optimizations to be centered around:

- **Memory usage.** The data structure the solver uses (TT) is quite large at 10 GiB, meaning that targeted optimizations to caching, NUMA awareness, and other techniques could carry significant weight.
- **Centralized scheduling.** Similar to the anti-pattern in asst4 with having a single thread dispatch work to other threads, this will result in poor performance when scaling to higher thread counts.
- **Pruning methodology.** Elhage argued that positional analysis can be used to prune and speculate the search pattern to effectively cut the amount of compute time to find a solution.

## Resources

- [Ultimate Tic-Tac-Toe (minimax.dev)](https://minimax.dev/docs/ultimate/)
- [Ultimate tic-tac-toe (Wikipedia)](https://en.wikipedia.org/wiki/Ultimate_tic-tac-toe)
- [2511.10339] Massively Parallel Proof-Number Search for Impartial Games and Beyond
- AlphaZero-based Proof Cost Network to Aid Game Solving (OpenReview)
- Possible Microsoft Azure credits for startups
- Leftover credits from previous classes / programs

## Goals and Deliverables

### Plan to achieve

- Reproduce Nelson Elhage's existing solver performance
  - Either from scratch, or build from the existing (reimplemented) model
  - Maybe use a different parallelization paradigm or platform
  - If we can create a CUDA version (with slightly worse performance), it may be possible to bruteforce in a more reasonable manner
- Profile and identify bottlenecks
- Implement at least two optimizations
- Evaluate parallel scaling
- Quantify improvements and preserve correctness

### Hope to achieve

- Speed up current SOTA techniques for solving UTTT by multiple orders of magnitude, weakly solving the game. (To weakly solve is to establish a proof of whether the first player has a winning strategy.)
- Explore neural-network guided search
- Develop a distributed solver

### Concerns from the meeting

- Will we be able to do something interesting / novel? Not sure, which is why our "plan to achieve" is just to replicate / reimplement the baseline Nelson Elhage solver.
- Will this also fall into the trap of the "parallel instances" just computing the same thing in isolation? The literature shows that cooperative proof number search is a real thing, so we have some basis to think we're good.

## Platform Choice

- GHC/PSC → NCSA for hella cores
- CMU Google Cloud credits are available (I think)
- AWS/Azure something; use friends' YC credits if possible
- Also have Modal credits for GPU

## Schedule

As Pranav has 1 final exam and no other projects or exams for the rest of the semester, he will be available to dedicate a significant number of hours to this project. Varun will also be able to dedicate a considerable amount of time to the project, although he has a regular number of projects and exams left in the semester. Taking this into consideration, we have created the following schedule.

| Week | Dates | Plan |
| --- | --- | --- |
| 1 | Nov 9–15 | Submit proposal. Build ultimattt on GHC/PSC and confirm it runs. Select a consistent method of benchmarking and run the baselines. |
| 2 | Nov 16–22 | Reproduce / reimplement baseline results and finalize core profiling. Begin design and setup for the neural-network-advised position analysis module and parallel transposition table optimizations. Intermediate milestone report due. |
| 3 | Nov 23–29 | Thanksgiving week—implement two optimizations and build candidate neural model for positional evaluation. Benchmark neural evaluation batching to explore parallelism and inference throughput. |
| 4 | Nov 30 – Dec 6 | Integrate neural move ordering into parallel DFPN search. Begin implementation of the distributed solver prototype across nodes, featuring coarse task leasing and work stealing. Complete feature freeze. |
| 5 | Dec 7–10 | Run full scaling sweeps for the combined distributed and neural-guided solver. Finalize efficiency plots, prepare final report, and present poster for competition on Dec 10. |
