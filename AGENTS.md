# AGENTS.md

## Project

**Project Name:** Hypercards  
**Game Title:** Racing Cards Championship

Hypercards is a research-oriented turn-based strategic card game developed as the experimental environment for a university thesis investigating reinforcement learning algorithms.

The primary research comparison is between:

- Deep Q-Network (DQN)
- Proximal Policy Optimization (PPO)
- Advantage Actor-Critic (A2C)

The three algorithms must be evaluated within the same game environment and under controlled experimental conditions.

---

## Purpose of This File

This document provides development instructions for AI coding agents working on the Hypercards repository.

AI agents must understand the project's research purpose before modifying the codebase.

The goal is not simply to create a fun game. The game is also a controlled experimental environment for reinforcement learning research.

Therefore, changes that affect gameplay, randomness, observations, actions, rewards, opponent behavior, or other experimental variables must be treated as research-impacting changes.

---

# 1. Core Development Principles

## 1.1 Preserve the Research Scope

Do not expand the project beyond the established thesis scope unless explicitly instructed by the project owner.

The current core scope is:

- 1v1 turn-based strategic card game
- Six rounds
- Three racing circuits
- 12-card hypercar deck
- Randomized card draws
- Increasing Energy by round
- Cards with Energy costs and Power values
- Card placement into circuits
- Simple card abilities
- Fixed heuristic opponent
- Terminal game reward
- DQN, PPO, and A2C reinforcement learning agents
- Standardized Gymnasium environment
- Controlled experimental evaluation

Do not introduce major systems such as:

- Deck building
- Player progression
- Leveling
- Shops
- Currency
- Multiplayer
- Matchmaking
- Self-play
- Procedurally generated maps
- Complex equipment systems
- Large-scale ability/combo systems
- Damage, health, tire, fuel, pit-stop, or weather simulation
- Animation-dependent gameplay mechanics

unless explicitly approved.

---

# 2. Do Not Change Research Variables Without Approval

The following are experimental variables or potentially experimental variables:

- Action space
- Observation space
- Reward function
- Opponent behavior
- Game rules
- Card Power
- Card Energy cost
- Circuit effects
- Number of rounds
- Number of circuits
- Deck composition
- Card draw rules
- Randomization
- Invalid-action handling
- Action masking
- Training interaction budget
- Evaluation procedure
- Random seed handling

An AI agent must not silently modify these systems for convenience or perceived gameplay improvement.

If a proposed code change would alter one of these variables:

1. Explain what would change.
2. Explain why the change is necessary.
3. Identify the potential effect on the experiment.
4. Ask for explicit approval before implementing the change if the change is not already specified by the project design.

---

# 3. Game Design Is Not Automatically Final

Some game values are intentionally not finalized yet.

In particular, the following may be refined during the game-balancing stage:

- Exact card Power values
- Exact Energy costs
- Exact circuit effects
- Exact circuit names
- Card ability values
- Card distribution and balance
- Randomization rules
- Heuristic weighting
- UI presentation

Do not invent final values merely because a value is currently missing.

Use clearly identified provisional values only when implementation requires them, and mark them as provisional.

---

# 4. Circuits

The game uses **three racing circuits**.

The term **circuit** should be used consistently throughout the code, documentation, UI, and research environment.

Do not refer to them as "bases", "lanes", or other alternative terminology unless explicitly requested.

The final circuits may use names inspired by real-world endurance racing/WEC circuits, potentially with randomized selection or assignment.

However, the exact circuit identities and their gameplay effects are not finalized.

Do not introduce real-world circuit names, layouts, or gameplay modifiers as permanent design decisions without explicit approval.

---

# 5. Game Architecture

The project should maintain a clear separation between:

1. Game rules and state
2. Graphical presentation
3. RL environment interface
4. RL algorithms
5. Training
6. Evaluation

The graphical game should not become a dependency of the headless reinforcement learning training environment.

The intended conceptual architecture is:

```text
Game Logic
    │
    ├── Godot Game / UI
    │
    └── Gymnasium Environment
            │
            ├── DQN
            ├── PPO
            └── A2C
```

Training must be capable of running without rendering the graphical game.

---

# 6. Godot Development

Godot is primarily responsible for:

- Game presentation
- User interface
- Scenes
- Visual card representation
- Human interaction
- AI demonstration
- Game visualization
- Results presentation

Do not place research training loops directly into Godot unless explicitly required.

Avoid tightly coupling game presentation code with RL training code.

The game should remain playable and testable independently of the training system.

---

# 7. Python / RL Development

Python is responsible for:

- Gymnasium environment
- RL-compatible game interface
- Training
- Evaluation
- Experiment scripts
- Statistical analysis
- Model management

The intended RL framework is:

- Python
- Gymnasium
- Stable-Baselines3
- PyTorch

The primary algorithms are:

- DQN
- PPO
- A2C

The same environment and game rules should be used when comparing the three algorithms.

---

# 8. Environment Requirements

The RL environment should provide:

- A discrete action space
- A numerical observation representation
- Legal-action handling
- Reset functionality
- Step functionality
- Reward output
- Episode termination
- Deterministic behavior when controlled by a seed

The environment should not depend on:

- GUI rendering
- Screen coordinates
- Mouse input
- Animation timing
- Real-time delays
- Human interaction

The environment exists for reinforcement learning experiments first.

---

# 9. Opponent

The default opponent is a fixed heuristic opponent.

The opponent is not another learning agent.

The heuristic should remain consistent between training and evaluation unless a research experiment explicitly specifies otherwise.

Do not replace the heuristic opponent with:

- Self-play
- Random-only behavior
- Another RL agent
- A dynamically learning opponent

without explicit approval.

Changes to heuristic behavior can change the difficulty of the environment and therefore affect experimental validity.

---

# 10. Rewards

The intended primary reward design is terminal:

```text
Win   = +1
Draw  =  0
Loss  = -1
```

Intermediate rewards are initially zero.

Do not add reward shaping merely to make an agent learn faster.

Reward modifications are research design changes and must be explicitly approved.

Reward hacking must be considered whenever modifying the reward system.

---

# 11. Randomness

Randomness is allowed where it is part of the game design.

Examples include:

- Deck shuffling
- Initial hand
- Card draws
- Potential heuristic tie-breaking

The core game rules should remain deterministic when the random seed is controlled.

Do not introduce unnecessary randomness into:

- Game rules
- Rewards
- Observations
- Opponent decisions
- Evaluation

unless explicitly required.

---

# 12. Action and Observation Design

The action space and observation space are part of the research environment.

Do not change them casually.

The current conceptual action structure is:

```text
Play Card + Target Circuit
OR
Pass
```

The environment may internally represent these choices as a discrete action index.

The observation should represent the game state numerically rather than relying on screenshots or computer vision.

The intended approach is a flat numerical observation representation suitable for an MLP-based policy/value network.

---

# 13. Invalid Actions

Invalid actions should be handled consistently.

Examples include:

- Playing a card that is not in the hand
- Playing a card that costs more Energy than currently available
- Selecting an otherwise illegal placement

The environment should expose legal-action information or use an appropriate invalid-action handling mechanism.

Do not solve invalid actions by silently changing the game state.

---

# 14. Code Quality

Prefer:

- Small, understandable functions
- Clear names
- Explicit data structures
- Modular systems
- Reusable game logic
- Minimal duplication
- Comments where reasoning is non-obvious

Avoid:

- Giant monolithic scripts
- Unnecessary abstraction
- Hidden global state
- Magic numbers scattered throughout code
- Duplicate implementations of the same game rule
- Hard-coded UI logic inside core game rules

Game rules should have a clear source of truth.

---

# 15. File Organization

Keep related systems separated.

Do not create new files or directories simply because they appear convenient.

Before introducing a new architectural component:

1. Check the existing project structure.
2. Check `ARCHITECTURE.md`.
3. Determine whether an existing component already performs the required responsibility.
4. Reuse existing structures when appropriate.

Avoid duplicate systems with overlapping responsibilities.

---

# 16. AI Agent Behavior

Before making significant changes, inspect:

- `README.md`
- `AGENTS.md`
- `REQUIREMENTS.md`
- `DESIGN.md`
- `ARCHITECTURE.md`

Do not assume that an omitted feature is an accidental omission.

Do not redesign existing systems without instruction.

Do not replace working implementations with completely different architectures without explaining the reason.

When requirements are ambiguous, prefer asking for clarification rather than inventing major behavior.

---

# 17. Implementation Strategy

Build incrementally.

The preferred development sequence is:

```text
1. Project foundation
2. Basic game state
3. Card data
4. Circuit system
5. Energy system
6. Player actions
7. Opponent
8. Round progression
9. Scoring
10. Complete playable game
11. Godot UI
12. Gymnasium environment
13. RL integration
14. Training
15. Evaluation
```

Do not attempt to implement the entire game, environment, and RL system in one step.

Each subsystem should be testable before the next major subsystem is built.

---

# 18. Testing

When modifying game rules:

- Test the affected rule directly.
- Test normal gameplay.
- Test edge cases.
- Test invalid actions.
- Verify that unrelated systems still behave correctly.

When modifying the RL environment:

- Verify `reset()`.
- Verify `step()`.
- Verify observations.
- Verify actions.
- Verify rewards.
- Verify termination.
- Verify deterministic behavior with fixed seeds.

When modifying experimental code:

- Record what changed.
- Consider whether previous experiment results remain comparable.

---

# 19. Documentation

Important design decisions should be documented.

If implementation differs from the documented design:

- Update the appropriate documentation.
- Do not silently allow the documentation and implementation to diverge.

Use:

- `README.md` for project overview and setup
- `REQUIREMENTS.md` for system requirements
- `DESIGN.md` for game rules and game design
- `ARCHITECTURE.md` for software architecture
- `AGENTS.md` for development-agent instructions

---

# 20. Git

Make focused commits.

Avoid combining unrelated changes into one commit.

Prefer commits such as:

```text
Add initial Godot project structure
Implement card data model
Implement circuit scoring
Add heuristic opponent
Add Gymnasium environment
Add DQN training configuration
```

Do not commit generated files that are intended to remain local, such as Godot's generated project cache.

Never create a second Git repository inside the project.

---

# 21. Thesis Integrity

This project is part of an academic research project.

Code must prioritize:

- Reproducibility
- Experimental consistency
- Traceability
- Clear documentation
- Controlled variables
- Honest evaluation

Do not optimize the system specifically to produce better results for one algorithm.

Do not cherry-pick successful training runs while hiding unsuccessful runs.

Do not modify the environment differently for DQN, PPO, and A2C unless the experimental design explicitly requires it.

The purpose of the project is to compare the algorithms fairly, not to make one algorithm win.

---

# 22. When Unsure

When a requested change conflicts with the documented research design:

**Stop and ask.**

When a value has not been finalized:

**Do not invent a permanent value.**

When a feature is outside the documented scope:

**Do not add it automatically.**

When a change could affect experimental validity:

**Identify the effect before implementing it.**

The project's research integrity takes priority over convenience.