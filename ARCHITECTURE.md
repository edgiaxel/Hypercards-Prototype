# Architecture

This document defines the software architecture of the **Hypercards Prototype**, establishing subsystem boundaries, component responsibilities, data flows, and interface contracts.

---

## Architectural Principles

The architecture is governed by four foundational principles derived from [`Phase6A.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6A.md) and [`Phase6B.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6B.md):

1. **Single Source of Truth**: The pure Python game engine is the sole authoritative authority for all game rules, state transitions, energy budgets, card placement legality, circuit activations, power calculations, and match outcomes.
2. **Strict Decoupling**: Core game logic has zero dependency on RL frameworks (`gymnasium`, `torch`, `stable-baselines3`) or UI engines (`godot`). The game engine is pure Python.
3. **Algorithm Interchangeability**: The Gymnasium environment exposes a single, standardized interface. DQN, PPO, and A2C interact with the exact same environment class, receiving identical observations, legal-action masks, and terminal rewards under identical interaction budgets.
4. **Headless Independence**: All training, evaluation, and statistical benchmarking pipelines run fully headless in standard Python without launching or requiring Godot.

---

## System Overview

```text
                    ┌────────────────────────┐
                    │      Godot 4.7.2       │
                    │ Presentation & UI      │
                    │ Human Play & Demos     │
                    │ Desktop Client         │
                    └───────────┬────────────┘
                                │ (JSON-RPC / Subprocess IPC / Stdio)
                                ▼
                    ┌────────────────────────┐
                    │   Hypercards Engine    │
                    │  (hypercards.core)     │
                    │ Pure Python Logic      │
                    │ Authoritative Rules    │
                    └───────────┬────────────┘
                                │ Internal Python API
                                ▼
                    ┌────────────────────────┐
                    │  Gymnasium Environment │
                    │   (hypercards.env)     │
                    │ Vector Obs & 28-Mask   │
                    │ Step / Reset Lifecycle │
                    └───────────┬────────────┘
                                │ Standard Gymnasium / Masked Env API
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
      ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
      │     DQN     │    │     PPO     │    │     A2C     │
      │ Value-Based │    │Policy-Grad. │    │Actor-Critic │
      └─────────────┘    └─────────────┘    └─────────────┘
```

---

## Python Environment

The Python environment serves as the authoritative foundation for both research execution and client simulation:

- **Runtime Target**: Host environment currently runs Python 3.14 (compatibility with PyTorch and SB3 to be verified in Phase 2).
- **Execution Modes**:
  - *Headless Research Mode*: Executes automated batch training, validation loops, and statistical evaluation runs.
  - *Server / IPC Mode*: Executes the Python engine as a backend subprocess, serving state updates and validating actions for the Godot frontend client.
- **Packaging Structure**:
  - `hypercards.core`: Pure game domain model, rule engine, and heuristic opponent.
  - `hypercards.env`: Gymnasium environment wrapper, observation builder, and action masker.
  - `hypercards.agent`: Agent adapters, SB3 integration, and masking handlers.
  - `hypercards.experiments`: Training scripts, seed orchestrators, evaluation suites, and statistical export tools.

---

## Game Engine

The game engine (`hypercards.core`) encapsulates all state representations and rule evaluation mechanics:

- **State Representation**:
  - `GameState`: Complete match snapshot, containing round counter (1–6), active circuits, player states, opponent states, and phase indicators.
  - `PlayerState`: Tracks deck (12 cards), hand (up to 9 slots), available Energy (1–6), and cards placed at circuits.
  - `Circuit`: Represents circuit identity, activation round, base power totals, and ability triggers.
  - `Card`: Represents card identity, cost, base power, ability type, trigger condition, and parameters.
- **Rule Resolution**:
  - Round progression: Round 1 activates Circuit 1; Round 2 activates Circuit 2; Round 3 activates Circuit 3; Rounds 4–6 maintain all active circuits.
  - Energy progression: Round $N$ allocates $N$ base energy. Unused energy is lost at round end unless modified by Circuit 5 (Americas Crown) or Card 9 (Porch 963).
  - Power calculation: Pure deterministic aggregation of base card powers plus applicable card and circuit modifiers.
  - Match resolution: At the end of Round 6, circuits are compared individually. The player winning $\ge 2$ circuits wins the match (+1); losing $\ge 2$ circuits loses (-1); otherwise Draw (0).

---

## Gymnasium Interface

The Gymnasium layer (`hypercards.env`) translates the game engine into the standardized RL environment API (`gymnasium.Env`):

- **Action Space**: `spaces.Discrete(28)`
  - Actions `0–26`: Card placement combinations (9 hand slots $\times$ 3 circuits).
  - Action `27`: `PASS / END TURN`.
- **Observation Space**: `spaces.Box(low=..., high=..., shape=(D,), dtype=np.float32)`
  - Fixed-size 1D numerical vector representing global match state, own circuit powers, opponent circuit powers, circuit metadata, own hand slot encodings, and opponent visible counts.
- **Action Masking**: Exposes a binary array of shape `(28,)` (`1` = legal, `0` = illegal) reflecting:
  - Slot occupancy (cannot play empty slot).
  - Energy affordability (effective cost $\le$ available energy).
  - Circuit activation status (cannot play on inactive circuit).
  - Action 27 (`PASS`) is always legal during a decision turn.
- **Lifecycle & Turn Semantics**:
  - An RL `step(action)` corresponds to a **single decision** by the RL agent, *not* an entire round.
  - An agent may place multiple cards in a single round across consecutive `step()` calls.
  - Selecting Action 27 (`PASS`) concludes the RL agent's turn for that round, triggers the heuristic opponent's decision sequence, executes round resolution, advances to the next round, and returns the next observation.

---

## RL Algorithm Layer

The RL layer consumes the Gymnasium environment to train and evaluate policies:

- **Target Algorithms**:
  - **DQN** (Deep Q-Network): Value-based, off-policy Q-learning with replay buffer and target network.
  - **PPO** (Proximal Policy Optimization): On-policy clipped surrogate policy gradient.
  - **A2C** (Advantage Actor-Critic): Synchronous, on-policy actor-critic baseline.
- **Policy Architectures**: Default SB3 two-layer Multi-Layer Perceptrons (MLP with $64 \times 64$ hidden units) mapping the numerical observation vector to action values/distributions.
- **Common Masking Contract**:
  - All algorithms must receive the exact same legal-action mask at every decision step.
  - The implementation must address the technical reality that SB3 provides native masking for PPO via `sb3-contrib.common.maskable`, but requires dedicated adaptation for DQN and A2C to ensure methodological equivalence without altering environment rules.

---

## Opponent Layer

The opponent is an integral component of the game environment rather than an external learning agent:

- **Non-Learning Baseline**: The opponent does not update parameters during training or evaluation.
- **Deterministic Heuristic Policy**:
  - Identifies legal actions available to the opponent using identical game rules.
  - Computes a deterministic heuristic score for each legal action based on expected power yield, energy efficiency, circuit deficits, and round context.
  - Employs a strict **deterministic tie-breaking rule** (e.g., highest heuristic score $\rightarrow$ highest power yield $\rightarrow$ lowest energy cost $\rightarrow$ lowest action ID).
- **Information Parity**: The heuristic opponent observes only legal public information and its own hand; it has no access to the RL agent's private hand or future RNG draws.

---

## Observation and Action Representation

### Action Mapping Table

| Action Range | Mapping Formula | Description |
| :--- | :--- | :--- |
| `0 – 2` | Hand Slot 1 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 1 onto selected circuit |
| `3 – 5` | Hand Slot 2 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 2 onto selected circuit |
| `6 – 8` | Hand Slot 3 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 3 onto selected circuit |
| `9 – 11` | Hand Slot 4 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 4 onto selected circuit |
| `12 – 14` | Hand Slot 5 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 5 onto selected circuit |
| `15 – 17` | Hand Slot 6 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 6 onto selected circuit |
| `18 – 20` | Hand Slot 7 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 7 onto selected circuit |
| `21 – 23` | Hand Slot 8 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 8 onto selected circuit |
| `24 – 26` | Hand Slot 9 $\rightarrow$ Circuits 1, 2, 3 | Place card from Hand Slot 9 onto selected circuit |
| `27` | `PASS / END TURN` | Conclude player actions for current round |

### Numerical Observation Vector
The observation vector avoids natural-language text and raw image matrices, providing normalized numerical encodings:
- **Global Fields**: Normalized current round ($r/6$), current energy ($e/6$), deck count remaining ($d/12$), hand count ($h/9$).
- **Circuit Fields ($\times 3$)**: Active flag ($0/1$), own power, opponent power, circuit ID / ability category encoding, ability parameter values.
- **Hand Slot Fields ($\times 9$)**: Occupancy flag ($0/1$), card ID, cost, base power, ability type, trigger condition, parameter values. Empty slots are padded with zeros.
- **Opponent Visible Fields**: Opponent visible circuit powers, opponent hand card count, opponent deck card count.

---

## Randomness and Reproducibility

- **Dual-RNG Separation**:
  - `Environment RNG`: Drives game-level stochastic events (deck shuffling, initial 3-card draw, per-round 1-card draw, circuit selection from the 6-circuit pool). Seeded explicitly via `env.reset(seed=...)`.
  - `Agent / Algorithm RNG`: Controls network initialization, exploration noise, replay buffer batch sampling, and policy rollouts.
- **Seed Methodology**:
  - Independent training seeds (e.g., 5 distinct seeds) evaluate training stability and variance.
  - Held-out evaluation seeds (e.g., 100 fixed seeds) present an identical set of stochastic match scenarios to all trained policies.

---

## Godot Client

Godot 4.7.2 serves strictly as a frontend client and human-computer interface:

- **Responsibilities**:
  - Rendering 2D visual assets, cards, race circuits, and scoreboards.
  - Handling human mouse/keyboard inputs (card dragging, clicking circuits, Ready button, Surrender button).
  - Executing human turn countdown timers (e.g., 30-second decision timer).
  - Loading frozen trained models (or connecting to the Python engine) for interactive human-vs-agent or agent-vs-agent demonstration modes.
- **Architectural Boundary**:
  - Godot communicates with the Python engine via an explicit serialization interface (e.g., JSON over stdio or local socket).
  - Godot requests valid action masks and sends intended user actions; the Python engine returns updated game state.
  - Godot **never** executes game rule calculations independently.

---

## Data Flow

```text
1. Env Reset(seed)
   └── Python Engine seeds Environment RNG
   └── Engine selects 3 circuits from 6 pool
   └── Engine shuffles 12-card decks, draws 3 cards per player
   └── Round 1 begins: Draw 1 card, activate Circuit 1, set Energy = 1

2. Agent Step(action)
   ├── Gymnasium verifies action against legal_action_mask
   ├── If Action in 0..26 (Card Placement):
   │   ├── Engine deducts energy, moves card from hand slot to circuit
   │   ├── Engine evaluates on-play card/circuit modifiers, updates power
   │   └── Env returns (next_obs, reward=0, terminated=False, info)
   └── If Action == 27 (PASS):
       ├── Opponent heuristic selects and executes its placement sequence
       ├── Engine resolves end-of-round triggers and power modifiers
       ├── If Round < 6:
       │   ├── Advance to Round R+1: Draw 1 card, activate circuit (if R+1 <= 3), allocate Energy = R+1
       │   └── Env returns (next_obs, reward=0, terminated=False, info)
       └── If Round == 6:
           ├── Final match scoring: compare powers across 3 circuits
           ├── Terminal reward assigned (+1 Win, 0 Draw, -1 Loss)
           └── Env returns (terminal_obs, reward, terminated=True, info)
```

---

## Training Flow

```text
[Research Config]
       │ (Fixed Budget, Matched Seeds)
       ▼
[Experiment Orchestrator]
       │
       ├── Spawn Run: DQN (Seed S_i)  ──► [Common Gymnasium Env] ──► TensorBoard / Logs
       ├── Spawn Run: PPO (Seed S_i)  ──► [Common Gymnasium Env] ──► TensorBoard / Logs
       └── Spawn Run: A2C (Seed S_i)  ──► [Common Gymnasium Env] ──► TensorBoard / Logs
                                                  │
                                                  ▼
                                       Checkpoints & Frozen Policies
```

- Each algorithm trains headlessly on the identical Gymnasium environment.
- Periodic checkpoints evaluate policy snapshots against the heuristic opponent.

---

## Evaluation Flow

```text
[Frozen Policy Checkpoint]
       │
       ▼
[Evaluation Harness]
       │
       ├── Evaluate on Held-Out Evaluation Seed Set (e.g., Seeds 1001–1100)
       │       └── Compute: Win Rate, Mean Return, Episode Lengths
       ▼
[Statistical Aggregation]
       ├── Mean & Standard Deviation across training seeds (Stability Metric)
       ├── Interquartile Mean (IQM) across evaluation episodes
       └── 95% Bootstrap Confidence Intervals
```

---

## Current vs Planned Components

| Component | Status | Implementation Target |
| :--- | :--- | :--- |
| **Specifications (`Phase6A.md`, `Phase6B.md`)** | **Completed** | Authoritative foundation |
| **Documentation Baseline (`README`, `AGENTS`, `ARCH`, `REQ`)** | **Current Stage** | Root documentation standards |
| **Python Core Engine (`hypercards.core`)** | Planned | Phase 5–7: Game state, rules, resolution |
| **Heuristic Opponent (`hypercards.core.opponent`)** | Planned | Phase 8: Deterministic baseline player |
| **Gymnasium Wrapper (`hypercards.env`)** | Planned | Phase 4, 9, 10: Action mask, vector obs, Gym API |
| **SB3 Algorithmic Integration (`hypercards.agent`)** | Planned | Phase 11: DQN, PPO, A2C masking adapters |
| **Training & Evaluation Suite (`hypercards.experiments`)** | Planned | Phase 12: Automated seed runs, metrics, stats |
| **Godot 4.7.2 Client (`godot/`)** | Planned | Phase 13: Presentation, UI, demo frontend |
