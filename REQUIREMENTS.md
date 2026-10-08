# REQUIREMENTS.md — Software and Experimental Requirements

This document translates the authoritative research methodology ([`Phase6A.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6A.md)) and game specification ([`Phase6B.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6B.md)) into concrete, verifiable technical requirements for the **Hypercards Prototype**.

---

## 1. Functional Game Engine Requirements (`GAME-xxx`)

- **`GAME-001`**: The engine SHALL simulate a two-player, turn-based card game over exactly 6 rounds.
- **`GAME-002`**: Each match SHALL initialize with a pool of 6 predefined fictional circuits and select exactly 3 distinct circuits without replacement.
- **`GAME-003`**: The 3 selected circuits SHALL activate progressively: Circuit 1 in Round 1, Circuit 2 in Round 2, Circuit 3 in Round 3, and all 3 circuits active for Rounds 4–6.
- **`GAME-004`**: Each player SHALL possess a fixed 12-card hypercar deck with fixed attributes (Cost, Base Power, Ability) as defined in Phase 6B Section 17.
- **`GAME-005`**: During Turn 0 (match initialization), each player SHALL draw 3 cards from their shuffled deck.
- **`GAME-006`**: At the start of each round (Rounds 1–6), each player SHALL draw exactly 1 card, resulting in an initial hand of 4 cards in Round 1 and a maximum possible hand of 9 cards in Round 6.
- **`GAME-007`**: The engine SHALL allocate $N$ base Energy at the beginning of Round $N$ (1 Energy in Round 1, scaling up to 6 Energy in Round 6).
- **`GAME-008`**: Unused Energy SHALL reset at the end of each round and SHALL NOT carry over to subsequent rounds unless explicitly permitted by an active circuit or card ability (e.g., Circuit 5 Americas Crown or Card 9 Porch 963).
- **`GAME-009`**: A player SHALL be permitted to deploy multiple cards within a single round as long as the card is in hand, the target circuit is active, and sufficient Energy remains.
- **`GAME-010`**: When a card is played, the card SHALL be removed from the player's hand, its Energy cost deducted from available Energy, and placed at the designated circuit.
- **`GAME-011`**: Card and circuit abilities SHALL be restricted strictly to Power manipulation and Energy manipulation.
- **`GAME-012`**: Circuit Power totals SHALL be calculated deterministically as the sum of base card powers plus all active card and circuit ability modifiers.
- **`GAME-013`**: At the conclusion of Round 6, each of the 3 circuits SHALL be evaluated independently: the player with higher Power wins the circuit; identical Power results in a circuit draw.
- **`GAME-014`**: Championship match resolution SHALL evaluate circuits won: winning $\ge 2$ circuits constitutes a Match Win; losing $\ge 2$ circuits constitutes a Match Loss; otherwise, the match is a Match Draw.

---

## 2. Environment Interface Requirements (`ENV-xxx`)

- **`ENV-001`**: The environment SHALL implement the standard `gymnasium.Env` interface (`reset(seed=..., options=...)` and `step(action)`).
- **`ENV-002`**: The environment SHALL support fully headless execution without graphical dependencies or windowing servers.
- **`ENV-003`**: An environment decision timestep (`step(action)`) SHALL correspond to a single decision by the RL agent (playing one card or choosing Pass), *not* an entire round.
- **`ENV-004`**: When the RL agent executes a valid card-placement action (Actions 0–26), the environment SHALL update state, deduct energy, evaluate on-play effects, and return `(next_obs, reward=0, terminated=False, truncated=False, info)`.
- **`ENV-005`**: When the RL agent executes Action 27 (`PASS / END TURN`), the environment SHALL prompt the opponent to complete its decisions, resolve the round, advance to the next round (or terminal resolution), and return `(next_obs, reward, terminated, truncated=False, info)`.
- **`ENV-006`**: The environment SHALL set `terminated=True` only upon completion of Round 6 match resolution.
- **`ENV-007`**: The environment SHALL maintain a single common implementation shared by all RL algorithms without algorithm-specific branching in game logic.

---

## 3. Action Space & Masking Requirements (`ACT-xxx`)

- **`ACT-001`**: The action space SHALL be a fixed discrete space: `gymnasium.spaces.Discrete(28)`.
- **`ACT-002`**: Actions `0` through `26` SHALL represent the 27 combinations of 9 hand slots $\times$ 3 circuits (Mapping: `action = slot_idx * 3 + circuit_idx` for `slot_idx` in `0..8` and `circuit_idx` in `0..2`).
- **`ACT-003`**: Action `27` SHALL represent `PASS / END TURN`, concluding the agent's actions for the current round.
- **`ACT-004`**: The environment SHALL compute a state-dependent binary action mask of shape `(28,)` with `dtype=np.int8` or `np.bool_`, where `1` indicates legal and `0` indicates illegal.
- **`ACT-005`**: A card placement action ($a \in [0, 26]$) SHALL be marked illegal if:
  - The designated hand slot is empty;
  - The target circuit is not yet active in the current round;
  - The card's effective Energy cost exceeds the player's currently available Energy.
- **`ACT-006`**: Action `27` (`PASS`) SHALL always be marked legal whenever it is the agent's turn to act.
- **`ACT-007`**: The environment SHALL expose the legal action mask via `info["action_mask"]` on `reset()` and `step()`, and/or through an environment method compatible with action-masking wrappers.
- **`ACT-008`**: If an illegal action is submitted to `step()`, the environment SHALL raise a clear exception or handle it according to the standardized invalid-action protocol without silent state corruption.

---

## 4. Observation Space Requirements (`OBS-xxx`)

- **`OBS-001`**: The observation space SHALL be a fixed-size 1D continuous vector: `gymnasium.spaces.Box(low=..., high=..., shape=(D,), dtype=np.float32)`.
- **`OBS-002`**: The observation vector SHALL contain structured numerical representations of:
  - Global match state (current round, current energy, deck size remaining, hand size);
  - Circuit states (for each of 3 circuits: active status, own power, opponent power, circuit ability category/parameters);
  - Hand slot states (for each of 9 hand slots: occupied flag, card cost, base power, ability type, ability condition, ability value, ability round);
  - Opponent public information (opponent circuit powers, opponent remaining deck count, opponent hand card count).
- **`OBS-003`**: Empty hand slots SHALL be represented using consistent neutral zero-padding.
- **`OBS-004`**: The observation vector SHALL NOT contain natural-language descriptions or text.
- **`OBS-005`**: The observation vector SHALL NOT leak unobservable private information, including the opponent's private hand contents, future card draw order, or internal PRNG state.

---

## 5. Reward Requirements (`RWD-xxx`)

- **`RWD-001`**: The environment SHALL assign a terminal reward of `+1.0` if the RL agent wins the championship match ($\ge 2$ circuits won).
- **`RWD-002`**: The environment SHALL assign a terminal reward of `-1.0` if the RL agent loses the championship match ($\ge 2$ circuits lost).
- **`RWD-003`**: The environment SHALL assign a terminal reward of `0.0` if the championship match ends in a draw.
- **`RWD-004`**: All intermediate decision steps (Rounds 1–5, and Round 6 prior to terminal resolution) SHALL receive a reward of `0.0`.
- **`RWD-005`**: The environment SHALL NOT apply step penalties (e.g., `-0.001` per step) or heuristic intermediate reward shaping in the primary research configuration.

---

## 6. Randomness & Reproducibility Requirements (`RNG-xxx`)

- **`RNG-001`**: All environment-level stochastic events (deck shuffling, initial card draw, round card draws, circuit pool selection) SHALL be governed by an explicit `Environment RNG`.
- **`RNG-002`**: Resetting the environment with a specific seed (`env.reset(seed=S)`) SHALL deterministically reproduce the exact sequence of circuit selections and card draws for that episode.
- **`RNG-003`**: The `Environment RNG` SHALL be strictly decoupled from the RL algorithm's internal training/exploration RNG.
- **`RNG-004`**: The evaluation harness SHALL maintain a standardized, held-out evaluation seed set (e.g., Seeds 1001–1100) that is identical across all algorithms and all evaluation checkpoints.
- **`RNG-005`**: The training pipeline SHALL support matched independent training seed sets (e.g., 5 independent runs per algorithm).

---

## 7. Opponent Requirements (`OPP-xxx`)

- **`OPP-001`**: The opponent SHALL be a fixed, non-learning deterministic heuristic agent.
- **`OPP-002`**: The opponent SHALL operate under the identical game rules, energy constraints, and legal action definitions as the player.
- **`OPP-003`**: The opponent SHALL evaluate legal moves using a fixed heuristic scoring function based on power yield, energy efficiency, circuit deficits, and round progression.
- **`OPP-004`**: The opponent SHALL employ a strict deterministic tie-breaking policy to ensure zero random jitter in opponent behavior given identical board states.
- **`OPP-005`**: The opponent SHALL NOT have access to the RL agent's private hand or future deck draws.
- **`OPP-006`**: The opponent's decisions during a round SHALL remain hidden from the RL agent until the round resolution phase.

---

## 8. RL Integration & Experimental Control Requirements (`RL-xxx`)

- **`RL-001`**: The test harness SHALL support training and evaluation for Deep Q-Network (DQN), Proximal Policy Optimization (PPO), and Advantage Actor-Critic (A2C).
- **`RL-002`**: All three algorithms SHALL be trained under an identical total environment interaction budget (total environment timesteps).
- **`RL-003`**: All three algorithms SHALL be evaluated on the identical held-out evaluation seed set.
- **`RL-004`**: Evaluation metrics SHALL record Win Rate (primary), Mean Episode Return (supporting), Sample Efficiency (timesteps to threshold), and Standard Deviation across training seeds (stability metric).
- **`RL-005`**: Aggregated performance reporting SHALL compute Interquartile Mean (IQM) and bootstrap confidence intervals across runs as specified in Phase 6A.
- **`RL-006`**: All algorithms SHALL receive the identical observation representation, discrete action mapping, and legal-action mask information.

---

## 9. Architecture & Modularity Requirements (`ARCH-xxx`)

- **`ARCH-001`**: Pure game logic (`hypercards.core`) SHALL have zero dependencies on `gymnasium`, `torch`, `stable-baselines3`, or `godot`.
- **`ARCH-002`**: The Gymnasium environment (`hypercards.env`) SHALL wrap the core game engine cleanly without embedding or rewriting game rules.
- **`ARCH-003`**: The Python engine SHALL serve as the authoritative single source of truth for all game rules and state transitions.
- **`ARCH-004`**: Game state objects SHALL be cleanly serializable to dictionaries / JSON for IPC communication with external clients.

---

## 10. Testing & Validation Requirements (`TEST-xxx`)

- **`TEST-001`**: Unit tests SHALL verify every card's energy cost, power contribution, and conditional ability trigger against Phase 6B Section 17 & 19.
- **`TEST-002`**: Unit tests SHALL verify all 6 circuit abilities and progressive activation schedules against Phase 6B Section 6–12.
- **`TEST-003`**: Unit tests SHALL verify that legal action masks correctly invalidate empty slots, unaffordable cards, and inactive circuits across all rounds.
- **`TEST-004`**: Unit tests SHALL verify deterministic reproducibility under identical random seeds.
- **`TEST-005`**: Unit tests SHALL verify championship match scoring: win condition ($\ge 2$ circuits), loss condition, and draw condition.
- **`TEST-006`**: Integration tests SHALL verify Gymnasium API compliance using standard environment checking tools (`gymnasium.utils.env_checker.check_env`).

---

## 11. Client & Presentation Requirements (`CLIENT-xxx`)

- **`CLIENT-001`**: The Godot client (version 4.7.2) SHALL function strictly as a frontend for rendering, animations, audio, and user input.
- **`CLIENT-002`**: The Godot client SHALL NOT independently calculate power scores, energy deductions, legal action masks, or match outcomes.
- **`CLIENT-003`**: The Godot client SHALL communicate with the authoritative Python engine via a structured interface (e.g., JSON-RPC or stdio IPC).
- **`CLIENT-004`**: The Godot client SHALL support human gameplay against the deterministic heuristic opponent.
- **`CLIENT-005`**: The Godot client SHALL support demonstration modes with frozen, pre-trained RL models loaded in inference mode.

---

## 12. Open Implementation Concerns & Technical Investigations (`OPEN-xxx`)

> [!WARNING]
> **Action-Masking Parity Across Algorithm Families**:
> The literature-derived research design ([`Phase6A.md`](file:///home/axel/Development/Hypercards-Prototype/Phase6A.md)) strictly dictates that valid action masking is a controlled variable common to all three algorithms (DQN, PPO, A2C).
> 
> However, standard RL libraries (including Stable-Baselines3) present an asymmetry:
> - `MaskablePPO` is natively provided via `sb3-contrib`.
> - Standard SB3 does not provide out-of-the-box `MaskableDQN` or `MaskableA2C`.
> 
> **Requirements for Resolution**:
> - **`OPEN-001`**: The development team SHALL NOT silently adopt divergent masking paradigms (e.g., masking PPO while allowing unmasked DQN with negative penalty shaping).
> - **`OPEN-002`**: The development team SHALL investigate and benchmark masking adapters (e.g., custom Q-head masking for DQN, masked categorical distribution policy for A2C, or wrapper-level action translation) to guarantee theoretical and empirical parity across all three algorithms.
> - **`OPEN-003`**: Any proposed masking architecture must be formally documented and verified prior to launching primary training experiments.
