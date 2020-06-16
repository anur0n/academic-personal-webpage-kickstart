---
title: "Paper Summary : DEEP NEUROETHOLOGY OF A VIRTUAL RODENT"
summary: 'Summary about the paper: DEEP NEUROETHOLOGY OF A VIRTUAL RODENT'
date: 2020-06-16T17:57:38-05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
---

### Summary
This paper of a [Virtual Rodent](https://openreview.net/pdf?id=SyxrxR4KPS) explores the neural activity for motor control part of a Virtual Rodent based on simulated environment. The Rodent is designed based on the real rodent measurements. It has 38 controllable Degree of Freedom (**DoF**), and has access to Proprioceptive information like angle joints, angular velocity etc, and raw egocentric RGB camera input.

In the paper they test the motor neural activity for four different activities which requires coordinated motor activity.


{{< figure src="rodent_tasks.png" title="Fig: Visualizations of four tasks the virtual rodent was trained to solve: (A) jumping over gaps, (B) foraging in a maze, (C) escaping from a hilly region, and (D) touching a ball twice with a forepaw with a precise timing interval between touches." align="center" >}}

Their system architecture of the single network for all the tasks are as below-

{{< figure src="rodent_architecture.png" title="_Fig: "Egocentric visual image inputs are encoded into features via a small residual network and proprioceptive state observations are encoded via a small multi-layer perceptron. The features are passed into a recurrent LSTM module The core module is trained by backpropagation during training of the value function. The outputs of the core are also passed as features to the policy module (with the dashed arrow indicating no backpropagation along this path during training) along with shortcut paths from the proprioceptive observations as well as encoded features. The policy module consists of one or more stacked LSTMs (with or without skip connections) which then produce the actions via a stochastic policy."_" align="center" >}}

Their analysis shows neural activities:

{{< figure src="rodent_analysis.png" title="_Fig: "(A) Example jumping sequence in gaps run task with a representative subset of recorded behavioral features. Dashed lines denote the time of the corresponding frames (top). (B) tSNE embedding of 60 behavioral features describing the pose and kinematics of the virtual rodent allows identification of rodent behaviors. Points are colored by hand-labeling of behavioral clusters identified by watershed clustering. (C) The first two principal components of different behavioral features reveals that behaviors are more shared across tasks at short, 5-25 Hz timescales (fast kinematics), but no longer 0.3-5 Hz timescales (slow kinematics)."_" align="center" >}}

video examples of a single policy solving episodes of each task: [gaps](https://youtu.be/rFelC_YbeLE), [forage](https://youtu.be/vBIV1qJpJK8), [escape](https://youtu.be/6d0SX56Cn6Q), and [two-tap](https://youtu.be/lBKwHzO-z_0).

### Good Thing
They use neuroscience techniques to understand neural net activities.

### For our use
We may use their approach to provide proprioceptor observations. Also, we their joint controlling mechanisms might be useful for our project.



Another blog about this paper can be found [here](https://spectrum.ieee.org/tech-talk/artificial-intelligence/machine-learning/ai-powered-rat-valuable-new-tool-neuroscience)

### Future Readings
 - [Deep residual learning for image recognition.](https://arxiv.org/pdf/1512.03385.pdf)
 - [A task-optimized neural network replicates human auditory behavior, predicts brain responses, and reveals a cortical processing hierarchy](https://www.cell.com/neuron/pdfExtended/S0896-6273(18)30250-2)
 - [t-SNE representation](https://towardsdatascience.com/visualising-high-dimensional-datasets-using-pca-and-t-sne-in-python-8ef87e7915b)
