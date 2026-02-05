In driving, tasks, dialog how do we set reward?
- how can we think about what is the expert trying to achieve (instead of blindly mimic like IL)

Goal classifiers
- predict success vs failure , use classifier output as reward signal
- issue: 
	- sparse reward (no intermediate)
	- might fool classifier but not complete task itself (policy can visit states not in D used to train classifier)
- how to prevent exploiting policy exploiting classifier OOD?
	- can be conservative on OOD states (label as negative) then retrain classifier (so the classifier decision boundary is only around where there's support and positive)
	- issue: can this overfit to positives, especially if small number of positives?
		- fix: regularize your classifier
	- issue: you keep adding negatives, it might not be balanced
		- fix: balance your dataset
	- issue: you might label states near the positives as negatives
		- if you balance your dataset, this might be ok b/c since half of dataset is true positive, for positive state it's still more likely positive than negative (b/c false neg <= 50% since true neg >= 0))
		- regularization will also help
	- can also label subset of data
- Q: can we use GAN? 
	- turns out this is a form of GAN! (can borrow stability techniques from GAN)
- can use all the states instead of goal state

Rewards from human preferences (good behavior / bad behavior), want to grade trajectory and not just goal state
- relative preferences easier for trajectory than absolute performance
- if $\tau_{win} > \tau_{lose}$ then we're saying $\sum\limits_{\tau_{win}} r > \sum\limits_{\tau_{lose}} r$ 
- how do we aggregate these preferences to learn a reward function?
	- maybe prob $\tau_{win} > \tau_{lose}$ model as $\sigma (r(\tau_w) - r(\tau_l))$ 
	- so $\max\limits_\theta \sum\limits_{\tau_w, \tau_l}  \sigma (r_\theta(\tau_w) - r_\theta(\tau_l))$ 
- can also use partial traj instead of full traj (ex. cleaning kitchen has many sub tasks)
- issue: this objective doesn't take into account how large the preference is
	- we need a lot of pairwise data to have transitivity 
	- or we can ask humans to rank
1. given dataset, from same starting state we sample k traj and ask humans to rank
2. compute reward using reward model
3. for all pairs per batch, compute gradient of win-loss
4. update reward model
- we can do this within the loop of online RL
	- ex. hard to specify how to do a backflip without using preferences. Same with driving to weight different factors (like collision avoidance vs staying within lane vs keeping speed). 
- LLMs too
	- interesting that she frames SFT as Imitation Learning (for a given prompt, this is how you should respond)
	- can also use AI feedback, especially on specific axis of preferences ex. which of these responses is less harmful
		- key insight: critique is easier than generation

