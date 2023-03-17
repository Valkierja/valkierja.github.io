---
title: Third-person view developing
poc: true
categories:
  - - GamePlay
tags: []
id: '190'
date: 2021-07-05 22:19:02
---

Normally,binding camera with Player into Third-person view equals to let camera be Player's child ,but in our case the Player moving way is rolling ,so the camera will roll with player ,which is not expected

## write a script to get and set camera offset relative to Player

declare GameObject instance to bind Player  
then declare a vector3 var to store offset

```
    public GameObject player;
    private Vector3 m_Offset;
```

at start function,store initial offset

```
    void Start()
    {
        m_Offset = transform.position - player.transform.position;
        // m_Offset = this.transform.position - player.transform.position;
    }
```

then after Player moved,calculate frame(relative to physical system)

```
  void FixedUpdate()
    {
        transform.position = player.transform.position + m_Offset;
    }
```