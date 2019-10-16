---
title: 'Obstacle Tower Challenge'
subtitle: 'A short summary on the new AI agent benchmarking environment.'
summary: A short summary on the new AI agent benchmarking environment.
authors:
- admin
tags:
# - Academic
categories:
# - Demo
date: "2019-10-01T00:00:00Z"
lastmod: "2019-10-01T00:00:00Z"
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Placement options: 1 = Full column width, 2 = Out-set, 3 = Screen-width
# Focal point options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
# image:
#  placement: 2
#  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
#  focal_point: ""
#  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
\justify
AI researchers have been using different game environments for a long time. Video games requires more frequent decision making, often in real time. So, from board games like checker and chess to Atari 2600 based Arcade Learning Environments (ALE) and doom based VizDoom have helped in the development and testing of AI Agents’ performance. But these have their own limitation and limited capabilities for developing generalized AI Agents. Like Atari has deterministic game levels, VizDoom has primitive graphics and limited randomization. Obstacle Tower Challenge is a new benchmark environment is specifically developed to overcome the limitations of previous game-based AI benchmarks, offering a broad and deep challenge. It provides following major features:



{{< figure src="/img/posts/obstacle-tower-challenge/obstacle-environment.png" title="Figure 1: Obstacle Tower at different floor levels." >}}

-	**High Visual Fidelity:** Provides 3D environment with real-time lighting and shadows and much detailed textures

 
 {{< figure src="/img/posts/obstacle-tower-challenge/obstacle-floor-plan.png" title="Figure 2: Obstacle Tower floor plans." >}}
 
-	**Procedurally generated rooms and floors:** Floors and rooms are procedurally generated, which requires generalization of agent to perform well.
-	**Physics Driven Interactions:** Interaction between Agent and other environment objects are controlled by real time 3D physics System.
-	**Procedurally Generated Visuals:** The visuals like textures, lighting, object geometry are different for different floors as well as in same floor in different times. Which also requires Agents to be generalized.

## Obstacle Tower Environment
Its environment is generated procedurally at multiple levels of interaction. 

### Episode Dynamics:
It consists of up to 100 floors and starting floor is zero. All floors of the environment are treated as a single finite episode. Each floor contains at the least a starting and ending room. Each room can contain a puzzle to solve, enemies to defeat, obstacles to evade, or a key to open a locked door. The layout of the floors and the contents of the rooms within each floor becomes more complex at higher floors. Within an episode, agent can only go to higher floors, and not to return to lower floors.
The episode terminates when the agent collides with a hazard such as a pit or enemy, when the timer runs out, or when the agent arrives at the top floor of the environment. The timer is set at the beginning of the episode, and completing floors as well as collecting time orbs increase the time left to the agent. In this way a successful agent must learn a behavior which is a trade off between collecting orbs and quickly completing floors of the tower in order to arrive at the higher floors before the timer ends.

It provides an image of the environment of size 168x168 RGB pixels and some other non-visual environment states like no of keys in possession of the Agent, the remaining time.

It supports 54 possible actions with combination of forward/backward/no-op movement, left/right/no-op movement, clockwise/counterclockwise rotation of the camera/no-op, and no-op/jump.

It can be configured to give Sparse Reward (+1 for completing a floor) or Dense Reward (+0.1 is provided for opening doors, solving puzzles, or picking up keys)

### Procedural Generation of Floors:
Each floor of the tower is procedurally generated with different lighting, textures, room layout and floor plan. This ensures that agents must have learned general purpose representations of the task at the levels of vision, low-level control, and high-level planning.

Each floor can have different appearance with one of 5 themes (Ancient, Moorish, Industrial, Modern, and Future) and different lighting conditions.

Floor layout is generated in two parts: mission graph and layout grid. Mission graph is generated using graph grammar technique and this graph is then transformed into a 2D layout grid. This is used for virtual scene generation.

For Room layout generation a template based system is used. Different room types, such as Puzzle or Key have their own set of templates from which specific room configurations are drawn. In Puzzle room, agent must push a block from a starting location to a goal location, while avoiding intermediate obstacles. In the Key rooms, there is a key somewhere in the room, along with potential obstacles.


## Evaluation Criteria:
It provides three possible evaluation schemes:
-	 No Generalization : Evaluate the performance of the agent on a single, fixed version of the Obstacle Tower.
-	Weak Generalization: Agents should be trained on a fixed set of 100 seeds for the environment configurations. They should then be tested on a held-out set of five randomly selected tower configuration seeds not in the training set.
-	Strong Generalization: In addition to the requirements for weak generalization, agents should be tested on a held-out visual theme which is separate from the ones on which it was trained.

Through all these environmental dynamicity Obstacle Tower challenge provides a meaningful challenge to current and future AI agents. This environment provides four axes of challenge for AI agents: vision, control, planning, and generalization. This benchmark combines all such axes of complexity.

