---
title: "Paper Summary - The Winograd Schema Challenge"
summary: 'Summary about the paper on Winograd challenge by - Hector J. Levesque et. al'
date: 2021-02-12T13:33:00+05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Paper Reading
draft: false
math: false
---

### Summary
This article summarizes the titled paper on [The Winograd Schema Challenge](https://cs.nyu.edu/faculty/davise/papers/WSKR2012.pdf)

This paper proposes an alternative to turing test, but it follows Turing's idea to judge only the behaviour as measure of intelligence. One drawback of the Turing test is that the final judgement for the same set of questions and answers will vary for different judges. They tried to avoid such a case. This challenge will provide quantitive measure of the 'Intelligence'.

This challenge is, as they claim, a variant of Recognizing Textual Entailment (RTE). The RTE provides set of questions which can be answered with yes/no only. This test asks whether a given sentence(A) 'Entails' the other given sentence (B). But in RTE the assumption about 'Entailment' was not proper. So, in winograd challenge the propose to generate questions using following 4 properties-

 -  _Two parties are mentioned in a sentence by noun phrases. They can be two males, two females, two inanimate objects or two groups of people or objects._
 - _A pronoun or possessive adjective is used in the sentence in reference to one of the parties, but is also of the right sort for the second party. In the case of males, it is “he/him/his”; for females, it is “she/her/her” for inanimate object it is “it/it/its,” and for groups it is “they/them/their.”_
 - _The question involves determining the referent of the pronoun or possessive adjective. Answer 0 is always the first party mentioned in the sentence (but repeated from the sentence for clarity), and Answer 1 is the second party._
 - _There is a word (called the special word) that appears in the sentence and possibly the question. When it is replaced by another word (called the alternate word), every- thing still makes perfect sense, but the answer changes._

The questions will have only 1 answer out of 2 given options. For Example- 
 - The trophy doesn’t fit in the brown suitcase because it’s too big. What is too big?
        Answer 0: the trophy 
        Answer 1: the suitcase
 - Joan made sure to thank Susan for all the help she had given. Who had given the help?
    - Answer 0: Joan 
    - Answer 1: Susan

To avoid using statistical learning they used the 4th rule of using some special word and alternative word. If the alrernative word is used, the answer will change totally. Like the following example-
- The trophy doesn’t fit in the brown suitcase because it’s too ⟨ ⟩. What is too ⟨ ⟩?
    - Answer 0: the trophy
    - Answer 1: the suitcase
    ----
    - special: big
    - alternate: small
    
**Good things:** This test doesn't require human judges. And it can be designed in a way that requires Knowledge and commonsense reasoning. They tried to ensure, that without reasoning, only statistical learnings are not enough

**Limitation:** Even if this test is passed, the passing agent still can't do everything like human.

