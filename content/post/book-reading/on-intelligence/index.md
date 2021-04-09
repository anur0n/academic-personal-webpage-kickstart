---
title: "Book Review : On Intelligence"
summary: "A short review of chapters 3, 4, 5, 6 from the book by Jeff Hawkins"
date: 2021-04-09T12:01:38-05:00
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
authors:
- admin
tags:
- Book Reading
draft: false
featured: false
image:
    focal_point: ""
    placement: 1
    preview_only: true
---

### Summary

This book amazingly presents some top-down mechanisms on how the human brain works, how it is so intelligent.

**The Brain**

Chapter 3 discusses about the brain structure, the _neocortex_ mainly. Human neocortex is a six layer structure with billions of neurons connected in some hierarchical way. The connections probably sum up to trillions. Though the there is a hierarchy of layers, it is not literal hierarchy. Mostly based on the information flow direction. Like visual pattern signals reach first at V1 layer, then passed to V2 and so on. There are different functional regions in the brain that perform different functions like visual, auditory, touch etc. But if these regions are rewired to different sensory inputs, they should work fine. This suggests, _"the brain works in some universal cortical algorithm"_. There are also lateral connection between different areas of the hierarchy. So, this is more like a heterarchy. There are some association regions, where different sensory connections are combined. For example, in one layer, both vision and touch areas might be connected. This are called association area. The brain receive only patterns of signals. These patterns are stored memory.

{{< figure src="cortical-hierarchy.png" title="Figure: Cortical Hierarchy. This is not an accurate representation. It's more of a conceptual depiction" >}}

**Memory**

In chapter 4, the author discuss memory functionality. Neurons are very slow compared to current computers. Yet, it outperforms the super fast computers in many aspects. The author believes the memory is behind this speed of neurons. Brain doesn't compute, it recalls from the memory. Hence it is fast. The entire cortex is a memory system, not a computer. And the signals are stored with an invariant representation. This way, it can recognize a previously experienced signals even though each time the raw signal is different in many ways from the stored representations. There are 4 attributes of neocortical memory - 
- It stores sequence of patterns
- Recalls patterns auto-associatively
- stores patterns in an invariant form
- stores patterns in a hierarchy

**Prediction**

The final important concept is _Prediction_, which the book disses in chapter 5. The brain only see patterns, and based on that pattern and memory, it predicts what would be the next sensory inputs. Based on previous experiences from memory, it predicts the immediate future events/sensory patterns. If some prediction doesn't match with the next sensory inputs, it becomes surprised and puts focus/attention on new experience.

**My thoughts**

The book presents interesting concept of hierarchical (More like heterarchi) predictive memory. Although in some points author presents his 'Belief' without actual findings. These ideas should be explored to build Human Level AI.

Also, while reading about memory, I was thinking-

How the memory is stored in invariant forms? Is it like the concept like hash(Not exactly)?

How do we start thinking without sensory inputs, if everything is predictive based on sensory inputs?

If we want to follow the connection like brain, how can we implement various connection between regions (Although universal algorithm)
