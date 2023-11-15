---
date: "2023-01-26"
draft: false
title: Draw a 2D silhouette in Unity
ShowCanonicalLink: true
canonicalURL: https://gerardgascon.medium.com/using-raw-images-to-create-infinite-backgrounds-67f2bb34c295

cover:
    alt: Cover
    image: img/covers/draw-2d-silhouette.png
tags: ["Unity", "Shaders"]
categories: ["Tutorial"]
---

I am developing a game, and recently it required to have silhouettes for when the player is behind an obstacle. To my surprise, I found no tutorials covering this at all. Even though there exist some, they only cover a small part of what any top-down 2D game would need. So after some trial and error I found a way, with some room for improvement, that solves those cases found in other tutorials.

---

## ****Setup****

Before starting with the silhouette, we need a basic sprite sorting system, we could use a [SortingGroup](https://docs.unity3d.com/Manual/class-SortingGroup.html) along with the Transparency Sort Mode. While experimenting with this, I stumbled across some problems with the masking and ended up creating my own sorting based on the default **Order In Layer**.

To make this, I use the y position of the sprite as a reference to change the Order In Layer:

```csharp
using UnityEngine;

[RequireComponent(typeof(SpriteRenderer))]
public class SpriteZRenderer : MonoBehaviour {

    [SerializeField, Min(0)] int precision = 50;
    [SerializeField] int sortingOffset;

    SpriteRenderer renderer;
    void Awake() => renderer = GetComponent<SpriteRenderer>();
    void Update() => renderer.sortingOrder = (int)(transform.position.y * -precision) + sortingOffset;
}
```

Please note that SpriteRenderer’s Order In Layer is a **16-bit signed integer**, therefore you **can only have 65,536 different values**, the more precision you give to this script, the less range of vertical movement you have. The precision means how many layers you will have for each unit. In this example, I use a precision of 50, so I could go up to around 655 units before overflowing.

After that, all the sprites that use this system will need to have the pivot set on their base position, like in this example:

{{< figure src="/img/tutorials/draw-2d-silhouette/Untitled.png#center" title="Example of character having the pivot on its feet" >}}

![Gif](/img/tutorials/draw-2d-silhouette/Gif1.gif#center)

After adding this script to each sprite that we want it to sort, we can now create the mask that will draw the silhouette.

## ****Silhouette Shader****

To add a color to the silhouette, I could use a sprite with the shape of the object, and apply any color I want to that, but a shader is a bit more interesting and it still doesn’t limit us in that part. For testing, I use this basic shader:

![Untitled](/img/tutorials/draw-2d-silhouette/Untitled%201.png#center)

To make this work I use a Sprite Graph from the URP and add two exposed properties, the Texture2D, whose reference must be **_MainTex** in order to work with the SpriteRenderer and the Color, whose reference must be **_VertexColor**.

After doing that, we need to create a material and assign it to a child copy of the obstacle that will have the silhouette projected on and set the Mask Interaction to **Visible Inside Mask**. Also, these children will need to have the **SpriteZRenderer’s Sorting Offset set to 1**, in order to make the silhouette always appear on top of the obstacle.

![Untitled](/img/tutorials/draw-2d-silhouette/Untitled%202.png#center)

Note that my script’s inspector is slightly different, but the functionality is the exact same as the snippet I’ve shown before.

## ****Silhouette Sprite Mask****

Finally, we have to create a Sprite Mask that will make the silhouette visible when behind an object.

![Untitled](/img/tutorials/draw-2d-silhouette/Untitled%203.png#center)

This mask will be present in any movable object that will have its silhouette projected onto the obstacles. After that, we will end up with a result like this one:

![Gif](/img/tutorials/draw-2d-silhouette/Gif2.gif#center)

---

I hope this small tutorial helped someone that, like me, was overengineering this simple effect and could find a simple solution to that.

{{< figure src="/img/tutorials/draw-2d-silhouette/Untitled%204.png#center" title="Screenshot of the final result" >}}
