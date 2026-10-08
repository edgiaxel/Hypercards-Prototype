# Hypercards Prototype

## Overview

The **Hypercards Prototype** is an experimental research codebase developed for an academic thesis investigating reinforcement learning (RL) in turn-based strategic decision environments. The project provides a controlled, turn-based card game environment—themed around fictional endurance hypercar racing—specifically designed to conduct a rigorous, multi-family algorithmic comparison among three foundational RL paradigms:

- **Deep Q-Network (DQN)** — Value-based, off-policy reinforcement learning
- **Proximal Policy Optimization (PPO)** — Policy-gradient, on-policy reinforcement learning
- **Advantage Actor-Critic (A2C)** — Actor-critic, on-policy synchronous baseline

This repository is an **academic research prototype**, not a commercial product. The core engineering priorities are experimental control, deterministic reproducibility, architectural decoupling, and observable validation of research hypotheses.

---

## Research Context

Existing literature provides substantial evidence comparing RL algorithms across video games and continuous control benchmarks, as well as domain-specific studies in card and board games. However, a methodological gap exists in controlled, multi-family comparisons of DQN, PPO, and A2C operating within a **single discrete turn-based game environment** under strictly standardized experimental conditions.

The Hypercards research framework addresses this gap across four central dimensions:

1. **Implementation Feasibility**: How DQN, PPO, and A2C can be integrated into the same turn-based environment while preserving action-masking validity.
2. **Decision Performance**: Comparative terminal win rates and mean episode returns against a standardized opponent.
3. **Learning Efficiency**: Sample efficiency and learning trajectory over standardized environment interaction budgets.
4. **Learning Stability**: Outcome consistency and variance across independent random training seeds, evaluated with robust statistics (standard deviation, interquartile mean, bootstrap confidence intervals).

---

## Project Status

The repository is currently in the **pre-implementation / prototype preparation stage**. 

At this baseline:
- The authoritative research specification ([`Phase6A.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6A.md)) and game/environment specification ([`Phase6B.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6B.md)) have been established.
- The documentation baseline ([`README.md`](file:///home/axel/Development/Hypercards-Prototype/README.md), [`AGENTS.md`](file:///home/axel/Development/Hypercards-Prototype/AGENTS.md), [`ARCHITECTURE.md`](file:///home/axel/Development/Hypercards-Prototype/ARCHITECTURE.md), and [`REQUIREMENTS.md`](file:///home/axel/Development/Hypercards-Prototype/REQUIREMENTS.md)) is active.
- A local Python virtual environment exists, but core game logic, Gymnasium wrappers, RL training pipelines, and Godot client scenes have **not yet been implemented**.

---

## Authoritative Specifications

All implementation decisions must conform to the established documentation hierarchy:

| Document | Authority Level | Scope & Core Responsibility |
| :--- | :--- | :--- |
| [**`Phase6A.md`**](file:///home/axel/Development/Hypercards-Prototype/Phase6A.md) | **Research Authority** | Research questions, independent/dependent/controlled variables, evaluation protocol, interaction budgets, random-seed methodology, statistical aggregation (IQM, bootstrap CI), and masking principles. |
| [**`Phase6B.md`**](file:///home/axel/Development/Hypercards-Prototype/Phase6B.md) | **Game & Environment Authority** | Fictional hypercar game rules, 12-card roster, 6-circuit pool, energy progression (1–6), round structure (6 rounds), hand management (9 slots), 28-action discrete mapping, action masking rules, observation encoding, heuristic opponent, and terminal rewards (+1 / 0 / -1). |
| [**`ARCHITECTURE.md`**](file:///home/axel/Development/Hypercards-Prototype/ARCHITECTURE.md) | **Software Architecture Authority** | System boundaries, decoupling of pure Python engine from Gymnasium and RL frameworks, headless execution architecture, and Godot client presentation boundary. |
| [**`REQUIREMENTS.md`**](file:///home/axel/Development/Hypercards-Prototype/REQUIREMENTS.md) | **Requirements Authority** | Concrete, traceable functional and technical requirements (`ENV-xxx`, `ACT-xxx`, `OBS-xxx`, `RWD-xxx`, etc.). |
| [**`README.md`**](file:///home/axel/Development/Hypercards-Prototype/README.md) | **Project Orientation** | Repository orientation, environment setup, and development status. Does not override authoritative specifications. |

> [!CAUTION]
> **Conflict Resolution Rule**:
> If any discrepancy or ambiguity is discovered between documents or during implementation, agents and developers must **not** guess or silently choose a solution. The conflict must be documented, surfaced, and formally clarified against `Phase6A.md` and `Phase6B.md`.

---

## Repository Structure

```text
Hypercards-Prototype/
├── .venv/                  # Local Python virtual environment
├── Phase6A.md              # Authoritative Research Design & Methodology
├── Phase6B.md              # Authoritative Game & Environment Specification
├── README.md               # Project overview and orientation (this file)
├── AGENTS.md               # Behavioral rules and constraints for AI agents
├── ARCHITECTURE.md         # Software architecture and module boundaries
└── REQUIREMENTS.md         # Traceable technical implementation requirements
```

*Note: Source code directories (`hypercards/`, `tests/`, `godot/`) will be introduced in subsequent roadmap phases.*

---

## Development Environment

The active local development host configuration is:

- **Operating System**: Linux (Kubuntu)
- **Active Workspace**: `/home/axel/Development/Hypercards-Prototype`
- **Virtual Environment**: `/home/axel/Development/Hypercards-Prototype/.venv`
- **Current Host Python**: `Python 3.14`
- **Current Host pip**: `pip 25.1.1`
- **Godot Engine Executable**: `/home/axel/Documents/Godot_v4.7.2-stable_linux.x86_64`
- **Godot Engine Version**: `4.7.2-stable`

> [!IMPORTANT]
> **Python 3.14 Dependency Compatibility Note**:
> `Python 3.14` is the currently active development runtime. Compatibility with key deep reinforcement learning dependencies (e.g., PyTorch, Gymnasium, and Stable-Baselines3) has **not yet been verified** and constitutes an upcoming setup task. The runtime will be validated or pinned to a compatible Python version during Phase 2.

---

## Architecture Overview

The system is strictly decomposed into decoupled layers to guarantee that research experiments remain independent of frontend visualization:

```text
                      ┌──────────────────────┐
                      │     Godot 4.7.2      │
                      │ Presentation & Client│
                      │ (Human play & demos) │
                      └──────────┬───────────┘
                                 │ Inter-process / API
                                 ▼
                      ┌──────────────────────┐
                      │   Hypercards Core    │
                      │ Pure Python Engine   │
                      │ Authoritative Rules  │
                      └──────────┬───────────┘
                                 │
                                 ▼
                      ┌──────────────────────┐
                      │ Gymnasium Interface  │
                      │  Action Masking &    │
                      │  Vector Observation  │
                      └──────────┬───────────┘
                                 │
               ┌─────────────────┼─────────────────┐
               ▼                 ▼                 ▼
             DQN               PPO               A2C
```

1. **Pure Game Engine (`hypercards.core`)**: Implements authoritative game state, rules, resolution, and heuristic opponent in standard Python without any ML dependencies.
2. **Gymnasium Environment (`hypercards.env`)**: Exposes a standard `gymnasium.Env` interface with a `Discrete(28)` action space, structured numerical vector observations, and state-dependent action masks.
3. **RL Algorithms**: Standardized DQN, PPO, and A2C agents interact with the common environment under identical budgets and reward structures.
4. **Godot Client**: Acts purely as a presentation, visualization, and human-input client. Godot **never** computes game rules, power totals, or legal action masks independently.

---

## Prototype Development Roadmap

Development proceeds sequentially through the following planned stages:

1. **Documentation Baseline** *(Current Phase)*: Establish `README.md`, `AGENTS.md`, `ARCHITECTURE.md`, and `REQUIREMENTS.md`.
2. **Python / Dependency Compatibility Verification**: Validate Python 3.14 compatibility with PyTorch, Gymnasium, and Stable-Baselines3, or pin an appropriate runtime.
3. **Python Project Skeleton**: Establish package structure, linting, configuration, and test harnesses.
4. **Minimal Gymnasium Environment**: Create stub environment adhering to the Gymnasium API standard.
5. **Core Game State**: Implement immutable and mutable state objects (players, decks, hands, energy).
6. **Cards and Circuits**: Implement the 12-card hypercar roster and 6-circuit pool with structured ability metadata.
7. **Game Engine / Resolution**: Implement round sequencing, placement execution, deterministic power calculation, and championship scoring.
8. **Deterministic Heuristic Opponent**: Implement the fixed, non-learning opponent with deterministic tie-breaking.
9. **Action Masking Mechanism**: Implement state-dependent validation producing the 28-element legal action mask.
10. **Observation Construction**: Implement flat/structured numerical vector encoding for MLP policies.
11. **Stable-Baselines3 Integration**: Connect DQN, PPO, and A2C, resolving invalid-action masking parity across all three algorithm families.
12. **Training & Evaluation Prototype**: Implement headless training pipelines, seed scheduling, checkpointing, and evaluation metrics (win rate, return, IQM, bootstrap CI).
13. **Godot Client / Presentation Layer**: Build the 2D desktop visualization in Godot 4.7.2 connecting to the authoritative Python engine for human play and model demonstration.

---

## Running the Project

> [!NOTE]
> **No Executable Application Yet**:
> Because the project is currently in the pre-implementation documentation baseline stage, there is no runnable game client or RL training script at this time. Execution commands and CLI entry points will be documented as the core packages are implemented.
