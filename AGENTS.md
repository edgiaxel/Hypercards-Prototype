# AGENTS.md — Implementation Guidelines for AI Coding Agents

This document establishes operational constraints, behavioral rules, and engineering standards for AI coding agents (including Gemini, Greg, Antigravity, and Claude) working on the **Hypercards Prototype**.

---

## 1. Specification Authority & Documentation Hierarchy

Every implementation decision must strictly conform to the established documentation hierarchy:

1. [**`Phase6A.md`**](file:///home/axel/Development/Hypercards-Prototype/Phase6A.md) — **Authoritative for Research Methodology**
   - Controls: Research problem, research questions, algorithm comparison scope (DQN vs. PPO vs. A2C), independent/dependent/controlled variables, interaction budget parity, random seed protocol, evaluation metrics (win rate, mean return, sample efficiency, standard deviation, IQM, bootstrap confidence intervals), and theoretical masking principles.
2. [**`Phase6B.md`**](file:///home/axel/Development/Hypercards-Prototype/Phase6B.md) — **Authoritative for Game & Environment Specification**
   - Controls: Hypercards game rules, 12-card roster, 6-circuit pool, 6 rounds, 1–6 energy progression, hand/deck mechanisms, 28-action discrete mapping, state-dependent invalid-action masking, structured numerical observation space, deterministic heuristic opponent behavior, match resolution, and terminal rewards (+1 / 0 / -1).
3. [**`ARCHITECTURE.md`**](file:///home/axel/Development/Hypercards-Prototype/ARCHITECTURE.md) — **Authoritative for Software Architecture**
   - Controls: Subsystem boundaries, decoupling of pure Python engine from Gymnasium and RL frameworks, headless execution architecture, and client integration protocols.
4. [**`REQUIREMENTS.md`**](file:///home/axel/Development/Hypercards-Prototype/REQUIREMENTS.md) — **Authoritative for Concrete Functional Requirements**
   - Contains traceable requirement identifiers (`ENV-xxx`, `ACT-xxx`, `OBS-xxx`, `RWD-xxx`, `RNG-xxx`, `OPP-xxx`, `RL-xxx`, `ARCH-xxx`, `TEST-xxx`, `CLIENT-xxx`, `OPEN-xxx`).
5. [**`README.md`**](file:///home/axel/Development/Hypercards-Prototype/README.md) — **Project Orientation**
   - High-level overview and setup notes; does not override specifications.

---

## 2. Conflict Resolution Protocol

> [!CAUTION]
> **Zero Silent Assumptions Rule**:
> If an agent detects any ambiguity, discrepancy, contradiction, or omission between specifications, or between specifications and external libraries (e.g., Stable-Baselines3, Gymnasium, PyTorch):
> 
> 1. **DO NOT silently choose an arbitrary solution.**
> 2. **DO NOT compromise research rigor or experimental controls to bypass engineering hurdles.**
> 3. **DO NOT guess author intent.**
> 4. **Document the conflict explicitly**, present the exact trade-offs to the user, and request clarification before proceeding with code changes.

---

## 3. Implementation Discipline & Absolute Prohibitions

### Absolute Prohibitions
- **DO NOT invent game rules**: Every mechanic, card cost, base power, circuit bonus, and turn transition must trace directly to `Phase6B.md`.
- **DO NOT alter card or circuit parameters**: Do not adjust numbers for balance or convenience unless explicitly instructed by the user.
- **DO NOT create algorithm-specific game environments**: There must be **ONE common Gymnasium environment** shared identically across DQN, PPO, and A2C. Never create "HypercardsDQNEnv", "HypercardsPPOEnv", or custom environment forks per algorithm.
- **DO NOT duplicate game rules in Godot**: Godot is strictly a presentation and input frontend. It must never independently compute power, decide legal action masks, manage RNG seeds, or determine round winners.
- **DO NOT silently drop or weaken action masking**: Valid action masking is a fundamental controlled variable across all three algorithms.
- **DO NOT leak unobservable information**: The observation vector must never expose the opponent's hidden hand, future deck draws, or internal RNG states.

### Implementation Discipline
- **Prefer small, verifiable changes**: Build incrementally and validate correctness at each stage rather than generating massive architectural boilerplate in a single pass.
- **Write unit tests for game mechanics**: Every card ability, circuit effect, energy transition, power calculation, and masking rule must be backed by isolated unit tests.
- **Avoid premature abstraction**: Implement clear, concrete data structures and functions before designing complex generic frameworks.
- **Avoid building the entire system before validating the core**: Verify the pure Python engine before building the Gymnasium environment; verify the Gymnasium environment before launching RL training.

---

## 4. Software Engineering Principles

- **Strict Decoupling of Engine and RL Frameworks**:
  - The pure game engine (`hypercards.core`) must depend only on standard Python libraries (e.g., standard library, `dataclasses`, `enum`, `typing`, `math`).
  - `hypercards.core` must have **zero dependencies** on `gymnasium`, `torch`, `stable-baselines3`, or `godot`.
- **Authoritative Python Environment**:
  - The Python engine is the single source of truth for all game logic, state transitions, validation, and scoring.
  - Client interfaces (Godot or CLI) must interact with the engine via clearly defined serialization boundaries (e.g., JSON / IPC).
- **Separation of Determinism and Stochasticity**:
  - The rules engine is 100% deterministic given a state and an action.
  - All stochastic events (deck shuffle, initial card distribution, round card draws, circuit selection) must be explicitly driven by an isolated, seedable pseudo-random number generator (`Environment RNG`).
  - The `Environment RNG` must be kept separate from the RL agent's training/policy stochasticity (`Agent RNG`).

---

## 5. Research Integrity & Experimental Controls

- **Preserve Controlled Variables**:
  Under `Phase6A.md`, algorithm comparison is valid only if all other variables remain strictly controlled:
  - Identical environment rules and card/circuit parameters
  - Identical observation encoding and action mapping
  - Identical action-masking mechanism
  - Identical terminal reward structure (`+1 / 0 / -1`)
  - Identical deterministic heuristic opponent
  - Identical interaction budgets (total timesteps)
  - Matched training seed sets and identical held-out evaluation seed sets
- **Never Weaken Requirements for Implementation Convenience**:
  If a deep learning library (such as Stable-Baselines3) lacks native support for a research requirement (e.g., uniform action masking across all three algorithms), surface this as a technical issue. Do not secretly alter the experiment to suit library limitations.

---

## 6. Prototype Philosophy & Engineering Priorities

This repository is an exploratory research prototype for a thesis. When making trade-offs, prioritize:

1. **Correctness & Rule Fidelity**: The game mechanics and RL interfaces must behave exactly as specified.
2. **Observability & Diagnostics**: Provide rich state inspection, logging, and validation hooks to trace why an action was taken or rejected.
3. **Reproducibility**: Identical seeds must yield identical episode trajectories and heuristic opponent decisions across test runs.
4. **Debuggability**: Clear assertions, type hints, informative error messages, and transparent data structures are favored over dense optimizations.
5. **Validation of Research Assumptions**: Instrumentation, diagnostic counters, and temporary verification scripts are encouraged to confirm that experimental controls hold.

Production polish, high-fidelity rendering, premature multi-threading, and commercial game feature bloat are explicitly out of scope.

---

## 7. Open Technical Investigations & Caveats

AI agents must remain aware of the following ongoing technical considerations:

1. **Python 3.14 Compatibility**:
   - The host environment currently runs `Python 3.14`. Key scientific packages (PyTorch, Stable-Baselines3) may have pre-built wheel limitations on Python 3.14. Compatibility verification is mandatory before writing dependency lockfiles.
2. **Action-Masking Parity across SB3 Algorithms**:
   - While `sb3-contrib` provides `MaskablePPO`, Stable-Baselines3 lacks native, unified action-masking implementations for DQN and A2C out of the box.
   - Agents must treat this as an open investigation item. Any proposed wrapper, custom policy, or logit-masking adapter must be evaluated for research equivalence before adoption.
