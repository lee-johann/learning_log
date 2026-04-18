Guest Lecture Aviral Kumar (conservative q-learning first author)
- also training LM to self correct via RL at Deepmind https://arxiv.org/abs/2409.12917

Why does next token prediction pretraining work?
- one perspective: $|\hat{p_\theta}(y|x) - P^*(y|x)| \propto \frac{1}{|D(y|x)|^\alpha}$ 
	- error between learned model and ground truth model decreases with more data
- issue: data is limited (embodied AI, high quality math data)
- you end up with answers that look correct but aren't

Outline (focused on math due to time limitation):
- Classical RL (two of his papers https://arxiv.org/abs/2406.14532 and https://arxiv.org/abs/2410.08146)
	- // first paper says SFT is line IL, instead we want advantage, so get reward model to learn bad examples too
	- imitation learning
	- offline RL
	- online RL
- takeaway: training w/ RL can improve efficiency of learning
- Modern RL
	- online RL
	- "meta"-actions and long lengths
- takeaway: still the old recipes

## classical 
Math problem: think of problem as initial state, each step towards the answer as an action, reward if answer correct
	- this is like a sparse reward MDP w/ deterministic dynamics (no external noise for next state)

Data scaling
- 3 paradigms: 
	- SFT correct solution
	- SFT on many correct solution traces (on policy rollouts filtered for correctness), called RFT (reject (incorrect ones) FT)
		- // dedup here ex. sample 2N then keep N that's most diverse
	- full blown RL (also use bad solutions)
	
- plot test error vs # questions
	- RFT gets 2x improvement over SFT, but overall rate is slow (if we fit D^alpha we find alpha ~ -0.05 for math)
	- but if we keep scaling SFT on-policy, you eventually degrade (after 30k questions error flat and sometimes even goes back up)
- // maybe we can think of if you train too much on a particular set of initial states, you find it harder to generalize to new initial states?
- explanation: spurious steps (incorrect steps, that you recover from during training (// maybe it's b/c we filter out wrong answers) but not test time) derail the model (IL's analogous problem is called causal confusion)
- how to address this: assign "credit" to each step to filter bad steps (paper: RL on Incorrect synthetic data scales efficiency eight-fold)
	- starting from step x of the trace, what's our odds of getting to the right answer (value function of the trace up till now)
	- now calculate Q value of each action (define as V of next state after action)
	- // he argues should use another policy to calculate this, but some NLP ppl use same policy
	- very related to trajectory stitching
	- // since no r at each step and transition dynamics static, they use A=Q_now - Q_prev in the paper
	- then checked did this step cause A to go up or down?
	- (Q function the NLP folks call PRM, process reward model, the other way is to use rollouts to estimate A)
	- helps localize which step makes us wrong (much better data efficiency)
- // i wonder if we IL enough will we get same LLM "understanding" of the world?

- using this advantage for training
	- option 1: filter steps for positive advantages (even if overall traj incorrect, we keep good steps) <– like advantage weighted regression but filter instead of e^a
		- results: helped w/ scaling

- how to use advantage for offline RL (DPO)
	- option 2: retain partial rollouts for training
	- now you don't have to throw out bad rollouts, instead can take a bad rollout and form preference pair with a good rollout (if same prefix)
	- // can prompt model to produce logic in steps, usually complies

- Online RL
	- basic recipe with 0/1 reward (called outcome-reward model) for correctness using policy gradient ($\nabla log \; π \cdot r(x,y)$)
		- other work uses PPO (learned value func), GRPO (rollout based estimation of advantages)
	- per-step advantages, predict step advantage with reward model (process advantage verifier)

- RL training objectives similar now, but actions space changed
	- "meta" actions like revisiting an answer, verification, backtracking, planning
	- ^ think of this as new action space
	- // output length increases a lot