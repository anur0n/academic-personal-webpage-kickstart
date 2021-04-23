---
title: "Paper Summary : Attention is all you need."
summary: 'Summary about the paper on Transformer Networks by Pierre-Ashish Vaswani et. al. from Google'
date: 2021-04-23T11:37:38-05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
math: true

---

### Summary
This [paper](https://arxiv.org/pdf/1706.03762.pdf) presented the Transformer Network based on Self attention instead of RNN or CNN, which is used build up NLP models like BERT or GPT.


**Transformer Model architecture:**
{{< figure src="model.png" align="center"  title="Figure 1: Transformer model Architecture">}}

The model consists of Encoder and Decoder module.

**Encoder:**
There are total 6 stack of identical layers. Every layer has a Multi-Head self-attention layer and a Feed-forward Layer. Each of these sub layers have residual connection (to tackle vanishing gradient problem). Each layer goes through layer normalization.

**Decoder:**
SImilar to decoder as seen in figure 1, but has one additional sub layer for masked attention layer. The masking makes sure the model doesn't see the future word positions.

{{< figure src="attention.png" align="center"  title="Figure 2: Scaled Dot-Product attention (Left) and Milti-Headed attention (Right)">}}

**Attention:**
It is a quantitative score for the releavance of each word with respect to other words in the sentence. Multi-head attention allows the model to jointly attend to information from different representation subspaces at different positions. It also enables parallel processing by removing sequential dependencies.

**Positional Encoding:**
Since there is no recurrence or convolution in the network, to inject the positional information they applied positional encoding to the input (Embedding of original input) with following formula-
$$ PE_{(pos,2i)} = sin(pos/1000^{2i/d_{model}}) $$
$$ PE_{(pos,2i+1)} = cos(pos/1000^{2i/d_{model}}) $$

**My thoughts:**
Since they are removing sequential dependencies this model takes the whole input in parallel which made if faster to train. But I'm not clear how it can handle sequential dependent applications like when the whole history is needed to be considered (like- time series) like mentioned [here](https://www.tensorflow.org/tutorials/text/transformer).
