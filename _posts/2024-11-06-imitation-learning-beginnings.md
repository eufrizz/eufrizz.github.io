---
layout: post
title:  "First forays into imitation learning for robotic manipulation"
date:   2024-09-06 12:00:00 +10
categories: robotics arm mujoco manipulation trajectory imitation learning deep neural network
comments: true
excerpt: Getting into imitation learning from the basics.
assets: /assets/images/imitation-learning-beginnings/
preview-image: ""
---
# Why imitation learning?
Deep learning is pushing forward the frontier of tasks possible with robotic arms, making previously impossible tasks, like folding shirts or tying shoelaces, possible. Manipulation tasks are very difficult to model due to the complexity of contact forces, deformable objects, and visual appearance in the real world, which is why learning based methods, which can implicitly learn these models from lots of data, are giving state of the art results. Imitiation learning (IL), where the model learns from expert demonstrations of a task, and reinforcement learning (RL), where the model explores by trial and error, are techniques that have both had success. Imitiation learning tends to be the dominant approach, as the expert demonstrations can much more quickly and efficiently get the model to learn desired behaviour, and reward functions can be very complex to define by hand, especially as tasks get more sophisticated.

Imitation learning has two main approaches: behavioural cloning (BC) and inverse reinforcement learning (IRL). BC is a simple supervised learning technique, where given the current observations of the robot and environment, the model is trained to predict the next action the expert takes. This breaks down when observations not in the data are encountered at test time, which is a common occurrence if things go a little off track - and techniques have been developed to help this. IRL involves training a network to estimate a reward function from the expert's actions, i.e. deducing the "goals" that are shaping the expert's decisions. A standard RL algorithm can then be used on this estimated reward function to find the optimal policy.


# Goal
To get my feet wet with imitiation learning for robotic manipulation, I decided to opt for the simplest possible approach, doing simple behavioural cloning, using a basic multilayer perceptron (MLP) as the actor network (i.e. the network learns to make decisions), and pretrained Resnet-18 networks for perception. The chosen task was simply to grasp (and optionally lift) a block in simulation, and I generated the demonstrations using a script (instead of recording human teleoperation), allowing for quick data collection and iteration.

# Environment setup
In previous blog posts, I have been playing around with an Xarm Lite6, and I was keen to get this project running in real life on the one I had access to. However, the [Mujoco Menagerie](https://github.com/google-deepmind/mujoco_menagerie) model did not have a gripper modelled for it, so I had to go and do a bit of CAD. Thankfully the .step files of the gripper were available from the Xarm website, so it was just a matter of separating the fingers, exporting the meshes, and setting it up in the Mujoco model XML. It ended up being more time consuming than expected (as usual) but I did gain good insight into how Mujoco's modelling works, and I was able to contribute the model back to the community.

The environment was set up with [Gymnasium](https://github.com/Farama-Foundation/Gymnasium), as most IL/RL projects are, and I used [ACT](https://github.com/tonyzhaozh/act) and [LeRobot](https://github.com/huggingface/lerobot) as references. I also wanted to have it compatible with LeRobot for the future. I chose a simple red cube for the object to grasp, so that it stood out and was simple to grasp with the limited gripper, and position of the arm and the position of the cube were randomised. The gripper is a little unusual in that it is pressure actuated, so it's an all or nothing actuation which I modeled as a discrete variable with 3 states - off, closed and open. This was embedded with a 3-bit one hot encoding for the network.

- Action space: 7DOF - 6 joint angles and 1 gripper action
- Observation space: 6 joint angles, 3rd person camera (320x240), 1st person wrist camera (320x240)

# Data collection

Because the task is simple, and the cube position is known in simulation, it was easiest to write a script to generate the training data. This ended up being incredibly useful as, even thought the initial set up took a while, being able to rerecord data at the press of a button made it really easy to iterate and test new ideas. The script took the following steps:
1. Generate and follow a screw trajectory from the intial (random) position to just above the box
2. Open the grippers and lower down around the box
3. Close the grippers once at the necessary height
4. Lift upwards to a predetermined height

Instead of adding collision avoidance to the script, I decided to take the quick and dirty option of simply discarding/redoing any trajectories that failed. They failed around 20% of the time, so it was no big deal just to generate a few more and get on with the imitation learning.

Two important steps to ensure the validity of the data were:
1. Run an open loop replay, i.e. play the recorded actions from a trajectory in the environment, and ensure that the grasp is reproducible with the data. This is great for catching any bugs in the data collection and/or environment setup - we can't train a successful model on data that doesn't work!
2. Plot/visualise the recorded data. Make sure the camera views take in the full scene, and the joint angles look correct. I spent a week training with no results, until I plotted the trajectories and noticed that the joint angle observations were static (a bug in the environment where I needed to do a deepcopy).


<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/scripted_lift_1.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <!-- <div class="caption">45 random, 5 fixed, 20 epochs</div> -->
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/scripted_lift_2.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <!-- <div class="caption">40 random, 5 fixed, 20 epochs</div> -->
    </div>
</div>
<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/scripted_lift_3.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <!-- <div class="caption">40 random, 5 fixed, 20 epochs</div> -->
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/scripted_lift_5.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <!-- <div class="caption">40 random, 5 fixed, 20 epochs</div> -->
    </div>
</div>
<div style="text-align: center;" class="caption"> A sample of the data generated by the scripted policy</div>

# Method

- For maximum simplicity, a simple behavioural cloning approach was taken. The model is trained on individual observation-action pairs, in random order, to minimise the loss between the predicted next action and the recorded next action. The next action was chosen to be the one 0.1s ahead.
- The model architectures tried were MLPs (usually around a size of 64x64x64), with ReLU activation. For perception, a pretrained Resnet-18 was used for each camera, with weights unfrozen.
- Two cameras were used in the data. A third person side-on camera, and a 1st-person wrist camera fixed to the gripper.
- Joint angles were used the observation and action space. Cartesian coordinates could be easier for the model to learn, but the inverse kinematics is susceptible to singularities, and the model should be able to learn the robot kinematics with enough demonstrations. It would be good to compare the differences in future, but state of the art approaches have been successful with both.
- Joint angles were normalised to the range [-1, 1] for the model
<div style="text-align: center;">
    <img src="{{page.assets}}/network_diagram.svg" style="width: 80%">
</div>
<div style="text-align: center;" class="caption"> A diagram of the model architecture</div>

# Experiments and results

## Initial mistakes
For brevity, I won't bother going through the results whilst I was testing the system with flaws that were obvious in hindsight. But so you don't make the same mistakes, I will briefly touch on them:
1. As previously mentioned, the joint angle observations weren't being recorded correctly. Validate the data by running open loop replays, and plotting it!
2. I was training on the current action instead of the next action. This directs the model's action towards the goal, stops it getting into a potential feedback loop, and is done in all IL papers.
3. The vision encoder weights were frozen. I thought "it's already learned good visual representations, why bother training?", but thankfully [this great tutorial](https://supervised-robot-learning.github.io) explained that it performs much better when fine tuned on data from the target domain e.g. it learns to recognise the object.

## Making the task more tractable
These were some other steps I took to maximise the model's chances
1. Using a wrist camera. This is important, and I noticed a marked improvement with how close the robot could get to the block when it came in view of the wrist camera. The 2D side cam image doesn't provide great information on the relative positioning of the gripper to the object, especially in the depth axis. Papers often find much better performance with a wrist camera. This is just extra, useful information that can only help the model.
2. Object choice. A cube is easy to grip. The larger, the better, and in a contrasting colour, to make it easier for the model to see where the object is.

## Evaluation
The rollouts shown are controlled at 30Hz, and are each 5s long. For each experiment, 15 rollouts are done, and the average reward is recorded for each. The mean and standard deviation of the average reward across all 10 rollouts are then used to evaluate performance on each task. The videos from the rollouts are concatenated into one video, i.e. a 50s video where each 5s is a different rollout.
The reward structure is as follows:
- 0-1 (continuous) - distance from grippers to object, where 0 is >10cm away, and 1 is touching; and grippers not touching ground
- 2 - one gripper touching the object and grippers not touching ground
- 3 - two grippers touching the object and grippers not touching ground

And additionally for the lift task:
- 4 - two grippers touching the object, grippers not touching ground, and object not touching ground (i.e. lifted)
- 5 - two grippers touching the object and grippers above 20cm

But as you will see we hardly every get to 2 let alone higher.

## Experiment 1: Task/data choice
Because we are learning to imitate the data, the data dictates the task that the policy learns.

### Grasp and lift with randomised arm and cube position
I started with this task because I thought it would be fairly simple. By randomising the position of the object and the arm, I hoped it would force the robot to learn how to detect the object, and how to control the arm. It turned out to be a little more difficult than expected, and I was unable to get it to perform well.
<div style="text-align: center;">
<video width="50%" autoplay loop controls playsinline>
  <source src="{{page.assets}}/task_choice/grasp_and_lift.mp4" type="video/mp4">
</video>
</div>
<div style="text-align: center;" class="caption"> Grasp and lift task</div>

It seemed to commonly fail by getting stuck, usually up high or just above the object, and never actually lowering down to the block. I hypothesised that this was due to the multimodality of the task - i.e. the reaching down to grasp and lifting up are two distinct/opposite actions, and it was averaging them out. Thus I decided to make the task easier by just focussing on the the grasping portion.

### Grasp only - randomised trajectories
Unfortunately this did little to improve performance. Clearly there were bigger issues than the multimodal data. The policy seemed to only vaguely go in the direction of the object. So to make the task easier again, I decided to hone in on a fixed task instead of randomly initialising.
Here is how well the same model as before, with 50 randomised trajectories, does on the fixed task. It gets to roughly the right location, but never manages to actually pick up the object.
<div style="text-align: center;">
<video width="50%" autoplay loop controls playsinline>
  <source src="{{page.assets}}/task_choice/grasp_random_task_epoch_20.mp4" type="video/mp4">
</video>
</div>
<div style="text-align: center;" class="caption"> Grasp only task</div>

#### Grasp only - fixed trajectory
To make things simpler again, I scaled it back to a single, fixed task, where the starting joint angles and object position are the same every time. This is a scripted trajectory showing the fixed task:
<div style="text-align: center;">
<video width="50%" autoplay loop controls playsinline>
  <source src="{{page.assets}}/scripted_fixed_task.mp4" type="video/mp4">
</video>
</div>
<div style="text-align: center;" class="caption"> Fixed task, completed by expert (scripted) demonstration</div>

I first tried a model trained on the randomised trajectories to see how well and consistently it performed on the fixed task:
<div style="text-align: center;">
<video width="50%" autoplay loop controls playsinline>
  <source src="{{page.assets}}/task_choice/grasp_fixed_task_50r_epoch_20.mp4" type="video/mp4">
</video>
</div>
<div style="text-align: center;" class="caption"> Fixed task, with policy trained on randomised trajectories</div>

It shows some promise, but it can't quite reach all the way down and pick the object up.

Thus to make it easier again, I trained a model with only data of the fixed trajectory. This gives an improvement, it gets near to the block consistently, and almost manages to grasp it. This is obviously a pretty limited model as it can only (barely) accomplish one task, so offers no advantage over a script.
<div style="text-align: center;">
<video width="50%" autoplay loop controls playsinline>
  <source src="{{page.assets}}/task_choice/fixed_task_50f_epoch_20.mp4" type="video/mp4">
</video>
</div>
<div style="text-align: center;" class="caption"> Fixed task, with policy trained only on this task</div>


### Mixture of random and fixed
In order to see how the network would respond to adding in some randomised trajectories, I experimented with different blends of fixed and random trajectories in the data. Ideally, the model would be able to accomplish the fixed task easily as it is in the training data, and then adapt to some more random tasks. Unfortunately, this adaptation never occurred, and the models struggled with the fixed task at any level of randomised trajectories being blended in, as seen in these rollouts:

<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/task_choice/grasp_fixed_task_45r_5f_epoch_20.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">45 random, 5 fixed, 20 epochs</div>
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/task_choice/grasp_fixed_task_40r_10f_epoch_19.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">40 random, 5 fixed, 20 epochs</div>
    </div>
</div>
<div style="margin: 2px;text-align: center;">
    <video autoplay loop controls playsinline>
    <source src="{{page.assets}}/task_choice/grasp_fixed_task_25r_25f_epoch_19.mp4" type="video/mp4">
    Your browser does not support the video tag.
    </video>
    <div class="caption">25 fixed, 25 random, 20 epochs</div>
</div>
<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/task_choice/grasp_fixed_task_10r_40f_epoch_20.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">10 random, 40 fixed, 20 epochs</div>
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/task_choice/grasp_fixed_task_5r_45f_epoch_20.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">5 random, 45 fixed, 20 epochs</div>
    </div>
</div>

This is a lot to take in at once, so here are the evaluation results. I've included the results at epoch 10 and 20 as they differ a bit. As a reminder, the evaluations are done on 15 rollouts. These are all done on the fixed task, except the last column which is done on randomised positions.
<div style="text-align: center;">
    <img src="{{ page.assets }}/task_choice_results.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> Task choice results, with different blends of fixed and random trajectories in the dataset, at two different epochs</div>

At epoch 10, the blend of 5 random, 45 fixed does slightly better than the purely fixed task (though with high variance). However, at epoch 20 this it managed to unlearn everything and get 0 reward. The purely fixed trajectory data managed to improve with more epochs, and achieved the best overall performance. The 45 random, 5 fixed task managed to have a small amount of success at epoch 20, and you can see its performance in the first of the videos above.

Overall, the model was unfortunately unable to generalise to random tasks. Some of the numerous other experiments attempted are included in the appendix, including training on a larger dataset, for longer, and with larger networks.

## Model architecture
I didn't manage to get much improvement in performance with different model architectures. Here's a comparison with an extra hidden layer to make the MLP 64x64x64x64 (referred to as 64^4 in the diagram). More testing on architecture for the random task is included in the appendix.
<div style="text-align: center;">
    <img src="{{ page.assets }}/network_arch_results.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> Comparing results from the standard 64x64x64 MLP with 64x64x64x64</div>


<!-- 64^3 w dropout -->

## Training time
Increased training time on the purely fixed dataset resulted in improving performance.

<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/train_time_fixed_task/train_time_epoch_8_fixed.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">8 epochs</div>
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/train_time_fixed_task/train_time_epoch_16_fixed.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">16 epochs</div>
    </div>
</div>
<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/train_time_fixed_task/train_time_epoch_32_fixed.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">32 epochs</div>
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{page.assets}}/train_time_fixed_task/train_time_epoch_64_fixed.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">64 epochs</div>
    </div>
</div>

The results are graphed here:

<div style="text-align: center;">
    <img src="{{ page.assets }}/train_time_results_fixed.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> Performance on the fixed dataset/task with increasing train time</div>

And the loss here. We see a strange spike in loss around 3.7k steps (~45 epochs)
<div style="text-align: center;">
    <img src="{{ page.assets }}/train_time_loss_fixed.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> The loss curve for the fixed dataset. 2.6k steps is 32 epochs. (The change in colour can be ignored, it is due to continuing the training in a different run.)</div>


# Discussion
_Coming soon_

<!-- 
- No work too well
    - More demos of actual grasp perhaps
    - Lots of noise in results, maybe due to noise in loss

Overall this naive approach was clearly 

- Future work
    - Try ACT/BeT -->

# Appendix
A lot of effort was expended trying to get the model to work on the randomised task. Here are some of the things I tried.

### Dataset size
I tried doubling the dataset from 50 to 100 randomised trajectories, and expected that the model would have a much easier time generalising with all this extra data. Similarly, I also tried halving the dataset to 25. I also doulbed/halved the number of epochs of training so that they are compared at the same step length, and the larger datasets do not get the unfair advantage of extra training.  For completeness, I then took the largest dataset (100) and trained it to 40 epochs so we cold also compare them at the same epoch.
Alas, the results were quite inconsistent and counterintuitive.
<div style="text-align: center;">
    <img src="{{ page.assets }}/dataset_size_results.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> Comparing results from different size datasets of randomised trajectories</div>

For the random tasks, the results are all very low and with high variance, so there's probably not much meaning in them. For some reason, the smallest dataset did far and away the best on the fixed task. To me this doesn't make a lot of sense - these models are trained on randomised trajectories, so I would expect the larger dataset to be able to adapt to the fixed task better. Perhaps the small dataset got lucky with demonstrations that were somewhat similar to the fixed task, so it was able to transfer easily. Whereas, models trained on the larger dataset are just unable to learn the distribution well, thus fail at everything. This got me thinking that perhaps a larger network was needed...


### Network architecture
In the case that learning was limited by the network size, I tried adding up to two more hidden layers i.e. from 64x64x64 to 64^4 and 64^5. I then trained them on the dataset of 50 randomised trajectories, and compares the results on the random and the fixed task, at 20 and 40 epochs.
<div style="text-align: center;">
    <img src="{{ page.assets }}/network_arch_random_results.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> Comparing results from different size datasets of randomised trajectories</div>
There was no clear result here, results were poor across the board.

### All in
Just to give the randomised task the best chance, I put together the largest dataset (100), the largest model (64^5) and trained for 60 epochs. The policy evaluated across every 10 epochs is shown here, on the random and the fixed task:
<div style="text-align: center;">
    <img src="{{ page.assets }}/all_in_results.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> The combination of largest network, largest dataset across its epochs</div>

All rewards are very low and with high variance, so it seems nothing is really being learned.

<div style="text-align: center;">
    <img src="{{ page.assets }}/all_in_loss.png" style="width: 100%">
</div>
<div style="text-align: center;" class="caption"> The loss curve for the all in experiment</div>

Here are some samples:
<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{ page.assets }}all_in/all_in_64_5_epoch_10_random_init.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">10 epochs</div>
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{ page.assets }}/all_in/all_in_64_5_epoch_20_random_init.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">20 epochs</div>
    </div>
</div>
<div style="display: flex">
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{ page.assets }}/all_in/all_in_64_5_epoch_40_random_init.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">40 epochs</div>
    </div>
    <div style="margin: 2px;text-align: center;">
        <video autoplay loop controls playsinline>
        <source src="{{ page.assets }}/all_in/all_in_64_5_epoch_60_random_init.mp4" type="video/mp4">
        Your browser does not support the video tag.
        </video>
        <div class="caption">60 epochs</div>
    </div>
</div>