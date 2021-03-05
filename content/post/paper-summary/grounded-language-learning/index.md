---
title: "Paper Summary - Grounded Language Learning in a Simulated 3D World"
summary: 'Summary about the paper on Grounded Language Learning by - Karl Moritz Hermann et. al from Deepmind'
date: 2021-02-19T13:33:00+23:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
math: false
---

### Summary

This [pape](https://arxiv.org/pdf/1706.06551.pdf) proposes a language grounding model based on a simulated 3D environment. It used an existing 3D environment from Deepmind, and proposed a model to learn language (word-object/color/size mapping).

**Simulated 3D World**
The agent can move around in the environment and receives visual input and texial instruction to follow like- _pick the pink striped ladder_. The environment consists of one or two rooms with different shaped/colored objects of varied size as show in following figure. Objects and rooms are configured programmatically with random sampling.

{{< figure src="environment.png" title="Fig: Top view of the room layout (left most) and agent views on different positions (Right 3 images). Black path is agents trajectory for the instruction _pick the red object next to the green object_" align="center" >}}

**Agent Model Design**
The agent model consists of four modules. A vision module (**V**), Language module (**L**), Mixing module (**M**), and Action module (**A**). It also has auxiliary unsupervised models consisting a Temporal autoencoding (**tAE**), Language prediction module (**LP**), Reward prediction module (**RP**) and a Value replay (**VR**) modules. The environment provide positive rewards for completing the instruction and negative reward for picking up wrong objects.

{{< figure src="model.png" title="Fig: Model design." align="center" >}}

**V** encodes the current visual input and **L** encodes instruction string. Mixing module **M** determines how these signals are combined before they are passed to two-layer LSTM action module **A**. The **tAE** module predics the nect visual environment based on prior visual input and choosen action. **LP** module estimates instruction words, given the visual observation.

**Experiments**
The paper showed many experiments and showed promising results.
{{< figure src="model combination.png" title="Fig: Results of different module combination. The agent with only reinforcement learning didn't show any learning behaviour. It performed best when all unsupervised auxiliary networs were combined." align="center" >}}

{{< figure src="curriculum learning.png" title="Fig: Results of curriculum learning. It showed, agent performs better with training on previous level." align="center" >}}
    
**Good things**
They combined reinforcement learning and unsupervised networks to achieve language grounding. The agent to some extend learnd the semantics and were able to generalize well on new words.

**Limitation**
The model still is dependent on the explicit rewards from the environment. Designing explicit rewards for more complex environment will be troublesome. Hence the model may not be applicable for human level AI.

Some demo of this work can be found [here](https://www.youtube.com/watch?v=wJjdu1bPJ04).

