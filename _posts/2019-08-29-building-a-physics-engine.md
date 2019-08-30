---
layout: post
title: "Building a Physics Engine"
category: c++, physics
series: physics engine
tags:
    - c++
    - gamedev
    - physics
---

I've recently picked up the task of building a small physics engine from scratch. My objective in this project was mostly to learn about engine design, but also to practice good coding habits; e.g. I've implemented a git design system rather than just throwing everything into the master branch like some kind of animal. With that being said, I've picked a rather simple rendering library, SDL2, as to be able to focus more on the design and physics of the actual engine logic.  

## Design

Resources are managed by *managers*, which all carry an instance of the physics engine and through that can communicate to other managers. The actual physics engine is in charge of containing all of the managers, initialize and destroy them when appropriate, as well as holding the main game loop. Currently I have managers for text rendering, window logic and rendering, and physics logic and computations.  

An other focus on the design the ease of use of the physics engine API. All functions that should be accessed by the user is kept in a separate namespace and only one file is required to be imported to have the physics engine running. This API namespace gives the user the ability to easily configure physics parameters (gravity, drag coefficient, etc...), window settings, and other general settings. Furthermore, users can play with physics settings and object properties at run-time through JSON files.

## The importance of numbers

One of the first challenges I've faced was my carelessness with numbers. Early on in the development I've decided to represent internally my positions and velocities as `double`. As such, after all computations are done and store, when it came to the rendering step, there was an implicit conversion from `double` to `int`, since points on the screen can of course not be `double`. This does not seem to be too much of a problem, however, unbeknown to me, implicit conversion actually rounds to *negative infinite*, rather than to *zero* as I've naively assumed. This means a number such as `8.4` would round to `8`, but the value `-8.4` would round to `-9`! This is a complete disaster. This meant that the object is essentially going up faster than it is going down (origin is at the top left). Thus we get the perceived effect of a lack of conservation of energy, as seen on the left gif below. This is particularly annoying, since certain numerical schemes can in fact not preserve energy (e.g. Euler's method) or a lack of conservation may be due to a poor implementation of an otherwise stable algorithm. Thus this subtle and easy to fix bug can lead to hours of wasted time analyzing one's implementation of their numerical scheme. The fix is of course simple, one simply has to explicitly cast all values. Explicit casting does round to zero and thus the problem is avoided. In general it is good practice to explicitly cast any values, even if we expect nothing terrible to happen.  

<img src="https://media.giphy.com/media/QVJgOgbOQ4xNBe9pLU/giphy.gif" width="85%" height="85%" />

<img src="https://media.giphy.com/media/YMMl56u9wKVgovdfKU/giphy.gif" width="85%" height="85%">

In general I've attempted to make the physics engine as realistic as possible. However to sometimes smooth out the animations, particularly when particles arrive at a rest state, odd animations glitches would occur where the static and dynamic forces of pushing on each other would make the ball un-physically vibrate forever. As such, I've implemented this "cheat" where at low velocities, the particles' velocity would round down to zero, allow particles to "snap" into position and achieve a proper rest state, as seen in the gif below.

<img src="https://media.giphy.com/media/L12i6JZ6NVCdQcxC27/giphy.gif" />

## Optimization

It is important to early on in the development of a game engine to optimize all aspects of your software. This can be both increasing compile time computations to relieve some run-time computations as well as implementing better algorithms rather than the easy and naive ones. For example, consider collision detection. The naive approach would be to check if every particle has collided with an other particle. This means we have a quadratic complexity. Can we do better? Yes.  

The main problem with this naive approach is that particles that are far away are never going to collide, and we know that. Yet we still spend resources checking every single possible combination. As such, what if we limited the actual region of the collision checking to an area around the target particle? Indeed this would mean we only check actual possible collision candidates. We can do this by recursively splitting the screen area into smaller areas which are individually checked. The granularity of the areas would be proportional to the total number of particles. This is commonly referred to as *quadtree collision detection*. A neat animation of the process can be found [here](http://www.mikechambers.com/blog/2011/03/21/javascript-quadtree-implementation/). 

## Future

I've been working on the project on and off for about a month now. Future work will include N-body simulations as well as more shapes such as triangles and rectangles. Even more future work may include attempts at fluid and electrodynamic simulations as well as destructible objects.  
