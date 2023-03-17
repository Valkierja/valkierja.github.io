---
title: Move Player
poc: true
categories:
  - - GamePlay
tags: []
id: '177'
date: 2021-07-03 16:28:09
---

## install input system

windows->package manager->input system->install

there should be a warning thrown, noticing that a pack will install

choose YES

## add input system component

click player

in player's inspector ->add component->player input->create action

then select a position,such as /asset/Input Action to restore the action set

after you create it you need to choose the new action set you've just created in player input-> action

now WADS is bound to Vector2(a vector that usually is applied to force function ) changing action function by default

but now the player is still not movable,until you write a C# script

## write your first C# script

click player,then

in inspector->add component->add new script to create new C# script

then go to edit->preference->External Tools to choose your favorite editor

to me ,it's Rider editor

open your new script with Rider ,you will see there has been some function:

```
    void Start()
    {
    }
    private void Update()
    {
​
    }
```

the Start() function will be called when game's first frame is rendered,and never be called excepting you actively call it

the Update function will be called once per frame**(before the next frame is rendered)**

and we will use another important function fixedUpdate() instead of Update()

we will see why soon,for now we just need to delete Update(),and write FixedUpdate() instead

OK let's get down

first we need to get playerObject's info when game start, the info like regidbody

```
private Rigidbody myBody
void Start()
{
    myBody = GetComponent<Rigidbody>();
}
```

then,the OnMove function will read key every time player tap key and store the taping info to a Vector2

```
void OnMove(InputValue inputEvent)  
{
    Vector2 movementVector = inputEvent.Get<Vector2>();
    axisX = movementVector.x;
    axisY = movementVector.y;
}
```

then ,fixedUpdate will apply the vector to force

```
private void FixedUpdate()
{
    Vector3 movement3 = new Vector3(axisX, 0.0f, axisY);
    myBody.AddForce(movement3);
}
```

now you might notice that we keep axisZ==0.0(float),there are few points need to be noticed

1.  in vector3 the member var Matrix is (X,Z,Y) ,not (X,Y,Z)
2.  type of arg of AddForce is vector3 ,not vector2;we need to new one vector3
3.  FixedUpdate() will be called every time game need calculate physic system behavior ,noting that physic system calculation is not related to frame Rendering

![](https://www.ksroido.art/wp-content/uploads/2021/07/image-1.png)

ok,save your script back to unity

click"play",now you see the ball is moving!!!

but there is a problem : the speed is too slow to improve player's experience

## change ball-speed and make speed can be changed in unity,not in script editor

add var speed;

```
public float speed = 0;
​
​
private void FixedUpdate()
{
    Vector3 movement3 = new Vector3(axisX, 0.0f, axisY);
    myBody.AddForce(movement3 * speed);
}
```

public var can be changed in unity Menu

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172057643.png)

keep increase and test it until speed is fast enough to improve player's experience