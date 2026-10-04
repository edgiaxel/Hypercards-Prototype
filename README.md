# Racing Cards Championship

A research-oriented reinforcement learning project for comparing **DQN, PPO, and A2C** in a custom turn-based card game environment.

The project combines a playable card game prototype with a headless reinforcement learning environment. The same game rules and evaluation conditions are used across the three algorithms so their performance can be compared under controlled conditions.

---

## Research Goal

The primary goal is to investigate how different reinforcement learning algorithms perform when learning to play the same discrete, turn-based strategic game.

The main algorithms are:

- **DQN** — Deep Q-Network
- **PPO** — Proximal Policy Optimization
- **A2C** — Advantage Actor-Critic

The algorithms will use the same:

- Game environment
- Observation representation
- Action semantics
- Opponent behavior
- Reward structure
- Interaction budget
- Evaluation procedure
- Hardware and software environment
- Experimental seeds

The final experimental configuration will be frozen before the main comparison is conducted.

---

## Game

**Racing Cards Championship** is a 1v1 turn-based card game inspired by the structure of games such as Marvel Snap, but themed around motorsport and hypercars.

A match consists of:

- 6 rounds
- 3 scoring **circuits**
- 12-card decks
- A starting hand of 4 cards
- Increasing Energy from Round 1 to Round 6
- Random card draws
- Cards with different Energy costs and Power values
- Simple card abilities
- A fixed heuristic opponent

Players deploy cards to circuits and attempt to win a majority of the three circuits.

The exact card values, circuit effects, reward design, and other balance-related details are **provisional during prototyping** and may change as the environment is tested.

---

## Project Architecture

The project is divided into several major components:

```text
                    ┌─────────────────────┐
                    │     Game Logic      │
                    │  Rules / State /    │
                    │ Actions / Scoring   │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
        ┌────────▼────────┐        ┌────────▼────────┐
        │     Godot       │        │    Gymnasium    │
        │ Presentation /  │        │ RL Environment  │
        │ UI / Demo       │        │   Headless      │
        └─────────────────┘        └────────┬────────┘
                                           │
                                  ┌────────▼────────┐
                                  │ Reinforcement   │
                                  │    Learning     │
                                  └────────┬────────┘
                                           │
                              ┌────────────┼────────────┐
                              │            │            │
                            DQN          PPO          A2C
                              │            │            │
                              └────────────┼────────────┘
                                           │
                                  ┌────────▼────────┐
                                  │    Evaluation   │
                                  │ & Comparison    │
                                  └─────────────────┘
```

### Godot

Godot is responsible primarily for:

- Game presentation
- Menus
- Battle interface
- Card visualization
- Human-playable prototype
- AI demonstration
- Final presentation

### Python / Gymnasium

Python is responsible primarily for:

- Headless game environment
- RL interaction
- Observation generation
- Action handling
- Reward calculation
- Training
- Evaluation

### Stable-Baselines3 / PyTorch

Stable-Baselines3 is used for the reinforcement learning implementations, with PyTorch providing the underlying machine learning framework.

---

## Repository Structure

The project is organized so that game logic, the RL environment, training, evaluation, and presentation remain separated.

The structure will evolve during development, but the intended organization is approximately:

```text
Hypercards-Prototype/
│
├── .venv/
│
├── godot/
│   ├── scenes/
│   ├── scripts/
│   ├── assets/
│   └── ...
│
├── python/
│   ├── game/
│   ├── environment/
│   ├── training/
│   ├── evaluation/
│   └── ...
│
├── tests/
│
├── AGENTS.md
├── REQUIREMENTS.md
├── DESIGN.md
├── ARCHITECTURE.md
└── README.md
```

The exact directory structure may change as implementation progresses.

---

## Reinforcement Learning Environment

The RL environment is designed around a discrete action space.

Conceptually, the agent can:

```text
Play Card → Target Circuit
```

or:

```text
Pass
```

The environment must only expose legal actions to the agent through the selected invalid-action handling approach.

The observation is intended to be a flat numerical representation containing information such as:

- Current round
- Current Energy
- Circuit Power
- Available cards
- Card costs
- Card Power
- Card abilities
- Remaining cards

The initial reward concept is:

```text
Win   = +1
Draw  =  0
Loss  = -1
```

with no intermediate reward.

These elements remain subject to refinement during prototyping and will be finalized before the main experiment.

---

## Opponent

The RL agent plays against a **fixed heuristic opponent** rather than another learning agent.

The opponent evaluates legal actions using game-state information such as:

- Expected immediate circuit advantage
- Resource efficiency
- Strategic deficits

The highest-scoring legal action is selected.

This keeps the opponent behavior controlled across experiments and avoids introducing self-play as an additional research variable.

---

## Experimental Comparison

The final experiment follows the general structure:

```text
                    SAME GAME
                        │
             ┌──────────┼──────────┐
             │          │          │
            DQN        PPO        A2C
             │          │          │
             └──────────┼──────────┘
                        │
                SAME EVALUATION
                        │
               STATISTICAL ANALYSIS
```

The primary comparison will consider factors such as:

- Win / success rate
- Episode return
- Episode length
- Learning curves
- Sample efficiency
- Training stability
- Variance
- Training time
- Computational behavior

Multiple random seeds will be used to reduce the influence of individual training runs.

---

## Development Philosophy

This repository is currently a **prototype**.

The prototype is intentionally used to discover what works before the final research environment is frozen.

The development process is:

```text
Experiment
    ↓
Observe
    ↓
Evaluate
    ↓
Refine
    ↓
Document
    ↓
Freeze
    ↓
Run Final Experiment
```

This means that during early development, elements such as:

- Card statistics
- Circuit effects
- Reward design
- Action representation
- Observation representation
- Opponent heuristics
- Game balance

may change.

Once the final experimental configuration is established, these variables will be documented and frozen so that DQN, PPO, and A2C can be compared under consistent conditions.

---

## Current Status

### Documentation

- [x] Research direction
- [x] Requirements
- [x] Game design
- [x] System architecture
- [x] Development guidelines
- [ ] Final experimental specification

### Prototype

- [ ] Core game state
- [ ] Card system
- [ ] Circuit system
- [ ] Energy system
- [ ] Turn / round system
- [ ] Action system
- [ ] Heuristic opponent
- [ ] Scoring
- [ ] Gymnasium environment
- [ ] Environment tests
- [ ] First random-agent test

### Reinforcement Learning

- [ ] DQN integration
- [ ] PPO integration
- [ ] A2C integration
- [ ] Training pipeline
- [ ] Evaluation pipeline
- [ ] Multi-seed experiments
- [ ] Statistical comparison

### Godot

- [ ] Main menu
- [ ] Battle prototype
- [ ] Card UI
- [ ] Circuit UI
- [ ] Human gameplay
- [ ] AI demonstration
- [ ] Final visual assets

---

## Documentation

The repository contains several documents describing different aspects of the project:

| Document | Purpose |
|---|---|
| `README.md` | Project overview and quick reference |
| `AGENTS.md` | Development and AI-agent guidelines |
| `REQUIREMENTS.md` | Functional and research requirements |
| `DESIGN.md` | Game and gameplay design |
| `ARCHITECTURE.md` | Technical system architecture |

These documents should remain consistent with one another as the project evolves.

---

## Technology Stack

### Game / Presentation

- **Godot 4.7.2**
- GDScript

### Machine Learning

- **Python**
- **Gymnasium**
- **Stable-Baselines3**
- **PyTorch**

### Development

- Git
- GitHub
- Antigravity IDE
- Linux / Windows development environments

---

## Research Principle

The central principle of this project is:

> **One game. One environment. Multiple algorithms. Controlled comparison.**

The game exists to provide a consistent environment in which different reinforcement learning algorithms can be trained and evaluated.

The goal is not to build the largest or most complicated card game possible.

The goal is to build a sufficiently strategic, reproducible, and well-defined environment that allows a meaningful comparison between **DQN, PPO, and A2C**.