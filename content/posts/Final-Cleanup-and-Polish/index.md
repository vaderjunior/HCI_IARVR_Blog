+++
date = '2026-02-03T18:00:00+01:00'
draft = false
title = 'Final cleanup: Aang sprites, better interaction controls, and a hidden HUD you can summon'
+++

Today was mostly “polish day”, but it fixed a bunch of things that were bothering me for weeks.

## 1 Replacing the placeholder avatar with Aang + airball sprites

Until now my “avatar” was literally a cylinder on a sphere. It worked for logic, but it looked like a science project.

So I switched the visuals to sprites:

* the **cylinder** became an **Aang sprite**
* the **sphere** became an **airball sprite**
* the physics stayed the same (AvatarRoot collider + rigidbody), only visuals changed

**Important thing I learned:** I can’t just drag any PNG into a SpriteRenderer. It needs to be imported as a Sprite first.

**Steps that worked for me:**

* Drag PNG into `Assets/`
* Click the PNG → Inspector → set **Texture Type = Sprite (2D and UI)**
* Apply
* Then drag it into the SpriteRenderer slot


Also I kept meshes hidden because I only want sprites visible. If any mesh renderer comes back during runtime, I force-hide it again.



## 2 Interaction task: fixing backward translation (tilt back actually pulls it towards me)

In the object interaction task, translation was working forward, but “tilt back” was refusing to move backward.
I was pinching and doing the “three fingers up / Jesus Christ” wrist bend, but nothing happened.

Turns out my logic was reading `rightHand.transform.forward`, and that axis doesn’t always flip the way my brain expects when I tilt my wrist back.

So I changed the translation direction to use `rightHand.transform.up` instead (projected onto XZ).
That made the gesture match what I naturally do:

* pinch + tilt forward → push away
* pinch + tilt back → pull towards me
* pinch + tilt right/left → slide right/left

And I updated the neutral calibration to also use the same axis so the comparison stays consistent.



## 3 HUD: still running, but only visible when I trigger it

When I start the parkour I see big UI text in my face (time, coins, etc.).
I wanted it to keep updating in the background, but only show up when I want.

I tried hiding the canvas, but Oculus UI stuff can be weird and it didn’t reliably disappear.
The version that finally worked was: **disable the TMP text renderers**, not the logic.

So `ParkourCounter` keeps updating `timeText.text` and `coinText.text`, but the text is invisible until I toggle it on.

Gesture-wise, I avoided pinch because pinch is already used for starting/exiting tasks. I made it something separate (hands together + relaxed), and it toggles HUD on/off like a switch.

{{< figure src="PLACEHOLDER_hud_toggle_script.png" caption="(add screenshot) HUD toggle script attached to the HUD Canvas" >}}

{{< video src="PLACEHOLDER_hud_toggle_demo.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

## 4 Quick recap of what improved today

* Aang + airball sprites replaced the cursed cylinder + sphere visuals
* Object interaction “tilt back” finally works (backward translation fixed)
* HUD is now optional: it runs always, but only shows when I summon it

It felt like a small day, but honestly these were the kind of annoying issues that made testing feel messy. Now it feels way cleaner.

Next up: real playtesting on the parkour level again and checking if anything feels unfair (coin collection especially), now that visuals and UI are not fighting me anymore.

Cheers,
Ajay
