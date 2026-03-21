Recap: estimating value of state
1. V Monte Carlo (D~π): $\sum\limits_{t'=t}^T r(s_t, a_t)$ 
2. V Bootstrapped / TD updates (D~π): $r(s_t,a_t) + \gamma \hat{V_\phi^{π_\theta}}(s')$ 
3. Q with TD updates ($s_t,a_t\sim π_{old}, a_t\sim π$): $r(s_t,a_t) + \gamma \hat{Q^{π_\theta}}(s_{t+1}, a_{t+1})$
	1. note that b/c we use $\hat{Q}$ as the label, we need it to be trained on actions that resemble the actions we're evaluating on (i.e. policy difference can't be big)
- SAC weighs likelihood by this, Q-learning skips this to learn optimal Q

Why offline RL
- leverages collected datasets
- online policy collection may be unsafe (ex. self driving)
- reuse previously online data
- formally, $\max\limits_\theta E_{\tau \sim p_\theta(\tau)} [r(\tau)]$
	- expectation is over learned policy $π_\theta$ but data collected by unknown policy $π_\beta$ so there's a distribution shift, but we don't get to sample from the learned policy

What happens if we use off policy algorithms? 
- $\hat{Q}$ might not be accurate b/c $π_\beta$ actions might not cover $π_\theta$ actions
	- these out of distribution actions won't have a change to get corrected b/c we don't run the policy 
	- given that Q is randomly initialized, actions out of the data support can be over or under optimistic, but max Q will exploit where Q is over-optimistic by chance
	- after policy update this is even worse b/c enforce fake high value Qs
	- fundamental issue: learned policy too different from behavioral policy
- question: can't I constrain to seen actions?
	- ans: yes some methods to this, what's hard is that some states might be similar but not exactly the same so what's OOD is not straight forward
	- but the idea of constraining policy to not differ to much and avoiding OOD actions is on point

How to mitigate over estimation?
- Imitation learning can work b/c it only uses actions in the data (so avoids OOD)
	- but offline dataset might not be optimal, we should be able to beat the behavioral policy through reward information
	- should be able to "stitch" together good behaviors (ex. one traj good first half, the second good second half, and they met somewhere in the middle; vanilla IL will take average action)
- baseline method: IL but only on good rewards

What if we weighted each transition depending on how good the action is (instead of the overall traj)
- $\max\limits_\theta \sum\limits_{s,a \in D} log \, π(a|s) e^{A(s,a)}$ <– same as IL by weighed by advantage A
	- recall A(s,a) = Q(s,a) - V(s)
	- this helps with stitching b/c now action per state Q
	- aside: can prove this objective ^ approximates max Q s.t. KL(policy || behavioral policy) < $\epsilon$ 
	- if our NN represents similar states similarly, similar s should have similar V(s), so should be able to generalize
	- A is of the behavioral policy
- // Chelsea said online will look at reverse KL, whereas offline forward KL
	- forward KL is E[log p(x)/q(x)], reverse is q(x)/p(x). Suppose p(x) is true while q(x) is approximate: 
	- In forward if p(x) = 0, q(x) doesn't matter. Whereas if p(x)>0, q(x) matters, so we'll ensure that everywhere there's p, q will be close (but there might be places with q but no p)
		- we avoid p(x)>0 when q(x) = 0 (this loss will be massive)
		- this is like everywhere there's an answer I should guess, although it's ok to guess anything where there's no answer
	- In reverse KL, if q(x)=0, there's no penalty on p(x). But if q(x)>0, then p's difference matters. So we'll ensure that everywhere there's q, there's p (but where there's q(x)=0, p can be anything)
		- i think it's a bit like every guess has to be right, although you can miss many places with answers
	- // since we can change p(x) and not q(x), then forward is wherever I guess there must be target (but I can choose to make p(x)=0). Reverse is everywhere there's a target I need a guess, but where there's no target I can guess anything)
- How to estimate A (of behavioral policy)?
	- V1 (advantage weighted regression): 
		- estimate V $\sum\limits_{t'=t}^T r(s_{t},a_{t})$ (monte carlo)
		- previously we used $A(s_t, a_t) = r(s_t,a_t)+V^{π_\theta}(s_{t+1})-V^{π_\theta}(s_t)$ but this needs accurate V for temporal difference
		- instead can use $A(s_t, a_t) = \sum\limits_{t'=t}^T r(s_{t'},a_{t'})-V^{π_\theta}(s_t)$ 
		- suppose $π_\beta$ is deterministic, what do we learn? Q=V so A=0, e^0=1 so we get vanilla IL (since we don't see deviation examples, how can we do better)
		- // there's usually a $\alpha$ in $e^{\alpha A}$, this $\alpha$ controls how spiky the distribution is
		- pros: simply, avoids OOD
		- cons: monte carlo estimation is noisy, A is over $π_\beta$ not $π_\theta$ , so can only improve so much over dataset
	- V2 (advantage-weighted actor-critic)
		- can we estimate A using TD updates, without querying Q on OOD actions
		- estimate Q like actor critic, but constrain from a~π to a~D: $r(s,a) + \gamma E_{a'\sim D}[\hat{Q^π_\phi}(s',a')]$ 
		- but still estimates Q over $π_\beta$ and not $π_\theta$ , how can we estimate Q for a policy that's better than $π_\beta$?
	- V3 (implicit Q-learning)
		- V(s) is the average, but there's a distribution of values starting at that state (depending on future trajectory). Can we fit some better percentile instead of the mean?
		- Idea: asymmetric loss function, downweigh loss if >0 (expectile regression)
			- penalized less for overestimating V (note that all seen data is within support)
			- we're estimating V for a policy better than $π_\beta$ so it makes sense that it's higher
		- // Q is lower variance than V

How to mitigate overestimation in offline RL? While still estimating value of the learned policy? (Conservative Q Learning)
- what if we just push down on large OOD Q-values? Conservative when OOD, not as conservative when in support
- $\min\limits_\phi \max\limits_{\mu} L_{critic} + \alpha E_{s \sim D, a \in \mu}\hat{Q}(s,a)$ 
	- optimize critic, then minimize Q for actions that have a high Q-value
	- we can show this lower bounds true Q function
- issue: we push down all Q values, but we have support on in-distribution actions
	- fix: add a term to cancel this out when in-distribution  $-\alpha E_{s \sim D, a \in D}\hat{Q}(s,a)$ 
- alpha controls how pessimistic you are (the lower it is, the more you use regular critic loss)
- how do we think about $\mu$? 
	- we want it to have large Q values, but also want to consider multiple actions 
	- so can add regularizer to Q to consider a broader set of actions (higher entropy)

