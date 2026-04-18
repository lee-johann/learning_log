Goal of IL:
- Given demonstration trajectories D, sampled from unknown policy $π_{expert}$ , goal is to learn policy at a similar level by mimicking D
Basic IL (problamatic):
- Basic level (deterministic trained policy): 
	- train $π_\theta$ (NN) s.t. $\min\limits_{\theta}\sum_{s,a\in D} ||\hat{a}-a||^2$ , where $\hat{a} = π_\theta (s)$
	- train this by regular SGD
	- problem: D might not be deterministic (for same state, have multiple distributions of actions in the dataset), in this case the trained policy will go to mean of the distributions (which might not be valid)
		- ex. 30% turns into left lane, 70% right lane, mean ends up between lanes
Learning distributions with NNs:
- output params of distributions over actions given state
- generative models for policies: max the likelihood of the demonstrations $\min\limits_\theta -logπ_\theta (a|s)$
	- mixture of gaussians: output mean, SD, weight per gaussian
	- discretize + autoregressive: 
		- ex. for action = [steering, acceleration], you bin the steering angles, then output prob for each bin (like a classification problem), next we output distribution over acceleration given steering angle
		- autoregression is over dimensions of the action set <– ordering you pick by hand (ex. for robot arm you go down the kinematics chain), but joint distribution over all is the same regardless of order (one at a time is more data efficient b/c avoids all the pairwise probs if model the whole joint distribution at the same time)
		- to train, for first P($a_{t1}$) it's cross entropy, for second P($a_{t2}|a_{t1}$) you use $a_{t1}$ from D instead and cross entropy. You backprop each loss. 
		- Q: why discretize a continuous variable if cross entropy doesn't differentiate between near or far bin? A: near or far bin is usually easy to learn, but how fine the discretization is still a problem, can also smooth around labels (so they're not 1-hot vectors)
	- diffusion over actions (not in video, TODO myself)
- expressive power of distribution depends on your NN outputs, separate from the expressive power of the function (complexity of your NN)
- Most SOTA models use imitation learning + expressive policies (diffusion has PI, NVDA Groot, Figure Helix; discretize + autoregressive has OpenVLA, Waymo, Wayve)
- Pros & cons:
	- offline 
	- doesn't need a reward func
	- but needs a lot of data for reliable performance
// I wonder how you find what gaps there are in the learned distribution there is from these static examples, surely RLHF has a similar problem
What can go wrong in IL:
- Compounding errors from covariate shift: as you stray away from states in the dataset, you might not have enough data there (i.e. distribution of states visited by policy diff from expert) and be more likely to make mistakes there, thus drifting even further
- solution: collect corrective data (if you stray off, how to get back)
	- rollout policy, then query expert at those states (expensive), add data back to dataset <– called DAgger (dataset aggregation)
	- sometimes hard to tell exactly what steering angle should be used, so instead when drift off expert fully takes over and completes task (partial demonstrations after policy makes mistake)
		- easier for providing corrections
		- do note when to take control
	- // i wonder if you rly need a human expert for correction, could it be a simpler problem and thus use another algo?
- 