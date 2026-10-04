# DESIGN.md

# Hypercards — Game Design Document

## 1. Game Overview

**Project:** Hypercards  
**Game Title:** Racing Cards Championship

Racing Cards Championship is a single-player, turn-based strategic card game centered around fictional hypercars and racing circuits.

The player competes against a fixed heuristic opponent across multiple racing circuits. Players deploy hypercar cards using limited Energy and attempt to achieve the highest Power across the circuits.

The game is designed primarily as a reinforcement learning research environment.

The game therefore prioritizes:

- Clear state representation
- Discrete actions
- Strategic decisions
- Controlled randomness
- Reproducibility
- Simple but meaningful game mechanics
- Compatibility with reinforcement learning

The game is intentionally small in scope.

---

# 2. Design Philosophy

The game should be:

### Simple enough to model

The game must have a state that can be represented numerically and an action space that reinforcement learning algorithms can reasonably explore.

### Strategic enough to be interesting

The best action should not always be immediately obvious.

Players should have to consider:

- Current circuit strength
- Remaining Energy
- Available cards
- Opponent positioning
- Future rounds
- Card abilities
- Which circuits are worth contesting

### Controlled enough for research

The game should avoid unnecessary mechanics that introduce uncontrolled experimental variables.

The goal is not to reproduce the complexity of a full commercial card game.

---

# 3. Prototype Philosophy

The first implementation is a **gameplay and research prototype**.

The prototype is intended to answer:

- Does the game work?
- Are the rules understandable?
- Is the action space manageable?
- Are decisions strategically meaningful?
- Can an RL agent interact with the game?
- Does the reward design work?
- Are cards balanced enough to create meaningful decisions?
- Does the heuristic opponent provide an appropriate challenge?

The prototype does not need final-quality graphics.

---

# 4. Prototype Presentation vs Final Presentation

The project will use two presentation stages.

## 4.1 Logic Prototype

The initial prototype may use:

- Simple colored panels
- Rectangles
- Text labels
- Basic buttons
- Placeholder card graphics
- Placeholder circuit graphics
- Simple background
- Basic animations or no animations

The purpose is to demonstrate functionality.

The prototype should not be delayed because final art is unavailable.

---

## 4.2 Final Presentation

After the gameplay and research design have stabilized, visual assets can be replaced or improved.

Potential final assets include:

- Main menu background
- Racing-themed background
- Hypercar illustrations
- Card artwork
- Circuit artwork
- Logos
- UI elements
- Icons
- Buttons
- Result screens
- Animations
- Visual effects

Final assets are a presentation concern and should not change the underlying game rules.

---

# 5. Match Structure

The initial match structure is:

```text
Players:        2
Rounds:         6
Circuits:       3
Deck:           12 cards
Starting hand:  4 cards
```

The RL player controls one side.

The opponent uses a fixed heuristic strategy.

---

# 6. Racing Circuits

The match contains three racing circuits.

Each circuit represents a separate scoring area.

The player and opponent each accumulate Power in every circuit.

At the end of the match, the Power values are compared.

---

## 6.1 Circuit Identity

The final circuit identities have not yet been finalized.

The design may eventually use:

- Real-world endurance/WEC circuit names
- A randomized selection of circuits
- Fictionalized circuit names inspired by endurance racing
- Another naming system

No final decision should be assumed until the prototype design is finalized.

---

## 6.2 Circuit Effects

Each circuit may have a unique gameplay modifier.

The current design direction is for circuit-specific effects to create different strategic priorities.

For example, a circuit could influence:

- Card Power
- Energy cost
- Late-round performance
- Other carefully controlled card properties

Exact effects are provisional.

The final circuit effects must be selected through prototype testing and documented before the experimental environment is frozen.

---

# 7. Cards

Each player uses a deck of hypercar cards.

Cards represent fictional high-performance racing machines.

Each card may contain:

- Name
- Manufacturer
- Energy cost
- Power
- Card type
- Ability

---

# 8. Card Design Philosophy

Cards should create meaningful trade-offs.

A card should not simply be:

> "Higher number = always better."

Players should need to decide:

- When to play a card
- Where to play it
- Whether to save Energy
- Whether to contest a losing circuit
- Whether to secure a winning circuit
- Whether a card's ability is more valuable now or later

---

# 9. Initial Card Concept

The prototype may begin with a 12-card deck inspired by the initial Phase 6B concept.

The initial conceptual cards are:

| # | Card | Initial Concept |
|---|---|---|
| 1 | Vortex V12 | Low-cost standard card |
| 2 | Asterion R8 | Low-cost efficient card |
| 3 | Tempest R9 | Mid-cost standard card |
| 4 | Valkor V12 | Overtake-oriented card |
| 5 | Auron LMH | Strong mid/high-cost card |
| 6 | Cerberus X | Late-round card |
| 7 | Veltrix 01 | High-power card |
| 8 | Stratos R | Losing-circuit / comeback card |
| 9 | Imperia LMH | High-power card |
| 10 | Phoenix X | Endurance-oriented card |
| 11 | Titan Hyperion | Very high-cost card |
| 12 | Zenith LMH | Final-round card |

These are **prototype concepts rather than permanently finalized statistics**.

The exact Power, Energy costs, abilities, and balance may change substantially during development.

---

# 10. Card Categories

The prototype may use three broad conceptual categories:

### Standard

Straightforward cards primarily defined by their Energy cost and Power.

### Tactical

Cards whose value depends on the current game state.

### Strategic

Cards whose value depends strongly on timing, circuit choice, or later rounds.

These categories are descriptive rather than strict gameplay classes.

---

# 11. Energy System

Energy is the primary resource used to deploy cards.

The initial concept gives the player more Energy as the match progresses.

Initial concept:

```text
Round 1 → 1
Round 2 → 2
Round 3 → 3
Round 4 → 4
Round 5 → 5
Round 6 → 6
```

Unused Energy does not initially carry into the following round.

This system is provisional and may be adjusted during prototype balancing.

---

# 12. Hand and Deck

Each player begins with an initial hand.

The initial concept uses:

```text
Deck:          12 cards
Starting hand: 4 cards
```

Cards are drawn during the match.

The initial concept draws one additional card at the beginning of each round.

Cards leave the player's hand when played.

---

# 13. Player Actions

The primary action is:

> **Play a card onto a circuit.**

An action therefore consists conceptually of:

```text
Card
+
Target Circuit
```

The player may also:

```text
Pass
```

The exact action encoding used by the RL environment will be defined separately from the visual interface.

---

# 14. Multiple Cards Per Round

The initial design does not impose a hard "one card per round" restriction.

A player may deploy multiple cards during a round as long as sufficient Energy remains and the actions are legal.

This allows Energy management to become part of the strategic decision-making process.

---

# 15. Invalid Actions

An action is invalid when it violates the current game state.

Examples:

- Card is not in the player's hand.
- Player does not have enough Energy.
- Card cannot legally be placed in the selected circuit.
- The action is no longer valid because the game state changed.

Invalid actions must not modify the game state incorrectly.

The RL environment should provide an appropriate mechanism for handling or masking invalid actions.

---

# 16. Round Structure

A match consists of six rounds in the initial design.

A conceptual round proceeds as:

```text
Start Round
    ↓
Draw Card
    ↓
Receive Round Energy
    ↓
Determine Legal Actions
    ↓
Player / RL Agent Selects Action
    ↓
Opponent Selects Action
    ↓
Resolve Actions
    ↓
Update Game State
    ↓
Next Round
```

The exact internal ordering may be refined during implementation.

---

# 17. Decision Timing

The game is conceptually designed around simultaneous strategic decision-making.

The player selects an action without seeing the opponent's current decision.

The opponent independently selects an action.

The actions are then resolved.

The underlying implementation may process these actions sequentially while hiding the opponent's decision until resolution.

This is intended to reduce excessive first-player advantage while keeping the environment implementation manageable.

---

# 18. Opponent

The opponent is a fixed heuristic agent.

It does not learn during the primary experiment.

The opponent evaluates legal actions according to game-state information.

Potential considerations include:

- Immediate circuit advantage
- Resource efficiency
- Current deficit
- Resulting strategic position

The heuristic may use deterministic decisions or controlled tie-breaking randomness.

The exact heuristic will be developed and tested during the prototype stage.

---

# 19. Scoring

At the end of the sixth round, the Power of both players is compared in each circuit.

Each circuit produces one of three outcomes:

```text
Player wins circuit
Opponent wins circuit
Circuit is tied
```

The initial match-level concept is:

```text
Win 2+ circuits → Match Win
Lose 2+ circuits → Match Loss
Otherwise       → Match Draw
```

The exact treatment of circuit ties may be refined during prototype testing.

---

# 20. Reward Design

Reward design is intentionally provisional.

The initial research concept uses a terminal reward:

```text
Win   → +1
Draw  →  0
Loss  → -1
```

Intermediate rewards are initially zero.

However, the prototype exists specifically to test whether this reward structure is appropriate.

Potential alternatives may be investigated if the initial reward produces:

- Poor learning
- Sparse-reward problems
- Unintended strategies
- Reward exploitation
- Insufficient learning signal

Any final reward modification must be documented and applied consistently across all algorithms.

---

# 21. Randomness

Controlled randomness is part of the game.

Potential sources include:

- Deck shuffle
- Starting hand
- Card draws
- Heuristic tie-breaking

The core game rules should remain deterministic when the random seed is controlled.

Randomness should not be introduced merely to make the game appear more complex.

---

# 22. Observation Design

The RL agent should receive a numerical representation of the current game state.

The conceptual observation may contain:

```text
Current round
Current Energy
Own circuit Power
Opponent circuit Power
Cards currently available
Card costs
Card Power
Relevant ability information
Cards remaining
```

The final representation will be determined during environment implementation and prototype testing.

The initial target is a flat numerical vector suitable for an MLP.

---

# 23. Action Space Design

The conceptual action space consists of:

```text
Play Card × Target Circuit
+
Pass
```

For example, if four cards are currently available and three circuits exist:

```text
Card 1 → Circuit 1
Card 1 → Circuit 2
Card 1 → Circuit 3

Card 2 → Circuit 1
Card 2 → Circuit 2
Card 2 → Circuit 3

Card 3 → Circuit 1
Card 3 → Circuit 2
Card 3 → Circuit 3

Card 4 → Circuit 1
Card 4 → Circuit 2
Card 4 → Circuit 3

Pass
```

This produces 13 conceptual actions for a four-card hand.

The exact implementation may use a fixed action encoding or another representation suitable for the finalized environment.

---

# 24. Game Flow

The complete conceptual game flow is:

```text
Main Menu
    │
    ├── New Championship
    │       ↓
    │   Create Match
    │       ↓
    │   Shuffle Decks
    │       ↓
    │   Draw Starting Hand
    │       ↓
    │   Round 1
    │       ↓
    │   Round 2
    │       ↓
    │   Round 3
    │       ↓
    │   Round 4
    │       ↓
    │   Round 5
    │       ↓
    │   Round 6
    │       ↓
    │   Calculate Circuit Results
    │       ↓
    │   Calculate Match Result
    │       ↓
    │   Result Screen
    │
    └── AI Demonstration
            ↓
        Select Model
            ↓
        Load Model
            ↓
        Run Match
            ↓
        Display Result
```

---

# 25. AI Demonstration

The graphical game may eventually provide an AI Demonstration mode.

The player may select:

- DQN
- PPO
- A2C

The selected trained model will play against the fixed heuristic opponent.

The demonstration exists primarily for:

- Visualization
- Demonstration
- Debugging
- Thesis presentation

It is not itself the primary training environment.

---

# 26. Main Menu

The intended main menu may contain:

```text
RACING CARDS
CHAMPIONSHIP

[ New Championship ]

[ AI Demonstration ]

[ Results ]

[ Settings ]

[ Exit ]
```

The exact visual design is not finalized.

---

# 27. Battle Interface

The battle interface should communicate the important game state clearly.

Potential elements include:

### Player area

- Player name
- Energy
- Hand
- Circuit Power

### Opponent area

- Opponent status
- Circuit Power
- Relevant visible information

### Circuit area

- Three circuit panels
- Player Power
- Opponent Power
- Circuit identity
- Circuit effect

### Game information

- Current round
- Remaining cards
- Action controls
- Match status

The UI should prioritize clarity over visual complexity.

---

# 28. Card Visual Design

The final card design has not yet been created.

The prototype may use a simple placeholder card containing:

```text
+-----------------------+
|      CARD NAME        |
|                       |
|    [PLACEHOLDER ART]  |
|                       |
| Cost: X     Power: Y  |
|                       |
| Ability:              |
| Description...        |
+-----------------------+
```

Final card artwork can be introduced later.

The visual design should not become a dependency for implementing or testing game logic.

---

# 29. Asset Plan

The project currently has no finalized art assets.

This is acceptable during the prototype phase.

The initial asset categories are:

### Required for prototype

- Basic UI panels
- Buttons
- Placeholder card representation
- Basic background
- Text
- Simple circuit representation

These can be created using Godot's built-in UI nodes and simple shapes.

### Required for final presentation

- Main menu artwork
- Game background
- Hypercar artwork
- Card artwork
- Circuit artwork
- Icons
- UI styling
- Result-screen artwork
- Optional visual effects

The exact asset list will be refined after the gameplay prototype is functional.

---

# 30. Hypercar Art Direction

The game uses fictional hypercar designs.

The cars should be visually distinct and recognizable.

Potential design inspirations may include:

- Modern endurance prototypes
- Le Mans Hypercar aesthetics
- LMDh-style proportions
- High-performance concept cars
- Futuristic racing prototypes

The game should avoid directly copying copyrighted real-world vehicle designs or manufacturer branding.

Final visual direction will be established separately from the game logic.

---

# 31. Prototype Asset Strategy

The project should not block programming because final assets are unavailable.

Recommended order:

```text
Functional placeholder
        ↓
Gameplay testing
        ↓
Mechanic refinement
        ↓
RL environment testing
        ↓
Final game design freeze
        ↓
Asset production
        ↓
Visual polish
```

The prototype should therefore use deliberately simple visuals.

---

# 32. Prototype Configuration

The following are considered **provisional**:

- Exact card statistics
- Exact card abilities
- Exact circuit names
- Exact circuit effects
- Exact reward design
- Exact opponent heuristic
- Exact action encoding
- Exact observation encoding
- Exact visual design
- Exact asset set

These should not be treated as final experimental methodology until explicitly frozen.

---

# 33. Final Design Freeze

Before large-scale DQN/PPO/A2C experiments begin, the following must be frozen:

```text
Game Rules
Card List
Card Statistics
Card Abilities
Circuit Configuration
Energy Rules
Draw Rules
Action Space
Observation Space
Invalid Action Handling
Opponent Behavior
Reward Function
Randomness
Seed Protocol
Evaluation Protocol
```

The frozen configuration becomes the basis for the final controlled experiment.

---

# 34. Design Development Philosophy

The design process follows:

```text
Design
  ↓
Prototype
  ↓
Playtest
  ↓
RL Experiment
  ↓
Analyze
  ↓
Modify
  ↓
Repeat
  ↓
Freeze
```

The prototype is therefore not expected to be perfect.

Its purpose is to discover what works before the research environment is finalized.