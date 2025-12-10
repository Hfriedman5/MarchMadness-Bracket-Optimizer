# March Madness Optimal Bracket Optimizer

This repository contains a Julia notebook that builds and optimizes NCAA March Madness brackets using probabilistic modeling, integer programming, and a custom genetic algorithm over groups of brackets. 

## Project Overview

The notebook starts from a full \(64 \times 64\) win-probability matrix between teams (real BPI-style probabilities and synthetic variants) and computes round-by-round advancement probabilities for every team. These probabilities feed into both an exact mixed-integer optimization model (using JuMP + Gurobi) and simulation-based heuristics to construct high-scoring, valid tournament brackets under standard scoring rules. 

## Core Components

- **Probability Inputs:**  
  - Loads serialized real and synthetic probability matrices `P_real` and `P_synthetic` representing win probabilities between any pair of 64 teams.
  - Builds supporting dictionaries `T` and `E` that encode the bracket tree and possible opponents each round. 

- **Advancement Probability Model:**  
  - `calculate_Wir(P, E)` computes for each team \(i\) and round \(r\) the probability that team \(i\) reaches and wins that round via a recursive dynamic program.  
  - Results are stored as `W_real` and `W_synthetic` for downstream optimization and heuristics.

- **Exact Optimal Bracket (JuMP + Gurobi):**  
  - `optimal_bracket(W, n_teams=64)` defines a binary integer program with variables $x_{i,r}$ indicating whether team \(i\) is picked to win round \(r\).  
  - Maximizes $$\sum_{i,r} 2^{r-1} W[(i,r)] x_{i,r}$$ subject to bracket-validity constraints: consistency across rounds, correct number of winners per round, and unique winners per game.

## Heuristics, Simulation, and Scoring

- **Greedy Bracket:**  
  - `greedy_bracket(P)` walks the tournament forward, always picking the higher-probability team (or a deterministic tie-break) in each game.

- **Tournament Simulation:**  
  - `simulate_game`, `simulate_round`, `create_initial_matchups`, `create_next_matchups`, and `simulate_tournament` simulate full stochastic tournaments from a probability matrix. 

- **Scoring & Validity:**  
  - `calculate_score(predicted, actual)` scores a bracket under standard March Madness scoring (1, 2, 4, 8, 16, 32 points by round).
  - `if_valid` / `is_valid` verify bracket structure: correct number of teams per round, proper progression of winners, and valid round-one pairings. 

## Genetic Algorithm over Bracket Groups

- **Representation & Diversity:**  
  - Brackets are represented as a vector of rounds, each a vector of team indices, with helpers for conversion and equality checks (`are_brackets_identical`, `is_unique`).

- **Mutation & Crossover (Bracket-Level):**  
  - `mutate_bracket` randomly changes a pick in a later round and repairs downstream rounds while preserving validity.
  - `crossover_brackets` performs single-point crossover between two parent brackets and repairs offspring.

- **Group-Level GA Loop:**  
  - `initialize_group` builds a diverse group of valid brackets around a base bracket.
  - `mutate_group` and `crossover_groups` evolve groups of brackets, maintaining only valid and unique children.
  - `average_max_score_over_simulations_valid` evaluates a group’s fitness as the average, over many simulated tournaments, of the best score achieved by any bracket in the group. 
  - `run_ga_groups` runs the GA with elitism, tournament selection, crossover, and mutation, logging per-generation best fitness and returning the best group found.

## Data Export and Environment

- **JSON Export:**  
  - Selected bracket groups are saved to `March_madness.json` using the `JSON` package, after validity checks.

- **Language & Packages:**  
  - Julia (kernel: `Julia 1.11.2`) with `Gurobi`, `JuMP`, `LinearAlgebra`, `Random`, `Statistics`, `Serialization`, and `JSON`.
  - Requires a working Gurobi installation and license, plus `Prob_real.dat` and `Prob_synthetic.dat` in the working directory.

## How to Run

1. Install Julia (≥ 1.11) and add the listed packages.
2. Configure Gurobi and ensure the Julia–Gurobi integration works.
3. Place `Prob_real.dat` and `Prob_synthetic.dat` alongside the notebook (or update paths). 
4. Open `march-madness.ipynb` in Jupyter and run cells sequentially to: load probabilities, compute `W_*`, solve `optimal_bracket`, run greedy/GA solvers, and export brackets.

---

## License

This project is open-source and available under the MIT License.

### The MIT License (MIT)

Copyright (c) 2025 [Hannah Friedman]

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

The Software is provided “as is”, without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages or other liability, whether in an action of contract, tort or otherwise, arising from, out of or in connection with the Software or the use or other dealings in the Software.
