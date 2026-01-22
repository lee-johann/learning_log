## Intro
Deep RL's difference to traditional ML
- learn behavior π(a|s) <– learning from exp is more indirect way to some objective
- data not i.i.d. b/c a affects future s
Why
- actions have consequences (ex. after a song on spotify, the next. song probably shouldn't be the same song)
	- autonomous agents, human-facing AI, labels not accurate or differentiable, sys affects future
- direct supervision data might not be possible (like feedback for coding agent)
	- can help discover new solutions
	- helpful for guiding image generation language alignment, chip design, post-training LMs etc.
Open research
- rep of success -> reward learning
- generalizing behavior across scenarios
	- leverage large diverse datasets -> offline RL
	- transfer from other tasks -> multitask/meta RL
- long horizon tasks -> hierarchy, reasoning
- practice fully autonomously -> reset-free RL

## Intro to modeling behavior and RL
How to rep exp as data? 
- Trajectory seq (s or o,a), reward
- difference between s and o
	- dynamics P(s2 | s1, a1) <– transition usually markov
	- think of o as a func of s, might not be complete (ex. cameras miss action behind you)
	- sometimes past o is helpful b/c most immediate o doesn't have full context (ex. chat agent), you don't need this if you have s and transition markov
- ex. 
	- robot: s is RHB images, joint positions, joint velocities; a is next joint position; $\tau$ at some Hz; r is task completion
	- chatbot: o is user's most recent message, a is chatbot's next message, $\tau$ is conversation trace (variable length), reward is user feedback
- // for sparse rewards you sometimes jump-start it through a demo
- reward not deterministic b/c: s can be stochastic (ex. other cars), policy not always deterministic
	- therefore model reward $\max\limits_{π} E_{\tau \sim p(\tau)}[\sum_{s_t, a_t \in \tau}{r(s_t, a_t)}]$ 
	- ^ sometimes we weigh rewards differently across time ex. using some discount factor

why stochastic policy?
- helps with exploration
- data is stochastic (ex. data from multiple ppl who behave differently)
	- can think of generative model over a given s/o

how good is a policy? 
- value $V^π (s)$ is the expected reward starting from s if you follow policy π

types of algorithms in the rest of the course
- IL: mimic π that achieves high reward
- policy gradient: directly differentiate the objective (max expected sum of rewards)
- actor-critic: estimate value of cur π and use it to make π better
- value-based: estimate value of optimal π
- model-based: learn to model the dynamics, and use it for planning or π improvement

**why so many algs (in ML only GD)? Algos make different trade-offs**
- how easy is it to collect data with policy (ex. simulator easy, human-interactions expensive)
- how easy are different forms of supervision (demos, detailed rewards)
- how important is stability and ease of use (vs data efficiency) <– if data efficient have more time to change hyper-parameters
- actions space dimensionality, continuous vs discrete
- is the dynamics model easy to learn