---
title: "Sim2Real for Robot Manipulation is finally scaling"
date: 2026-09-10
draft: false
ShowToc: false
description: 'Years ago, robot quadrupeds got their "randomize" button which kickstarted sim2real. Now, robot manipulation finally has one too.'
summary: 'Years ago, robot quadrupeds got their "randomize" button which kickstarted sim2real. Now, robot manipulation finally has one too.'
cover:
  image: "/images/molmobot-sim2real.gif"
  alt: "Manipulation policies trained in randomized simulation, running on real robots"
  relative: false
---

*Paper: [MolmoB0T: Large-Scale Simulation Enables Zero-Shot Manipulation](https://arxiv.org/abs/2603.16861) · [Project page](https://allenai.github.io/MolmoBot/)*

When I started my PhD in 2021, the state of robot learning was roughly this: we had beautiful simulations, we had policies that did impressive things inside them, and we had very little that worked on an actual robot.

Transfer to real hardware was sparse. You could get it to work if you constrained the action space enough, or leaned on a bag of tricks, and even then success rates were low. Plenty of serious people doubted whether simulation policies would ever transfer at all.

## Then robot locomotion got a button

The breakthrough came from legged locomotion, and from quadrupeds in particular.

Marco Hutter's group at ETH Zurich, the group behind ANYmal, worked out large-scale simulation and sim2real on GPUs in Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning <a class="cite" href="#ref-1">[1]</a>. That paper blew sim2real for locomotion **wide open**.

Two things came out of it. The first was making RL fast: thousands of simulations running in parallel, all gathering experience at once. That part got the attention.

<figure>
  <video src="/images/quadruped-parallel.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="Thousands of simulated ANYmal quadrupeds learning to walk in parallel across stairs and rough terrain."></video>
  <figcaption>
    <p>Thousands of ANYmal robots learning to walk in parallel on a single GPU, each on
    different terrain <a class="cite" href="#ref-1">[1]</a>.</p>
  </figcaption>
</figure>

The second is the one I keep coming back to. Once you have a policy that walks passably in simulation, you press a button called **randomize**. Now you are training across many simulations that differ slightly in terrain, robot dynamics, friction, mass and actuator response. This is domain randomization <a class="cite" href="#ref-2">[2]</a>, applied to physics and at scale. Instead of learning to walk in one simulator, the robot learns to walk across a whole range of plausible ones.

What comes out transfers *directly*. You take the policy trained with the button pressed and you put it on the robot the next day.

It might seem obvious now, but this was a big breakthrough in 2021. The key idea was not a new architecture or objective. It was treating the sim-to-real gap as a range of conditions to train across, rather than an error to engineer away. Once real-world physics is inside the training distribution, real-world performance follows.

The recipe also **scaled**. Anyone could randomize their simulation, generate more data and get better transfer to the real robot, without needing a new insight for every robot, task or lab.

## Why manipulation didn't get one

Since then, researchers have been trying to bring the same capability to manipulation. For a long time, it mostly did not work.

Some manipulation works from depth alone, grasping being the obvious case (including some of my own PhD work <a class="cite" href="#ref-3">[3]</a>). Depth is a fairly narrow input and fairly easy to simulate. The real goal, though, is visual policies that map pixels directly to joint positions or torques, which is the most likely route to solving manipulation.

Vision is much harder to randomize. You need more randomization, and the space you have to cover is far larger: a quadruped's proprioception lives in a small, well-understood space, while a camera in somebody's kitchen can see almost anything.

So visual policies mostly took a different approach: simulation as a rough starting point, with heavy reliance on teleoperation. That has led to great progress in imitation learning from real-world data, but teleoperation data is expensive. It is paid for in human hours, and human hours do not have a randomize button.

Meanwhile, simulation for manipulation was left behind, widely seen as something that does not really work or does not pay dividends.

## Manipulation gets its button

During my year-long research scientist internship at the Allen Institute for AI, I wanted to work on exactly this. Ai2 had already seen visual sim2real work for household *navigation*, and the obvious question was whether the recipe extended to manipulation and mobile manipulation. Having done my PhD on robot learning for homes, 2D and 3D vision policies and mobile manipulation, I jumped at the opportunity.

With [MolmoB0T](https://allenai.github.io/MolmoBot/) <a class="cite" href="#ref-4">[4]</a> ([Ai2's announcement](https://allenai.org/blog/molmobot-robot-manipulation)), I think the answer is yes.

It builds on [MolmoSpaces](https://github.com/allenai/molmospaces) <a class="cite" href="#ref-5">[5]</a>: 10K to 100K procedurally generated households built from Objaverse assets, with human-authored doors and articulated objects. On top of that, we use fairly standard data generators that rely on ground-truth simulation information such as object poses, collision bodies and bounding boxes. They solve pick-and-place, door opening and articulated-object opening with IK controllers, collision-aware IK and cuRobo motion planning.

None of these ingredients is exotic. Combined, they give manipulation its randomize button: data generated at scale across textures, lighting, object placement and dynamics, with enough diversity to put real-world observations in distribution.

<figure>
  <img src="/images/molmospaces.gif" loading="lazy"
       alt="Procedurally generated household environments from MolmoSpaces.">
  <figcaption>
    <p>Procedurally generated households in MolmoSpaces, the environments the
    randomization runs over <a class="cite" href="#ref-5">[5]</a>.</p>
  </figcaption>
</figure>

<figure>
  <video src="/images/molmobot-data.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="Simulated robots performing manipulation tasks across many procedurally generated households, zooming out to thousands of episodes."></video>
  <figcaption>
    <p>MolmoBot data generation: the same tasks generated across thousands of
    randomized households <a class="cite" href="#ref-4">[4]</a>.</p>
  </figcaption>
</figure>

For a sense of scale: MolmoBot-Data is 1.8 million trajectories across 94,000 generated houses and 11,000 objects, roughly 5,800 robot-hours of experience. DROID <a class="cite" href="#ref-6">[6]</a>, one of the largest real-world manipulation datasets, is 76,000 trajectories from 350 hours of teleoperation.

## What actually changes

The policies transfer directly to real robots. In some of the demos on the [project page](https://allenai.github.io/MolmoBot/), the desk the robot is mounted on is raised and lowered *mid-rollout*, and the policy keeps going. It was trained on data with randomized robot heights, so this is just another variation.

For me, the key point is not a success rate but a change in how you work.

Until now, if you designed a simulation policy for manipulation, you eventually had to put the simulator aside and start over on the real robot: rigging teleoperation, collecting demonstrations, paying in human hours.

Now you press randomize. Overnight you get scaled-up, domain-randomized data for your task. You train. You come back in the morning with a policy that runs on your real manipulator or mobile manipulator.

This changes what a manipulation simulator is for. It is no longer just a demo environment but somewhere you can train deployable policies, including for whole-body tasks like opening doors and cupboards, which are some of the least pleasant things to collect real data for.

It is an exciting time to be working on physical AI, and simulation is definitely back. Shoutout to concurrent work from RAI, AnyTask <a class="cite" href="#ref-7">[7]</a>, which reaches a similar conclusion from a different direction.

<figure>
  <video src="/images/molmobot-table-height.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="A robot arm putting blue tape in a wooden box while the desk it is mounted on is raised and lowered."></video>
  <figcaption>
    <p>"Put the blue tape in the wooden box" while the desk moves up and down <a class="cite" href="#ref-4">[4]</a>.</p>
  </figcaption>
</figure>

<figure>
  <video src="/images/molmobot-camera-demo.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="A real robot arm putting a banana in a brown bowl, shown from a third-person camera and the policy's own exo and wrist cameras."></video>
  <figcaption>
    <p>"Put the banana in the brown bowl." When the brown bowl is out of view the
    policy heads for the black one, then changes course once it sees the right
    bowl <a class="cite" href="#ref-4">[4]</a>.</p>
  </figcaption>
</figure>

## So is sim2real solved?

Not entirely.

For rigid-body manipulation and mobile manipulation, partly yes. We now have a recipe that gets to around 80% success, which can then be fine-tuned with real data. That is a very different starting point from zero.

{{< figure-svg src="zero-shot-results.svg"
    caption="Zero-shot pick-and-place on a Franka FR3: 120 real-world trials, no real-world fine-tuning. MolmoBot-Pi0 keeps the π0 <a class='cite' href='#ref-8'>[8]</a> architecture and swaps in simulation data, so much of the gain comes from the data. The baseline is π0.5 <a class='cite' href='#ref-9'>[9]</a> trained on DROID <a class='cite' href='#ref-6'>[6]</a>. Results from <a class='cite' href='#ref-4'>[4]</a>." >}}

For deformable and soft objects, and for more complex objects in general, it remains to be seen.

What excites me now is how far this recipe can scale. The question is no longer whether policies transfer at all, but how many axes of diversity we can scale up. MolmoSpaces is a great household simulator, and there is still a lot of headroom in it.

Stay tuned for more on this!

---

MolmoBot was accepted as an Oral at the Conference on Robot Learning (CoRL) 2026! You can read the paper [here](https://arxiv.org/abs/2603.16861), and see more on the [project page](https://allenai.github.io/MolmoBot).

<section class="references" aria-labelledby="references">
<h2 id="references">References</h2>
<ol>
  <li id="ref-1">N. Rudin, D. Hoeller, P. Reist, M. Hutter. <a href="https://arxiv.org/abs/2109.11978">Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning</a>. Conference on Robot Learning (CoRL) 2021.</li>
  <li id="ref-2">J. Tobin, R. Fong, A. Ray, J. Schneider, W. Zaremba, P. Abbeel. <a href="https://arxiv.org/abs/1703.06907">Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World</a>. IROS 2017.</li>
  <li id="ref-3">S. Jauhri, I. Lunawat, G. Chalvatzaki. <a href="https://arxiv.org/abs/2306.07392">NeuGraspNet: Learning Any-View 6DoF Robotic Grasping in Cluttered Scenes via Neural Surface Rendering</a>. RSS 2024.</li>
  <li id="ref-4">S. Jauhri*, A. Deshpande*, M. Guru*, R. Hendrix*, et al. <a href="https://arxiv.org/abs/2603.16861">MolmoB0T: Large-Scale Simulation Enables Zero-Shot Manipulation</a>. Conference on Robot Learning (CoRL) 2026.</li>
  <li id="ref-5">Y. Kim, W. Pumacay, O. Rayyan, et al. <a href="https://arxiv.org/abs/2602.11337">MolmoSpaces: Large-Scale Open Ecosystem for Robot Manipulation and Navigation</a>. RSS 2026.</li>
  <li id="ref-6">A. Khazatsky et al. <a href="https://arxiv.org/abs/2403.12945">DROID: A Large-Scale In-The-Wild Robot Manipulation Dataset</a>. RSS 2024.</li>
  <li id="ref-7">R. Gong, X. Zhang, J. Shang, et al. <a href="https://arxiv.org/abs/2512.17853">AnyTask: An Automated Task and Data Generation Framework for Advancing Sim-to-Real Policy Learning</a>. arXiv 2025.</li>
  <li id="ref-8">K. Black et al. <a href="https://arxiv.org/abs/2410.24164">π0: A Vision-Language-Action Flow Model for General Robot Control</a>. arXiv 2024.</li>
  <li id="ref-9">Physical Intelligence. <a href="https://arxiv.org/abs/2504.16054">π0.5: a Vision-Language-Action Model with Open-World Generalization</a>. arXiv 2025.</li>
</ol>
</section>
