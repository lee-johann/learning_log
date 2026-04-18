Issues with IL: can't outperform expert, doesn't allow improvement from practice

Online RL outline: init policy (randomly if reward dense, IL, heuristics), run policy, then improve policy from rollouts

Policy gradient
- obj J($\theta$) = expected reward over rollouts, to $\max\limits_\theta$ J, $\nabla_{\theta}J(\theta)=\int p_\theta (\tau) r(\tau) d\tau =\nabla_{\theta} E_{\tau \sim P_\theta(\tau)}[r(\tau)]$ $=E_{\tau \sim P_\theta(\tau)}\left[\left(\sum_\limits{i=1}^{T}\nabla_\theta \, log \, π_\theta(a_{t}|s_{t})\right)\,\left(\sum_\limits{i=1}^{T}r(s_{t},a_{t})\right)\right]$ 
		- note: the first term is the same as the gradient in the IL objective. Therefore think of this as increasing likelihood of data, weighted by the reward
			- inc prob if high reward, lower if neg
	- this is called REINFORCE or vanilla policy gradient
	- In practice, instead of $E_{\tau \sim P_\theta(\tau)}$ we use $\frac{1}{N}\sum\limits^{N}_{i=1}$  
	- Gradient update: sample, calculate $\nabla_{\theta}J(\theta)$, then $\theta \leftarrow \theta + \alpha \nabla_{\theta} J(\theta)$ 
	- side note: $\nabla_\theta$ calculated with automatic differentiation in Torch

Issue: gradient might be noisy ex. large step backward + small step forward is negative reward overall
- improvement: instead of reward over all states cumulatively, we grad starting from t (what's the best outcome given where I'm at)
	- $\sum_\limits{i=1}^{T}r(s_{t},a_{t})$ becomes $\sum_\limits{t'=t}^{T}r(s_{t'},a_{t'})$
	- b/c actions now only impact rewards in the future

Issue: policy is encouraged to do everything with pos reward, instead of best action ex. fall forward still pos reward even if walk / run forward is an option
- more broadly, we're sensitive to the scale of the reward
- critique: this is local minimum, if you let it run enough it'll run more b/c bigger gradient
	- comment: true, but this makes learning take longer
- improvement: replace $\int p_\theta (\tau) r(\tau) d\tau$ with $\int p_\theta (\tau)(r(\tau)-b) d\tau$ where $b=\sum\limits_{i=1}^{N}r(\tau)$ 
	- in other words, reward above-average actions
	- adding or subtracting constants doesn't change max wrt $\theta$
	- if we had expectation over policy instead of sum over rollouts, we wouldn't need this
		- think of this as unbiased way of reducing expectation estimate's variance
	- // optimal baseline derived from differentiation isn't average reward, but avg is close enough
- but still needs: dense rewards, large batches (might miss out on best trajectory)
// i wonder can we assign intermediate rewards using other models ex. for folding shirt, completion rate can be done using GPT mini, doesn't seem that hard of a problem

Implementation speed:
- the $\nabla_\theta \, log \, π_\theta(a_{t}|s_{t})$ term need N\*T backward passes, slow
- solution: implement surrogate obj that has same gradient: replace with $log \, π_\theta(a_{t}|s_{t})$, which is cross-entropy for discrete action policy, MSE for Gaussian policy

Issue: expectation is under current policy, so every gradient step we need to recollect data (instead we want to take more than 1 gradient step per batch of data)
- def: on-policy uses only cur policy's data; off-policy can use past policy's data
- importance sampling: $E_{x\sim p(x)}[f(x)] = E_{x\sim q(x)}[\frac{p(x)}{q(x)}f(x)]$ therefore can use samples from old policy and weight it by the relative likelihood of the sample under the new policy
	- assumes enough support for q(x) i.e. q(x) != 0
	- this estimation works best when p and q similar
		- common choice is to constrain the policy changes during gradient updates to help with this $D_{KL}(π_{new} || π_{old}) \leq \delta$
- therefore $J(\theta) = E_{\tau \sim \bar{p}}(\tau) \left[ \frac{p_\theta (\tau)}{\bar{p}(\tau)} r(\tau) \right]$ for old policy $\bar{p}$ 
	- caveat: over large t this $π(a_t | s_t)$ product either vanishes or explodes, instead approx by using $π(s_{i,t} , a_{i,t})$ 

