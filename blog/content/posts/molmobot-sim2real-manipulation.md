---
title: "Sim2Real for Robot Manipulation Is Finally Scaling"
date: 2026-10-20
draft: false
ShowToc: false
description: 'Years ago, robot quadrupeds got their "randomize & learn" button which kickstarted sim2real. Now, robot manipulation finally has one too.'
summary: 'Years ago, robot quadrupeds got their "randomize & learn" button which kickstarted sim2real. Now, robot manipulation finally has one too.'
# Share-card image for X/LinkedIn: a 1200x630 still, since the cover GIF is over their 5 MB limit.
socialImage: "/images/molmobot-sim2real-card.jpg"
cover:
  image: "/images/molmobot-sim2real.gif"
  alt: "Manipulation policies trained in randomized simulation, running on real robots"
  relative: false
---

When I started my PhD in 2021, the state of embodied AI was roughly this: we had cool simulation demos, we had robot policies that could learn impressive behaviors in simulation, and we had very little that worked on an actual robot.

Transfer from sim to real hardware was sparse. You could get it to work if you constrained the action space enough, or learned on top of some pre-existing robot skills, and even then success rates were low. Plenty of serious people doubted whether simulation policies would even transfer at all and argued that real-world data was the only relevant data.

## Robot Locomotion and the "Randomize & Learn" Button

The breakthrough came from robot locomotion, specifically quadrupeds <a class="cite" href="#ref-1">[1]</a>. Researchers trained quadrupeds to walk in simulation, but randomized simulation rollouts such that each quadruped was trained in a slightly different world: different terrains (slopes, stairs, rough ground), ground friction, random shoves and noisy sensors. The complexity of these changes in environment was also oncreased over time as the trained policy improved.

<!-- This was domain randomization <a class="cite" href="#ref-2">[2]</a>, applied to physics and at scale. Instead of learning to walk in one simulator, the robot learns to walk across a whole range of plausible ones. -->

<!-- Two things came out of it. The first was making RL data generation fast and scalable: thousands of simulations running in parallel, all gathering experience at once. -->

<figure>
  <video src="/images/quadruped-parallel.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="ANYmal robots learning to walk in large-scale physics simulations."></video>
  <figcaption>
    <p>ANYmal robots learning to walk in large-scale physics simulations<a class="cite" href="#ref-1">[1]</a>.</p>
  </figcaption>
</figure>

This was domain randomization <a class="cite" href="#ref-2">[2]</a>, applied to physics adn dynamics and at scale, let's call it **large-scale physics randomization**. The big picture was that now they could transfer and run the same policy on the real robot with **no further training**. This recipe blew sim2real for locomotion **wide open**.

<!-- The second is the one I keep coming back to: the "randomize and learn" button. Every simulated robot gets a slightly different world: its own terrain (slopes, stairs, rough ground, getting harder as it improves), its own ground friction, random shoves and noisy sensors. One policy then learns to walk in all of them at once. This is domain randomization <a class="cite" href="#ref-2">[2]</a>, applied to physics and at scale. Instead of learning to walk in one simulator, the robot learns to walk across a whole range of plausible ones. -->

The recipe transferred to several locomotion problems and meant that, as a researcher, you could train a quadruped policy in simulation, press a "randomize & learn" button, see a large number of simulations roll out with slightly different physics, terrain settings, and then see a robust policy learned and working on your **real** robot the next day! Essentially, once real-world physics was inside the training distribution of randomized physics, real-world performance followed.

It might seem obvious now, but this was a big breakthrough back then, and went a long way towards getting the classical robotics community to believe in learned robot policies.
<!-- The key idea was not a new architecture or objective. It was treating the sim2real gap as a range of conditions to train across, rather than an error to engineer away.  -->

<!-- The recipe also **scaled**. Anyone could randomize their simulation, generate more data and get better transfer to the real robot, without needing a new insight for every robot, task or lab. -->

## But Manipulation Needs Visual Input and Realistic Environments

Since then, researchers have been trying to bring the same capability to robot manipulation. But there hasn't been the same amount of success.

Locomotion could get away with geometric perception, such as depth cameras or lidar. Manipulation usually can't: it needs fine-grained visual understanding of the scene, because many tasks depend on what things look like, not just their shape (for example, folding laundry and sorting it by color). Some tasks, like grasping, do work from depth alone <a class="cite" href="#ref-3">[3]</a>, and realistic depth is fairly easy to simulate. But the north star for manipulation is visual policies that map pixels directly to robot actions, since that is the most likely route to solving general-purpose manipulation.

Vision is also much harder to randomize <a class="cite" href="#ref-2">[2]</a>. For a walking robot, randomizing terrain and physics was enough: a quadruped's dynamics live in a smaller space. A camera in somebody's kitchen can see almost anything. So a visual policy needs whole environments that look realistic, with rooms, furniture, clutter, materials and lighting, and enough variety to cover the homes the robot will actually work in. Building and randomizing that many realistic scenes is a much harder task.

As a result, simulation for manipulation was largely left behind, widely seen as something that does not really work or does not pay dividends, and mainly useful for benchmarking.

<figure>
  <img src="/images/molmospaces.gif" loading="lazy"
       alt="Procedurally generated household environments from MolmoSpaces.">
  <figcaption>
    <p>Procedurally generated households from MolmoSpaces <a class="cite" href="#ref-4">[4]</a>: the kind of
    realistic, varied environments a visual policy needs.</p>
  </figcaption>
</figure>

## Pressing the Button for Manipulation

During my year-long research scientist internship at the Allen Institute for AI, I wanted to work on exactly this but for mobile manipulation. Mobile manipulation raises the bar further: a mobile manipulator can move through the entire scene, so the whole home has to be built and randomized, not just a tabletop. Ai2 had already seen visual sim2real work for household *navigation*, and the question was whether the recipe extended to manipulation and mobile manipulation.

[MolmoB0T](https://allenai.github.io/MolmoBot/) <a class="cite" href="#ref-5">[5]</a> is our attempt at answering that. It builds on [MolmoSpaces](https://github.com/allenai/molmospaces) <a class="cite" href="#ref-4">[4]</a>: over 230K indoor environments, from handcrafted homes to procedurally generated multi-room houses, with 48K manipulable objects built from Objaverse assets and human-authored doors and articulated objects.

On top of that, we use fairly standard data generators that rely on ground-truth simulation information such as object poses, collision bodies and bounding boxes. They solve pick-and-place, door opening and articulated-object opening with IK controllers, collision-aware IK and cuRobo <a class="cite" href="#ref-6">[6]</a> motion planning.

None of these ingredients is new. What we changed is the scale and the amount of variation: data generated across textures, lighting, object placement and dynamics, with the aim of making real-world observations look like just another sample from the training distribution.

<figure>
  <video src="/images/molmobot-data.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="Simulated robots performing manipulation tasks across many procedurally generated households, zooming out to thousands of episodes."></video>
  <figcaption>
    <p>MolmoBot data generation: the same tasks generated across thousands of
    randomized households <a class="cite" href="#ref-5">[5]</a>.</p>
  </figcaption>
</figure>

For a sense of scale: MolmoBot-Data is 1.7 million trajectories across more than 94,000 environments and 11,000 objects, roughly 5,700 robot-hours of experience. Every episode is randomized on top of that: where objects are placed and how they are oriented, the number, position, color and intensity of the lights, the surface textures (both procedural and real-world texture maps), the camera poses, and physical parameters such as friction, object mass and joint damping.

## The New Workflow for Sim2Real Manipulation

The policies run on real robots without any real-world fine-tuning. In one of the demos, the desk the robot is mounted on is raised and lowered *mid-rollout*, and the policy keeps going. The training data had randomized robot heights, so to the policy this is just another variation it has already seen.

The bigger change is in how you build a manipulation policy. Even a policy prototyped in simulation usually had to be redone on the real robot with teleoperated demonstrations, so the simulator ended up as a testbed rather than a source of training data.

With enough randomization, the workflow starts to look like the locomotion one:

1. **Write a data generator** for the task in simulation. The simulator knows every object's pose and geometry, so a scripted planner can solve the task without any learning.
2. **Press "randomize": generate data at scale.** Run the generator across thousands of households, randomizing the objects, textures, lighting, camera poses and robot setup in every episode. This is the step that large-scale households and randomization make possible, and it is the manipulation version of locomotion's "randomize physics" button.
3. **Train a visual policy via imitation learning** on the generated demonstrations.
4. **Deploy it zero-shot** on a real arm or mobile manipulator.

When the policy fails on the real robot, you can often fix the simulation, for example by randomizing something you missed, and generate again, instead of collecting more demonstrations. A new task means writing a new generator rather than recording new demonstrations.

This helps most for whole-body tasks like opening doors and cupboards, which are tedious to teleoperate. Instead, a cuRobo-based whole-body motion planner can be tweaked to solve the task in simulation, and the same recipe then transfers the resulting policy zero-shot to a real mobile manipulator.

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
    <p>"Put the banana in the brown bowl." When the brown bowl is out of view the policy heads for the black one, then changes course once it sees the right bowl <a class="cite" href="#ref-5">[5]</a>.</p>
  </figcaption>
</figure>

## So Is Sim2Real Solved?

Not so fast!

For rigid-body manipulation and mobile manipulation, partly. In our pick-and-place tests the policy reaches about 80% success zero-shot, which is a useful starting point for fine-tuning with real data.

{{< figure-svg src="zero-shot-results.svg"
    caption="Zero-shot pick-and-place on a Franka FR3: 120 real-world trials, no real-world fine-tuning. MolmoBot-Pi0 keeps the π0 <a class='cite' href='#ref-8'>[8]</a> architecture and swaps in simulation data, so much of the gain comes from the data. The baseline is π0.5 <a class='cite' href='#ref-9'>[9]</a> trained on DROID <a class='cite' href='#ref-10'>[10]</a>. Results from <a class='cite' href='#ref-5'>[5]</a>." >}}

For deformable and soft objects, and for more complex objects in general, it remains to be seen. But the results are promising!

The open question for me is how far this scales and how many more we can add. There is still a lot left to vary.

## Promising Recent Progress

MolmoBot is not the only recent result pointing this way. AnyTask <a class="cite" href="#ref-11">[11]</a> from RAI automates task and data generation for sim2real policy learning, and SimVLA <a class="cite" href="#ref-12">[12]</a> trains a mobile-manipulation policy entirely in procedurally generated houses and runs it zero-shot in real homes.

A complementary direction is large-scale reinforcement learning in simulation, distilled into a policy that sees only RGB. Zhang et al. <a class="cite" href="#ref-13">[13]</a> scale RL to over a million parallel environments by sampling simulator resets at the edge of what the policy can currently do, and transfer zero-shot to real assembly tasks such as gear insertion. DoorMan <a class="cite" href="#ref-14">[14]</a> from NVIDIA uses a similar recipe to teach a humanoid to open real doors.

<figure>
  <img src="/images/sim2real-balanced-data-diet.jpg" loading="lazy"
       alt="A robot gripper inserting a gear in simulation, next to a real UR5e robot doing the same.">
  <figcaption>
    <p>Gear insertion in simulation (left) and zero-shot on a real UR5e (right) <a class="cite" href="#ref-13">[13]</a>.</p>
  </figcaption>
</figure>

<figure>
  <img src="/images/sim2real-doorman.jpg" loading="lazy"
       alt="Three simulated humanoids opening doors with different wall, door and lighting randomization, next to a real humanoid opening a door.">
  <figcaption>
    <p>DoorMan in simulation with randomized doors, walls and lighting (left three) and opening a real door zero-shot (right) <a class="cite" href="#ref-14">[14]</a>.</p>
  </figcaption>
</figure>

It is an exciting time for sim2real in manipulation. Stay tuned for more on this!

---

MolmoBot was accepted as an Oral (top 3% of papers) at the Conference on Robot Learning (CoRL) 2026! You can read the paper [here](https://arxiv.org/abs/2603.16861), and see more on the [project page](https://allenai.github.io/MolmoBot).

<section class="references" aria-labelledby="references">
<h2 id="references">References</h2>
<ol>
  <li id="ref-1">N. Rudin, D. Hoeller, P. Reist, M. Hutter. <a href="https://arxiv.org/abs/2109.11978">Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning</a>. Conference on Robot Learning (CoRL) 2021.</li>
  <li id="ref-2">J. Tobin, R. Fong, A. Ray, J. Schneider, W. Zaremba, P. Abbeel. <a href="https://arxiv.org/abs/1703.06907">Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World</a>. IROS 2017.</li>
  <li id="ref-3">S. Jauhri, I. Lunawat, G. Chalvatzaki. <a href="https://arxiv.org/abs/2306.07392">NeuGraspNet: Learning Any-View 6DoF Robotic Grasping in Cluttered Scenes via Neural Surface Rendering</a>. RSS 2024.</li>
  <li id="ref-4">Y. Kim, W. Pumacay, O. Rayyan, et al. <a href="https://arxiv.org/abs/2602.11337">MolmoSpaces: Large-Scale Open Ecosystem for Robot Manipulation and Navigation</a>. RSS 2026.</li>
  <li id="ref-5">S. Jauhri*, A. Deshpande*, M. Guru*, R. Hendrix*, et al. <a href="https://arxiv.org/abs/2603.16861">MolmoB0T: Large-Scale Simulation Enables Zero-Shot Manipulation</a>. Conference on Robot Learning (CoRL) 2026.</li>
  <li id="ref-6">B. Sundaralingam, S. K. S. Hari, A. Fishman, C. Garrett, K. Van Wyk, et al. <a href="https://arxiv.org/abs/2310.17274">cuRobo: Parallelized Collision-Free Minimum-Jerk Robot Motion Generation</a>. arXiv 2023.</li>
  <li id="ref-7">C. Chi, S. Feng, Y. Du, Z. Xu, E. Cousineau, B. Burchfiel, S. Song. <a href="https://arxiv.org/abs/2303.04137">Diffusion Policy: Visuomotor Policy Learning via Action Diffusion</a>. RSS 2023.</li>
  <li id="ref-8">K. Black et al. <a href="https://arxiv.org/abs/2410.24164">π0: A Vision-Language-Action Flow Model for General Robot Control</a>. arXiv 2024.</li>
  <li id="ref-9">Physical Intelligence. <a href="https://arxiv.org/abs/2504.16054">π0.5: a Vision-Language-Action Model with Open-World Generalization</a>. arXiv 2025.</li>
  <li id="ref-10">A. Khazatsky et al. <a href="https://arxiv.org/abs/2403.12945">DROID: A Large-Scale In-The-Wild Robot Manipulation Dataset</a>. RSS 2024.</li>
  <li id="ref-11">R. Gong, X. Zhang, J. Shang, et al. <a href="https://arxiv.org/abs/2512.17853">AnyTask: An Automated Task and Data Generation Framework for Advancing Sim-to-Real Policy Learning</a>. arXiv 2025.</li>
  <li id="ref-12">K. Baik, Y. Lee. <a href="https://arxiv.org/abs/2610.11248">SimVLA: Zero-Shot Sim-to-Real VLA Learning for Mobile Manipulation</a>. arXiv 2026.</li>
  <li id="ref-13">O. Zhang, M. Guaman Castro, P. Yin, I. Dagnino, A. Gupta, R. Scalise, B. Boots. <a href="https://arxiv.org/abs/2610.12465">A Balanced Data Diet: Addressing the Exploration Bottleneck in Mega-Scale RL for Robot Control</a>. Conference on Robot Learning (CoRL) 2026.</li>
  <li id="ref-14">H. Xue, T. He, Z. Wang, Q. Ben, W. Xiao, et al. <a href="https://arxiv.org/abs/2512.01061">Opening the Sim-to-Real Door for Humanoid Pixel-to-Action Policy Transfer</a>. arXiv 2025.</li>
</ol>
</section>
