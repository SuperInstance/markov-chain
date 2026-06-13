# Markov Chain

A **Markov chain** is a stochastic process where the probability of transitioning to any future state depends only on the current state, not on the sequence of events that preceded it. This crate provides the building blocks for constructing and sampling from discrete-time Markov chains in Rust.

## Why It Matters

Markov chains power a surprising range of applications: predictive text input, Google's PageRank, queueing theory, reinforcement learning state transitions, and procedural content generation. Any system that can be modeled as "given where I am now, where might I go next?" is a candidate for Markov chain analysis.

## How It Works

A discrete-time Markov chain is defined by a **state space** S and a **transition matrix** P where P[i][j] gives the probability of moving from state i to state j. The fundamental property — the **Markov property** — is that P(X_{n+1} = j | X_n = i, X_{n-1}, ..., X_0) = P(X_{n+1} = j | X_n = i). This memorylessness dramatically simplifies analysis.

The **stationary distribution** π satisfies π = πP, meaning once the chain reaches π, it stays there. Finding π reveals the long-run behavior of the system.

## Quick Start

```rust
use markov_chain::TransitionMatrix;

// Build a simple weather model: Sunny <-> Rainy
let mut tm = TransitionMatrix::new(2);
tm.set(0, 0, 0.8); // Sunny -> Sunny: 80%
tm.set(0, 1, 0.2); // Sunny -> Rainy: 20%
tm.set(1, 0, 0.5); // Rainy -> Sunny: 50%
tm.set(1, 1, 0.5); // Rainy -> Rainy: 50%

let chain = tm.build();
let next_state = chain.step(0); // Sample from state 0
```

## API

- `TransitionMatrix::new(states)` — O(n²) space for n states
- `set(from, to, prob)` — O(1) transition assignment
- `step(current)` — O(n) sampling, next state from current
- `stationary_distribution()` — O(n³) via eigendecomposition

## Architecture Notes

See the [SuperInstance Architecture](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md) overview.

## License

MIT
