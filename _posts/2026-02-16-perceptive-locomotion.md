---
layout: post
title:  "Perception-aware RL locomotion with Mujoco/Mjlab"
date:   2026-02-16 12:00:00 +10
categories: RL humanoid unitree mujoco GPU locomotion depth
comments: true
excerpt: Training an end-to-end locomotion policy that can use depth sensor input, with RL
assets: /assets/images/perception-aware-locomotion/
preview-image: "/assets/images/perception-aware-locomotion/gap_jump.gif"
---

This is a quick guide on how I trained an RL locomotion policy with height sensor input on mjlab. Standard locomotion policies are blind, purely based on proprioception, which works wonderfully for flat surfaces, bumpy terrain, and small stairs, but more varied terrain is infeasible without knowing what's in front of you. Pure RGB sensor input requires a lot of compute and data to gain an understanding of the physical world, so a depth-only sensor is a much easier and more efficient way to perceive the geometry of the world around you and focus on the important information. Additionally, training with RL only works if you can train on millions of samples - this precludes heavy computation like vision encoders, and favours any methods that are lightweight and can be massively parallelised. Although we've been training models on GPUs for decades, it's only more recently that simulation environments have been able to run on them. With the [recent release](https://github.com/mujocolab/mjlab/discussions/539) of the GPU-accelerated RayCast sensor in mjlab, anyone can now run thousands of simultaneous simulations and train a perception-aware policy in a matter of hours, for a couple of dollars.

# 1. Choose parameters
I built off the Unitree G1 velocity-tracking demo in [mjlab](https://github.com/mujocolab/mjlab). I changed the height sensor a bit for better correspondence to the real world:
- Attach the sensor to the head instead of pelvis, where an RGB sensor would more likely be
- Have the sensor only looking forward, instead of forwards and back
- Reduce the resolution of the sensor from 0.1 to 0.15m spacing for faster training (less ray traces, less policy params)

I was considering doing a pinhole model sensor, like a depth camera, but went with the standard grid-pattern for easier learning. Would be good to try with the pinhole too, I'm guessing it would work similarly well.

# 2. Generate terrain
I wanted to force the policy to use the height sensor, so my first thought was along the lines of jumping between benches. After playing around a bit, the variable heading of the velocity tracking task made it clear that having gaps in every direction was a better task to train on, so it could practice overcoming gaps no matter which way it was oriented. Removing the floor in between also made for a clear failure signal, rather than getting stuck in a weird position when falling between raised platforms.

I randomised the platform size, gap between platforms, and relative height of platforms. It was important to start these parameters at an easy level, from basically flat ground (large platforms, small gaps, minimal height difference), to the more difficult target terrain (small platforms, large gaps) - starting it learning on gaps too large just made it too scared to take the risk of hopping across.

# 3. Train a base policy on rough terrain
I figured it would be easier to train it on gaps if I already had a policy that could stand, walk and run. This was already taken care of by the existing rough terrain training environment provided in mjlab (`Mjlab-Velocity-Rough-Unitree-G1`).

One unexpected thing did occur - when transferring the base policy to the gap environment, the spike in sensor inputs from out-of-range sensor readings (in the gaps) caused the policy to go crazy and fold over on itself, and it eventually learned to just avoid looking at gaps. I did a couple of things to condition these readings:
- normalising the sensor readings to the range [0, 1]
- reducing the maximum reading distance to 3m from 5m (for higher resolution in its useful range of 0-2m)
- introducing the policy to gaps in the beginning of its training. The rough environment had no gaps to begin with, so these high readings in the gaps were out of distribution and made adaptation much more difficult. I replaced the flat terrain with simple gap terrain to give it some exposure to the full range of sensor readings from the beginning.

It was important to match the difficulty of the gap task to the other terrains. Because the terrain difficulty is increased as the policy becomes more successful, and the gap task requires a reasonably different strategy to all the other terrains which are solvable with blind locomotion, jumping to a gnarly gap terrain too soon prohibits learning a good strategy to deal with the gap terrain. If you couldn't tell, I was quite overzealous on the gaps at first. I also separated variation in gaps from variation in height - having both at once made the task quadratically more difficult, rather than linearly like the other terrains.

I also found it didn't learn to run well by default, so I had to edit some of the reward weights:
- the reward for being upright was half the velocity tracking weight, so it learned to fall quickly forward when given higher velocity commands, instead of running forwards, prioritising velocity tracking over staying upright. Increasing the upright reward to match the velocity tracking reward fixed this.
- Increasing the standard deviation for upright error, allowing it to take a broader range of angles to be dynamic but stay upright.
- Reducing the action_error_l2 weight by a factor of 10. This is a penalty on rate of change of actions, intended to encourage slow, smooth actions, but prohibiting dynamic behaviour.
- Reduce the foot clearance reward by a factor of 10. This is intended to give it a nice gait for walking by targeting a step height of 10cm, but was prohibitive for running, jumping, and dealing with big changes in elevation.

Here's a video of how this policy performed:
<div style="text-align: center;">
<video width="75%" controls loop playsinline autoplay>
  <source src="{{page.assets}}/rough_policy.mp4" type="video/mp4">\
</video>
</div>
This took about 2.7k steps at 8192 envs - around 3 hours on a single A100.

# 4. Train on gap terrain
Taking on board the previous learnings about smooth ramping up of difficulty, training on the gap only terrain was very straightforward and the model learned quickly. I modified the rewards as follows:
- Reducing the pose constraint by 10x - this penalty encourages the robot to match a reference pose (upright, hands by side), but with more dynamic motions like jumping between platforms, this is limiting.
- Removing the foot swing height (encourages foot swing zenith to be at a determined height), and reducing foot clearance penalty by 10x again - to allow dynamic foot swings and jumps
- Increasing the std deviation for the upright constraint - allowing a larger range of angles, whilst still enforcing uprightness.

With 1k more steps (1 hr on A100), the policy learned to do this:
<div style="text-align: center;">
<video width="100%" controls loop playsinline autoplay>
  <source src="{{page.assets}}/gap_jump_final.mp4" type="video/mp4">\
</video>
</div>

# Notes on training
- The PPO algorithm is sensitive to rewards, so it takes quite a bit of playing around to encourage the right behaviour. Often it's a compromise - allowing more dynamic behaviour for jumping means that the normal walking gait ends up more erratic and less efficient than it could be.
- The terrain difficulty increases as the model improves, but eventually it levels off. I never found that letting it continue to train when it levelled off resulted in increased performance, just wasted GPU hours. Progressing to harder terrain required modifying the rewards/curriculum to break past.
- It was fairly easy to see which constraints were limiting behaviour by visually observing the behaviour of the most recent checkpoint, and looking at the magnitude of the reward. Usually you would see one constraint (e.g. `action_error_l2`) improving whilst the number of failures increased, and the velocity tracking and upright penalty worsened.
- I trained on Google Colab. 1hr of A100 time is about 54c, so to train the end policy cost around $2. It could definitely be done in less. This doesn't include the GPU hours used up on experimenting with different rewards and terrains, which was about 10x as much, but worth it for the learning experience!

# Train it yourself
Use this branch on my public fork of mjlab: [https://github.com/eufrizz/mjlab/tree/perception-walking](https://github.com/eufrizz/mjlab/tree/perception-walking). I basically just followed the simple instructions in the mjlab README.

```
uv run train Mjlab-Velocity-Rough-Unitree-G1 --env.scene.num-envs 4096
```


I cut it off early (~2.7k) once it wasn't showing much improvement with the task.

Then from that checkpoint, continue train the gap env:

```
uv run train Mjlab-Velocity-Gap-Unitree-G1 --env.scene.num-envs 4096 --agent.resume True --wandb-run-path <run-path-from-rough-training-run>
```

Whilst you train, it's useful to inspect the progress on your local machine with:


```
uv run play Mjlab-Velocity-Gap-Unitree-G1 --wandb-run-path <run-path>
```

Happy training!