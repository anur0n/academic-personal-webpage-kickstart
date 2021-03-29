---
title: "Paper Summary - SEDRo: Simulated Environment for Developmental Robotics"
summary: 'Summary about the paper on SEDRo- by Pothula et. al.'
date: 2021-03-26T22:00:00+05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
math: false
---

### Summary
This article summarizes the titled paper on [SImulated testbed for human level AI](https://arxiv.org/pdf/2009.01810.pdf)
This paper proposes a 3D Simulated environment for developing generalized intelligent agent. They suggest that the main reason behind non generalized learning agents are-
 - Targeting a single task rather than diverse task.
 - Use of refined and focused datasets rather than diverse and noisy datasets
 - Relying on explicit rewards rather than on other mechanisms 
 - Too many necessary components rather than a sufficient set of the learning mechanism.
 
 SEDRo provides 2 different environments.
 - Fetus: Simulates the womb
 - After birth: Simulates 0 to 12 months after birth.
 The learning agent is a human baby. It also provides a pre-programmed (To interact with the baby and provide the essential experience for human level AI) **Social partner**. 

 The proposed environment has 3 unique features:
 - _Open-ended tasks without extrinsic reward_: Provides no reward, no specific goal.
 - _Human-Like experience with social interaction_: Learning agent is given a human baby's environment. Agent model performance can be compared against human performance.
 - _Longitudinal development_: Unfolds agent capabilities(Motor strength, Vision, etc) according to a curriculum following milestones from developmental psychology.
 
 And finally SEDRo also provides a set of evaluation methods from developmental psychology research.
 
 **Concluding thoughts**:
 Overall the paper presents a novel idea. However how the essential experience will be chosen and developed remains a challenge.


