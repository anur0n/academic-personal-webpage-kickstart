---
title: "Paper Summary : World Model"
summary: 'Summary about the paper: World Model'
date: 2020-06-15T19:37:38-05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
---

### Summary
In the [**World Model** paper](https://arxiv.org/pdf/1803.10122.pdf), the authors proposed a generative neural network to tackle different RL environments. They divided the agent in two parts. First, a model is trained to capture the environment around the agent and create a model of the environment (World model). In second part, a controller model is trained to perform a task based on the world model. Both parts are trained separately.

{{< figure src="model_design.png" title="World Model design" align="center" >}}

The model comprises of a **Vision Model** (V: A VAE network) to compress visual observation to a smaller latent vector. Which is then passed to a **Memory model** (M: a RNN) to consider the current observation (z) and previous history (h) and predict the future z vector. The **Controller model** (C: a linear NN)  then uses that predicted next frame and choses an action (a). Here is the final flow diagram-

{{< figure src="model_flow.png" title="World Model flow" align="center" >}}

### Interesting
One interesting part about this paper is, they used the agent’s **_dream_** to train for an environment without the actual environment. Since the agent is predicting the future environment vector, it feeds the predicted frame data as the next input frame to the World Model network and simulates a virtual environment. This way the agent can be trained inside the ‘dream’ of the agent. The model was trained for the **vizdoom** in the dream and then connected to the actual Vizdoom environment. The agent was able to avoid the fires.

### Limitation
Some limitations of training in the dream is that sometimes the agent creates some state that is not possible in the actual environment. Like, in Vizdoom environment the agent sometimes extinguishes the fireballs which is not possible in the actual game. So, it performs well on the virtual environment but performs poorly on actual environment.

The interactive version of this paper can be found [here](https://worldmodels.github.io)

### Future Readings
 - [On Learning to think](https://arxiv.org/pdf/1511.09249.pdf)
 - [Brain’s prediction of the future based on our internal model of the world](https://academic.oup.com/cercor/article/25/6/1427/298681)
