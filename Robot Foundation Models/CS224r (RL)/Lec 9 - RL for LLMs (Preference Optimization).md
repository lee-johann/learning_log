Archit Sharma guest lecture (DPO 2nd author)

Why does pretraining teach?
- Knowledge, syntax, coreference, topic, sentiment, some reasoning (from predict next token)
- some notion of modeling what actions each type of human does (LM as Agent Models https://arxiv.org/abs/2212.01681)

How to turn LLMs into chatbots: instruction fine-tuning, preference optimization
- next token prediction != task completion (instructGPT https://arxiv.org/abs/2203.02155)
- instruction finetuning
	- found that generalization capability scales with model size
	- limitation: 
		- expensive to collect instruction, output pairs
		- some tasks don't have right answers ex. creative writing
		- all tokens penalized equally, but some errors are worse than others (ex. avatar is a adventure/fantasy TV show <– here advanture/fantasy doesn't rly matter, but TV vs book really matters)
		- human answers aren't optimal
	- // for differentiating between formatting and knowledge, you can train it on the wrong answers and see the impact on eval

RL from human preferences
- (still InstructGPT paper) RLHF: 
1. finetune as usual
2. collect rankings of responses and train reward model
3. optimize policy against reward model with RL (PPO)
- // mentions modeling reward helpful b/c sometimes reward itself is not differentiable, but our model is
- use reward model instead of humans for the RL step b/c human label expensive
- human scoring not consistent, so instead of raw score use preference
	- // i wonder if there's a more data-efficient/human-time-efficient in between representation
	- // also do we assume transitivity? 
		- he said no solution to this, human prefs are noisy, in fact preference prediction is usually 60-70% accuracy but somehow that improves task performance by a lot
- tokens are discrete, so we need RL b/c discrete tokens can't rly differentiate gradients through
	- if everything is continuous we don't need RL, can just backprop gradients all the way through
- but we want the reward model to grade outputs similar to support (for accurate reward)
	- add KL for diverging too far from pretrained model
	- earlier RLHF paper https://arxiv.org/abs/2009.01325
- RLHF can be complex and expensive (Bytedance paper explaining PPO's effect on the model https://arxiv.org/abs/2307.04964)

Direct Preference Optimization: simplifying RLHF
- can we use preference directly instead of using intermediate reward model
- objective we want to maximize is $E_{\hat{y} \sim p_\theta^{RL} (\hat{y}|x)}[RM(x,\hat{y})-\beta \; log \left( \frac{p_\theta^{RL}(\hat{y}|x)}{p_\theta^{PT}(\hat{y}|x)} \right)]$ where PT is pretrain, RM is reward model, x is instruction, y^ is completion
	- the closed form optimal solution implies $RM(x, \hat{y}) = \beta log \frac{p^*(\hat{y}|x)}{P^{PT}(\hat{y}|x)} + \beta log Z(x)$ where Z(x) is just a normalization term
	- plug this back into bradly terry reward preference model E[log sigmoid(RM(x, y_win) - RM(x, y_lose))] = $-E_{x, y_w, y_l \sim D} [log \; \sigma(\beta log \frac{p_\theta^{RL}(y^w|x)}{p_\theta^{PT}(y^w|x)}) - \beta log \frac{p_\theta^{RL}(y^l|x)}{p_\theta^{PT}(y^l|x)}))]$ to tune LM params
	- beta controls how fast the difference saturates the sigmoid (recall that sigmoid slope flattens to 0 around the two edges); can also view as controls how much you penalize
- // we lose the ability to grade data that isn't in the generation
- note that still no strength of preference (mitigation is that you give strongly prefer, weak prefer, weak not prefer, strong not prefer) and then only use the strong preferences
- ppl usually collect ranking and then turn into pairs, or train directly on ranked using a more advanced formulation of bradly terry
- also note that this is offline RL
- can learn latent human preferences like explanations etc.

limitations of RL + reward modelings
- human prefs unreliable
- reward hacking: as you RLHF, RM prediction is that answers get preferred more and more, true human preference is that it gets worse after some point
	- if you have to stop training at some point, that prevents scaling
	- now more moving towards rubrics
- chatbots rewarded for responses that seem authoritative and helpful instead of truthful https://openai.com/index/sycophancy-in-gpt-4o/

Open topics:
- RL expensive: 
	- RL AIF https://arxiv.org/abs/2212.08073
	- Fintune on own outputs 
		- self-taught reasoner https://arxiv.org/abs/2203.14465
		- LLM can self improve https://arxiv.org/abs/2210.11610
- RLHF happens at aggregate level, what if we personalized
	- Chelsea few-shot preference optim paper https://arxiv.org/abs/2502.19312
	- dataset of different groups' preferences https://arxiv.org/abs/2404.16019
- 