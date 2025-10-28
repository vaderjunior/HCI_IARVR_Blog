+++
date = '2025-10-27T17:33:36+01:00'
draft = false
title = 'My First Post'
+++

**Day 1: Setting up Unity, Meta tools, and starting the Roll-a-Ball game**

Phew — today was long, but honestly, pretty exciting.

I started by factory-resetting the headset because it was still logged into a previous account. I wanted to use my own Meta account, so that was necessary. After setting it up, I played around with it a bit (and so did my friends). My roommate ended up bowling in VR — I think we’re all easily impressed 😄

Then I switched into **Unity mode**. I’ve never worked with Unity or C# before, so everything is new. Before I dive into that though — **I set up this Hugo website yesterday!** I originally tried using a fancy theme, but the dependencies were melting my brain, so I gracefully gave up and went with the **Paper** theme that Yara recommended. Clean, simple, works. Perfect.

{{< figure src="website1.png" alt="First steps with the website" >}}

---

### 🕹️ Roll-a-Ball

The Roll-a-Ball project was provided on Moodle, but I wanted to *really* understand what was happening rather than drag-and-drop. So I followed the official Unity tutorial:

https://learn.unity.com/project/roll-a-ball

I worked through the part where you set up the ground and ball:

{{< video src="rollaball1.mp4" >}}

The tutorial is beginner-friendly and well explained. But…

---

### ❗ The great “Why is the ball not moving?” mystery

When I pressed Play, **the ball refused to move.**  
No errors. Just stubborn stillness.

So naturally I:

1. Adjusted speed values  
2. Re-imported Input Manager  
3. Googled way too many things  
4. Checked Reddit & StackOverflow  
5. Considered rewriting physics myself (joking… mostly)

After **25 minutes**, I finally noticed:

I **wasn’t pressing the arrow keys** to control the ball.  
I was *just staring at it.* Waiting. Expecting movement.  

So yes — the problem was me.  
The ball was fine. Gravity was fine. Unity was fine.  
**I** was the bug.

---

After that breakthrough, I:

- Set up the camera to follow the ball
- Added walls so the ball doesn’t fall off the platform

Next step is adding collectibles and finishing the game.

I also installed Oculus support in Unity — that setup was surprisingly smooth. But I’ll get into VR integration *after* finishing the game logic.

---

Alright, that's all for today.  
More tomorrow ✌️
