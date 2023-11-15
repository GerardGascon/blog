---
date: "2023-01-28"
draft: false
title: Cinemachine Scaler Extension
ShowCanonicalLink: true
canonicalURL: https://gerardgascon.medium.com/cinemachine-scaler-extension-c751797652c4

cover:
    alt: Cover
    image: img/covers/cinemachine-scaler.png
tags: ["Unity"]
categories: ["Tutorial"]
---

Today, I’ve developed another basic tool that could be useful to someone else. It is called the Cinemachine Scaler, an extension that changes the orthographic size to avoid hiding important sprites with the UI.

While testing some dialogues with Cinemachine, we’ve found that the Target Group made some characters appear under the dialogue’s UI. While this could be solved by increasing the radius of the character that is being occluded, it isn’t the most elegant solution. So then I developed this.

---

## ****Setup****

To start, we’ll need a script that will act as an extension for the Cinemachine Virtual Camera. In this case, I based my script on the Cinemachine Camera Offset that comes bundled with Cinemachine.

```csharp
using UnityEngine;
using Cinemachine;

[AddComponentMenu("")] // Hide in menu
[ExecuteAlways]
[SaveDuringPlay]
public class CinemachineScaler : CinemachineExtension {

    [Range(0f, 100f), SerializeField] float _percentage = 60f;
    [SerializeField] bool _showHelper = true;
    [SerializeField] bool _onTop;

    protected override void PostPipelineStageCallback(
        CinemachineVirtualCameraBase vcam,
        CinemachineCore.Stage stage, ref CameraState state, float deltaTime)
    {
        //Our scaler code will go here
    }

    void OnGUI() {
        if(Application.isPlaying || !_showHelper) return;

        //Draw a horizontal white line in the game view
        const float size = 3f;
        GUI.DrawTexture (new Rect (0, _percentage / 100f * Screen.height, Screen.width, size), Texture2D.whiteTexture);
    }
}
```

To start, the basic script will inherit the **CinemachineExtension** class, which requires the **PostPipelineStageCallback()** method, where our code will be placed.

In the **OnGUI()** method, we draw a horizontal line that will help us set the correct percentage of screen filled by the UI. This percentage works from top to bottom and with the boolean *top* we will define whether the UI is shown on the top or bottom of the horizontal line.

## ****Properties Structure****

To organize the things a bit, we create a separate script that will contain the properties structure:

```csharp
using UnityEngine;

public struct ScalerProperties {
    public Vector2 position;
    public float orthographicSize;

    public ScalerProperties(Vector2 position, float orthographicSize) {
        this.position = position;
        this.orthographicSize = orthographicSize;
    }

    public static ScalerProperties CalculateNewProperties(ScalerProperties previous, float percentageFilled, bool onTop = false) {
        float percentage = onTop
            ? Mathf.Max(0.001f, 1f - percentageFilled / 100f)
            : Mathf.Max(0.001f, percentageFilled / 100f);

        float newOrthographicSize = previous.orthographicSize / percentage;

        Vector2 newPosition = onTop
            ? previous.position + Vector2.up * (newOrthographicSize - previous.orthographicSize)
            : previous.position + Vector2.down * (newOrthographicSize - previous.orthographicSize);

        return new ScalerProperties(newPosition, newOrthographicSize);
    }
}
```

This simple structure only requires the position and orthographic size that the camera has. Inside the structure we’ll create a method called **CalculateNewProperties()** that will take the **previousProperties**, **percentageFilled** and **onTop** as parameters.

First, it ranges the percentage filled from 0.001 to 1 and inverts it in case it is on top. Then the new orthographic size is calculated from the previous one and the percentage. Finally, as the orthographic size is directly related to the height of the camera I can easily calculate the new position by getting the differences in size between the old and the new one.

## ****Implementation****

Finally, inside the **PostPipelineStageCallback** we have to write the following code:

```csharp
if (stage != CinemachineCore.Stage.Finalize) return;

ScalerProperties p = ScalerProperties.CalculateNewProperties(
    new ScalerProperties(state.CorrectedPosition, state.Lens.OrthographicSize),
    _percentage,
    _onTop
);
state.PositionCorrection = p.position - (Vector2)state.RawPosition;
state.Lens.OrthographicSize = p.orthographicSize;
```

This will first ensure that the code is only executed when the virtual camera has finished and right before sending its values to the **CinemachineBrain**.

The rest will just run the method from the **ScalerProperties** and apply those properties to the position correction and orthographic size.

![Untitled](/img/tutorials/cinemachine-scaler/Untitled.png#center)

After that, we can go to the editor and under the **Extensions section** we can select our script and set up the **Scaler** in there.

---

## ****Final Result****

{{< figure src="/img/tutorials/cinemachine-scaler/Untitled%201.png#center" title="Before the Cinemachine Scaler" >}}

I hope that this simple tutorial helps you solve a small detail in your project and makes it feel a bit more polished.

{{< figure src="/img/tutorials/cinemachine-scaler/Untitled%202.png#center" title="After the Cinemachine Scaler" >}}
