# ARCHITECTURE.md

# Hypercards — Software Architecture

## 1. Overview

Hypercards is divided into several major systems:

```text id="6h7f5e"
                    ┌──────────────────────┐
                    │       Godot         │
                    │   Game / UI Layer   │
                    └──────────┬───────────┘
                               │
                               │
                    ┌──────────▼───────────┐
                    │    Game Logic Layer  │
                    │ Rules / State / Data │
                    └──────────┬───────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
       ┌─────────────────────┐    ┌─────────────────────┐
       │ Human / Demo Control│    │ Gymnasium Environment│
       └─────────────────────┘    └──────────┬──────────┘
                                             │
                                  ┌──────────┼──────────┐
                                  │          │          │
                                  ▼          ▼          ▼
                                DQN        PPO        A2C
```

The architecture separates:

- Game rules
- Game state
- Graphical presentation
- Human interaction
- RL environment
- RL algorithms
- Training
- Evaluation

The purpose of this separation is to make the project easier to develop, test, modify, and reuse for the final thesis experiment.

---

# 2. Architectural Principles

The architecture follows several principles.

## 2.1 Separation of Concerns

Each system should have one primary responsibility.

For example:

- Game logic should not be responsible for drawing UI.
- UI should not determine game rules.
- RL algorithms should not implement game rules.
- Training scripts should not contain card definitions.
- Evaluation scripts should not modify the environment.

---

## 2.2 Single Source of Truth

Important game rules should have one authoritative implementation.

Examples include:

- Energy calculation
- Card legality
- Circuit scoring
- Match scoring
- Round progression
- Card drawing
- Game termination

The same rules should not be independently rewritten in Godot and Python.

---

## 2.3 Headless RL Environment

The reinforcement learning environment must be capable of operating without graphical rendering.

The RL training process should not require:

- Godot UI
- Mouse input
- Keyboard input
- Animation
- Rendering
- Real-time timing

This allows large numbers of episodes to be executed efficiently.

---

## 2.4 Research Reproducibility

The architecture must support:

- Controlled random seeds
- Configurable game parameters
- Reproducible environment behavior
- Consistent observations
- Consistent actions
- Consistent rewards
- Repeatable evaluation

---

# 3. High-Level System Architecture

The system can be viewed as six major layers.

```text id="n0r7sm"
┌───────────────────────────────────────────────┐
│                  Presentation                 │
│                                               │
│             Godot Game / UI                  │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│                  Game Layer                   │
│                                               │
│      Game State / Rules / Cards / Circuits   │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│                Environment Layer              │
│                                               │
│             Gymnasium Interface               │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│                 Agent Layer                   │
│                                               │
│          DQN / PPO / A2C via SB3             │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│                 Training Layer                │
│                                               │
│       Training / Checkpoints / Logging       │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│                Evaluation Layer               │
│                                               │
│     Metrics / Comparison / Statistical Data  │
└───────────────────────────────────────────────┘
```

---

# 4. Game Layer

The Game Layer contains the actual rules of Racing Cards Championship.

It should be independent from the visual presentation.

The Game Layer is responsible for:

- Match state
- Player state
- Cards
- Decks
- Hands
- Circuits
- Energy
- Rounds
- Actions
- Legal-action determination
- Card effects
- Opponent decisions
- Circuit scoring
- Match scoring
- Episode termination

---

# 5. Game State

The game state represents everything required to determine what can happen next.

Conceptually:

```text id="h8o5g4"
GameState
│
├── Current Round
├── Players
│   ├── Player State
│   │   ├── Hand
│   │   ├── Deck
│   │   ├── Energy
│   │   └── Circuit Power
│   │
│   └── Opponent State
│       ├── Hand
│       ├── Deck
│       ├── Energy
│       └── Circuit Power
│
├── Circuits
├── Current Phase
├── Random State
└── Match Result
```

The exact implementation language and class structure may differ between the prototype and final implementation.

---

# 6. Card Data

Card definitions should be represented as data rather than duplicated throughout gameplay code.

A card conceptually contains:

```text id="i7w6oh"
Card
├── ID
├── Name
├── Manufacturer
├── Energy Cost
├── Power
├── Card Type
└── Ability
```

Card data should be easy to modify during prototype balancing.

This allows the game to experiment with different:

- Power values
- Energy costs
- Abilities
- Card distributions

without rewriting core game logic.

---

# 7. Circuit Data

Circuits should similarly be represented as configurable data.

A circuit conceptually contains:

```text id="3fkwz9"
Circuit
├── ID
├── Name
├── Description
└── Effect
```

The exact circuit names and effects are provisional during prototyping.

Circuit behavior should be implemented through a clearly defined mechanism rather than hard-coded throughout unrelated game systems.

---

# 8. Rules Engine

The Rules Engine is responsible for applying the game's rules.

Potential responsibilities include:

```text id="9gyq8c"
calculate_energy()
draw_card()
get_legal_actions()
validate_action()
apply_action()
resolve_round()
calculate_circuit_scores()
calculate_match_result()
is_terminal()
```

These names are conceptual.

The final implementation may use different names or class structures.

The important requirement is that game rules remain centralized and testable.

---

# 9. Action System

Actions represent decisions available to a player or RL agent.

The primary conceptual actions are:

```text id="2upz0u"
Play Card → Circuit
Pass
```

The action system should provide:

- Action representation
- Action validation
- Legal-action determination
- Action execution
- Action encoding/decoding for the RL environment

The visual UI may represent an action using buttons and card selection, while the RL environment represents the same action using a discrete numerical action.

---

# 10. Opponent System

The opponent system controls the non-learning opponent.

The primary opponent is a fixed heuristic.

The opponent system should:

1. Receive the current game state.
2. Determine legal actions.
3. Evaluate candidate actions.
4. Select an action.
5. Return the selected action to the game system.

The heuristic itself should remain separate from the core game rules.

This allows the heuristic to be modified without rewriting the game engine.

---

# 11. Godot Presentation Layer

Godot is responsible primarily for presentation and interaction.

The Godot layer includes:

- Main menu
- Battle interface
- Card UI
- Circuit UI
- Energy display
- Round display
- Result screen
- AI demonstration
- Settings
- Visual effects
- Animations

The UI should read game state rather than independently maintaining a second version of the game state.

---

# 12. Godot Scene Structure

The exact scene hierarchy is not yet finalized.

A conceptual structure may eventually resemble:

```text id="f0y8vl"
Main
│
├── MainMenu
│
├── Game
│   ├── OpponentArea
│   ├── CircuitArea
│   │   ├── Circuit1
│   │   ├── Circuit2
│   │   └── Circuit3
│   ├── PlayerArea
│   ├── Hand
│   ├── EnergyDisplay
│   ├── RoundDisplay
│   └── ActionControls
│
├── ResultScreen
│
└── Settings
```

This is a conceptual structure, not a requirement to create all nodes immediately.

---

# 13. Prototype UI

The prototype UI should prioritize functionality.

Initial UI elements may use:

- Basic Godot Control nodes
- Panels
- Labels
- Buttons
- Containers
- Simple shapes
- Placeholder card artwork

Final artwork is not required for the first functional prototype.

---

# 14. Python Environment Layer

The Python environment provides the standardized interface between the game logic and reinforcement learning algorithms.

The environment is responsible for translating between:

```text id="9zppf5"
Game State
     ↕
Gymnasium Observation / Action
```

It should expose the standard environment operations required by Stable-Baselines3.

Conceptually:

```text id="y8gc8d"
reset()
step(action)
observation_space
action_space
```

Additional helper methods may be implemented as required.

---

# 15. Environment Responsibilities

The Gymnasium environment should:

- Initialize a game
- Reset the game
- Provide an observation
- Accept an action
- Validate or handle the action
- Advance the game
- Generate a reward
- Determine termination
- Return the next observation

The environment should not contain presentation code.

---

# 16. Observation Adapter

The RL agent should not directly consume complex game objects.

An observation adapter should translate the internal game state into the numerical observation expected by the model.

Conceptually:

```text id="1eq6v5"
Game State
     │
     ▼
Observation Adapter
     │
     ▼
Numerical Observation
     │
     ▼
RL Algorithm
```

The observation representation should contain the information necessary for the agent to make decisions.

---

# 17. Action Adapter

The RL environment should translate the agent's discrete action into an internal game action.

Conceptually:

```text id="e7f6pf"
RL Action Index
      │
      ▼
Action Adapter
      │
      ▼
Game Action
      │
      ▼
Game Rules
```

For example:

```text
Action 0
→ Play Card 1 on Circuit 1

Action 1
→ Play Card 1 on Circuit 2

Action 2
→ Play Card 1 on Circuit 3

...

Action N
→ Pass
```

The exact action indexing scheme is provisional.

---

# 18. Reward Adapter

The environment converts the final game outcome into an RL reward.

The initial conceptual reward is:

```text id="b7s6cz"
Win   → +1
Draw  →  0
Loss  → -1
```

The prototype may investigate alternative reward designs.

The reward implementation should be isolated so that reward experiments do not require rewriting unrelated game logic.

Once the final reward is frozen, the same reward definition must be used for DQN, PPO, and A2C.

---

# 19. RL Algorithm Layer

Stable-Baselines3 is intended to provide the primary implementations of:

```text id="q5u3hz"
DQN
PPO
A2C
```

The algorithms should interact with the same Gymnasium environment.

Conceptually:

```text
                  Gymnasium Environment
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
         DQN            PPO            A2C
```

The algorithm implementation itself should not contain game-specific rules.

---

# 20. Training Layer

The Training Layer is responsible for running experiments.

Potential responsibilities include:

- Model initialization
- Hyperparameter configuration
- Training timesteps
- Random seed configuration
- Checkpointing
- Logging
- Model saving
- Experiment metadata

Training scripts should not duplicate game rules.

---

# 21. Configuration

Experiment configuration should be separated from implementation where practical.

Potential configuration values include:

```text id="bqfrq8"
Algorithm
Random Seed
Training Timesteps
Environment Configuration
Learning Parameters
Evaluation Parameters
Model Save Location
Logging Location
```

This makes experiments easier to reproduce.

---

# 22. Evaluation Layer

The Evaluation Layer is responsible for evaluating trained models.

Potential metrics include:

- Win rate
- Draw rate
- Loss rate
- Episode return
- Episode length
- Learning curves
- Sample efficiency
- Training time
- Training throughput
- Variance
- Confidence intervals
- Interquartile mean where appropriate

The exact statistical analysis will be defined separately in the research methodology.

The evaluation system must use the same finalized environment configuration for all algorithms.

---

# 23. Experiment Flow

The intended research workflow is:

```text id="4zv7yr"
Game Configuration
        │
        ▼
Gymnasium Environment
        │
        ├──────────────┐
        │              │
        ▼              ▼
      Training       Evaluation
        │              │
        ▼              ▼
    DQN / PPO / A2C
        │
        ▼
   Saved Models
        │
        ▼
   Evaluation Runs
        │
        ▼
 Metrics / Results
        │
        ▼
 Statistical Analysis
```

---

# 24. Godot and Python Relationship

Godot and Python have different primary responsibilities.

## Godot

```text id="t8ghj1"
Presentation
UI
Human Interaction
Visualization
AI Demonstration
```

## Python

```text id="9z2zai"
RL Environment
Training
Evaluation
Experimentation
Data Collection
```

The two systems should communicate through clearly defined interfaces rather than tightly coupling every component.

---

# 25. Prototype Integration Strategy

During early development, the project does not need to immediately achieve full Godot ↔ Python integration.

Development should proceed incrementally.

### Stage 1

Build and test the game logic.

### Stage 2

Build a basic Godot interface around the game.

### Stage 3

Build the Python/Gymnasium environment.

### Stage 4

Test the environment independently.

### Stage 5

Integrate Stable-Baselines3.

### Stage 6

Train DQN, PPO, and A2C.

### Stage 7

Connect trained models to Godot for demonstration.

This minimizes debugging complexity.

---

# 26. Recommended Development Boundary

The architecture should ideally allow:

```text
Godot
   │
   │
   ▼
Game Representation / Visualization

Python
   │
   │
   ▼
Research Environment / Training
```

The research environment should be usable without opening Godot.

Likewise, the Godot game should be capable of running without launching a training process.

---

# 27. Data Flow During a Human Match

A human-controlled match conceptually follows:

```text id="twv1ko"
Player Input
    ↓
Godot UI
    ↓
Game Action
    ↓
Game Rules
    ↓
Updated Game State
    ↓
Godot UI
```

The UI displays the resulting state.

The UI does not independently calculate the result.

---

# 28. Data Flow During RL Training

An RL episode conceptually follows:

```text id="n7r0xk"
RL Agent
    │
    │ action
    ▼
Gymnasium Environment
    │
    ▼
Game Logic
    │
    ▼
Updated Game State
    │
    ├── observation ──→ RL Agent
    │
    └── reward ───────→ RL Agent
```

This cycle continues until the episode terminates.

---

# 29. Data Flow During AI Demonstration

When a trained model is demonstrated through Godot:

```text id="d8g3ct"
Godot
  │
  ▼
Game State
  │
  ▼
Observation Adapter
  │
  ▼
Trained RL Model
  │
  ▼
Action
  │
  ▼
Action Adapter
  │
  ▼
Game Logic
  │
  ▼
Updated Game State
  │
  ▼
Godot UI
```

The demonstration should use the same game rules as the research environment.

---

# 30. Testing Architecture

Testing should occur at multiple levels.

## Unit Testing

Test individual systems such as:

- Card calculations
- Energy calculations
- Circuit scoring
- Action validation
- Reward calculation

## Game-Level Testing

Test:

- Complete matches
- Round transitions
- Card drawing
- Match scoring
- Invalid actions
- Terminal states

## Environment Testing

Test:

- `reset()`
- `step()`
- Observation shape
- Action validity
- Reward output
- Episode termination
- Seed reproducibility

## Training Testing

Test:

- Model initialization
- Environment compatibility
- Short training runs
- Checkpoint saving
- Model loading

---

# 31. Error Handling

Errors should fail clearly rather than silently corrupting the game state.

Examples include:

- Invalid action
- Invalid card ID
- Invalid circuit ID
- Invalid game phase
- Invalid Energy state
- Missing configuration
- Malformed model
- Invalid environment state

Development builds should provide sufficient information to diagnose these problems.

---

# 32. Configuration and Balance Separation

Gameplay balance values should not be scattered throughout the code.

Values such as:

- Card Power
- Card Energy cost
- Circuit modifiers
- Number of rounds
- Starting hand size
- Deck size
- Reward values

should be represented in a way that makes prototype experimentation easy.

The architecture should allow these values to eventually be frozen into a documented experimental configuration.

---

# 33. Assets and Presentation

Assets are considered a presentation layer.

The game should not depend on final artwork to function.

Placeholder assets may be used during development.

The architecture should allow visual assets to be replaced without changing:

- Game rules
- Game state
- RL environment
- Reward system
- Action system
- Observation system

This allows visual development to occur independently from research development.

---

# 34. Prototype-to-Final Transition

The architecture is intended to survive the transition from prototype to final experiment.

The process is:

```text id="cvb5b7"
Prototype
    │
    ├── Test Game Logic
    ├── Test Cards
    ├── Test Circuits
    ├── Test Rewards
    ├── Test Opponent
    └── Test RL Interface
            │
            ▼
      Design Refinement
            │
            ▼
      Configuration Freeze
            │
            ▼
     Final Environment
            │
            ├── DQN
            ├── PPO
            └── A2C
```

Prototype code may be reused when it satisfies the finalized requirements.

Prototype-specific experiments should not automatically be considered part of the final methodology.

---

# 35. Final Experimental Boundary

Once the final configuration is frozen, the following should be treated as controlled experimental components:

```text
Game Rules
Card Configuration
Circuit Configuration
Opponent
Action Space
Observation Space
Reward Function
Randomness
Seed Protocol
Environment Version
Software Version
Hardware Configuration
Training Budget
Evaluation Protocol
```

Changes to these components after freezing should be documented as experimental changes.

---

# 36. Architectural Goal

The final architecture should allow the following statement to be true:

> DQN, PPO, and A2C interact with the same game, using the same state representation, action semantics, opponent, reward definition, and evaluation procedure, while differing primarily in their reinforcement learning algorithm.

This separation is the foundation of the controlled comparison.

---

# 37. Summary

The intended architecture is:

```text id="4r7l6z"
                    RACING CARDS CHAMPIONSHIP
                              │
                              ▼
                       ┌─────────────┐
                       │ Game Logic  │
                       └──────┬──────┘
                              │
               ┌──────────────┴──────────────┐
               │                             │
               ▼                             ▼
          Godot Layer                 Gymnasium Layer
               │                             │
               ▼                             ▼
          Human / UI                    RL Interface
                                             │
                                  ┌──────────┼──────────┐
                                  ▼          ▼          ▼
                                 DQN        PPO        A2C
                                  │          │          │
                                  └──────────┼──────────┘
                                             ▼
                                         Training
                                             │
                                             ▼
                                        Evaluation
                                             │
                                             ▼
                                          Results
```

The central architectural principle is:

**One game. One environment. Multiple algorithms. Controlled comparison.**