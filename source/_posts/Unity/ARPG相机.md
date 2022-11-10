---
title: ARPG相机
tags: Unity
abbrlink: 34a2bdf1
date: 2022-11-10 16:35:56
---

ARPG相机

```C#
using UnityEngine;
using System.Collections;

/*****************摄像机*************/

public class ARPGcamera : MonoBehaviour
{

    public Transform target;
    //public string targetname = "zhujue(Clone)";
    public float targetHeight = 1.2f;
    public float distance = 4.0f;
    public float maxDistance = 6;
    public float minDistance = 1.0f;
    public float xSpeed = 250.0f;
    public float ySpeed = 120.0f;
    public float yMinLimit = -10;
    public float yMaxLimit = 70;
    public float zoomRate = 80;
    public float rotationDampening = 3.0f;
    private float x = 20.0f;
    private float y = 0.0f;
    public Quaternion aim;
    public float aimAngle = 8;


    void Start()
    {
        var angles = transform.eulerAngles;
        x = angles.y;
        y = angles.x;
    }

    void Update()
    {
        if (Input.GetMouseButton(1))
        {
            x += Input.GetAxis("Mouse X") * xSpeed * 0.02f;
            y -= Input.GetAxis("Mouse Y") * ySpeed * 0.02f;
        }
        distance -= (Input.GetAxis("Mouse ScrollWheel") * Time.deltaTime) * zoomRate * Mathf.Abs(distance);
        distance = Mathf.Clamp(distance, minDistance, maxDistance);
        y = ClampAngle(y, yMinLimit, yMaxLimit);
        Quaternion rotation = Quaternion.Euler(y, x, 0);
        transform.rotation = rotation;
        aim = Quaternion.Euler(y - aimAngle, x, 0);

        Vector3 position = target.position - (rotation * Vector3.forward * distance + new Vector3(0, -targetHeight, 0));
        transform.position = position;
    }

    static float ClampAngle(float angle, float min, float max)
    {
        if (angle < -360)
            angle += 360;
        if (angle > 360)
            angle -= 360;
        return Mathf.Clamp(angle, min, max);
    }
}
```

