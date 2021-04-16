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
---

### Summary
In [this](http://www.pyoudeyer.com/ims.pdf) paper, the author provides one mechanism of Intrinsic Motivation which evaluates the progress of learning by comparing similar situations. Situation categorization, dividing situations in different regions, is one unique contribution of the paper. They call the system _Intelligent Adaptive Curiosity_ (IAC).

**IAC:**

It has a memory to store all the experiences (in form of Vector exemplars) that the agent faces. When number of experiences reaches a threashold (e.g. 250), the region is split into two sub-regions based on the threshold on features. Basically this splitting works like a decision tree boundary. 
