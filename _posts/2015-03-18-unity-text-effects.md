---
layout: post
title: Unity Text Effects
date: 2015-03-18 06:58
author: ray
comments: true
categories: [Blog]
tags: [GUI, Text, Unity, Unity3D]
---
<strong>Gradient</strong>
<pre>using UnityEngine;
using System.Collections;
using System.Collections.Generic;
using UnityEngine.UI;

[AddComponentMenu("UI/Effects/Gradient")]
public class Gradient : BaseVertexEffect {
    [SerializeField]
    private Color32 topColor = Color.white;
    [SerializeField]
    private Color32 bottomColor = Color.black;

    public override void ModifyVertices(List vertexList) {
        if (!IsActive()) {
            return;
        }

        int count = vertexList.Count;
        float bottomY = vertexList[0].position.y;
        float topY = vertexList[0].position.y;

        for (int i = 1; i &lt; count; i++) {             float y = vertexList[i].position.y;             if (y &gt; topY) {
                topY = y;
            }
            else if (y &lt; bottomY) {
                bottomY = y;
            }
        }

        float uiElementHeight = topY - bottomY;

        for (int i = 0; i &lt; count; i++) {
            UIVertex uiVertex = vertexList[i];
            uiVertex.color = Color32.Lerp(bottomColor, topColor, (uiVertex.position.y - bottomY) / uiElementHeight);
            vertexList[i] = uiVertex;
        }
    }
}</pre>
