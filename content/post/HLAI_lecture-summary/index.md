---
title: "Summary of the lecture of 'Towards Human Level AI (HLAI)'"
summary: 'This lecture was provided by Dr. Deokgun Park for HLAI class.'
date: 2021-05-03T11:37:38-05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
draft: false
---
In our last class of Human Level Artificial Intelligence (HLAI), Professor Dr. Park presented on 'Towards Human Level AI'. He discussed on how we can approach towards building human level intelligent robot. The talk was divided in three parts - The theory, Test, and Model.

### The theory
In this part, he discussed related terms and background on intelligence. To build an _'Intelligent'_ agent we first need to define or understand clearly, what is _'Intelligence'_. Based on the [Universal Intelligence](https://arxiv.org/pdf/0712.3329.pdf) - 

>Intelligence measures an agent’s ability to **achieve goals** in a **wide range of environments**.

He the discussed three levels of intelligence:
- **Level 1**: In this level there is no individual learning. It is refined by evolution. (Ex- Earthworm). The problem with this kind is that, evolution is too slow.
- **Level 2**: Animals in this level can learn from direct experience. And the learning can be refined based on rewards. (Ex-rats, dogs). The problem is they cannot pass their learning to next generation.
- **Level 3**: In this level, the learning is possible from indirect experience too. Like using language. (Ex. Human). This learning can also be shared with others with language.

He then presented formal definition of HLAI as:

>An agent has a human-level artificial intelligence if _there exists a symbolic description for every feasible sensory input and agent action sequence_, such that the agent _can update the behavior policy equally_ whether it receives the symbolic description or it experience the sensory input and the action sequence itself.

### The test (or task)
There has been some 'Sufficient' tests proposed for evaluating HLAI like turing test, robot college student test, kitchen test, AI preschool test. But these tests were very difficult. And there has been some eaier but 'not sufficient' tests like Winograd schema challenge, imagenet challenge.

Dr. Park suggested that language acquisition is a function of environment and capability of the agent. He then proposed the **Language Acquisition Test for HLAI** as-

>Given the proper **environment**, if the agent can learn **language**, we can say it has a **capability** for human-like artificial intelligence. 

{{< figure src="language_acq.png">}}

He then proposed a simulated environment [SEDRo](https://arxiv.org/abs/2009.01810) for training and evaluating learning agent. SEDRo has three key characteristics:

- Open-ended tasks with No reward: Because learning based on environment rewards is not transferable.
- Human-like experience: Because we don't know what are the sufficient conditions necessary for HLAI. Only human babies achieve human level intelligence. Also learning based on human like experience makes agent behavior more interpretable.
- Longitudinal development: Advanced skills require many combined basic skills. So learning in a timeline will ensure, the agent learns incrementally.

SEDRo also provides tests based on developmental psychology experiments.

### The Model:
He proposed modulated Heterarchical Prediction Memory (mHPM) model.

First, lets see hierarchical prediction memory model.

{{< figure src="model.png">}}

Higher layer provides context and lower layer provides summary. The autoregressor makes prediction based on the input from lower layer or sensory inputs and context from upper layer. The Pooler generates an abstract summary based on sequential inputs which is fed to the upper layer. When the lower layer learns from 4 cycles, the upper layer learns 1 cycle containing summary of 4 cycle lower layer summary. Each autoregressor and pooler in each layer learns and update locally.

He then suggested that we can explor heterarchical connection with this predictive memory to associate inputs/contexts from different regions. Like shown below-

{{< figure src="heterarchical_model.png">}}

Now for modulation the behavior is modulated based on some reward. As shown in picture below-

{{< figure src="modulation.png">}}

Then finally there needs to be some auxiliary module in the model-
- Instinct (Pre-built behavior): weight agnostic models can be explored.
- Reward Center (To guide behavior): Implemented like curiosity. No reward from environment.
- Hippocampus as episodic memory writer: A loopy structure to repeatedly predict


### Future experiment plan
Finally he shared the current status and future experiment plan summarized below.
{{< figure src="plan.png">}}
