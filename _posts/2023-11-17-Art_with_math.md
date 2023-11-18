---
title: Art with Math
author: gozepolat
date: 2022-11-17 05:30:00 -0500
math: true
categories: [Artificial Intelligence, Deep Learning]
tags: [research, deep learning, phd]
---

## A Droplet in the Ocean

Yunus Emre says "I am the drop that contains the ocean".  It may be the case that a limited set of rules is responsible for everything around us. They may apply in various scales over and over again.  

{% raw %}
<video autoplay loop muted playsinline>
  <source src="/assets/img/fractal.webm" type="video/webm">
</video>
{% endraw %}

I like imagining Yunus Emre's ocean as a fractal. I came up with this fractal using the below recurrence relation:

![fractal_eqn.png]({{site.baseurl}}/assets/img/fractal_eqn.png)

## What is Perona-Malik Diffusion?

Perona-Malik diffusion is a well known regularization technique that preserves the edges while smoothing down the texture.
It is simply defined as:

$$ I_t = C(x,y,t) \nabla ^2I + \nabla C . \nabla I $$

where $$ C $$ is the diffusion coefficient, and $$ I $$ is the diffusing image. This formula comes from the energy functional:

$$ E[I] = \frac{1}{2} \int_{\Omega} g\left( \| \nabla I(x)\|^2 \right)\, dx $$

where $g$ is a monotonically decreasing, edge seeking function such as:

$$ g\left(\|\nabla I\|\right) = e^{-\left(\|\nabla I\| / K\right)^2} $$.

During the minimization, we normally use a constant kernel to calculate the gradients (i.e. the Laplace operator for $$ \nabla ^2 $$) at each step.

## Perona-Malik with PyTorch

The amount of flexibility and ease of use makes PyTorch
a great choice for both research and prototyping. Moreover, it is quite easy to use PyTorch for things other than deep learning where you still need GPU acceleration; hence my experiments on Perona-Malik!

I started implementing a vanilla version of Perona-Malik using PyTorch, where I initialized a convolutional layer as my Laplace operator. The convolutional kernel is initialized as below:

```python
[0.0,  1.0,  0.0]
[1.0, -4.0,  1.0]
[0.0,  1.0,  0.0]
```

## Minimization Art

Thanks to the utilization of GPU, the speed I got from such a simple implementation was pleasant.
Yet, using PyTorch for nothing but GPU acceleration seemed like a waste... Let's change the edge seeking property of $$ g $$,
with something to be learned from the loss function using gradient descent. Depending on what we want from the transformations, it is possible to define various loss functions. Here is what I picked to generate all kinds of cool reaction-diffusion transformations:

$$ E[I, K] = \int (I_t(x) - I_0(x)) ^2 + \|K_t * I_{t-1}(x)\| + I_t(x) ^2 dx $$

where $$ I $$ is the image and $$ K $$ is the custom kernel that is being learned. ($I_t$ is the diffusing image, and $$ I_0 $$ is the original image.) This loss function has the fidelity component $$ I_t - I_0 $$ to keep the transformation closer to the original image,
it penalizes the case where edges disappear by learning a $$ K $$ that quickly increases intensity. Using this loss function,
and after finding the right diffusion rate vs learning rate, I observed aesthetically pleasing diffusion reaction transformations
which were able to temporarily preserve the edges to some extent:

{% raw %}
<video autoplay loop muted playsinline>
  <source src="/assets/img/profile.webm" type="video/webm">
</video>
{% endraw %}

### Samples

![stars.jpg]({{site.baseurl}}/assets/img/stars.jpg)

Feel free to check [this repo](https://github.com/gozepolat/minimization_art) for more details. Enjoy!
