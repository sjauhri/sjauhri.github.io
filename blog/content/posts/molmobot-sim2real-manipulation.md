---
title: "Sim2Real for Robot Manipulation is finally scaling"
date: 2026-09-10
draft: false
ShowToc: false
description: 'Years ago, robot quadrupeds got their "randomize & learn" button which kickstarted sim2real. Now, robot manipulation finally has one too.'
summary: 'Years ago, robot quadrupeds got their "randomize & learn" button which kickstarted sim2real. Now, robot manipulation finally has one too.'
cover:
  image: "/images/molmobot-sim2real.gif"
  alt: "Manipulation policies trained in randomized simulation, running on real robots"
  relative: false
---

*Paper: [MolmoB0T: Large-Scale Simulation Enables Zero-Shot Manipulation](https://arxiv.org/abs/2603.16861) · [Project page](https://allenai.github.io/MolmoBot/)*

When I started my PhD in 2021, the state of embodied AI was roughly this: we had beautiful simulations, we had robot policies that could learn impressive behaviors in sim, and we had very little that worked on an actual robot.

Transfer from simulation to real hardware was sparse. You could get it to work if you constrained the action space enough, or learned on top of some pre-existing robot skills, and even then success rates were low. Plenty of serious people doubted whether simulation policies would ever transfer at all.

## Robot locomotion and the "randomize & learn" button

The breakthrough came from robot locomotion, specifically quadrupeds <a class="cite" href="#ref-1">[1]</a>. Researchers trained quadrupeds to walk in simulation, but randomized simulation rollouts such that each quadruped was trained in a slightly different world: different terrain (slopes, stairs, rough ground, getting harder as it improves), ground friction, random shoves and noisy sensors. One policy then learns to walk in all of them at once. Crucially, this meant that they could now transfer and run the same policy on the real robot with **no further training**. That recipe blew sim2real for locomotion **wide open**.

<!-- This was domain randomization <a class="cite" href="#ref-3">[3]</a>, applied to physics and at scale. Instead of learning to walk in one simulator, the robot learns to walk across a whole range of plausible ones. -->

<!-- Two things came out of it. The first was making RL data generation fast and scalable: thousands of simulations running in parallel, all gathering experience at once. -->

<figure>
  <video src="/images/quadruped-parallel.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="ANYmal robots learning to walk in large-scale physics simulations."></video>
  <figcaption>
    <p>ANYmal robots learning to walk in large-scale physics simulations<a class="cite" href="#ref-1">[1]</a>.</p>
  </figcaption>
</figure>

<!-- The second is the one I keep coming back to: the "randomize and learn" button. Every simulated robot gets a slightly different world: its own terrain (slopes, stairs, rough ground, getting harder as it improves), its own ground friction, random shoves and noisy sensors. One policy then learns to walk in all of them at once. This is domain randomization <a class="cite" href="#ref-3">[3]</a>, applied to physics and at scale. Instead of learning to walk in one simulator, the robot learns to walk across a whole range of plausible ones. -->

The recipe transferred to several locomotion problems and meant that, as a researcher, you could train a quadruped policy in simulatio, press a "randomize" button, see a large number of simulations roll out with slightly different physics, terrain settings, and then see a robust policy learned and working on your **real** robot the next day.
<!-- What comes out transfers *directly*. You take the policy trained with the button pressed and you put it on the robot the next day. -->

<!-- It might seem obvious now, but this was a big breakthrough in 2021. The key idea was not a new architecture or objective. It was treating the sim-to-real gap as a range of conditions to train across, rather than an error to engineer away. Once real-world physics is inside the training distribution, real-world performance follows. -->

<!-- The recipe also **scaled**. Anyone could randomize their simulation, generate more data and get better transfer to the real robot, without needing a new insight for every robot, task or lab. -->

## But Robot Manipulation needs Vision

Since then, researchers have been trying to bring the same capability to manipulation. For a long time, it mostly did not work.

Some manipulation works such as grasping can work from depth alone <a class="cite" href="#ref-2">[2]</a>, but depth is a fairly narrow input and fairly easy to simulate. The real goal, though, is visual policies that map pixels directly to joint positions or torques, which is the most likely route to solving manipulation.

Vision is much harder to randomize <a class="cite" href="#ref-3">[3]</a>. You need more randomization, and the space you have to cover is far larger: a quadruped's proprioception lives in a small, well-understood space, while a camera in somebody's kitchen can see almost anything.

So visual policies mostly took a different approach: learning from demonstrations with heavy reliance on teleoperation. That has led to great progress in imitation learning from real-world data <a class="cite" href="#ref-4">[4]</a>, but teleoperation data is expensive. It is paid for in human hours, and human hours do not have a randomize button.

Meanwhile, simulation for manipulation was left behind, widely seen as something that does not really work or does not pay dividends.

## Manipulation gets its button

During my year-long research scientist internship at the Allen Institute for AI, I wanted to work on exactly this but for mobile manipulation. Ai2 had already seen visual sim2real work for household *navigation*, and the obvious question was whether the recipe extended to manipulation and mobile manipulation.

[MolmoB0T](https://allenai.github.io/MolmoBot/) <a class="cite" href="#ref-5">[5]</a> is our attempt at answering that.

It builds on [MolmoSpaces](https://github.com/allenai/molmospaces) <a class="cite" href="#ref-6">[6]</a>: 10K to 100K procedurally generated households built from Objaverse assets, with human-authored doors and articulated objects. On top of that, we use fairly standard data generators that rely on ground-truth simulation information such as object poses, collision bodies and bounding boxes. They solve pick-and-place, door opening and articulated-object opening with IK controllers, collision-aware IK and cuRobo motion planning.

None of these ingredients is new. What we changed is the scale and the amount of variation: data generated across textures, lighting, object placement and dynamics, with the aim of making real-world observations look like just another sample from the training distribution.

<figure>
  <img src="/images/molmospaces.gif" loading="lazy"
       alt="Procedurally generated household environments from MolmoSpaces.">
  <figcaption>
    <p>Procedurally generated households in MolmoSpaces, the environments the
    randomization runs over <a class="cite" href="#ref-6">[6]</a>.</p>
  </figcaption>
</figure>

<figure>
  <video src="/images/molmobot-data.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="Simulated robots performing manipulation tasks across many procedurally generated households, zooming out to thousands of episodes."></video>
  <figcaption>
    <p>MolmoBot data generation: the same tasks generated across thousands of
    randomized households <a class="cite" href="#ref-5">[5]</a>.</p>
  </figcaption>
</figure>

For a sense of scale: MolmoBot-Data is 1.8 million trajectories across 94,000 generated houses and 11,000 objects, roughly 5,800 robot-hours of experience. For comparison, the real-world DROID dataset <a class="cite" href="#ref-7">[7]</a> is 76,000 trajectories from 350 hours of teleoperation.

## What actually changes

The policies run on real robots without any real-world fine-tuning. In one of the demos, the desk the robot is mounted on is raised and lowered *mid-rollout*, and the policy keeps going. The training data had randomized robot heights, so this is a variation it has already seen.

The practical difference is in the workflow. A manipulation policy developed in simulation usually had to be redone on the real robot with teleoperated demonstrations.

With enough randomization, you can instead generate data for a task in simulation, train on it and run the result on a real arm or mobile manipulator.

This helps most for whole-body tasks like opening doors and cupboards, which are tedious to teleoperate.

<figure>
  <video src="/images/molmobot-table-height.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="A robot arm putting blue tape in a wooden box while the desk it is mounted on is raised and lowered."></video>
  <figcaption>
    <p>"Put the blue tape in the wooden box" while the desk moves up and down <a class="cite" href="#ref-5">[5]</a>.</p>
  </figcaption>
</figure>

<figure>
  <video src="/images/molmobot-camera-demo.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="A real robot arm putting a banana in a brown bowl, shown from a third-person camera and the policy's own exo and wrist cameras."></video>
  <figcaption>
    <p>"Put the banana in the brown bowl." When the brown bowl is out of view the
    policy heads for the black one, then changes course once it sees the right
    bowl <a class="cite" href="#ref-5">[5]</a>.</p>
  </figcaption>
</figure>

## So is sim2real solved?

Not entirely.

For rigid-body manipulation and mobile manipulation, partly. In our pick-and-place tests the policy reaches about 80% success zero-shot, which is a useful starting point for fine-tuning with real data.

{{< figure-svg src="zero-shot-results.svg"
    caption="Zero-shot pick-and-place on a Franka FR3: 120 real-world trials, no real-world fine-tuning. MolmoBot-Pi0 keeps the π0 <a class='cite' href='#ref-8'>[8]</a> architecture and swaps in simulation data, so much of the gain comes from the data. The baseline is π0.5 <a class='cite' href='#ref-9'>[9]</a> trained on DROID <a class='cite' href='#ref-7'>[7]</a>. Results from <a class='cite' href='#ref-5'>[5]</a>." >}}

For deformable and soft objects, and for more complex objects in general, it remains to be seen.

The open question for me is how far this scales: which kinds of variation matter most, and how many more we can add. There is still a lot left to vary in MolmoSpaces.

## Other recent progress

MolmoBot is not the only recent result pointing this way. AnyTask <a class="cite" href="#ref-10">[10]</a> from RAI automates task and data generation for sim-to-real policy learning, and SimVLA <a class="cite" href="#ref-11">[11]</a> trains a mobile-manipulation policy entirely in procedurally generated houses and runs it zero-shot in real homes.

Another route is large-scale reinforcement learning in simulation, distilled into a policy that sees only RGB. Zhang et al. <a class="cite" href="#ref-12">[12]</a> from the University of Washington scale RL to around a million parallel environments and transfer zero-shot to real assembly tasks such as gear insertion. DoorMan <a class="cite" href="#ref-13">[13]</a> from NVIDIA uses a similar recipe to teach a humanoid to open real doors.

<figure>
  <img src="/images/sim2real-balanced-data-diet.jpg" loading="lazy"
       alt="A robot gripper inserting a gear in simulation, next to a real UR5e robot doing the same.">
  <figcaption>
    <p>Gear insertion in simulation (left) and zero-shot on a real UR5e (right) <a class="cite" href="#ref-12">[12]</a>.</p>
  </figcaption>
</figure>

<figure>
  <img src="/images/sim2real-doorman.jpg" loading="lazy"
       alt="A simulated first-person camera view of a humanoid hand at a door handle, next to a real humanoid robot opening a door.">
  <figcaption>
    <p>DoorMan's simulated RGB view (left) and the real humanoid opening a door (right) <a class="cite" href="#ref-13">[13]</a>.</p>
  </figcaption>
</figure>

It is an exciting time for sim2real in manipulation. Stay tuned for more on this!

---

MolmoBot was accepted as an Oral (top 3% of papers) at the Conference on Robot Learning (CoRL) 2026! You can read the paper [here](https://arxiv.org/abs/2603.16861), and see more on the [project page](https://allenai.github.io/MolmoBot).

<section class="references" aria-labelledby="references">
<h2 id="references">References</h2>
<ol>
  <li id="ref-1">N. Rudin, D. Hoeller, P. Reist, M. Hutter. <a href="https://arxiv.org/abs/2109.11978">Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning</a>. Conference on Robot Learning (CoRL) 2021.</li>
  <li id="ref-2">S. Jauhri, I. Lunawat, G. Chalvatzaki. <a href="https://arxiv.org/abs/2306.07392">NeuGraspNet: Learning Any-View 6DoF Robotic Grasping in Cluttered Scenes via Neural Surface Rendering</a>. RSS 2024.</li>
  <li id="ref-3">J. Tobin, R. Fong, A. Ray, J. Schneider, W. Zaremba, P. Abbeel. <a href="https://arxiv.org/abs/1703.06907">Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World</a>. IROS 2017.</li>
  <li id="ref-4">C. Chi, S. Feng, Y. Du, Z. Xu, E. Cousineau, B. Burchfiel, S. Song. <a href="https://arxiv.org/abs/2303.04137">Diffusion Policy: Visuomotor Policy Learning via Action Diffusion</a>. RSS 2023.</li>
  <li id="ref-5">S. Jauhri*, A. Deshpande*, M. Guru*, R. Hendrix*, et al. <a href="https://arxiv.org/abs/2603.16861">MolmoB0T: Large-Scale Simulation Enables Zero-Shot Manipulation</a>. Conference on Robot Learning (CoRL) 2026.</li>
  <li id="ref-6">Y. Kim, W. Pumacay, O. Rayyan, et al. <a href="https://arxiv.org/abs/2602.11337">MolmoSpaces: Large-Scale Open Ecosystem for Robot Manipulation and Navigation</a>. RSS 2026.</li>
  <li id="ref-7">A. Khazatsky et al. <a href="https://arxiv.org/abs/2403.12945">DROID: A Large-Scale In-The-Wild Robot Manipulation Dataset</a>. RSS 2024.</li>
  <li id="ref-8">K. Black et al. <a href="https://arxiv.org/abs/2410.24164">π0: A Vision-Language-Action Flow Model for General Robot Control</a>. arXiv 2024.</li>
  <li id="ref-9">Physical Intelligence. <a href="https://arxiv.org/abs/2504.16054">π0.5: a Vision-Language-Action Model with Open-World Generalization</a>. arXiv 2025.</li>
  <li id="ref-10">R. Gong, X. Zhang, J. Shang, et al. <a href="https://arxiv.org/abs/2512.17853">AnyTask: An Automated Task and Data Generation Framework for Advancing Sim-to-Real Policy Learning</a>. arXiv 2025.</li>
  <li id="ref-11">K. Baik, Y. Lee. <a href="https://arxiv.org/abs/2610.11248">SimVLA: Zero-Shot Sim-to-Real VLA Learning for Mobile Manipulation</a>. arXiv 2026.</li>
  <li id="ref-12">O. Zhang, M. Guaman Castro, P. Yin, I. Dagnino, A. Gupta, R. Scalise, B. Boots. <a href="https://arxiv.org/abs/2610.12465">A Balanced Data Diet: Addressing the Exploration Bottleneck in Mega-Scale RL for Robot Control</a>. Conference on Robot Learning (CoRL) 2026.</li>
  <li id="ref-13">H. Xue, T. He, Z. Wang, Q. Ben, W. Xiao, et al. <a href="https://arxiv.org/abs/2512.01061">Opening the Sim-to-Real Door for Humanoid Pixel-to-Action Policy Transfer</a>. arXiv 2025.</li>
</ol>
</section>
