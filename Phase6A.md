### **PHASE 6A — Literature-Derived Research Design**

We establish everything the 155-paper review gives us:

**Research problem → RQs → algorithms → experimental philosophy → controls → training methodology → seeds → evaluation → statistical methodology → environment interface principles.**

# **PHASE 6 — Part A**

## **What the literature already establishes for D1**

### **6.1 Research problem**

**The literature gives us the basis for this:**

> **There are existing comparisons of RL algorithms, and there are RL studies in turn-based games, but there remains a methodological gap in controlled, multi-family comparison of DQN, PPO, and A2C within a single discrete turn-based game environment under standardized experimental conditions.**

---

# **6.2 Research questions**

**The literature supports questions around:**

1. **How can the DQN, PPO, and A2C algorithms be implemented in the same turn-based game environment?**  
2. **How do DQN, PPO, and A2C differ in performance when making decisions in a turn-based game environment?**  
3. **How do DQN, PPO, and A2C differ in learning efficiency, based on the number of interactions with the environment during training?**  
4. **How do the learning outcomes of DQN, PPO, and A2C compare across repeated training runs with different random seeds?**

**The exact wording is ours, but the dimensions come from the literature.**

---

# **6.3 Algorithms**

**This is one of the strongest things we can lock.**

### **DQN**

**Value-based, off-policy RL.**

### **PPO**

**On-policy policy-gradient method using a clipped surrogate objective.**

### **A2C**

**Actor-critic baseline.**

**The D1 bibliography explicitly identifies these three as the core comparison and provides separate theoretical foundations for them.** 

**Yuuna: So yeah, DQN/PPO/A2C are fucking locked.**

**Yuuko: For the research comparison, yes.**

### **Independent variable**

**RL algorithm**

* **DQN**  
* **PPO**  
* **A2C**

### **Controlled variables**

**All three use:**

* **the same Hypercards environment**  
* **the same game rules**  
* **the same observation representation**  
* **the same action space**  
* **the same legal-action mechanism/action masking**  
* **the same reward function**  
* **the same fixed deterministic heuristic opponent**  
* **the same interaction budget**  
* **the same hyperparameter-setting policy**  
* **the same hardware/software environment**  
* **matched training seeds**  
* **the same held-out evaluation seed set**

---

# **6.4 Environment interface**

**The literature strongly supports using a standardized Gymnasium environment.**

**Gymnasium provides the standard environment interface and compatibility with RL tooling such as Stable-Baselines3 to define hyperparameters setup.** 

**So:**

**Headless training environment → Gymnasium-compatible API**

**is a very defensible methodological choice.**

**But:**

> ***What the game actually is*** **is still ours.**

---

# **6.5 Action space**

**Literature strongly supports the general direction:**

> **Discrete action space**

**because we're specifically investigating these algorithms in a turn-based game with discrete decisions.**

**The turn-based literature also demonstrates discrete action/state representations and constrained action spaces across multiple game domains.** 

**But the papers do NOT decide our exact actions.**

**They cannot tell us:**

> **Up / Down / Attack / Defend / Heal**

**because that's our game.**

---

# **6.6 Action masking**

**This one is interesting.**

**The literature gives us a strong methodological basis for:**

> **Invalid actions should be handled explicitly rather than allowing agents to blindly select illegal actions.**

**Huang & Ontañón specifically support invalid-action masking in rule-governed environments.** 

**So we can likely establish:**

**The environment should have a well-defined mechanism for invalid actions.**

# **4\. DQN is actually very straightforward conceptually**

**DQN produces one Q-value for every action.**

**Suppose:**

**Q-values:**

**Action 0 →  3.2**

**Action 1 →  5.8**

**Action 2 →  9.7   ← ILLEGAL**

**Action 3 →  4.1**

**Action 4 →  7.3**

**Mask:**

**\[1, 1, 0, 1, 1\]**

**We effectively do:**

**Action 0 →  3.2**

**Action 1 →  5.8**

**Action 2 → \-∞**

**Action 3 →  4.1**

**Action 4 →  7.3**

**Therefore:**

**argmax \= Action 4**

**instead of illegal Action 2\.**

**But there's a very important second part.**

**When DQN calculates:**

> **"What is the best action in the next state?"**

**we must also mask that calculation.**

**SB3's current DQN implementation normally computes the next-state Q-values and takes the maximum across actions. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/master/_modules/stable_baselines3/dqn/dqn.html?utm_source=chatgpt.com)**

**We would change the conceptual operation from:**

**max(Q(next\_state, ALL actions))**

**to:**

**max(Q(next\_state, LEGAL actions))**

**And Gymnasium's own masking tutorial explicitly demonstrates this principle for Q-learning. [Gymnasium](https://gymnasium.farama.org/tutorials/training_agents/action_masking_taxi/?utm_source=chatgpt.com)**

**So DQN requires masking in:**

### **During exploration**

**random action ∈ legal actions**

### **During greedy action selection**

**argmax(Q) over legal actions**

### **During target calculation**

**max(Q\_next) over legal next actions**

**That's important.**

---

# **5\. PPO is different**

**PPO doesn't produce Q-values like DQN.**

**Its actor produces something like:**

**Action probabilities:**

**A0 → 0.12**

**A1 → 0.18**

**A2 → 0.30  ← illegal**

**A3 → 0.20**

**A4 → 0.20**

**We don't want:**

**A2 → 30%**

**So the mask modifies the action distribution:**

**A0 → 0.12**

**A1 → 0.18**

**A2 → 0**

**A3 → 0.20**

**A4 → 0.20**

**Then the remaining legal probabilities are normalized.**

**Conceptually:**

                **PPO policy**

                     **│**

              **action logits**

                     **│**

                     **▼**

              **apply mask**

                     **│**

                     **▼**

          **legal-action distribution**

                     **│**

                     **▼**

                  **sample**

**And this isn't some crazy theoretical hack. The invalid-action-masking literature specifically describes masking invalid actions in policy-gradient algorithms by restricting sampling to valid actions, and provides theoretical justification for the approach. [arXiv](https://arxiv.org/abs/2006.14171?utm_source=chatgpt.com)**

---

# **6\. And A2C does essentially the same thing**

**A2C also uses an actor-critic architecture.**

**Its actor produces an action distribution.**

**So:**

**A0 → 20%**

**A1 → 10%**

**A2 → 35% ← illegal**

**A3 → 15%**

**A4 → 20%**

**Mask:**

**A0 → 20%**

**A1 → 10%**

**A2 → 0%**

**A3 → 15%**

**A4 → 20%**

**Then normalize the remaining legal probabilities.**

**We don't just mask the action when choosing it.**

**The mask needs to be consistently applied when the algorithm evaluates the action distribution during training.**

**Why?**

**Because PPO and A2C calculate things such as:**

> **log probability of the selected action**

**and entropy of the action distribution.**

**SB3's PPO training currently calls the policy's `evaluate_actions()` to obtain values, log probabilities and entropy. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/master/_modules/stable_baselines3/ppo/ppo.html?utm_source=chatgpt.com)**

**A2C does the same thing. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/master/_modules/stable_baselines3/a2c/a2c.html?utm_source=chatgpt.com)**

**So our masked distribution needs to be used consistently there too.**

**Otherwise we'd have:**

**action selected using masked distribution**

             **↓**

**training evaluates it using unmasked distribution**

---

# **6.7 Reward design**

**The literature strongly warns us about two things:**

* **sparse rewards can make exploration difficult;**  
* **poorly designed intermediate rewards can create reward hacking.**

**The D1 bibliography specifically includes work on reward shaping, potential-based shaping, reward hacking, and sparse-to-dense transitions.** 

**Therefore, literature tells us:**

> **Reward design must be explicit, justified, and identical across algorithms.**

**But it does NOT tell us:**

> **"Axel's game should give \+10 for killing an enemy."**

**That's entirely ours.**

**So reward principles \= literature.**  
**Reward values/formula \= us.**

### **Reward philosophy**

**Start simple:**

> **Win \= \+1**  
> **Loss \= −1**  
> **Draw \= 0**  
> **Intermediate steps \= 0**

---

# **6.8 Training step**

**This is important.**

**The literature strongly supports equal interaction budgets when comparing algorithms.**

**So we should establish:**

> **Every algorithm receives the same number of environment interactions.**

**Greg's Phase 5 variable specification already proposed equal training timesteps as a controlled variable, supported by the reproducibility literature.** 

**An episode consists of multiple environment timesteps and does not necessarily contain a fixed number of timesteps.**

---

# **6.9 Random seeds**

### **The game contains stochasticity:**

* **initial card draw**  
* **subsequent card draws**  
* **deck shuffle**  
* **circuit selection/ordering if randomized**  
* **starting condition**  
* **stochastic heuristic decisions, if we choose them**

## **What does a seed do?**

**Yuuka: Say this:**

> **A random seed initializes the pseudorandom number generator, allowing stochastic environment behavior to be reproduced.**

**That's it.**

**For example:**

**Seed 42**

   **↓**

**Environment RNG**

   **↓**

**Episode 1 → scenario A**

**Episode 2 → scenario B**

**Episode 3 → scenario C**

**...**

## **Training seeds**

**Suppose we choose:**

**42**

**123**

**256**

**512**

**1024**

**Each seed represents an independent training run.**

**So:**

**DQN \+ seed 42 \+ fixed training budget**

**DQN \+ seed 123 \+ fixed training budget**

**...**

**And exactly the same seed set for:**

**PPO**

**A2C**

---

## **Evaluation seeds**

**These should ideally be separate from training seeds.**

**For example:**

**Training seeds:**

**42, 123, 256, 512, 1024**

**Evaluation seeds:**

**1001–1100**

**Then:**

**DQN → evaluation seeds 1001–1100**

**PPO → evaluation seeds 1001–1100**

**A2C → evaluation seeds 1001–1100**

**Yuurei: Thus, every algorithm encounters the same set of randomized evaluation scenarios.**

**Fixed deterministic heuristic opponent.**

**Algorithm \= variable**

**Opponent \= controlled**

### **Deterministic heuristic policy as the baseline.**

**For every decision:**

1. **Determine legal actions.**  
2. **Evaluate each legal action.**  
3. **Select the highest-scoring action.**  
4. **If two or more actions have exactly equivalent heuristic value, use a controlled tie-break.**

**The current Phase 6B already proposes evaluating legal actions according to strategic value and potentially using a small tie-break. Phase6B**

### **Why deterministic first?**

**Because it makes your experimental environment easier to explain.**

**If the opponent's behavior is deterministic given the same game state:**

> **Same state → same opponent action.**

**That removes another source of unnecessary uncertainty.**

**If later we discover deterministic behavior makes the game too predictable, we can introduce controlled stochastic tie-breaking.**

### **Environment RNG**

**Controls stochastic environment events:**

**Seed**

 **│**

 **├── Deck shuffle**

 **├── Initial card distribution**

 **├── Card draws**

 **├── Randomized circuit ordering, if used**

 **└── Any explicitly stochastic game event**

**The core game rules remain deterministic.**

**That's actually an important distinction.**

**We are not making the environment deterministic.**

**We're making its randomness controlled and reproducible.**

---

# **6.10 Evaluation**

**The literature strongly pushes us away from:**

> **"Run it once → average reward → DQN wins."**

**Instead, the evaluation should account for variability.**

**The D1 literature specifically supports:**

* **multiple seeds**  
* **confidence intervals**  
* **IQM**  
* **robust statistical comparison**  
* **standardized evaluation episodes**  
* **reproducible protocols**

**Agarwal et al. specifically motivates IQM and bootstrap confidence intervals, while Henderson et al., Colas et al., and related work address reproducibility and seed variance.** 

**So we can establish:**

### **Evaluation framework**

## **A. Learning Performance**

**Primary metric: Win Rate**

**$WinRate=\frac{Number\ of\ Wins}{Total\ Evaluation\ Episodes}$**

**This is the clearest measure of actual game performance.**

### **2\. Episode return**

**Yuuri: This was confusing you earlier...**

**Yuuko: And now it's easy.**

### **Return \= total reward accumulated during one episode.**

**Your reward is:**

**Round 1 → 0**

**Round 2 → 0**

**Round 3 → 0**

**Round 4 → 0**

**Round 5 → 0**

**Round 6 → \+1**

**Then:**

> **Episode return \= \+1**

**If the agent loses:**

**0 \+ 0 \+ 0 \+ 0 \+ 0 \- 1**

**Episode return:**

> **−1**

**If draw:**

> **0**

**So with your current reward design, episode return is closely related to game outcome.**

**Do not pretend win rate and mean return are completely independent metrics.**

**With ±1/0 terminal reward, they're mathematically closely related.**

**So I would make:**

> **Win rate \= primary performance metric**

**and:**

> **Mean episode return \= supporting performance metric**

**That's cleaner.**

### Mean Episode Return

**This is simply:**

> **The average episode return across many evaluation episodes.**

**Suppose:**

**Episode 1 → \+1**

**Episode 2 → \+1**

**Episode 3 → \-1**

**Episode 4 → \+1**

**Episode 5 →  0**

**Then:**

**$MeanReturn=\frac{1+1-1+1+0}{5}=0.4$**

**So:**

> **Mean episode return \= 0.4**

# **B. Learning efficiency**

**This should have two measurements.**

### **1\. Learning curve**

**Performance plotted against environment timesteps.**

**For example:**

**50k**

**100k**

**150k**

**200k**

**...**

**500k**

**At each checkpoint, evaluate the current model.**

**Then:**

**Win rate**

   **│**

   **│          PPO**

   **│       ╭────────**

   **│     ╭─╯**

   **│   ╭─╯**

   **│  ╱       DQN**

   **│ ╱    ╭────────**

   **│╱  ╭──╯**

   **└──────────────────**

          **Timesteps**

**This answers:**

> **How does performance develop as training progresses?**

**Phase 6A already establishes learning curves as part of learning behavior. Phase6A**

### **2\. Sample efficiency**

**Define it as:**

> **The number of environment timesteps required to reach a predefined performance threshold.**

**For example, if we eventually choose:**

> **70% win rate**

**then:**

**DQN → 420k timesteps**

**PPO → 280k**

**A2C → 350k**

**PPO would be more sample-efficient under that threshold.**

**Important: the actual threshold should be determined/justified before the final experiment. We don't want to choose 70% after seeing the graphs because one algorithm happens to cross it.**

---

# **C. Learning stability**

**How consistently does an algorithm produce similar learning outcomes when it is trained multiple times under different random seeds?**

**If I train DQN five times, does DQN reliably produce roughly similar results, or does it sometimes become amazing and sometimes completely shit itself?**

**Same for PPO and A2C.**

**That's why we need multiple independent training seeds.**

**For each algorithm:**

**Seed 42**

**Seed 123**

**Seed 256-**

**Seed 512**

**Seed 1024**

**Each produces an evaluation result.**

**Then analyze the distribution across independent runs.**

> # **1\. Mean**

**Mean answers:**

> **"What is the typical result?"**

**For DQN:**

> **72, 74, 70, 73, 71**

**Mean:**

> **\\\[ \\frac{72+74+70+73+71}{5}=72\\% \\\]**

**So:**

> **DQN's average final win rate \= 72%**

**But mean tells us nothing about how spread out those results are.**

**That's why mean alone isn't stability.**

> ---

> # **2\. Standard deviation**

**THIS is the simplest actual stability measure for your thesis.**

**Standard deviation asks:**

> **How far do the individual training runs tend to spread away from their mean?**

**Imagine:**

> ### **DQN**

> **70   71   72   73   74**

>              **↑**

>            **mean**

**Small spread.**

> ### **PPO**

> **55        62              81       84   89**

>                      **↑**

>                    **mean**

**Huge spread.**

**Therefore:**

> **Lower standard deviation \= more consistent training outcomes.**

> **Higher standard deviation \= more variable training outcomes.**

**For your thesis, this is extremely easy to explain.**

> ---

> # **So if we want ONE actual stability metric...**

> ## **I recommend:**

> ### **Standard deviation of the final evaluation performance across independent training seeds.**

**That's your fucking stability metric.**

**For example:**

| Algorithm | Mean Win Rate | SD |
| ----- | ----- | ----- |
| **DQN** | **72.0%** | **1.58%** |
| **PPO** | **74.2%** | **14.30%** |
| **A2C** | **69.0%** | **1.58%** |

**Interpretation:**

* **PPO has the highest average.**  
* **But PPO is much more variable.**  
* **DQN is much more consistent.**  
* **A2C is also consistent.**

**That is a meaningful thesis result.**

# **4\. IQM**

**Ah yes.**

**The fucking IQM.**

**Yuuki: Sounds like an exam.**

**Yuuko: IQM stands for Interquartile Mean.**

**It's basically a robust average.**

**Instead of simply:**

> **"Average everything."**

**it removes the most extreme observations from the upper and lower ends and averages the middle portion.**

**Why?**

**Because RL results can sometimes have weird outliers.**

**For example:**

**20%**

**22%**

**24%**

**25%**

**26%**

**27%**

**91%**

**That 91% could be an unusually lucky run.**

**A normal mean gets pulled upward by it.**

**IQM says:**

> **"Let's focus more on the middle of the distribution."**

**So IQM is useful for robust aggregation of performance.**

**And this is exactly why our Phase 6A says:**

> **don't call IQM "the stability metric."**

**Instead, IQM is a robust aggregation statistic, while variability is separately analyzed. Phase6A**

---

# **5\. NOW: Bootstrap Confidence Interval**

## **Imagine we say:**

**DQN's observed average win rate:**

> **72%**

**But you only ran five training seeds.**

**You obviously don't know the "true" average performance DQN would produce across every possible training run.**

**So statistics can say:**

> **"Based on our observed data, the plausible range for the underlying performance is approximately this."**

**That's what a confidence interval is trying to communicate.**

**For example, conceptually:**

**DQN mean \= 72%**

          **├───────────────┤**

        **68%             76%**

**You might report:**

> **72% with a 95% confidence interval of 68–76%.**

**The interval expresses uncertainty around an estimate.**

**It does NOT directly tell us:**

> **"DQN is stable."**

## **Primary stability analysis**

### **Standard deviation across independent training seeds**

**Each algorithm:**

**Seed 1**

**Seed 2**

**Seed 3**

**Seed 4**

**Seed 5**

**After evaluation:**

**Win rate per training run**

**Then calculate:**

**Mean**

**Standard deviation**

**Report something like:**

> **DQN achieved a mean win rate of 72.0% ± 1.6% across five independent training seeds.**

**That ± 1.6% is the standard deviation.**

**Then another evaluation using IQM and Confidence interval**

---

# **6.11 Experimental controls**

**This is probably the most important thing the literature gives us.**

**The comparison should keep constant:**

* **environment**  
* **observation representation**  
* **action semantics**  
* **training interaction budget**  
* **evaluation protocol**  
* **random-seed protocol**  
* **hardware/software stack**  
* **general hyperparameter policy**  
* **identical game**  
* **identical rules**  
* **identical observation representation**  
* **identical action semantics**  
* **identical reward**  
* **identical opponent**  
* **identical training budget**  
* **matched training seeds**  
* **same software/hardware environment**

**The D1 variable specification explicitly lays these out.** 

**So our fundamental experiment becomes:**

            **SAME GAME**  
                 **│**  
        **┌────────┼────────┐**  
        **↓        ↓        ↓**  
       **DQN      PPO      A2C**  
        **│        │        │**  
        **└────────┼────────┘**  
                 **↓**  
       **SAME EVALUATION**  
                 **↓**  
       **STATISTICAL ANALYSIS**

---

# **6.12 Hyperparameters**

# **Stable-Baselines3 defaults**

**I checked the SB3 2.9.0 documentation, since that's the version you're using.**

**And yes, the defaults are explicitly documented.**

## **DQN**

**Current SB3 2.9.0 defaults include:**

| Hyperparameter | Default |
| ----- | ----- |
| **Learning rate** | **0.0001** |
| **Replay buffer** | **1,000,000** |
| **Learning starts** | **100 steps** |
| **Batch size** | **32** |
| **Tau** | **1.0** |
| **Gamma** | **0.99** |
| **Train frequency** | **4 steps** |
| **Gradient steps** | **1** |
| **Target update interval** | **10,000 steps** |
| **Exploration initial ε** | **1.0** |
| **Exploration final ε** | **0.05** |
| **Exploration fraction** | **0.1** |
| **Max gradient norm** | **10** |

**SB3 documents these defaults directly. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/v2.2.1/modules/dqn.html?utm_source=chatgpt.com)**

**And because you're using a vector observation and discrete action space, DQN's default `MlpPolicy` uses a two-layer fully connected network with 64 units per layer. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/master/_modules/stable_baselines3/dqn/policies.html?utm_source=chatgpt.com)**

**So roughly:**

**Observation**

    **↓**

**64**

    **↓**

**64**

    **↓**

**Q-values for 13 actions**

---

# **PPO**

**SB3 2.9.0 defaults:**

| Hyperparameter | Default |
| ----- | ----- |
| **Learning rate** | **0.0003** |
| **n\_steps** | **2048** |
| **Batch size** | **64** |
| **Epochs** | **10** |
| **Gamma** | **0.99** |
| **GAE λ** | **0.95** |
| **Clip range** | **0.2** |
| **Entropy coefficient** | **0.0** |
| **Value-function coefficient** | **0.5** |
| **Max gradient norm** | **0.5** |
| **Advantage normalization** | **True** |

**These are the SB3 PPO defaults. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/_/downloads/en/master/pdf/?utm_source=chatgpt.com)**

**And PPO's default MLP is also:**

**Observation**

      **↓**

    **64**

      **↓**

    **64**

   **↙   ↘**

**Actor   Critic**

**SB3 documents the default 64×64 architecture for PPO/A2C/DQN with 1D observations. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/master/guide/custom_policy.html?utm_source=chatgpt.com)**

---

# **A2C**

**SB3 2.9.0 defaults:**

| Hyperparameter | Default |
| ----- | ----- |
| **Learning rate** | **0.0007** |
| **n\_steps** | **5** |
| **Gamma** | **0.99** |
| **GAE λ** | **1.0** |
| **Entropy coefficient** | **0.0** |
| **Value-function coefficient** | **0.5** |
| **Max gradient norm** | **0.5** |
| **RMSProp epsilon** | **0.00001** |
| **RMSProp** | **True** |
| **gSDE** | **False** |
| **Advantage normalization** | **False** |

**These are directly documented by SB3. [Stable Baselines3 Docs](https://stable-baselines3.readthedocs.io/en/v2.9.0/modules/a2c.html?utm_source=chatgpt.com)**

**Again, default MLP:**

**Observation**

      **↓**

    **64**

      **↓**

    **64**

   **↙   ↘**

**Actor   Critic**

---

# **6.13 Software architecture**

**The literature strongly supports:**

**Game logic → standardized environment interface → RL algorithms**

**Gymnasium provides the interface standard, and game/RL integration work provides precedent for connecting game environments with RL frameworks.** 

**So:**

### **Headless environment for training**

**Yes.**

### **GUI/game for demonstration**

**Potentially yes.**

### **Exact engine?**

**Not decided.**

### **Godot vs Python/Pygame vs something else?**

**Not decided.**

