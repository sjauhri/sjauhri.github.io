---
title: "Hello World"
date: 2026-09-08
draft: false
tags: ["meta"]
math: false
description: "First post — what this blog is about."
---

Welcome to my blog. I'm Snehal, a postdoctoral researcher at the [3D AI Lab](https://www.3dunderstanding.org/) at TU Munich.

I'll use this space to write about research ideas, paper breakdowns, and things I find interesting in:

- 3D scene understanding and generation
- Reinforcement learning for robotics
- Embodied AI

## What to expect

Posts will vary in depth — some short notes, some long technical deep-dives with math and code.

For example, inline math works: the policy gradient estimator is $\nabla_\theta J(\theta) = \mathbb{E}_\tau [R(\tau) \nabla_\theta \log \pi_\theta(\tau)]$.

And display math:

$$
\mathcal{L}(\theta) = \mathbb{E}_{(s,a) \sim \pi_\theta} \left[ \log \pi_\theta(a|s) \cdot A^\pi(s,a) \right]
$$

Code too:

```python
import torch

def policy_gradient_loss(log_probs, advantages):
    return -(log_probs * advantages).mean()
```

Stay tuned.
