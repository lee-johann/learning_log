PPO
- start with base actor-critic
- modification 1: importance weighting to take multiple gradient steps
- Recall that surrogate objective $\hat{J}(\theta') \approx \sum_i \frac{π_{\theta'}(a_i|s_i)}{π_\theta(a_i | s_i )} \hat{A}^{π_\theta}(s_i,a_i)$ 
	- we increase likelihood (relative to the base policy) of high advantage actions 
	- the gradient of this is equal to the gradient we desire (we give torch this expression, torch does the grad)

Issue: $\hat{A}$ might not be accurate
- $\hat{A}$ is relative to old policy $π_{\theta}$ so if we take too many steps, it's estimate might be inaccurate
	- do try to visually check the policy trajectories to make sure it isn't doing anything stupid like under-exploring on suboptimal path b/c some advantage is high
- note there's two problems in this, first is that the advantage estimate is based off of limited data , second that this data's distribution follows the old policy
- lastly, b/c the first term in J is the relative probs, we're incentivized to differ significantly from the old policy

Solution: encourage new policy to be similar to old
- Idea 1: add KL divergence as penalty to objective
	- // do note now we need to keep old params, this costs mem (but can cache)
- Idea 2: clip the ratio between $\frac{π_{\theta'}}{π_\theta}$ between some $1-\epsilon, 1+\epsilon$ 
	- now if the policy differs by above the threshold, further increases don't upweight the advantage
	- we bound the importance weights

Issue: we don't want to clip if policy decreases prob of good action by a lot, or increases prob of bad action by a lot
- Sol: lower-bound the objective to the original: using min(original, clipped)
- https://spinningup.openai.com/en/latest/algorithms/ppo.html when A>0 we're min(original, 1+e), when A<0, we're min(original, 1-e)

Estimating the advantage:
- Generalized Advantage Estimate: use n-step returns idea to estimate advantage (less variance than 1 step estimate, more bias than all steps till t=T)
- usually pick high decay, low n

Overall workflow
1. sample from $π_\theta$
2. fit $\hat{V}^{π_\theta}_{\phi}$ to rewards in data
3. evaluate $\hat{A}^π$ using GAE
4. update policy with M gradient steps on surrogate objective $\hat{J}(\theta') \approx \sum_\limits{t,i} min\left(\frac{π_{\theta'} (a|s)}{π_\theta (a|s)} \hat{A}^{π_\theta}, clip\left(\frac{π_{\theta'}(a|s)}{π_\theta (a|s)}, 1-\epsilon, 1+\epsilon \right)\hat{A}^{π_\theta}\right)$ 
- typical hyperparams: 2000 timesteps in a batch of data, 10 epochs when update policy (~M=300, 64 bs), clipping range e=0.2, 500 iterations

Q: now we use 1 batch of data, multiple gradient steps. But how can we reuse data from previous batches?
- idea 1: maintain a buffer of past data "replay buffer"
- idea 2: remove on-policy assumptions from algs

Adjustments to use a replay buffer
- in step 1 add data to buffer
-  step 1.5 sample a batch from the replay buffer
- in step 3 average the objective over the replay buffer minibatch

issue: in step 3 $\hat{V}^{π_\theta}_{\phi}$ is wrt to current policy $π_\theta$ and not a mixture of your past policies that are in the replay buffer
- idea: what if we fit Q instead of V? Because Q takes a as input, we can switch up the a
- // but doesn't Q = r(s,a) + Q_t+1 still have it's a~π?
	- answer: in the next timestep, we can use the new a (under the new policy) instead of the a from the dataset (under old policy)
	- caveat: for estimated Q to be accurate, we need to have taken similar actions in the dataset
- approach:
	- sample $(s_i, a_i, s_i')$ from the buffer, where s' is next state
	- sample $\bar{a}_i' \sim π_\theta$ 
	- calculate $Q^{π_\theta}(s_i,a_i) \approx r(s_i, a_i) + \gamma Q^{π_\theta}(s_i',\bar{a}_i) = y_i$ 
	- use NN to fit y
		- but need sufficient action coverage to be accurate
		- aside: other methods use multiple $\bar{a}$ to get closer to expectation over π

issue: in step 4 the action in the batch is not the action that $π_\theta$ would take, so $\nabla_\theta \; log \; π_\theta (a | s)$ isn't right, nor is A
- fix 1: use Q instead of A for convenience
	- this is ok b/c we have a lot more data in buffer, prev motivation for A was variance
- fix 2: use $a^π \sim π_\theta$ instead of past policy's actions
	- this is likely good b/c current policy is probably better than past

Issue: s didn't come from $p_\theta (s)$
- but this isn't too bad, we want optimal policy on $p_\theta (s)$ , but we get optimal policy on a broader state distribution

This is called soft actor critic (SAC)
- full alg:
	1. take action $a \sim π_\theta (a|s)$, get (s, a, s', r), store in replay buffer R
	2. sample a batch {s_i, a_i, r_i, s'_i} from R
	3. update $\hat{Q}^π_\phi$ using targets $y_i = r_i + \gamma \hat{Q}^π_\phi(s_i', a_i')$
	4. compute $\nabla_\theta J(\theta) \approx \frac{1}{N} \sum_i \nabla_\theta \; log \; π_\theta (a^π_i | s_i) \hat{Q}^π(s_i, a_i^π)$ where $a_i^π \sim π_\theta (a | s_i)$ 
	5. update $\theta \leftarrow + \alpha \nabla_\theta J(\theta)$
- pro: more data efficient (reaches reward with less samples)
- con: harder to tune hyperparams compared to PPO, therefore less stable
	- therefore simulators like PPO, b/c data efficiency isn't a problem (if sim2real isn't a problem)