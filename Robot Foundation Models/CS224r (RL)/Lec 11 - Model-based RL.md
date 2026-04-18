High level recap of algorithms in this class so far:
![[lec11_method_recap.png]]

- model based RL can be applied to all previous methods

outline:
- key idea: TODO
- learning a dynamics model
- Using a dynamics model for planning & data generation
- case study in dexterous robotic manipulation

Key idea: can we learn a "simulator": predict $s_{t+1} | s_t, a_t$ 
- either model the physics (if you can observe the physical entities), or video prediction conditioned on actions
- learn $f(s,a)=s'$ from $D={s_i,a_i,s_i'}$ then run RL method inside "simulator" $f$
- what might go wrong: data coverage / out-of-distribution actions, simulator might not be accurate (can get exploited)

Learning a dynamics model
- options
	- model e2e
	- model in lower dimensional space $z_t=encoder(s_t)$, predict $z_{t+1}$ instead of $s_{t+1}$ if computational cost too high to model whole video (maybe some parts of obs doesn't matter ex. color of road)
	
using the model: if you can predict reward of future state, 
	- approach 1 backprop from rewards into actions: $\max\limits_{a_{t:t+h}} \sum\limits_{t'=t}^{t+H}{r_{t'}}$ //24:00-26:30 don't understand
	- approach 2: sampling based
	- 