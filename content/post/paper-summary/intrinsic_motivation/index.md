---
title: "Paper Summary : Intrinsic Motivation Systems for Autonomous Mental Development"
summary: 'Summary about the paper on Intrinsic Motivation by Pierre-Yves Oudeyer et. al.'
date: 2021-04-16T11:37:38-05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
math: true

---

### Summary
In [this](http://www.pyoudeyer.com/ims.pdf) paper, the author provides one mechanism of Intrinsic Motivation which evaluates the progress of learning by comparing similar situations. Situation categorization, dividing situations in different regions, is one unique contribution of the paper. They call the system _Intelligent Adaptive Curiosity_ (IAC).

**IAC:**

It has a memory to store all the experiences (in form of Vector exemplars) that the agent faces. When number of experiences reaches a threashold (e.g. 250), the region is split into two sub-regions based on the threshold on features. Basically this splitting works like a decision tree boundary. The features are sensory inputs (_S(t)_), and motor outputs (_M(t)_). Combining these two _SM(t)_ is used as sensorymotor context.

{{< figure src="feature_splitting.png" align="center" >}}

For each region this system assigns a **Local** expert _E_. It is responsible for prediction of next sensory inputs _S(t+1)_ for that local region R based on _SM(t)_. The decrease in mean error rate in prediction is used as **Learning Progress** as well as **Reward**. So, this works as intrinsic reward. This reward and S, M vectors can be used in Reinforcement Learning too.

The paper also presents two experiments with this system. One on virtual and one on Sony Aibo. These experiments were tested for small domain of tasks.

**My thoughts:**

The idea about dividing the input space into smaller sub sections seems interesting. Also, the part of using decrease of error is interesting. But assigning a local expert to predict only for that region seems limitation. What happens when there is multiple regions involved for a single action decision? Which expert will handle the bigger picture? Also for language or audio space, how will it work when temporal context is needed to understand current inputs?
