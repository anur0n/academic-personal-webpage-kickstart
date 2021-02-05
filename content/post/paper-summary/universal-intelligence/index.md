---
title: "Paper Summary - Universal Intelligence: A Definition of Machine Intelligence"
summary: 'Summary about the paper on Universal intelligence by - Shane Legg and Marcus Hutter'
date: 2021-02-05T11:53:00+05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
math: true
---

### Summary
This article summarizes the titled paper on [universal intelligence paper](https://arxiv.org/pdf/0712.3329.pdf)

One problem of working with Artificial Intelligence is that there is no universally accepted definition of the term ‘**_Intelligence_**’. As R. J. Sternberg quoted-

“_Viewed narrowly, there seem to be almost as many definitions of intelligence as there were experts asked to define it._”

This paper tried to address this issue. After reviewing many definitions of ‘_Intelligence_’ they came up with an informal definition first –

“_Intelligence measures an agent’s ability to achieve goals in a wide range of environments._”

This definition works well with the concept of reinforcement learning mechanism. From this informal definition they came up with a formal definition for machines. The formal definition is:

$$\Upsilon(\pi) := \sum_{\mu\epsilon E} 2^{-K(\mu)} V_\mu^\pi$$

This is **_Universal Intelligence_** of the agent $\pi$. Where $V_\mu^\pi$ is the expected reward for agent $\pi$ in the environment defined by $\mu$. $K(\mu)$ is the Kolmogorov complexity of the environment $\mu$. And $2^{-K(\mu)}$ is the algorithmic probability distribution over the space of the environments.

Basically, this equation performs weighted sums of the expected reward of an agent $\pi$, with weights being the inverse of the complexity of the environments. The more complex environments are less likely. The agent is evaluated over a wide range of environments and their achievements (rewards) are summed up. The more this value of universal intelligence the more intelligent the agent is.
In the paper, they also reviewed many human intelligent test as well as machine intelligence tests. They nicely summarized the comparison of the machine tests according the following properties-

 - **Valid**- Capture just intelligence and not some related quantity or only a part of intelligence. 
 - **Informative**- Result should be a scalar value, or perhaps a vector. 
 - **Wide range**- Cover very low levels of intelligence right up to superhuman intelligence. 
 - **General**- Applicable to everything from a fly to a machine learning algorithm. 
 - **Dynamic**- Take into account the ability to learn and adapt over time. 
 - **Unbiased**- Not be biased towards any particular culture, species, etc. 
 - **Fundamental**- Not need to be changed from time to time due to changing technology and knowledge. 
 - **Formal**- Specified with the highest degree of precision possible. 
 - **Objective**- Not dependent on subjective assessments such as the opinions of human judges. 
 - **Fully Defined**- Has the test/definition been fully defined, or are parts still unspecified? 
 - **Universal**- Is the test/definition universal, or is it anthropocentric? 
 - **Practical**- Can be performed quickly and automatically, while from a definition it should be possible to create an efficient test. 
 - **Test vs. Def**- Whether the proposal is of a test, more of a definition, or something in between.


{{< figure src="test_comparison.png" title="Table : Machine intelligence test comparisons: ⬤ (bigger circle) means “yes”, ● (smaller circle) means “debatable”, · (dot) means “no”, and '?' means unknown. " align="center" >}}


**Good things:** Overall, the authors nicely captured the major discussions about intelligence and gathered different tests and provided a formal definition or measurement scale for machine intelligence.

**Limitation:** One major limitation is their formal definition is not practical yet because, that the Kolmogorov complexity function K is not computable  

