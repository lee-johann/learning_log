https://www.youtube.com/watch?v=i2qSxMVeVLI

Objective
- gaussian p(z) -> decoder $\theta$ -> generation target $p_{data} (x)$ 
	- $\theta^* = argmax \prod\limits_{i=1}^m p_\theta (x^i) \approx argmax \; E_{x\sim p_{data}}[log \; p_\theta (x)] = argmin \; D_{KL}(p_{data} || p_\theta)$ 
	- but to turn p(z) into p(x), we need ground truth latent encoder p(z|x) for $log \; p(x) = log \frac{p(x,z)}{p(z|x)}$ 
- instead estimate encoder q(z|x)
	- then use log p(x) = $E_{q(z|x)} \left[ log\frac{p(x,z)}{q(z|x)} \right] +D_{KL}(q(z|x) || p(z|x))$ 
		- since KL divergence >= 0, use log p(x) >= E
		- p(x) is the evidence, therefore E is called evidence lower bound (ELBO)
- latent variable models
	- in VAE, observed is x, latent is z, we max E_q(z|x)
	- in diffusion, observed is x_0, latent is x_1, ..., x_T, similarly we max E_q(x1:xT | x0)
		- note that x_0...x_T is encoding (adding noise), $\hat{x}_{T-1}$...$\hat{x_0}$ is decoding

- diffusion
	- define $q(x_1:x_T|x_0) = \prod\limits_{t=1}^{T} q(x_t|x_{t-1})$ 
		- where q(x_t | x_t-1) is a gaussian over x_t with mean scaler\*x_t-1 and variance scaler_2 \*I
		- this encoding ensures that after some number of timesteps, the image becomes gaussian noise

- after some math, we can break $E_{q(x_1:x_T|x_0)} \left[ log\frac{p(x_0:x_T)}{q(x_1:x_T|x_0)} \right]$ down into 3 terms:
	- prior matching: $-D_{KL}(q(x_T|x_0) || p(x_T))$ 
		- the latent will be similar to the gaussian at the end of the encoding
	- reconstruction: $E_{q(x_1|x_0)} [ log \; p_\theta (x_0 | x_1)$ 
	- denoising matching: $-\sum\limits_{t=2}^{T} E_{q(x_t|x_0)}[D_{KL}(q(x_{t-1}|x_t,x_0) || p_\theta(x_{t-1}|x_t))]$
		- $q(x_t|x_0)$ is the prob of a noisy distribution at timestep t given a clean image = (with some math) $N(x_t; \sqrt{\hat{a_t}}x_0, (1-\hat{a_t}I))$ where $x_t=\sqrt{\hat{a_t}}x_0, (1-\hat{a_t}\epsilon)$,  $\hat{a_t} = \prod\limits_{i=1}^t a_i$, $\epsilon \sim N(\epsilon; 0, I)$
		- $q(x_{t-1}|x_t,x_0)$ says suppose we know the clean image, and the noisy version of it after t encoder steps, what's the probability of the last noisy image x_t-1
			- this tells us how to denoise x_t when knowing the ground truth x_0
			- after some math, we can show this is also some calculatable gaussian with mean between x_0 and x_t
		- we use this ^ to guide our denoising network $p_\theta(x_{t-1}|x_t)$ 
			- we can show this is some gaussian too
		- since both q and p are gaussian, min KL means minimizing mean between the two

Training
- in the training process, we take clean image and gaussian, encode it x steps x_t, then compare $\mu_\theta(x_t, t)$ and $\mu_q (x_t, x_0)$ with L2 loss
	- interpretation 1: 
		- recall that $\mu_\theta(x_t, t)$ = $a(x_t) + b(\hat{x}_0(x_t,t))$ and $\mu_q (x_t, x_0)$ = $a(x_t) + b(x_0)$ 
		- we already know x_t, so this loss minimizes $\hat{x}_\theta(x_t, t) - x_0$
		- therefore, we are asking the model $\theta$ to predict the clean image x_0 given the noisy one x_t
	- interpretation 2:
		- recall that  $\mu_\theta(x_t, t)$ = $a_2(x_t) + b_2(\epsilon)$ and $\mu_q (x_t, x_0)$ = $a_2(x_t) + b_2(\hat{\epsilon}_\theta(x_t, t))$ 
		- similarly, we know x_t, so this loss minimizes $\hat{\epsilon}_\theta(x_t,t) - \epsilon$
		- therefore, this loss asks the denoising network to predict the noise (and not the ground truth)
	- interpretation 3:
		- recall Tweedie's formula $E[\mu_z|z] = z+ \sum_z \nabla_z log \; p(z)$ 
		- similarly, we get that the loss minimizes $\nabla log \; p(x_t) - s_\theta (x_t, t)$ 
			- Why can the log prob increase? since $\nabla_{x_t} log \; p(x_t) = -1 (a_3 \epsilon)$ , and the noisy image x_t comes from adding noise to the clean image x_0, -1 means we move in the opposite direction of the noise (i.e. denoise), so it makes sense we increase the log prob
	- we take many small steps to get more accurate estimates
		- these steps trace a path (not necessarily straight) from noise back to the sample

guidance in the generation process
- unconditioned: predict local likelihood gradient $\nabla log \; p(x_t)$
- conditioned: predict conditional score $\nabla log \; p(x_t|y) = \nabla log \; p(x_t) + \nabla \; log(y|x_t)$
	- conditional = unconditional + adversarial gradient of a classifier
	- use normal image to text classifier for p(y|x_t)
		- since classifiers usually trained on cleaned images x_0, we use estimated x_0 (using x_t and estimated noise $\hat{\epsilon}$) (bit blurry, but classifiers usually robust to blur)
	- alternatively, you can turn p(y|x_t) into p(x_t|y) + p(x_t), to get rid of the classifier (classifier-free guidance)

resolution: how to generate high res images?
- cascade diffusion: text-to-image diffusion model -> upscale model
- latent diffusion: VAE naturally compresses image then upscales, we pass diffusion model -> decoder of VAE to upscale
- end-to-end diffusion: active research area

speed
- diffusion slow b/c denoising has many steps
- DDIM (2021) paper finds deterministic representation of equations
- progressive distillation reduces the number of steps: teacher is 1 step diffusion n-> n+1, student predicts teacher's n+2 steps
- guided distrillation unifying models (Meng et al 2023)
- consistency model (song et al 2023)
- aside: can use LoRA on diffusion models
