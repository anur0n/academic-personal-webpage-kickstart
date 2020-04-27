---
title: 'Controlling UnityML environment Scenes for different experiments with python'
# subtitle: ''
summary: This article discusses ways to control the UnityML game/environment and switch scenes from python script.

authors:
- admin

tags:
- UnityML
- Environment Control
# - Academic
categories:
# - Demo
date: "2020-04-27T10:00:00Z" 
# "2020-01-01T10:00:00Z"
lastmod: "`r format(Sys.time(), '%d %B %Y')`"
# "2020-01-01T10:00:00Z"
featured: true
draft: false

projects: []
---

Suppose, our environment has multiple scenes to support different experiments in the same environment. Based on different configurations we want to load different scenes. One simple approach to handle this requirement is to **Build** different executables for different scenes.

But I wanted to keep one single executable and control it programmatically from python script. So far, I couldn’t find a direct way to do this. Instead I used another API called ‘**Side Channels**’ provided by **UnityML**. Details about Side Channels can be found [here](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Python-API.md#communicating-additional-information-with-the-environment). 
Basically side channels provide a communication channel. When the simulation/training is running, we can pass data between unity environment and python script. We can also create custom channels if we need more control following [this](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Custom-SideChannels.md).

Let’s start with implementations in unity-

## Unity Side implementations
In unity, add a new script (*Right click -> Create -> C# Script*), ang give it a name. I name it _EnvironmentController_. Attach the script to any object of the environment, or add it to any empty object, so that the script is executed when the game/environment runs.

Let’s just first define an enum with the scene/experiments-

```c#
enum Experiment
{
    Main = 1,
    PaperRod = 2
}
```
Now we use the _Awake()_ method of the MonoBehaviour script to get access to the Float Properties channel using the **Academy** instance (a Singleton class) like below. We use the _Awake()_ method to fetch the scene before the script starts running.

```C#
private float scene;
public void Awake()
{
    var floatChannel = Academy.Instance.FloatProperties;
    scene = floatChannel.GetPropertyWithDefault("sceneToLoad", (float) Experiment.Main);
}
```

From the floatChannel we can retrieve the data that was passed through python API. This works like a key-value dictionary with a default value. We are retrieving the scene no. with ‘**sceneToLoad**’ key, and setting a default to our Main scene enum value, in case there is no value passed from python API.

Now that we have the scene no, we use the _Start()_ method to load the scene using Unity’s **SceneManager**. 

```C#
private void Start()
{
    if (scene == (float) Experiment.PaperRod && !SceneManager.GetActiveScene().name.Equals("Paper_Rod_Experiment"))
    {
        SceneManager.LoadScene("Paper_Rod_Experiment", LoadSceneMode.Single);
    }
    else if (scene == (float)Experiment.Main && !SceneManager.GetActiveScene().name.Equals("MainScene"))
    {
        SceneManager.LoadScene("MainScene", LoadSceneMode.Single);
    }
}
```

Here we are just checking if the current scene is not the one we want to change with **_SceneManager.GetActiveScene()_**. If it is a different scene we are using the **_SceneManager.LoadScene_** method to load the scene with the exact scene names.

That’s it. Easy way to change the scene from python API. Now, we just need to export the unity app. Be sure to include all your scenes when exporting through the build setting window _(File->Build Settings)_. To add scenes to the exported app/binary, add all the scenes (Scenes need to be added in the hierarchy window.

{{< figure src="/img/posts/unity-ml_scene_controlling_from_python/build_window.png" title="Add scenes before building" align="center" >}}

Then build the app binary.

## Python Side Implementation

Similar to Unity first we will define some enum for the scenes in python side also-

```python
import enum

class Experiments(enum.Enum):
    Main = 1
    PaperRod = 2
```

To pass the controlling data through python APIs and FloatPropertiesChannel we use 

```python
channel = FloatPropertiesChannel()
channel.set_property("sceneToLoad", float(Experiments.Main.value))
```

We initialize a **FloatPropertiesChannel** object. Then we set the scene value with key ‘**sceneToLoad**’ exactly the same as we used in unity side.

Now we will pass this channel in the environment creation step-

```python
env = UnityEnvironment(base_port = 5005, file_name=env_name, \
      side_channels = [engine_configuration_channel, channel])
```

We pass our channel as the side_channels argument. To learn about the details on how to start the environment with UnityML follow this documentations.

Here is the output of two enum (Main, PaperRod) values passed as FloatPropertiesChannel.


| {{< figure src="/img/posts/unity-ml_scene_controlling_from_python/output_main.png" width="516" height="516" >}} | {{< figure src="/img/posts/unity-ml_scene_controlling_from_python/output_paper_rod.png" width="516" height="516" >}} |
| ------------------| ------------------------------ |
| `hugo`            | Build your website.            |
| `hugo serve -w`   | View your website.             |

Hope this helps someone.
