---
title: "MolmoBot: Has Sim2Real for Manipulation Finally Been Unlocked?"
date: 2026-09-10
draft: false
description: "Locomotion got a randomize button in 2021 and sim2real fell over. Manipulation never got one — until now."
summary: "Locomotion got its randomize button in 2021. Manipulation finally has one too."
cover:
  image: "/images/molmobot-sim2real.gif"
  alt: "Manipulation policies trained in randomized simulation, running on real robots"
  relative: false
---

When I started my PhD in 2021, the state of robot learning was roughly this: we had beautiful simulations, we had policies that did impressive things inside them, and we had very little that worked on an actual robot.

Transfer to real hardware was sparse. You could get it to work if you constrained the action space enough, or leaned on a bag of tricks, and even then success rates were low. Plenty of serious people doubted whether simulation policies would ever transfer at all. "It works in simulation" was the kind of sentence that made reviewers narrow their eyes.

## Then locomotion got a button

The breakthrough did not come from manipulation. It came from legged locomotion — quadrupeds specifically.

Marco Hutter's group at ETH Zurich, the group behind ANYmal, worked out large-scale simulation and sim2real on GPUs in [Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning](https://arxiv.org/abs/2109.11978). That paper blew sim2real for locomotion **wide open**.

Two things came out of it. The first was making RL fast: thousands of simulations running in parallel, all gathering experience at once. That part got the attention.

The second is the one I keep coming back to. Once you have a policy that walks passably in simulation, you press a button called **randomize**. Now you are training across many simulations that differ slightly from each other — terrain, robot dynamics, friction, mass, actuator response. Physics domain randomization, at scale. The robot stops learning to walk in *your* simulator and starts learning to walk in the space of plausible simulators.

{{< figure-svg src="randomize-button.svg"
    caption="The whole recipe. Randomization is not a detail of the training setup — it is the step that makes the real world in-distribution for the policy." >}}

What comes out transfers *directly*. You take the policy trained with the button pressed and you put it on the robot the next day.

This seems obvious in hindsight, which is usually a sign that something important happened. It is worth being precise about why it mattered. It was not a clever architecture or a new objective. It was the realization that the domain gap is not a wall you engineer your way through, but a distribution you can simply cover. Make real-world physics in-domain and real-world performance follows.

And crucially, the recipe **scaled**. Anyone could randomize harder, generate more data, and get more transfer. No new insight required per robot, per task, per lab. That is a very different thing from a result — that is a recipe.

## Why manipulation didn't get one

Naturally everyone then asked whether the same trick would work for manipulation. For a while, the answer was mostly no.

Some manipulation does work from depth alone — grasping being the obvious case, including [some of my own PhD work](https://sites.google.com/view/neugraspnet). Depth is a fairly narrow, fairly simulatable input. But the actual goal is visual policies: pixels in, joint positions or torques out. That is the paradigm most likely to solve manipulation.

And vision is a much harder thing to randomize. It is not just that you need *more* randomization — it is that the space you are trying to cover is enormous. A quadruped's proprioception lives in a modest, well-understood space. The set of things a camera might see in somebody's kitchen does not.

So the field took the other road: use simulation as a rough starting point, then rely heavily on teleoperation. That road has produced genuinely great work in imitation learning from real-world data. It has also been expensive, because teleop data is paid for in human hours, and human hours do not have a randomize button.

Meanwhile simulation for manipulation quietly went out of fashion. Not disproven exactly — just widely treated as something that does not pay dividends.

## Manipulation gets its button

During my year-long research scientist internship at the Allen Institute for AI, I wanted to work on exactly this. Ai2 had already seen visual sim2real work for household *navigation*, and the obvious question was whether the recipe extended to manipulation and mobile manipulation. Having spent a PhD on robot learning for homes, on 2D and 3D vision policies, and on mobile manipulation, I did not need much convincing.

With [MolmoB0T](https://allenai.github.io/MolmoBot/) ([paper](https://arxiv.org/abs/2603.16861), [Ai2's announcement](https://allenai.org/blog/molmobot-robot-manipulation)), I think the answer is yes.

The substrate is [MolmoSpaces](https://github.com/allenai/molmospaces) (RSS 2026): 10K to 100K procedurally generated households, built from Objaverse assets with human-authored doors and articulated objects. On top of that, fairly standard data generators — the kind that work well when you have ground-truth object poses, collision bodies and bounding boxes to lean on — solving pick-and-place, door opening, and opening articulated objects.

<figure>
  <img src="/images/molmospaces.gif" loading="lazy"
       alt="Procedurally generated household environments from MolmoSpaces.">
  <figcaption>
    <p>MolmoSpaces: procedurally generated households. A randomize button needs
    something to randomize <em>over</em>.</p>
  </figcaption>
</figure>

None of those ingredients is exotic. That is rather the point. What you get by combining them is the randomize button for manipulation: generate at scale across textures, lighting, object placement and dynamics, and you get the diversity that finally puts real-world observations in distribution.

<figure>
  <video src="/images/molmobot-data.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="Simulated robots performing manipulation tasks across many procedurally generated households, zooming out to thousands of episodes."></video>
  <figcaption>
    <p>The button, pressed: the same tasks generated across thousands of randomized
    households. Video from the <a href="https://allenai.github.io/MolmoBot/">MolmoB0T project page</a>.</p>
  </figcaption>
</figure>

For a sense of scale: MolmoBot-Data is 1.8 million trajectories across 94,000 generated houses and 11,000 objects, roughly 5,800 robot-hours of experience. DROID, one of the largest real-world manipulation datasets, is 76,000 trajectories from 350 hours of teleoperation.

## What actually changes

The policies transfer directly. There are demos on the [project page](https://allenai.github.io/MolmoBot/) where the desk the robot is mounted on is raised and lowered *mid-rollout* and the policy simply carries on, because a policy trained across tens of thousands of slightly wrong worlds has seen worse.

<figure>
  <video src="/images/molmobot-table-height.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="A robot arm putting blue tape in a wooden box while the desk it is mounted on is raised and lowered."></video>
  <figcaption>
    <p>"Put the blue tape in the wooden box," while the desk moves up and down
    underneath. The training data started the robot at random heights, so this is
    just one more variation. Video from the <a href="https://allenai.github.io/MolmoBot/">MolmoB0T project page</a>.</p>
  </figcaption>
</figure>

<figure>
  <img src="/images/molmobot-sim2real.gif" loading="lazy"
       alt="MolmoB0T policies opening a door and a drawer in randomized simulation (top) and executing manipulation tasks on a real robot (bottom).">
  <figcaption>
    <p>Trained entirely in randomized simulation. Running on real hardware.</p>
  </figcaption>
</figure>

<figure>
  <video src="/images/molmobot-camera-demo.mp4" autoplay muted loop playsinline preload="metadata"
         aria-label="A real robot arm putting a banana in a brown bowl, shown from a third-person camera and the policy's own exo and wrist cameras."></video>
  <figcaption>
    <p>"Put the banana in the brown bowl." When the brown bowl is out of view the
    policy heads for the black one, then changes course once it sees the right
    bowl. Video from the <a href="https://allenai.github.io/MolmoBot/">MolmoB0T project page</a>.</p>
  </figcaption>
</figure>

But the result I care about is not a success rate. It is a change in what you do on a Tuesday.

Until now, building a simulation policy for manipulation meant eventually throwing the simulator away and starting the real work: rigging teleop, collecting demonstrations, paying in human hours. The simulator was a prototype you discarded.

Now you press randomize. Overnight you get scaled-up, domain-randomized data for your task. You train. You come back in the morning with a policy that runs on your real manipulator or mobile manipulator.

That is a different relationship with your simulator. It stops being a demo environment and becomes infrastructure — including for whole-body problems like opening doors and cupboards, which were among the least appealing things to collect real data for.

Simulation is back. Shoutout to concurrent work from RAI, [AnyTask](https://arxiv.org/abs/2512.17853), arriving at a similar place from a different direction — always a good sign that the thing is real and not just our particular pipeline.

## So is sim2real solved?

No, and I would be suspicious of anyone who said otherwise.

For rigid-body manipulation and mobile manipulation, partly yes: there is now a recipe that gets you to something like 80% success, which you then fine-tune with real data. That is a genuinely different starting point from zero.

{{< figure-svg src="zero-shot-results.svg"
    caption="Zero-shot pick-and-place on a Franka FR3: 120 real-world trials, no real-world fine-tuning. MolmoBot-Pi0 keeps the π0 architecture and swaps in simulation data, so much of the gain comes from the data. Numbers from the [MolmoB0T paper](https://arxiv.org/abs/2603.16861)." >}}

For deformables, soft bodies, and the long tail of objects that do not behave like convex rigid lumps, it remains to be seen. Cloth does not care about your friction randomization.

What I find most interesting is that the bottleneck has moved. It is no longer "can this transfer at all" — it is how many axes of diversity we can scale. MolmoSpaces is a good household simulator and there is a lot of headroom left in it.

More on that soon.
