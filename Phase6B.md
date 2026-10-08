# **PHASE 6B — GAMEPLAY AND ENVIRONMENT SPECIFICATION**

## **1\. Purpose of Phase 6B**

Phase 6B defines the actual game environment that will be implemented and subsequently used as the common environment for the reinforcement-learning experiments described in Phase 6A.

The specification covers:

1. Game concept  
2. Match structure  
3. Game objects  
4. Circuit system  
5. Hypercar card system  
6. Energy system  
7. Hand and deck system  
8. Round progression  
9. Player decision mechanics  
10. Human gameplay interface  
11. Reinforcement-learning interaction  
12. Action space  
13. Invalid-action masking  
14. Observation/state representation  
15. Card and circuit ability representation  
16. Opponent behavior  
17. Match resolution  
18. Reward connection  
19. Randomness and seed-controlled stochasticity  
20. AI demonstration modes  
21. Environment architecture  
22. Scope limitations

The objective is to produce a **complete but controlled turn-based strategic card game** that provides enough decision-making complexity for comparing DQN, PPO, and A2C without introducing unnecessary game systems that could confound the experiment.

---

# **2\. Game Concept**

The game, provisionally named **Hypercards**, is a turn-based strategic card game themed around fictional hypercar racing.

The game takes structural inspiration from games such as Marvel Snap, particularly the combination of:

* a finite number of rounds,  
* a limited resource per round,  
* a hand of cards,  
* multiple competing locations,  
* cards that contribute power,  
* strategic placement,  
* and a final comparison of location scores.

However, Hypercards does **not** reproduce Marvel Snap's cards, characters, locations, abilities, or other copyrighted game content.

The game instead uses an original fictional hypercar championship setting.

The game is designed around a simple strategic question:

> **How should a player allocate limited Energy and available cards across several competing circuits in order to maximize the probability of winning the championship?**

The game is therefore not intended to simulate real motorsport physics.

It is a **strategic abstraction of racing**.

---

# **3\. Research Role of the Game**

The game serves as the environment in which the three reinforcement-learning algorithms are compared:

* Deep Q-Network (DQN)  
* Proximal Policy Optimization (PPO)  
* Advantage Actor-Critic (A2C)

The game environment is shared across the algorithms.

The primary comparison therefore remains:

> **DQN vs PPO vs A2C under the same game rules, state representation, action representation, reward structure, opponent, and experimental conditions.**

The game itself is not the independent variable.

The game is the **controlled environment** in which the independent variable—the RL algorithm—is evaluated.

---

# **4\. Match Structure**

Each match consists of:

* **2 players**  
  * one RL-controlled player or human player,  
  * one computer-controlled opponent;  
* **6 rounds**;  
* **3 active circuits per match**;  
* **12 cards in each player's deck**;  
* **3 starting cards**; 4 in turn 1 at the start of the game  
* **1 additional card drawn at the beginning of each round**;  
* **Energy increasing from 1 to 6**;  
* **multiple card placements permitted within a round as long as sufficient Energy is available**;  
* **a Pass/End Turn action**;  
* **final comparison of Power across the three circuits**.

The standard research configuration is:

> **RL agent vs fixed deterministic heuristic opponent.**

The opponent is not another learning agent during the primary experiment.

---

# **5\. Game Components**

Each match consists of the following components.

## **5.1 Player**

The player possesses:

* a 12-card deck,  
* a hand,  
* Energy,  
* Power accumulated at each circuit,  
* and the ability to place cards.

## **5.2 Opponent**

The opponent possesses the same fundamental resources:

* a 12-card deck,  
* a hand,  
* Energy,  
* circuit Power,  
* and legal card-placement actions.

The opponent uses a fixed heuristic decision policy.

## **5.3 Circuits**

Each match contains three circuits selected from a larger six-circuit pool.

Each circuit has:

* a name,  
* a Power value for each player,  
* an ability,  
* and an activation/reveal round.

## **5.4 Cards**

Each deck contains the same fixed set of 12 fictional hypercar cards.

Each card has:

* card ID,  
* name,  
* Energy cost,  
* base Power,  
* ability type,  
* ability parameters.

---

# **6\. Circuit Pool**

Rather than using only three permanently fixed circuits, the game contains a pool of **six fictional circuits**.

At the beginning of each match, **three distinct circuits are selected from the six-circuit pool**.

This creates controlled environmental variation while maintaining only three simultaneously active scoring areas.

The circuit concepts are inspired by real endurance-racing venues, but their names and gameplay effects are fictional.

The six circuits are:

| ID | Fictional Circuit | Inspiration | Ability |
| ----- | ----- | ----- | ----- |
| C1 | **Le Womans Circuit** | Le Mans | Final-round Power bonus |
| C2 | **Fiji Speedway** | Fuji Speedway | Energy efficiency |
| C3 | **Spa-Rainbow Circuit** | Spa-Francorchamps | Underdog Power bonus |
| C4 | **Monzi Autodromo** | Monza | High-cost Power bonus |
| C5 | **Americas Crown Raceway** | Circuit of the Americas | Energy recovery/management |
| C6 | **Nürburg Ringlet** | Nürburgring | Late-race Power scaling |

The parody names deliberately avoid using the actual circuit names as the game's fictional locations.

The underlying design principle is more important than the names:

> Each circuit creates a different strategic incentive while remaining within the restricted domains of **Power manipulation and Energy manipulation**.

---

# **7\. Circuit 1 — Le Womans Circuit**

**Inspiration:** Circuit de la Sarthe / Le Mans

### **Ability: Final Push**

> During Round 6, cards placed at this circuit receive **\+2 Power**.

The effect applies to eligible cards according to the circuit's defined trigger.

The ability is intended to create a late-game strategic incentive.

A player may:

* establish an early lead elsewhere,  
* conserve strong cards,  
* then commit heavily to Le Womans during the final round.

---

# **8\. Circuit 2 — Fiji Speedway**

**Inspiration:** Fuji Speedway

### **Ability: Energy Efficient**

> Cards played at this circuit cost **1 less Energy**, with a minimum effective cost of 1\.

This creates an Energy-management opportunity.

For example:

A card normally costing 3 Energy may require only 2 Energy when deployed here.

The reduction affects the Energy cost required to legally deploy the card.

---

# **9\. Circuit 3 — Spa-Rainbow Circuit**

**Inspiration:** Spa-Francorchamps

### **Ability: Overtake**

> A card receives **\+2 Power** when deployed to this circuit while the player's current Power at the circuit is lower than the opponent's Power.

This encourages the agent to contest circuits that it is currently losing rather than simply reinforcing circuits it already controls.

---

# **10\. Circuit 4 — Monzi Autodromo**

**Inspiration:** Monza

### **Ability: High-Speed Machine**

> Cards with an Energy cost of 4 or greater receive **\+2 Power** when deployed here.

This creates a strategic relationship between expensive cards and circuit selection.

A high-cost card that might be inefficient elsewhere can become valuable at Monzi.

---

# **11\. Circuit 5 — Americas Crown Raceway**

**Inspiration:** Circuit of the Americas

### **Ability: Energy Reserve**

> If a player has unused Energy at the end of a round while occupying this circuit, **1 unused Energy may be carried into the next round**, subject to the environment's maximum Energy limit.

This circuit therefore creates an exception to the normal no-carryover Energy rule.

The exception is intentionally localized to the circuit.

It creates an additional strategic decision:

> Should the player spend all Energy now, or deliberately preserve Energy for a stronger future action?

This ability should be implemented carefully because it modifies the normal Energy progression.

---

# **12\. Circuit 6 — Nürburg Ringlet**

**Inspiration:** Nürburgring

### **Ability: Endurance Build-Up**

> Cards deployed here during Rounds 5 and 6 receive **\+1 Power**.

This provides a simpler late-game advantage than Le Womans and allows the two circuits to remain strategically distinct.

---

# **13\. Circuit Selection**

At the beginning of an episode:

1. The six-circuit pool is initialized.  
2. Three distinct circuits are selected without replacement.  
3. The selected circuits are assigned to Circuit Slot 1, Circuit Slot 2, and Circuit Slot 3\.  
4. Their identities and abilities are included in the game state.  
5. The circuits become active progressively during the match.

The circuit selection is controlled by the environment's random seed.

Therefore:

> The same environment seed produces the same circuit-selection sequence under the same implementation.

---

# **14\. Circuit Visibility and Activation**

The game distinguishes between **knowing a circuit's identity** and **having that circuit become active**.

The three selected circuits and their abilities are visible to the player/agent from the beginning of the match.

However, they become active according to the round progression.

### **Round 1**

Circuit Slot 1 becomes active.

### **Round 2**

Circuit Slot 2 becomes active.

### **Round 3**

Circuit Slot 3 becomes active.

### **Rounds 4–6**

All three circuits remain active.

This means the agent knows the available strategic rules before they become active, allowing it to plan around future circuit effects.

The state therefore explicitly represents:

* circuit identity,  
* circuit ability,  
* circuit activation status,  
* current Power,  
* and relevant ability parameters.

---

# **15\. Why Progressive Circuit Activation?**

Progressive activation provides another long-term planning element without requiring a larger action space.

The player can observe:

> "This circuit will become active next round, and its ability favors high-cost cards."

The agent therefore has an opportunity to preserve resources for future strategic use.

This creates temporal decision-making while keeping the game mechanically simple.

---

# **16\. Card Deck**

Each player uses the same fixed 12-card hypercar deck.

There is:

* no deck building,  
* no card purchasing,  
* no card upgrading,  
* no card rarity system,  
* no random card generation.

The fixed card pool ensures that the comparison between DQN, PPO, and A2C occurs within the same underlying card distribution.

The stochasticity comes from **shuffle and draw order**, rather than changing the composition of the deck.

---

# **17\. Hypercar Card Roster**

The cards are inspired by real endurance prototypes and hypercars, including vehicles from the 2026 FIA WEC Hypercar field and IMSA GTP field. The real-world inspiration is deliberately transformed into fictional parody names rather than one-to-one representations. [FIAWEC](https://www.fiawec.com/en/car/2026?utm_source=chatgpt.com)

The 12-card roster is:

| \# | Fictional Hypercar | Inspiration | Cost | Base Power | Ability |
| ----- | ----- | ----- | ----- | ----- | ----- |
| 1 | **Aston Martian Valykrie** | Aston Martin Valkyrie | 1 | 2 | None |
| 2 | **Beemer M Hyper Vee** | BMW M Hybrid V8 | 2 | 3 | Energy Saver |
| 3 | **Cadillack V-Series-RR** | Cadillac V-Series.R | 2 | 4 | Underdog Power |
| 4 | **Ferrary 499Punto** | Ferrari 499P | 3 | 5 | High-Energy Power |
| 5 | **Toyoda TR010-ish** | Toyota TR010/GR010 lineage | 3 | 6 | Late-Race Power |
| 6 | **Alpina A424-ish** | Alpine A424 | 2 | 3 | Cost Reduction |
| 7 | **Peujot 9X-8.5** | Peugeot 9X8 | 4 | 7 | Circuit Power |
| 8 | **Genesys GMR-001-ish** | Genesis GMR-001 | 4 | 8 | Underdog Power |
| 9 | **Porch 963** | Porsche 963 | 5 | 9 | Energy Reserve |
| 10 | **Akura ARX-07-ish** | Acura ARX-06 | 4 | 7 | Efficient Placement |
| 11 | **Lambargar SC-63-ish** | Lamborghini SC63 | 5 | 10 | Final-Round Power |
| 12 | **Vanwool Van-Derp 680** | Vanwall Vandervell 680 | 6 | 12 | High-Cost Bonus |

The exact numeric values should be treated as **initial game-balance parameters** and verified through implementation testing before the final experimental configuration is frozen.

The important structural constraint is:

> Card abilities must remain within **Power manipulation and Energy manipulation**.

No complicated card-generation, movement, destruction, copying, resurrection, deck manipulation, or board-locking mechanics are required.

---

# **18\. Card Ability Categories**

Card abilities are restricted to a small number of controlled categories.

## **18.1 No Ability**

The card simply contributes:

> Base Power

This provides a baseline card type.

---

## **18.2 Power Modification**

Examples:

> \+2 Power during Round 6\.

or:

> \+2 Power when deployed to a circuit currently being lost.

---

## **18.3 Energy Modification**

Examples:

> Reduce effective Energy cost by 1\.

or:

> Preserve 1 Energy for the next round.

---

## **18.4 Conditional Power Modification**

An ability can depend on a simple observable condition.

Example:

> If the player's Power at the target circuit is lower than the opponent's Power, gain \+2 Power.

The condition is evaluated by the environment.

---

## **18.5 Round-Based Power Modification**

An ability can depend on the current round.

Example:

> \+3 Power when played during Round 6\.

This gives cards different temporal values without introducing complex mechanics.

---

# **19\. Example Card Abilities**

### **Aston Martian Valykrie**

**Cost:** 1  
**Power:** 2

**Ability:** None.

A basic low-cost card.

---

### **Beemer M Hyper Vee**

**Cost:** 2  
**Power:** 3

**Ability: Energy Saver**

> When played at a circuit that provides an Energy-cost reduction, the reduction is applied before Energy is deducted.

This card is designed to interact naturally with Energy-efficient circuits.

---

### **Cadillack V-Series-RR**

**Cost:** 2  
**Power:** 4

**Ability: Overtake**

> \+2 Power if the player's Power at the target circuit is lower than the opponent's Power before placement.

---

### **Ferrary 499Punto**

**Cost:** 3  
**Power:** 5

**Ability: Momentum**

> \+2 Power if the player's current Energy before placement is at least 4\.

This rewards committing a larger resource reserve.

---

### **Toyoda TR010-ish**

**Cost:** 3  
**Power:** 6

**Ability: Endurance**

> \+2 Power when played during Round 5 or Round 6\.

---

### **Alpina A424-ish**

**Cost:** 2  
**Power:** 3

**Ability: Efficiency**

> Its effective Energy cost is reduced by 1 when the target circuit has an Energy-reduction effect.

---

### **Peujot 9X-8.5**

**Cost:** 4  
**Power:** 7

**Ability: Circuit Specialist**

> \+2 Power when played at a circuit whose current score is lower than the opponent's.

---

### **Genesys GMR-001-ish**

**Cost:** 4  
**Power:** 8

**Ability: Recovery**

> \+2 Power when deployed to a circuit currently being lost.

---

### **Porch 963**

**Cost:** 5  
**Power:** 9

**Ability: Reserve**

> If played during a round in which the player has at least 1 unused Energy after all current placements, 1 Energy may be preserved for the following round.

This ability requires the Energy state to be explicitly represented.

---

### **Akura ARX-07-ish**

**Cost:** 4  
**Power:** 7

**Ability: Efficient Deployment**

> Effective Energy cost is reduced by 1 when deployed to a circuit with an Energy-management ability.

---

### **Lamborgini SC-63-ish**

**Cost:** 5  
**Power:** 10

**Ability: Final Attack**

> \+3 Power when played during Round 6\.

---

### **Vanwall Van-Derp 680**

**Cost:** 6  
**Power:** 12

**Ability: Heavy Machine**

> \+2 Power when played at a circuit that grants a bonus to high-cost cards.

---

# **20\. Ability Representation**

Abilities must not be represented to the RL model as natural-language descriptions.

The environment should internally represent them using structured numerical or categorical fields.

For example:

ability\_type  
ability\_trigger  
ability\_target  
ability\_value  
ability\_condition  
ability\_round

A card with:

> "+3 Power during Round 6"

could internally be represented approximately as:

ability\_type \= POWER\_MODIFIER  
trigger \= ROUND  
target \= SELF  
value \= \+3  
condition \= NONE  
round \= 6

A card with:

> "+2 Power when behind"

could be represented as:

ability\_type \= POWER\_MODIFIER  
trigger \= CONDITIONAL  
target \= SELF  
value \= \+2  
condition \= BEHIND\_AT\_TARGET\_CIRCUIT  
round \= ANY

This makes the ability information accessible to the MLP observation model.

---

# **21\. Important Ability Constraint**

Abilities must not require the agent to understand natural-language rules.

The environment itself evaluates the ability conditions.

The agent receives the structured information necessary to make decisions.

Therefore:

> **Game logic is deterministic; ability interpretation is performed by the environment; the agent learns the strategic consequences from state and reward.**

---

# **22\. Starting Hand**

At the beginning of every match:

* each player receives **3 cards**.

This is the initial hand before Round 1\.

The starting cards are drawn from the player's shuffled 12-card deck.

Therefore:

Deck \= 12 cards  
Starting hand \= 3 cards  
Remaining deck \= 9 cards

The initial draw is controlled by the environment's random seed.

---

# **23\. Card Drawing**

At the beginning of each round, each player draws **one additional card**.

The first draw therefore occurs when Round 1 begins.

The hand-size progression, assuming no cards are played, is:

| Stage | Maximum Hand |
| ----- | ----- |
| Initial state / Turn 0 | 3 |
| Round 1 | 4 |
| Round 2 | 5 |
| Round 3 | 6 |
| Round 4 | 7 |
| Round 5 | 8 |
| Round 6 | 9 |

Therefore:

> **The maximum possible hand size is 9 cards.**

This replaces the obsolete 4-card/13-action design from the earlier Phase 6B.

---

# **24\. Cards Leaving the Hand**

When a card is successfully deployed:

1. The card leaves the player's hand.  
2. The card is placed at the selected circuit.  
3. Its Energy cost is deducted.  
4. Its Power and ability effects are applied according to the game's resolution rules.

An empty hand slot becomes available.

The implementation should maintain stable hand-slot identities rather than renumbering the action space dynamically.

---

# **25\. Stable Hand Slots**

The game uses **nine maximum hand slots**.

A card can occupy one of:

Slot 1  
Slot 2  
Slot 3  
Slot 4  
Slot 5  
Slot 6  
Slot 7  
Slot 8  
Slot 9

If a card is played, its slot becomes empty.

Newly drawn cards are inserted into an available hand slot according to a deterministic implementation rule.

The exact slot-filling rule should remain consistent throughout all experiments.

This allows the action space to remain fixed.

---

# **26\. Energy System**

Energy represents the primary limited resource.

The standard Energy progression is:

| Round | Base Energy |
| ----- | ----- |
| 1 | 1 |
| 2 | 2 |
| 3 | 3 |
| 4 | 4 |
| 5 | 5 |
| 6 | 6 |

The player's current Energy is included in the observation.

---

# **27\. Energy Does Not Normally Carry Over**

Under the default game rule:

> Unused Energy is lost at the end of the round.

For example:

Round 3  
Available Energy \= 3

Player spends 1

Remaining \= 2

End of Round 3  
→ 2 unused Energy is normally lost

This encourages the player to consider whether available Energy should be used now or preserved through card choice rather than simply accumulating it indefinitely.

Circuit/card abilities may create controlled exceptions to this rule.

---

# **28\. Multiple Cards Per Round**

There is **no fixed card-per-round limit**.

A player may deploy multiple cards during one round as long as:

1. the cards are in the hand,  
2. the cards have not already been played,  
3. the target circuits are active,  
4. sufficient Energy remains,  
5. the corresponding actions are legal.

For example:

Round 5  
Energy \= 5

Card A → cost 2  
Card B → cost 1  
Card C → cost 2

Total \= 5

All three cards can be deployed during the same round.

This creates a meaningful resource-allocation problem.

---

# **29\. Turn 0**

Before Round 1 begins, the game performs its initialization phase.

### **Turn 0 / Match Initialization**

1. Shuffle each player's deck.  
2. Draw 3 cards for each player.  
3. Select 3 circuits from the 6-circuit pool.  
4. Display the selected circuits and their abilities.  
5. Initialize circuit Power to zero.  
6. Initialize round counter.  
7. Initialize Energy configuration.  
8. Initialize all other game-state variables.

No card placement occurs during Turn 0\.

Turn 0 is therefore an initialization state rather than a normal decision round.

---

# **30\. Round 1**

At the beginning of Round 1:

1. Each player draws one card.  
2. Each player therefore has up to 4 cards if no previous cards exist.  
3. Circuit Slot 1 becomes active.  
4. Each player receives 1 Energy.  
5. Players make their placement decisions.  
6. Players end their turn.  
7. Actions are resolved.  
8. The round ends.

---

# **31\. Round 2**

At the beginning of Round 2:

1. Each player draws one card.  
2. Circuit Slot 2 becomes active.  
3. Each player receives 2 Energy.  
4. Players make decisions.  
5. Players end their turn.  
6. Actions resolve.  
7. The round ends.

---

# **32\. Round 3**

At the beginning of Round 3:

1. Each player draws one card.  
2. Circuit Slot 3 becomes active.  
3. Each player receives 3 Energy.  
4. Players make decisions.  
5. Players end their turn.  
6. Actions resolve.  
7. The round ends.

At this point, all three circuits are active.

---

# **33\. Rounds 4–6**

Rounds 4, 5, and 6 continue the same general structure.

### **Round 4**

Energy \= 4

### **Round 5**

Energy \= 5

### **Round 6**

Energy \= 6

Round 6 is the final decision round.

After Round 6 is resolved, the match proceeds to final scoring.

---

# **34\. Human Decision Phase**

The human version of the game contains a visible decision interface.

The player may:

1. select a card,  
2. select a target circuit,  
3. deploy the card,  
4. repeat the process while Energy remains,  
5. press **READY** when satisfied.

The human player does not need to wait for the entire timer if the intended decisions have already been made.

---

# **35\. Human Ready Button**

The human interface contains a:

> **READY**

button.

When the human player presses READY:

* the player's current round decisions are locked,  
* no additional cards can be placed for that round,  
* the game waits for the opponent to finish,  
* and once both sides have committed, the round proceeds to resolution.

This provides the human version with an actual turn-completion mechanism.

---

# **36\. Human Decision Timer**

Each human decision phase has a fixed countdown.

A suitable initial configuration is:

> **30 seconds per round**

The exact duration is an implementation/UI parameter rather than a reinforcement-learning research variable.

If the timer expires:

### **If the player has already placed cards**

Those placements are locked and the round proceeds.

### **If the player has not placed a card**

The player effectively performs:

> **Pass**

The timeout therefore does not create an additional gameplay action.

---

# **37\. Human Retreat**

The human interface also contains:

> **RETREAT**

or:

> **SURRENDER**

The button immediately ends the match.

Retreat is primarily a human gameplay feature.

It is **not part of the primary RL action space**.

The training environment does not provide surrender as a normal learning action.

This prevents the agent from discovering an undesirable trivial policy such as:

> "Immediately surrender every episode."

---

# **38\. RL Agent Does Not Use the Human Timer**

The reinforcement-learning environment is not constrained by the human countdown.

The RL agent receives a game state and produces an action computationally.

There is therefore:

* no 30-second waiting period,  
* no wall-clock decision limit,  
* no READY action,  
* no human timeout.

This ensures that training performance is not influenced by arbitrary UI timing.

---

# **39\. RL Turn Completion**

Because the RL agent can perform multiple card placements during a round, the agent needs a mechanism to indicate:

> **I am finished placing cards this round.**

This is the purpose of the **Pass / End Turn** action.

For the RL environment:

> **PASS \= voluntarily end the current player's decision phase.**

Therefore, Pass does not necessarily mean:

> "I refuse to play anything."

It means:

> "I am finished making decisions for this round."

If the agent has already played several cards, Pass simply ends the remaining decision sequence.

---

# **40\. Action Space**

The maximum hand contains:

> **9 cards**

There are:

> **3 circuits**

Therefore:

9 hand slots × 3 circuits  
\= 27 card-placement actions

plus:

1 Pass / End Turn action

giving:

> **28 discrete RL actions.**

The action space is therefore:

Discrete(28)  
---

# **41\. Action Mapping**

The fixed action mapping is:

| Action ID | Meaning |
| ----- | ----- |
| 0 | Slot 1 → Circuit 1 |
| 1 | Slot 1 → Circuit 2 |
| 2 | Slot 1 → Circuit 3 |
| 3 | Slot 2 → Circuit 1 |
| 4 | Slot 2 → Circuit 2 |
| 5 | Slot 2 → Circuit 3 |
| 6 | Slot 3 → Circuit 1 |
| 7 | Slot 3 → Circuit 2 |
| 8 | Slot 3 → Circuit 3 |
| 9 | Slot 4 → Circuit 1 |
| 10 | Slot 4 → Circuit 2 |
| 11 | Slot 4 → Circuit 3 |
| 12 | Slot 5 → Circuit 1 |
| 13 | Slot 5 → Circuit 2 |
| 14 | Slot 5 → Circuit 3 |
| 15 | Slot 6 → Circuit 1 |
| 16 | Slot 6 → Circuit 2 |
| 17 | Slot 6 → Circuit 3 |
| 18 | Slot 7 → Circuit 1 |
| 19 | Slot 7 → Circuit 2 |
| 20 | Slot 7 → Circuit 3 |
| 21 | Slot 8 → Circuit 1 |
| 22 | Slot 8 → Circuit 2 |
| 23 | Slot 8 → Circuit 3 |
| 24 | Slot 9 → Circuit 1 |
| 25 | Slot 9 → Circuit 2 |
| 26 | Slot 9 → Circuit 3 |
| 27 | PASS / END TURN |

This action space remains fixed throughout the episode.

---

# **42\. Why the Action Space Is Fixed**

The number of legal actions changes from state to state.

However, the **action space itself does not change**.

For example, at one state:

Slot 1 \= occupied  
Slot 2 \= occupied  
Slot 3 \= empty  
Circuit 1 \= active  
Circuit 2 \= active  
Circuit 3 \= inactive  
Energy \= 1

Only some of the 28 actions are legal.

The environment therefore produces a legal-action mask.

This preserves stable action semantics for DQN, PPO, and A2C.

---

# **43\. Invalid-Action Mask**

The environment determines whether every action is legal.

An action may be invalid because:

* the hand slot is empty,  
* the card has already been played,  
* insufficient Energy exists,  
* the target circuit is not active,  
* another game rule prevents the placement.

The environment produces a binary action mask:

1 \= legal  
0 \= illegal

The mask changes according to the current game state.

---

# **44\. Common Masking Mechanism**

The action mask is part of the **common environment mechanism**.

It is not a research factor.

All three algorithms should receive the same legal-action information.

The implementation therefore adapts the algorithm interfaces where necessary so that:

> DQN, PPO, and A2C all operate using the same state-dependent legal-action set.

The purpose is to prevent algorithms from wasting decisions on impossible game actions.

---

# **45\. Agent and Human Action Difference**

The human player interacts with the game through:

* card selection,  
* circuit selection,  
* Ready,  
* Retreat.

The RL agent interacts with the environment through:

* card-slot/circuit action,  
* Pass.

The UI mechanics are therefore not identical to the RL action representation.

This is intentional.

The game client translates human interactions into the underlying game-state transitions.

---

# **46\. Decision Timestep**

An RL timestep represents:

> **one decision made by the RL-controlled player.**

It does **not** necessarily represent:

* one round,  
* one card,  
* or one complete match.

A single round may therefore contain several RL timesteps if the agent plays multiple cards before choosing Pass.

For example:

Round 5

Timestep 1:  
Card A → Circuit 1

Timestep 2:  
Card B → Circuit 3

Timestep 3:  
Card C → Circuit 2

Timestep 4:  
PASS

→ Round ends

This distinction is important when measuring environment timesteps during training.

---

# **47\. Opponent Decision Process**

The opponent uses the same underlying legal-action rules.

The heuristic evaluates legal actions and selects one according to a deterministic scoring procedure.

The opponent does not learn during the primary experiment.

Its strategy therefore remains fixed throughout training and evaluation.

---

# **48\. Heuristic Opponent**

The opponent evaluates legal card-placement options according to factors such as:

1. expected Power contribution,  
2. Energy efficiency,  
3. current circuit deficit,  
4. circuit ability,  
5. card ability,  
6. strategic importance of the target circuit,  
7. remaining rounds.

The opponent then chooses the action with the highest heuristic value.

The exact numerical heuristic formula belongs to the implementation specification and should be fixed before the main experiment.

---

# **49\. Deterministic Tie-Breaking**

The primary heuristic opponent should use a **deterministic tie-breaking rule**.

For example:

1. highest heuristic score,  
2. highest resulting Power,  
3. lower Energy cost,  
4. lower action ID.

This avoids introducing unnecessary opponent randomness.

The game's primary stochasticity already comes from:

* deck shuffle,  
* initial card draw,  
* subsequent card draws,  
* circuit selection.

---

# **50\. Opponent Information**

The heuristic opponent should receive the same information that the RL agent is allowed to observe.

The opponent should not have access to:

* hidden implementation variables,  
* future random draws,  
* future card identities,  
* information unavailable to the RL agent.

This keeps the game strategically fair.

---

# **51\. Opponent Decision Visibility**

The opponent's current-round choices are not revealed to the RL agent before the RL agent finishes its own current-round decisions.

Internally, the environment may process the decisions sequentially.

However, the current opponent decision is hidden from the RL observation until the round reaches its resolution stage.

This prevents the RL agent from simply reacting to the opponent's already-selected move.

---

# **52\. Round Resolution**

Once both players have completed their current-round decisions:

1. The selected actions are committed.  
2. The current-round placements are revealed.  
3. Card effects are evaluated.  
4. Circuit effects are evaluated.  
5. Power totals are updated.  
6. Energy-related effects are applied according to their defined timing.  
7. The round ends.

The resolution order must be deterministic.

This is particularly important for Energy-related abilities.

---

# **53\. Ability Timing**

Each ability has an explicit trigger.

Possible triggers include:

* **ON PLAY**  
* **ROUND START**  
* **ROUND END**  
* **ROUND N**  
* **CONDITIONAL ON PLAY**  
* **FINAL ROUND**

The environment evaluates the trigger according to the defined game-state transition.

No ability should rely on ambiguous timing.

---

# **54\. Power Calculation**

Each circuit maintains separate Power totals:

Player Circuit 1 Power  
Opponent Circuit 1 Power

Player Circuit 2 Power  
Opponent Circuit 2 Power

Player Circuit 3 Power  
Opponent Circuit 3 Power

Power is calculated from:

Base card Power  
\+  
card ability modifiers  
\+  
circuit ability modifiers

The resulting total determines the winner of the individual circuit.

---

# **55\. Winning a Circuit**

At the end of Round 6:

Player Power \> Opponent Power  
→ Player wins circuit

Player Power \< Opponent Power  
→ Opponent wins circuit

Player Power \= Opponent Power  
→ Circuit draw

No arbitrary winner is assigned when both players have identical Power.

---

# **56\. Winning the Championship**

The standard objective is to win at least:

> **2 of the 3 circuits.**

Therefore:

### **Player wins**

Player wins two or three circuits.

### **Player loses**

Opponent wins two or three circuits.

### **Overall draw**

Neither player wins two circuits.

For example:

Player  
Circuit 1 → WIN  
Circuit 2 → LOSS  
Circuit 3 → DRAW

Result → DRAW  
---

# **57\. Overall Tie Resolution**

A circuit draw does not automatically determine the match winner.

If the circuit results are:

1 win  
1 loss  
1 draw

the overall result is:

> **DRAW**

This avoids introducing an additional arbitrary tiebreaking rule.

---

# **58\. Optional Aggregate Power Statistic**

Although the primary victory condition is based on circuits won, the game may also record:

> **Total Power across all three circuits**

This can be displayed on the result screen and used for analysis/debugging.

However:

> Total Power is not the primary victory criterion.

Winning the championship requires winning the circuit comparison.

---

# **59\. Reward Connection**

The game environment supplies the reward structure defined by the research design.

The primary reward is:

Win  \= \+1  
Draw \=  0  
Loss \= \-1

Intermediate decision steps receive:

0

Therefore a typical episode may look like:

Round 1 → 0  
Round 2 → 0  
Round 3 → 0  
Round 4 → 0  
Round 5 → 0  
Round 6 → \+1

or:

Round 6 → \-1

or:

Round 6 → 0

The detailed rationale for this reward design belongs to Phase 6A.

Phase 6B simply specifies how the game outcome produces the environment reward.

---

# **60\. No Step Penalty**

The game does not impose a negative reward for taking additional decision steps.

There is therefore no:

\-0.001 per timestep

penalty in the primary environment.

This keeps the objective aligned with:

> **Win the championship.**

rather than:

> **Finish the game as quickly as possible.**

---

# **61\. Observation / State**

The RL agent does not receive screenshots.

It receives a structured numerical representation of the current game state.

The observation is designed for an:

> **MLP-based policy/value network.**

The observation should contain enough information for the agent to understand:

* current match progress,  
* Energy,  
* circuit state,  
* card availability,  
* card properties,  
* card abilities,  
* circuit abilities,  
* opponent state,  
* and relevant strategic conditions.

---

# **62\. Observation: Global State**

The global observation contains information such as:

Current round  
Current Energy  
Maximum/base Energy  
Number of cards remaining in deck  
Number of cards in hand

Additional Energy-related state variables are included if an ability has modified Energy carryover or future Energy.

---

# **63\. Observation: Own Circuit State**

For each of the three circuits:

Own current Power  
Opponent current Power  
Circuit active/inactive  
Circuit identity  
Circuit ability type  
Circuit ability parameters  
Relevant current-round modifiers

This allows the agent to determine:

> "What is happening at each circuit?"

---

# **64\. Observation: Hand**

Each of the nine hand slots receives a structured representation.

For every slot:

Occupied?  
Card identity  
Energy cost  
Base Power  
Ability type  
Ability trigger  
Ability value  
Ability condition  
Ability round

If the slot is empty:

Occupied \= 0

and the remaining fields are represented consistently using the environment's chosen zero/neutral encoding.

---

# **65\. Observation: Opponent State**

The observation includes the opponent's **observable** game state.

For example:

Opponent circuit Power  
Opponent number of cards remaining  
Opponent known/visible board state

The agent does not receive the opponent's hidden hand contents.

This is important.

The opponent's hand remains private information.

---

# **66\. Observation: Circuit Information**

Because circuit identity and abilities are part of the known game configuration, the agent receives their structured representation.

For example:

Circuit 1:  
active \= 1  
ability\_type \= FINAL\_ROUND\_POWER  
ability\_value \= \+2

Circuit 2:  
active \= 1  
ability\_type \= ENERGY\_REDUCTION  
ability\_value \= \-1

Circuit 3:  
active \= 0  
ability\_type \= UNDERDOG\_POWER  
ability\_value \= \+2

The agent therefore knows what strategic opportunities will become available.

---

# **67\. Observation and State Updates**

The observation must be regenerated after every environment transition.

For example, after playing a card:

Energy decreases  
Hand slot becomes empty  
Circuit Power changes  
Opponent-visible board state remains updated  
Legal action mask changes

After drawing a card:

Hand slot becomes occupied  
Card identity becomes visible  
Cost becomes visible  
Power becomes visible  
Ability metadata becomes visible  
Legal action mask changes

After a circuit becomes active:

Circuit active flag changes  
Circuit-related legal actions change

The observation therefore always represents the current state.

---

# **68\. No Information Leakage**

The environment must not expose future information that would normally be unavailable to the player.

The observation must not contain:

* the opponent's hidden hand,  
* future card draws,  
* future random-number outcomes,  
* hidden implementation variables,  
* information generated after the current decision.

This prevents accidental information leakage.

---

# **69\. Randomness**

The game intentionally contains controlled stochasticity.

Random elements include:

1. initial deck shuffle,  
2. initial three-card draw,  
3. one-card draw at the beginning of every round,  
4. three-circuit selection from the six-circuit pool,  
5. circuit ordering.

The underlying game rules remain deterministic.

Thus:

> **Randomness determines the scenario; deterministic game rules determine the consequences.**

---

# **70\. Seed-Controlled Randomness**

The environment uses explicit random seeds.

The environment RNG controls game-level stochasticity such as:

deck shuffle  
card draws  
circuit selection  
circuit ordering

The algorithm's training randomness is handled separately according to the research design.

This separation helps prevent the environment's random sequence from being accidentally affected by unrelated algorithm operations.

---

# **71\. Same Seed and Reproducibility**

A seed initializes the pseudorandom process.

Using the same:

* game implementation,  
* environment configuration,  
* random seed,  
* and random-number consumption sequence

should reproduce the same stochastic game scenario.

However, the seed does not mean every episode in training is identical.

The environment's RNG continues generating subsequent random values as episodes progress.

Therefore:

> A seed provides reproducibility of a stochastic process, not a permanently identical episode.

The formal experimental seed methodology belongs to Phase 6A.

---

# **72\. Training vs Evaluation Randomness**

The environment should support separate training and evaluation seed sets.

For example:

Training seeds:  
S1, S2, S3, ...

Evaluation seeds:  
E1, E2, E3, ...

The same held-out evaluation scenarios can then be presented to DQN, PPO, and A2C.

This allows the algorithms to be compared under matched stochastic evaluation conditions.

---

# **73\. Fixed Deck, Randomized Order**

Every player receives the same 12-card pool.

The deck composition therefore does not change.

However:

> The order in which cards are drawn is randomized.

This creates variation between episodes without changing the fundamental game.

For example:

Episode A:  
Card 2 → Card 8 → Card 1 → ...

Episode B:  
Card 5 → Card 1 → Card 10 → ...

Episode C:  
Card 7 → Card 3 → Card 12 → ...

This prevents the agent from simply memorizing a single fixed sequence.

---

# **74\. Environment Architecture**

The environment should be separated from the graphical game client.

### **Research/training environment**

Python  
   ↓  
Gymnasium-compatible environment  
   ↓  
DQN / PPO / A2C

The environment operates headlessly during training.

### **Game client**

Godot  
   ↓  
Game interface  
   ↓  
Human input / trained model  
   ↓  
Game environment

Godot is therefore primarily responsible for:

* presentation,  
* animation,  
* menus,  
* user interaction,  
* and demonstration.

The RL environment remains the authoritative source of game rules.

---

# **75\. Environment Authority**

The Python environment is the source of truth for:

* Energy,  
* cards,  
* Power,  
* circuit effects,  
* card abilities,  
* legal actions,  
* scoring,  
* rewards,  
* random events,  
* and terminal conditions.

The Godot client must not independently implement alternative versions of those rules.

This avoids the situation where:

> "The game shown in Godot behaves differently from the environment used for training."

---

# **76\. Human Gameplay Mode**

The finished game should allow a human to play against:

> **the fixed heuristic opponent.**

This provides a baseline playable version of the game before the trained RL agents are integrated.

The human therefore experiences the same fundamental rules used by the research environment.

---

# **77\. Human vs Trained AI**

After training, the game can also allow the human player to select a trained model.

Possible opponents include:

DQN  
PPO  
A2C

The selected model is loaded in inference mode.

The model does not continue learning during the human match.

Therefore:

> Human vs DQN

means:

> a human playing against the frozen trained DQN policy.

Likewise:

> Human vs PPO

and:

> Human vs A2C.

---

# **78\. AI Demonstration Mode**

The game can also include an AI demonstration mode.

The user can select:

DQN  
PPO  
A2C

and watch the selected model play against the fixed heuristic opponent.

This mode is primarily a thesis artifact/demo feature.

It is not itself the primary statistical experiment.

---

# **79\. Frozen AI-vs-AI Demonstration**

The game can additionally support matches such as:

DQN vs DQN  
DQN vs PPO  
DQN vs A2C  
PPO vs DQN  
PPO vs A2C  
A2C vs DQN  
A2C vs PPO  
A2C vs A2C

The models are loaded in **inference/frozen mode**.

They do not update their parameters.

Therefore:

> DQN vs PPO in the demonstration mode is **not self-play and not continued learning**.

It is simply:

> **evaluation/inference using two already-trained policies.**

---

# **80\. Important Distinction: Research Experiment vs Demonstration**

The primary thesis experiment remains:

DQN → fixed heuristic opponent  
PPO → fixed heuristic opponent  
A2C → fixed heuristic opponent

under controlled experimental conditions.

AI-vs-AI matches are an optional demonstration/evaluation feature.

They should not silently replace the fixed-opponent experiment.

Otherwise, the opponent becomes another experimental variable.

---

# **81\. AI-vs-AI Limitations**

A frozen DQN vs PPO match does not tell us:

> "Which algorithm is universally better?"

It only demonstrates how the already-trained policies behave against each other.

The models are not adapting.

There is no:

DQN learns against PPO  
PPO learns against DQN

during the match.

Therefore, this mode should be described as:

> **cross-policy inference demonstration**

rather than self-play.

---

# **82\. Full Human Game Flow**

The complete human match is:

MAIN MENU  
    ↓  
NEW CHAMPIONSHIP  
    ↓  
Initialize 12-card decks  
    ↓  
Shuffle decks  
    ↓  
Draw 3 starting cards  
    ↓  
Select 3 circuits  
    ↓  
Display circuit information  
    ↓  
TURN 0 COMPLETE  
    ↓  
ROUND 1  
    ↓  
Draw 1 card  
    ↓  
Activate Circuit 1  
    ↓  
Energy \= 1  
    ↓  
Human places cards  
    ↓  
Opponent chooses actions  
    ↓  
Human presses READY  
    ↓  
Resolve round  
    ↓  
ROUND 2  
    ↓  
Draw 1 card  
    ↓  
Activate Circuit 2  
    ↓  
Energy \= 2  
    ↓  
...  
    ↓  
ROUND 3  
    ↓  
Activate Circuit 3  
    ↓  
...  
    ↓  
ROUND 6  
    ↓  
Final decisions  
    ↓  
Resolve  
    ↓  
Calculate circuit winners  
    ↓  
Calculate championship result  
    ↓  
WIN / DRAW / LOSS  
---

# **83\. Full RL Environment Flow**

The RL version is:

RESET  
    ↓  
Seed environment  
    ↓  
Shuffle deck  
    ↓  
Draw 3 cards  
    ↓  
Select 3 circuits  
    ↓  
Initialize state  
    ↓  
ROUND 1  
    ↓  
Draw card  
    ↓  
Activate Circuit 1  
    ↓  
RL observes state \+ action mask  
    ↓  
RL selects legal action  
    ↓  
Card placement OR PASS  
    ↓  
If card placement:  
    ↓  
Update state  
    ↓  
Return next observation  
    ↓  
RL chooses another action  
    ↓  
...  
    ↓  
PASS  
    ↓  
Opponent completes decision  
    ↓  
Resolve round  
    ↓  
ROUND 2  
    ↓  
...  
    ↓  
ROUND 6  
    ↓  
Final resolution  
    ↓  
Calculate result  
    ↓  
Reward:  
\+1 / 0 / \-1  
    ↓  
TERMINAL  
---

# **84\. Example RL Decision**

Suppose the agent is in Round 4\.

State:

Energy \= 4

Hand:  
Slot 1 → Cost 1 / Power 2  
Slot 2 → Cost 2 / Power 4  
Slot 3 → Cost 4 / Power 7  
Slot 4 → Empty  
Slot 5 → Cost 3 / Power 5  
...

The agent may choose:

Action 0

meaning:

> Slot 1 → Circuit 1

After the action:

Energy \= 3  
Slot 1 \= empty  
Circuit 1 Power increases

The environment returns the updated state.

The agent may then choose:

Action 16

meaning:

> Slot 6 → Circuit 2

or eventually:

Action 27

meaning:

> PASS / END TURN.

---

# **85\. Strategic Structure of the Game**

The game deliberately creates several simultaneous strategic pressures:

### **Card availability**

The agent does not control which card is drawn.

### **Energy**

The agent has limited resources per round.

### **Circuit control**

The agent must decide where Power should be invested.

### **Timing**

Some cards and circuits become stronger during later rounds.

### **Opportunity cost**

Playing a strong card now means it cannot be used later.

### **Opponent pressure**

The opponent's Power influences the value of certain actions.

### **Stochasticity**

Different episodes provide different card and circuit configurations.

Together, these create the long-term decision-making problem required by the thesis.

---

# **86\. Example Strategic Situation**

Suppose the state is:

| Circuit | Player | Opponent |
| ----- | ----- | ----- |
| Le Womans | 8 | 13 |
| Fiji | 14 | 10 |
| Spa-Rainbow | 6 | 12 |

The player is:

Losing Circuit 1  
Winning Circuit 2  
Losing Circuit 3

The player has 4 Energy.

A new card costs 4 and has 7 Power.

The agent must determine whether to:

* contest Circuit 1,  
* contest Circuit 3,  
* reinforce Circuit 2,  
* use a cheaper card,  
* or preserve the card for a later round.

There is no single hard-coded instruction saying which choice is correct.

The optimal decision depends on:

* current board state,  
* card abilities,  
* circuit abilities,  
* remaining rounds,  
* future Energy,  
* and the probability of eventual championship victory.

That is the strategic structure the RL algorithms must learn.

---

# **87\. Why the Game Is Suitable for RL**

The environment contains:

### **State**

A structured representation of:

* round,  
* Energy,  
* cards,  
* Power,  
* circuits,  
* abilities,  
* opponent-visible state.

### **Actions**

Discrete card-placement and Pass decisions.

### **State transitions**

Cards alter:

* hand,  
* Energy,  
* circuit Power,  
* and future available decisions.

### **Stochasticity**

Deck and circuit randomization.

### **Delayed reward**

The ultimate reward occurs at the end of the match.

### **Long-term consequences**

Using a card now changes what is available later.

Therefore, the environment contains a meaningful sequential decision problem rather than merely a collection of independent classification decisions.

---

# **88\. Research-Relevant Simplification**

Despite the strategic elements, the game deliberately excludes systems that are unnecessary for the research objective.

The primary environment does **not** include:

* deck building,  
* card collection,  
* card upgrades,  
* player levels,  
* currency,  
* shops,  
* equipment,  
* fuel,  
* tires,  
* pit stops,  
* vehicle damage,  
* HP,  
* weather,  
* procedural maps,  
* real-time physics,  
* multiplayer networking,  
* matchmaking,  
* complicated card generation,  
* card movement,  
* card destruction,  
* resurrection,  
* card copying,  
* card stealing,  
* card spawning,  
* dozens of interacting abilities,  
* or self-play training.

The purpose is to keep the environment sufficiently rich for sequential decision-making while remaining experimentally controllable.

---

# **89\. Real-World Racing Inspiration**

The game uses endurance racing as its visual and thematic foundation.

The fictional cars are inspired by real prototype machinery such as the Aston Martin Valkyrie, BMW M Hybrid V8, Cadillac V-Series.R, Ferrari 499P, Toyota prototype, Alpine A424, Peugeot 9X8, and Genesis GMR-001 represented in the 2026 WEC Hypercar field. [FIAWEC](https://www.fiawec.com/en/news/decouvrez-la-liste-des-engages-pour-la-saison-2026-du-fia-wec/8580?utm_source=chatgpt.com)

Additional prototype inspiration can come from IMSA GTP cars such as the Porsche 963 and Acura ARX-06. Porsche continues competing with the 963 in IMSA GTP in 2026, while Acura continues with the ARX-06. [IMSA](https://www.imsa.com/news/2025/10/07/porsche-affirms-imsa-gtp-commitment/?utm_source=chatgpt.com)

This provides the visual identity of the game without making the game a literal simulation of WEC or IMSA.

---

# **90\. Relationship Between Game and Thesis**

The game is not the research question by itself.

The research question concerns the behavior and performance of different RL algorithms when operating within the same controlled environment.

The relationship is therefore:

GAME DESIGN  
     ↓  
State  
     ↓  
Actions  
     ↓  
Transitions  
     ↓  
Reward  
     ↓  
Gymnasium Environment  
     ↓  
DQN / PPO / A2C  
     ↓  
Training  
     ↓  
Evaluation  
     ↓  
Algorithm Comparison

Phase 6B defines the first half of this pipeline.

Phase 6A defines how the resulting environment is used scientifically.

---

# **91\. Final Environment Definition**

The resulting research environment can be summarized as:

> **A finite-horizon, single-agent, discrete-action, stochastic, turn-based strategic card-game environment in which an RL agent manages randomly drawn hypercar cards and limited Energy to compete for Power across three progressively activated circuits over six rounds against a fixed deterministic heuristic opponent.**

The environment provides:

* a fixed 12-card deck,  
* 3 starting cards,  
* one card draw per round,  
* maximum hand size of 9,  
* 6 rounds,  
* 1–6 Energy,  
* 3 circuits selected from a 6-circuit pool,  
* Power-based circuit scoring,  
* controlled Power/Energy abilities,  
* a fixed 28-action space,  
* state-dependent action masking,  
* structured numerical observations,  
* controlled stochasticity,  
* terminal win/loss/draw outcomes.

---

# **92\. Final Action Model**

The final RL action structure is:

9 hand slots  
        ×  
3 circuits  
        \=  
27 card-placement actions

\+

1 PASS / END TURN

\=

28 discrete actions

Therefore:

Discrete(28)

is the intended fixed action space.

---

# **93\. Final Hand Model**

The final hand progression is:

Turn 0 → 3 cards

Round 1 → 4 cards maximum  
Round 2 → 5 cards maximum  
Round 3 → 6 cards maximum  
Round 4 → 7 cards maximum  
Round 5 → 8 cards maximum  
Round 6 → 9 cards maximum

provided the player never plays any cards.

Cards that are played leave the hand.

---

# **94\. Final Circuit Model**

The game contains:

6 possible circuits

but only:

3 circuits per match

are selected.

The three selected circuits are:

* known from the beginning,  
* activated progressively,  
* represented explicitly in the observation,  
* and controlled by the environment RNG.

This gives environmental variety without increasing the action space beyond the intended 28 actions.

---

# **95\. Final Card Model**

The game contains:

12 fixed hypercar cards

Each card contains:

ID  
Name  
Energy Cost  
Base Power  
Ability Type  
Ability Trigger  
Ability Parameters

Abilities remain restricted primarily to:

Power manipulation  
Energy manipulation

with simple conditional and temporal triggers.

---

# **96\. Final Human Model**

Human gameplay includes:

Card selection  
Circuit selection  
Ready button  
30-second decision timer  
Retreat button

The human may play multiple cards per round.

The Ready button ends the human's decision phase.

Timeout automatically commits existing placements or acts as Pass if nothing was played.

---

# **97\. Final RL Model**

RL gameplay excludes human-only interface controls.

The agent receives:

Observation  
\+  
Legal-action mask

and chooses:

Card Slot → Circuit

or:

PASS / END TURN

The agent does not wait for a human timer.

---

# **98\. Final Opponent Model**

The primary opponent is:

> **A fixed deterministic heuristic policy.**

It does not learn.

It does not adapt its parameters during training.

It does not use self-play.

It operates under the same fundamental game rules and observable information available to the RL agent.

---

# **99\. Final Training Configuration Boundary**

Phase 6B defines the game mechanics.

It does **not** lock the final:

* number of training timesteps,  
* number of seeds,  
* evaluation episodes,  
* checkpoint frequency,  
* exact statistical procedure,  
* algorithm hyperparameters,  
* final sample-efficiency threshold,  
* or final hardware benchmarking protocol.

Those belong to the experimental research design in Phase 6A and the finalized implementation protocol.

Likewise, exact numerical card balancing may be adjusted during implementation validation before the final experiment is frozen, provided the final configuration is documented and then kept constant across algorithms.

---

# **100\. Final Scope**

The complete Hypercards environment is therefore:

> **12 fictional hypercar cards × 6 possible circuits × 3 selected circuits per match × 6 rounds × 1–6 Energy × randomized card draws × controlled Power/Energy abilities × fixed heuristic opponent × 28-action discrete interface × common invalid-action masking × structured MLP-compatible observation × terminal win/loss/draw outcome.**

The core research comparison remains:

> **DQN vs PPO vs A2C**

with the same game environment.

The game exists to provide a controlled but strategically meaningful sequential decision-making problem.

---

# **101\. Implementation Principle**

The most important implementation principle is:

> **The game rules must be finalized as one authoritative environment before the three algorithms are trained.**

The implementation should therefore avoid creating:

DQN version of the game  
PPO version of the game  
A2C version of the game

Instead, there should be:

ONE Hypercards environment  
        ↓  
 ┌──────┼──────┐  
 ↓      ↓      ↓  
DQN    PPO    A2C

All three algorithms interact with the same environment definition.

This preserves the central purpose of the thesis:

> **comparing learning algorithms rather than comparing different game implementations.**

