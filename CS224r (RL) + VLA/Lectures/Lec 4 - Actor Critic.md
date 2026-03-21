defs
- (actor critic is basis for PPO)
- def value $V^π(s)$ future expected reward starting at s, then following π
- def $Q^π(s,a)$ future expected reward starting s, taking a, then following π
- def advantage $A^π(s,a)=Q^π(s,a)-V^π(s)$ how much better it is to take a than to follow policy π at state s

Inspiration
- policy gradient doesn't make efficient use of data b/c it's not specific enough about what's good what's bad
	- ex. partial success not utilized
- vailla policy gradient is $\nabla_{\theta}J(\theta) \approx \frac{1}{N}\sum_\limits{i=1}^{N} \left[\left(\sum_\limits{i=1}^{T}\nabla_\theta \, log \, π_\theta(a_{i,t}|s_{i,t})\right)\,\left(\sum_\limits{t'=t}^{T}r(s_{i,t'},a_{i,t'})\right)\right]$
	- we're going to change the estimate of the future reward if we take a in state s $\sum_\limits{t'=t}^{T}r(s_{i,t'},a_{i,t'})$
	- what if we use Q instead here ^ , the true expected reward of an action
	- reintroduce b, now instead use average Q, which is in fact V
	- but Q-V = A, so new function is $\nabla_{\theta}J(\theta) \approx \frac{1}{N}\sum_\limits{i=1}^{N} \left[\sum_\limits{i=1}^{T}\nabla_\theta \, log \, π_\theta(a_{i,t}|s_{i,t})\,A^π(s_{i,t},a_{i,t}) \right]$
- why this helps: instead of reward for 1 trajectory, we now look at what's the expected reward
- now, the training is 1) run policy 2) estimate expected return (this is new) 3) improve policy

How can we estimate V, Q, or A?
- $A = Q - V = r(s_t, a_t) + E_{s_{t+1}\sim p}[V^π(s_{t+1})] - V^π(s_t) \approx r(s_t, a_t) + V^π(s_{t+1}) - V^π(s_t)$ 
	- we use the approx b/c one sampled next state is an unbiased estimate of the expectation over all possible next states
	- therefore, if next a is pretty deterministic, it'll be good estimate, if next a is uncertain, it'll be a poorer estimate
- // question: do I have to re-estimate $V^π$ from scratch every time I change my policy? Isn't that kinda slow or inefficient if π doesn't differ by much across gradient updates?
	- Ans: we'll go over this later
- Version 1: monte carlo
	- $V \approx \sum_\limits{t'=t}^{T}r(s_{t'},a_{t'})$ <– single rollout approx of the expectation (if you have simulator that can reset world to $s_t$ then can consider multiple sample estimate) 
	- train supervised model to input s, output that sum
- Version 2: bootstrapping (also called temporal difference)
	- idea: use own estimates as well, not just raw r
	- instead of  $V^π\approx r(s_t, a_t) + V^π(s_{t+1})$ we use $r(s_t, a_t) + \hat{V}_\phi^π(s_{t+1})$ 
	- q: is there issue if own estimate sucks? Ans: not in practice, in theory can start with montecarlo then change to this
	- we aggregate info across trajectories by using own estimate (not just this trajectory, but also what's the value of future states that this trajectory passes through)
		- b/c you can arrive at that future state from a different starting point, and these two trajectories can have different endings, we want to use both in the calculation of the current state (MC only follows 1 trajectory, so the other traj which intersects with me in the future isn't used in my estimate of this current state)
-  Version 3: n-step returns
	- Is there some middle ground (b/c MC high var, bootstrapped has bias) <– use value estimate after N time steps instead of 1 time step 
	- MC: $\sum_\limits{t}^{T}r$, TD: $r_t + V_{t+1}$, how about  $\sum_\limits{t}^{t+n-1}r + V_{t+n}$ ?
	- helpful for small time steps like 10ms
	- In practice this works best. Notice t=1 is TD, t=N is MC

Aside: discount factors
- for long T, V can get very large, so discount future rewards can help
- also useful if V's variance increases a lot as you step into the future

