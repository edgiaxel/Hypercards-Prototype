# REQUIREMENTS.md

# Hypercards — Requirements Specification

## 1. Overview

**Project:** Hypercards  
**Game:** Racing Cards Championship

Hypercards is a prototype and research-oriented turn-based strategic card game designed to become the reinforcement learning environment for a university thesis.

The project will be used to develop, test, and refine the game mechanics before establishing the final controlled environment used for the reinforcement learning experiments.

The primary algorithms under investigation are:

- Deep Q-Network (DQN)
- Proximal Policy Optimization (PPO)
- Advantage Actor-Critic (A2C)

The three algorithms will ultimately be evaluated using the same finalized environment and controlled experimental conditions.

---

# 2. Prototype vs. Final Experimental Environment

The current implementation is a **prototype**.

The prototype exists to answer questions such as:

- Is the game actually playable?
- Are the rules understandable?
- Are the decisions strategically meaningful?
- Is the action space appropriate for reinforcement learning?
- Is the observation representation sufficient?
- Are the cards reasonably balanced?
- Are the circuits useful?
- Is the reward design appropriate?
- Is the heuristic opponent sufficiently challenging?
- Are there unintended strategies or reward exploits?
- Does the game produce enough meaningful variation between episodes?

Therefore, some requirements are intentionally flexible during the prototype phase.

The following may be experimentally modified during prototyping:

- Card Power
- Card Energy cost
- Card abilities
- Circuit identities
- Circuit effects
- Reward structure
- Opponent heuristic
- Card distribution
- Randomization
- Action representation
- Observation representation
- Other gameplay balancing parameters

Once the prototype has been sufficiently tested, a specific configuration will be selected and documented as the **final experimental environment**.

Changes made after the experimental configuration is frozen must be treated as research-impacting changes.

---

# 3. Research Requirements

## RQ-REQ-01 — Algorithm Support

The final environment shall support the following reinforcement learning algorithms:

- DQN
- PPO
- A2C

The algorithms shall interact with the same underlying game environment.

---

## RQ-REQ-02 — Controlled Comparison

The final experimental comparison shall use the same:

- Game rules
- Observation representation
- Action semantics
- Opponent behavior
- Reward definition
- Evaluation procedure
- Interaction budget
- Random-seed protocol

for all three algorithms unless a documented experimental reason requires otherwise.

---

## RQ-REQ-03 — Reproducibility

The environment shall support controlled random seeds.

Given the same:

- Environment configuration
- Initial conditions
- Random seed
- Agent configuration

the environment should produce reproducible behavior to the extent permitted by the software and hardware stack.

---

# 4. Core Game Requirements

## GAME-REQ-01 — Match Format

The game shall support a 1v1 match.

The intended configuration is:

```text
Players: 2
Rounds: 6
Circuits: 3
```

The exact values may be adjusted during prototype experimentation if testing demonstrates that a different configuration produces a more suitable research environment.

---

## GAME-REQ-02 — Turn-Based Structure

The game shall use a turn-based structure divided into a fixed number of rounds.

The initial design uses six rounds.

Each round shall provide players with opportunities to deploy cards using available Energy.

---

## GAME-REQ-03 — Circuits

The game shall contain three scoring circuits.

Each circuit shall maintain a Power value for each player.

The final winner is determined from the results across the circuits.

The exact identities and gameplay effects of the three circuits are provisional during the prototype phase.

---

## GAME-REQ-04 — Deck

Each player shall use a finite deck of hypercar cards.

The initial design uses:

```text
12 cards per deck
```

The exact card composition may be modified during prototyping.

---

## GAME-REQ-05 — Starting Hand

A player shall begin a match with an initial hand of cards.

The initial design uses:

```text
4 cards
```

The starting hand may be modified during prototype testing if necessary.

---

## GAME-REQ-06 — Card Drawing

Cards shall be drawn from the player's deck during the match.

The initial design draws one card at the beginning of each round.

The exact drawing mechanism may be refined during prototyping.

---

# 5. Energy Requirements

## ENERGY-REQ-01 — Limited Resource

The game shall use Energy as the primary resource for deploying cards.

Cards shall require Energy to be played.

---

## ENERGY-REQ-02 — Round-Based Energy

The initial design increases the player's available Energy according to the current round.

The initial configuration is:

```text
Round 1 → 1 Energy
Round 2 → 2 Energy
Round 3 → 3 Energy
Round 4 → 4 Energy
Round 5 → 5 Energy
Round 6 → 6 Energy
```

This configuration is provisional and may be modified during balancing.

---

## ENERGY-REQ-03 — Energy Reset

Unused Energy shall initially not carry over between rounds.

This behavior may be tested and revised during prototype experimentation.

---

# 6. Card Requirements

## CARD-REQ-01 — Card Identity

Each card shall have a unique identity.

A card may contain information such as:

- Name
- Manufacturer
- Energy cost
- Power
- Card type
- Ability

The exact card attributes are intentionally not finalized at this stage.

---

## CARD-REQ-02 — Card Cost

Cards shall have an Energy cost.

A card shall only be playable when the player has sufficient available Energy, subject to any circuit or card-specific effects.

---

## CARD-REQ-03 — Card Power

Cards shall have a Power value.

Power contributes to the player's score within the circuit where the card is deployed.

Exact Power values shall be determined during prototype balancing.

---

## CARD-REQ-04 — Card Abilities

Cards may contain simple abilities that modify their behavior or Power under specific conditions.

Abilities shall remain relatively simple during the initial prototype.

The prototype should prioritize strategic decision-making over complicated combinations.

---

# 7. Player Action Requirements

## ACTION-REQ-01 — Card Deployment

The player shall be able to deploy a card from their hand to a legal circuit.

---

## ACTION-REQ-02 — Circuit Selection

A card deployment action shall include a target circuit.

The target circuit may affect the resulting Power depending on the circuit's rules.

---

## ACTION-REQ-03 — Pass

The player shall have the option to pass when permitted by the game rules.

Passing should consume no card.

---

## ACTION-REQ-04 — Multiple Actions

The player may be able to deploy multiple cards during the same round when sufficient Energy remains.

The exact action limit shall be determined during prototype testing.

---

## ACTION-REQ-05 — Invalid Actions

The game shall prevent or reject invalid actions.

Examples include:

- Card not present in hand
- Insufficient Energy
- Invalid target circuit
- Action no longer legal because of the current game state

Invalid actions must not corrupt or unexpectedly modify the game state.

---

# 8. Opponent Requirements

## OPP-REQ-01 — Fixed Opponent

The initial environment shall use a fixed heuristic opponent.

The opponent is not intended to learn during the primary experiment.

---

## OPP-REQ-02 — Heuristic Decision Making

The heuristic opponent should evaluate available legal actions using game-state information.

Potential considerations include:

- Immediate circuit advantage
- Resource efficiency
- Current strategic deficit
- Resulting board position

The exact heuristic formula is a prototype design variable and may be refined.

---

## OPP-REQ-03 — Consistency

The same opponent configuration shall be used when comparing DQN, PPO, and A2C in the final experiment.

---

# 9. Scoring Requirements

## SCORE-REQ-01 — Circuit Scoring

At the end of the match, each circuit shall be evaluated by comparing the players' Power values.

---

## SCORE-REQ-02 — Match Result

The game shall determine a final match result based on the three circuit results.

The initial design uses a majority-of-three structure:

```text
Win two or more circuits → Match Win
Lose two or more circuits → Match Loss
One circuit win each + remaining circuit tied → Match Draw
```

The exact handling of tied circuits may be refined during prototyping.

---

## SCORE-REQ-03 — Draw

The game should support a draw state if the final rules permit an unresolved overall result.

A forced tiebreaker should not be introduced solely for convenience without considering its effect on the research environment.

---

# 10. Reward Requirements

## REWARD-REQ-01 — Reinforcement Learning Reward

The final environment shall provide a numerical reward to the reinforcement learning agent.

---

## REWARD-REQ-02 — Prototype Reward Experimentation

The reward function is intentionally **not finalized**.

The prototype shall be used to evaluate possible reward designs.

Candidate approaches may include:

- Terminal win/loss/draw reward
- Carefully justified intermediate rewards
- Other reward structures if required by observed learning behavior

Reward modifications must be evaluated for potential reward hacking and unintended incentives.

---

## REWARD-REQ-03 — Final Reward Consistency

Once the reward design is finalized for the thesis experiment, DQN, PPO, and A2C shall receive the same reward definition.

The reward function shall not be individually optimized for each algorithm.

---

# 11. RL Environment Requirements

## ENV-REQ-01 — Gymnasium Compatibility

The final RL environment shall implement a Gymnasium-compatible interface.

At minimum, it shall provide the expected environment operations for:

- Resetting an episode
- Performing an action
- Returning observations
- Returning rewards
- Determining episode termination

---

## ENV-REQ-02 — Headless Operation

The RL environment shall be capable of running without the Godot graphical interface.

Training must not require:

- Rendering
- Mouse input
- Keyboard input
- Animation playback
- GUI interaction

---

## ENV-REQ-03 — Discrete Action Space

The primary experimental environment shall use a discrete action space.

The conceptual action structure is:

```text
Play Card + Target Circuit
Pass
```

The exact numerical encoding of actions may be determined during implementation.

---

## ENV-REQ-04 — Numerical Observation Space

The environment shall provide a numerical representation of the game state.

The initial intended representation is a flat numerical vector suitable for an MLP-based RL model.

Potential information includes:

- Current round
- Current Energy
- Own circuit Power
- Opponent circuit Power
- Card availability
- Card cost
- Card Power
- Relevant card ability information
- Remaining cards

The exact observation representation may be refined during prototype experimentation.

---

# 12. Invalid Action Handling

## ENV-REQ-01 — Legal Action Awareness

The environment shall be capable of determining which actions are currently legal.

---

## ENV-REQ-02 — Action Masking / Handling

The prototype shall investigate an appropriate mechanism for preventing the agent from selecting invalid actions.

The exact implementation is not considered finalized until tested.

The mechanism should not introduce a separate experimental variable between algorithms.

---

# 13. Godot Requirements

## GODOT-REQ-01 — Graphical Game

Godot shall provide the graphical implementation of the game.

The graphical version should allow a human to observe and interact with a match.

---

## GODOT-REQ-02 — Game UI

The game shall eventually provide interfaces for:

- Main menu
- Starting a game
- Selecting an AI demonstration
- Viewing the match
- Viewing the result

The exact visual design is not finalized.

---

## GODOT-REQ-03 — AI Demonstration

The graphical application should eventually allow demonstration of trained DQN, PPO, and A2C agents.

This demonstration is separate from the headless training environment.

---

# 14. Prototype Requirements

The prototype must allow rapid experimentation.

It should be possible to modify and test:

- Card values
- Card abilities
- Circuit effects
- Reward design
- Opponent behavior
- Action representation
- Observation representation
- Game parameters

without requiring a complete rewrite of the project.

---

# 15. Prototype Validation

Before freezing the final experimental environment, the prototype should be tested for:

### Gameplay

- Rules function correctly.
- Players can complete a full match.
- Invalid actions are handled correctly.
- Matches terminate correctly.
- Scores are calculated correctly.

### Strategic Behavior

- Multiple reasonable strategies exist.
- The game is not trivially solved by one action.
- Cards have meaningful trade-offs.
- Circuit choices matter.
- Later-round decisions can affect the final result.

### RL Suitability

- The observation contains sufficient information.
- The action space is manageable.
- Episodes terminate reliably.
- Rewards correspond to meaningful outcomes.
- The environment does not contain obvious exploitable reward bugs.
- The heuristic opponent provides a consistent challenge.

---

# 16. Final Experimental Freeze

After prototype experimentation, a final environment configuration shall be established.

The final configuration should explicitly record:

- Card list
- Card Power values
- Card Energy costs
- Card abilities
- Circuit names
- Circuit effects
- Number of rounds
- Energy rules
- Card draw rules
- Action space
- Observation space
- Invalid-action mechanism
- Opponent heuristic
- Reward function
- Randomization rules
- Seed protocol
- Evaluation protocol

After the environment is frozen, changes to these components should be treated as changes to the experimental methodology rather than ordinary feature development.

---

# 17. Reusability

The prototype should be developed with future reuse in mind.

Potentially reusable components include:

- Game rules
- Card data structures
- Circuit logic
- Scoring logic
- Environment interface
- Observation representation
- Action representation
- Heuristic opponent
- Reward implementation
- Training infrastructure
- Evaluation scripts
- Documentation
- Test cases
- Configuration files

However, prototype code must not automatically be considered valid final experimental code.

Before reuse in the final experiment, each component must be reviewed to ensure that:

1. It matches the finalized design.
2. It does not contain prototype-only behavior.
3. Its behavior is documented.
4. Its effect on experimental validity is understood.

---

# 18. Definition of Prototype Completion

The prototype can be considered sufficiently complete when:

- A complete match can be played from start to finish.
- The game rules operate reliably.
- Cards and circuits can be modified without major architectural changes.
- The opponent can complete a match.
- The environment can represent the game state.
- The environment can execute legal actions.
- Episodes can terminate correctly.
- Rewards can be generated.
- Basic experiments can be performed with the environment.

At this stage, the project should move into balancing, validation, and final environment definition.

---

# 19. Definition of Final Environment Completion

The final experimental environment is complete when:

- The game configuration has been frozen.
- The reward design has been frozen.
- The action space has been frozen.
- The observation space has been frozen.
- The opponent has been frozen.
- Randomization and seed handling have been defined.
- DQN, PPO, and A2C can all interact with the same environment.
- The evaluation protocol has been defined.
- The environment can run headlessly.
- The implementation matches the documented design.
- The experiment can be reproduced using the documented configuration.

---

# 20. Guiding Principle

The prototype is not merely a preliminary version of the game.

It is the **experimental design and validation stage** of the research environment.

The project should therefore prioritize:

**Experiment → Observe → Evaluate → Refine → Document → Freeze**

rather than assuming that every initial design decision is final.