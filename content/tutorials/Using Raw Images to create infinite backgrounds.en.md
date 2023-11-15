---
date: "2023-01-29"
draft: false
title: Using Raw Images to create infinite backgrounds
ShowCanonicalLink: true
canonicalURL: https://gerardgascon.medium.com/draw-a-2d-silhouette-in-unity-1918a9fbd4d1

cover:
    alt: Cover
    image: img/covers/raw-image-background.png
tags: ["Unity", "Shaders"]
categories: ["Tutorial"]
---

Creating a game with a large background can cause some large messes like this one:

![Untitled](/img/tutorials/raw-image-background/Untitled.png#center)

A huge background that was made out of dozens of images. But after some workaround, I ended up discovering that you can replace that with a Raw Image and a simple script.

---

## ****Setup****

To start, we need to create a specific canvas for the Raw Image and set its **Render Mode to Camera**. Then we create a Raw Image child and set it to stretch.
{{< figure src="/img/tutorials/raw-image-background/Untitled%201.png#center" >}}

## ****Main Script****

After creating the Raw Image, we’ll add the script. And write the following code:

```csharp
using UnityEngine;
using UnityEngine.UI;

[RequireComponent(typeof(RawImage))]
public class Scrolling : MonoBehaviour {

    RawImage _rawImage;
    Camera _main;
    Vector2 _position, _size;

    void Awake() {
        _main = Camera.main;
        _rawImage = GetComponent<RawImage>();
        UpdateCamera();
    }

    void LateUpdate(){
        float height = 2 * _main.orthographicSize;
        float width = height * _main.aspect;

        _size = new Vector2(_main.aspect * _main.orthographicSize, _main.orthographicSize);

        _position = new Vector2(_main.transform.position.x / width * _size.x - _size.x / 2f,
        _main.transform.position.y / height * _size.y - _size.y / 2f);

        _rawImage.uvRect = new Rect(_position, _size);
    }
}
```

The idea of this script is to **modify the UV rect** of the Raw Image, but to do so we first need to know the real values of height and width. These are calculated with the orthographic size and the aspect of the camera. After that, the offset of the camera is calculated from the position of the camera and subtracting to it a portion of its size.

We could introduce some scale and parallax variables, and multiply them by the components of the UV rect, but for the sake of simplicity I didn’t do that.

## Background Shader

For this specific project, I wanted to create a color change for different levels. In this case, I created the following shader:

![Untitled](/img/tutorials/raw-image-background/Untitled%202.png#center)

This shader is composed by a Texture2D that **needs to have _MainTex as a reference** in order to be read from the Raw Image. Then I take the red and green channels to replace them for some alternative colors that change in each level.

---

## ****Final Result****

After having done that, we get the following result:

![Gif](/img/tutorials/raw-image-background/Gif.gif#center)

Thank you for reading and, as always, hope this helps you in your projects.
