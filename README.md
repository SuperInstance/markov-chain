# Markov Chain

A **Markov chain library** for modeling discrete-state stochastic processes where the next state depends only on the current state (the Markov property) — providing transition matrix construction, stationary distribution computation, and chain simulation.

## Why It Matters

Markov chains are the mathematical foundation for: Google PageRank (a random walk on the web graph), predictive text (what word comes next), queueing theory (M/M/1 queues), financial modeling (regime-switching models), and biological sequence analysis (HMMs are Markov chains with hidden states). The Markov property — P(next | history) = P(next | current) — is both the key simplification (making computation tractable) and the key limitation (ignoring long-range dependencies). This library provides the core tools: build a transition matrix, compute its eigenvector for the stationary distribution, and generate random walks through the state space. For SuperInstance, Markov chains model agent state transitions in the conservation framework.

## How It Works

**Transition matrix**: An N×N stochastic matrix P where `P[i][j]` = probability of transitioning from state i to state j. Each row sums to 1 (it's a probability distribution).

```
P = | 0.9  0.1  0.0 |    State 0: mostly stays, 10% → State 1
    | 0.3  0.4  0.3 |    State 1: volatile, spreads across all
    | 0.0  0.2  0.8 |    State 2: mostly stays, 20% → State 1
```

**n-step transitions**: P^n gives the probability of transitioning from i to j in exactly n steps. As n → ∞, P^n converges to a matrix where every row equals the stationary distribution π (if the chain is ergodic — irreducible and aperiodic).

**Stationary distribution**: The eigenvector of P^T corresponding to eigenvalue 1:
```
π · P = π     (left eigenvector)
Σ π_i = 1     (normalization)
```
This is the long-run proportion of time spent in each state. Computed via:
- **Power iteration**: π_{n+1} = π_n · P until convergence. O(N²) per iteration.
- **Direct eigendecomposition**: For small N, solve the linear system (P^T - I)π = 0 with constraint Σπ = 1.

**Simulation**: Given a starting state and the transition matrix, generate a random walk:
```
current = start
for each step:
    r = uniform(0, 1)
    cumulative = 0
    for j in 0..N:
        cumulative += P[current][j]
        if r < cumulative:
            current = j
            break
    yield current
```

**Complexity**:
| Operation | Time | Space |
|-----------|------|-------|
| Build matrix | O(N²) | O(N²) |
| One-step transition | O(N) | O(1) |
| Power iteration (to convergence) | O(N² × iterations) | O(N) |
| N-step simulation | O(steps × N) | O(1) |

**Convergence rate**: Determined by the spectral gap `1 - |λ₂|` where λ₂ is the second-largest eigenvalue (by magnitude). The mixing time is O(1/spectral_gap). Chains with λ₂ close to 1 mix slowly.

## Quick Start

```rust
use markov_chain::{MarkovChain, TransitionMatrix};

// Build a weather model: Sunny → Cloudy → Rainy
let mut p = TransitionMatrix::new(3);
p.set(0, 0, 0.7); p.set(0, 1, 0.2); p.set(0, 2, 0.1); // Sunny
p.set(1, 0, 0.3); p.set(1, 1, 0.4); p.set(1, 2, 0.3); // Cloudy
p.set(2, 0, 0.2); p.set(2, 1, 0.3); p.set(2, 2, 0.5); // Rainy

let chain = MarkovChain::new(p);
let stationary = chain.stationary_distribution();
println!("Long-run distribution: {:?}", stationary);

// Simulate 1000 steps
let walk = chain.simulate(0, 1000);
```

## API

| Type | Description |
|------|-------------|
| `TransitionMatrix::new(n)` | Create an N×N matrix |
| `.set(i, j, prob)` | Set transition probability |
| `MarkovChain::new(matrix)` | Wrap matrix with analysis methods |
| `.stationary_distribution()` | Compute π via power iteration |
| `.simulate(start, steps)` | Generate a random walk |

## Architecture Notes

This crate provides stochastic modeling for SuperInstance agent state transitions. The stationary distribution reveals the equilibrium behavior of agent fleets — directly connecting to **γ + η = C** where the stationary π represents the steady-state resource allocation. See [Architecture](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md).

## References

- Norris, J. *Markov Chains*, Cambridge UP (1997).
- Page, L. et al. "The PageRank Citation Ranking," Stanford Tech Report (1998).
- Levin, D. & Peres, Y. *Markov Chains and Mixing Times*, 2nd ed., AMS (2017).

## License

MIT
