---
title: "Airbending Locomotion + Interaction in VR Parkour"
description: "Final paper-style project page — hand-tracked airbending locomotion + interaction in the VR parkour course repo."
---

# Airbending Locomotion + Interaction in VR Parkour  
*Final project page (paper-style) — Ajay Jose*

## Abstract (quick version)
I built a VR locomotion + interaction system inspired by airbending (Avatar: The Last Airbender). The final technique uses hand tracking: left-hand tilt controls horizontal movement and direction, and a deliberate right-hand swirl triggers a single upward jump. The parkour course and the object-interaction tasks in the provided course repository were originally built for a first-person OVRCameraRig player, so a large part of the work was integrating my third-person “AvatarRoot” body back into that pipeline. This page explains the motivation, early prototypes, the final implementation, the hardest challenges, and a small user study plan.

---

# 1. Introduction & Motivation
Most VR movement is still thumbstick-based. It works, but it does not feel like you are “doing” the movement. I wanted something more embodied and playful: move like an airbender, using hands, while still being able to finish the course parkour track and the interaction tasks.

---

# 2. Problem Statement
The main issues I wanted to address:

- Typical locomotion is functional but not very expressive or in-world.
- Hand tracking input is noisy (hands move even when you do not mean to move).
- Comfort: unstable backwards motion and bouncy vertical motion can become uncomfortable quickly.
- Integration: the course logic (banners, coins, tasks) assumes the OVRCameraRig is the player, but in my setup the physical player is a separate AvatarRoot.

Goal: design a hand-driven locomotion + interaction technique that is predictable, comfortable, and integrates cleanly with the existing parkour system.

---

# 3. Solution (early versions → final system)

## 3.1 Inspiration: Air scooter idea
The starting point was the air scooter from Avatar. I mapped the fantasy to controls: left hand tilt to steer, right hand motion to create lift/jumps.

{{< video src="day-3--airscooter.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

Early movement concept demo:

{{< video src="day-3--locomotion2.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

## 3.2 Third-person AvatarRoot + follow camera
Instead of moving the OVRCameraRig directly (first person), I introduced a separate physical body:

- **AvatarRoot:** Rigidbody + CapsuleCollider (this is the real player body that collides with the world).
- **airball + avatar mesh:** visuals only.
- **OVRCameraRig:** follows behind AvatarRoot like a third-person camera.

Visuals used in the post:

![NoBounce physics material](images/week-5--nobounce.png)

![AvatarRoot + airball + third-person camera](images/week-5--image.png)

---

## 3.3 Locomotion iterations (controllers → hand tracking)
The first working locomotion prototype used controller rotation. Later I switched to full hand tracking and kept controllers only as fallback.

A key Unity detail that helped camera stability was moving follow logic into **LateUpdate** (camera reacts after movement for the frame).

### 3.3.1 Left-hand tilt movement + activation zone
Core idea: calibrate a neutral pose, then compare the current hand ‘up’ vector to the neutral ‘up’ vector on the XZ plane. That difference becomes the tilt direction and magnitude for movement.

Major roadblock: I wasted 2–3 days because the avatar still moved even when I thought my hand was “not visible”. The Quest cameras could still track my hand at the edge of the view. Fix: an activation zone — movement only works when the left hand is inside a small box in front of the chest.

{{< video src="week-6,7--zone.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

### 3.3.2 Camera follow: HMD yaw vs avatar yaw
A mistake in my first camera-follow version was using HMD yaw as the rig direction. In VR, that made the avatar feel like it orbits around me whenever I looked sideways. I changed it so the rig follows AvatarRoot.forward, and my head is free to look around inside the rig.

Before (HMD yaw driving the rig):

{{< video src="week-6,7--week_6_yaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

After (avatar yaw driving the rig):

{{< video src="week-6,7--week_6_no_yaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

## 3.4 Switching fully to hand tracking
After feedback in class, I switched from controllers to hand tracking. It felt more like actual airbending, but also made noise and accidental input more obvious. I added thresholds, gating, and simple states to keep it predictable.

![Hand tracking setup screenshot](images/winter-break-weeks--image.png)

---

## 3.5 Right hand: lift experiments → final jump
My first idea for vertical movement was continuous lift by swirling the right hand (a lift gauge that builds up and decays). In reality it created constant micro-bouncing because small accidental motions kept the gauge above zero.

{{< video src="winter-break-weeks--winterbreak_bouncebounce.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

Final choice: make it discrete. One deliberate swirl triggers one clean upward impulse (jump), then a cooldown prevents spamming. This felt much better and reduced discomfort.

{{< video src="winter-break-weeks--jump.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

## 3.6 Turning corners: yaw-relative mapping
On the parkour track, corners were tricky. If movement is only based on avatar forward, turning a 90° corner can feel like fighting the track. I used HMD yaw (yaw-only) to compute camera-relative forward/right vectors so that “tilt forward” means “move where I am looking”.

{{< video src="winter-break-weeks--lefthandwithyaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

## 3.7 Making the parkour system recognize AvatarRoot
After I got locomotion working, I realized something embarrassing: the parkour course did not care about my AvatarRoot. It still assumed OVRCameraRig was the player. So banners, coins, and tasks never triggered.

The fix had two parts:

- **TriggerRelay:** forward collisions from my moving body into the existing AvatarTriggerEnter() logic.
- **Layers:** separate the physics body from a trigger-only sensor, because the repo uses a collision matrix where “Default” and “locomotion” interact differently.

Debugging showed I was colliding with random objects but not the triggers I cared about:

![Debug collisions](images/jan-first-half--raandom.png)

Final structure:

- **AvatarRoot (Default layer):** physics body (ground, walls, gravity).
- **BannerHitbox child (locomotion layer, IsTrigger):** sensor that talks to banners/coins/tasks and calls TriggerRelay.

![AvatarRoot setup](images/jan-first-half--new_avatarRoot.png)

![BannerHitbox sensor](images/jan-first-half--BannerHitBox.png)

![Coins and stages working](images/jan-first-half--goodCoin.png)

Once this was done, coins and banners worked again:

{{< video src="jan-first-half--coinsWork.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

## 3.8 Interaction task (T-shape puzzle)
The interaction task had two hidden problems: (1) starting it reliably, and (2) disabling locomotion so I don’t drift away while interacting.

### 3.8.1 Start: pinch to start
I originally thought the small block was a physical button. Instead, I made it a station: entering shows UI, and an index pinch actually starts the task.

![Station prompt](images/jan-second-half--objintr1.png)

### 3.8.2 Mode switching: stop locomotion cleanly
My actual movement logic lives in my custom LocomotionTechnique script, so disabling the wrong component did nothing. I used a PlayerModeManager to toggle exactly the locomotion behaviours vs interaction behaviours, and also stop momentum on switch.

![Mode manager wiring](images/jan-second-half--parkourcontrol.png)

### 3.8.3 Controls during interaction
Final mapping during interaction mode:

- **Left index pinch + rotate wrist:** rotate the T object (camera-yaw aligned).
- **Right index pinch + tilt:** translate in XZ relative to POV.
- **Right middle pinch + hand up/down:** translate in Y.
- **Hold both index pinches for ~1s:** finish the task and return to locomotion.

Interaction demo:

{{< video src="jan-second-half--t-shape.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

---

# 4. Implementation Details (complex challenges)
This section focuses on the parts that were actually hard (not basic Unity steps).

## 4.1 Input mapping in POV space (yaw-only)
The most important improvement was mapping tilt direction relative to my POV. I compute yaw-only forward/right vectors from the HMD and then use dot products to convert the hand tilt direction into a movement direction. That keeps steering consistent even when I rotate my head in the world.

## 4.2 Noise control (gating, thresholds, smoothing)
Hand tracking is noisy, so I used: an activation zone, a deadzone/threshold for tilt, smoothing (accelerate into target velocity), and cooldowns for jump. Without these, the avatar moves even when you don’t mean to move.

## 4.3 Unity update order: LateUpdate for camera follow
Movement and physics update earlier in the frame. Putting camera follow in LateUpdate means the avatar has already moved, and the camera reacts to the final position each frame. This reduced jitter and felt more stable.

## 4.4 Layer/collider split to integrate with existing parkour logic
The repo’s layer collision matrix made it impossible for a single collider to both collide correctly with the ground and trigger the locomotion layer objects. The dual-body solution (physics body + trigger sensor) was the cleanest way to integrate without rewriting the repo.

## 4.5 Mode switching reliability (and the “avatar didn’t come back” bug)
Disabling scripts is simple, but it can create lifecycle bugs. I had a case where the avatar renderers stayed hidden after the puzzle because the script that should re-enable them got disabled first. The fix was enabling the avatar again inside OnDisable().

---

# 5. Evaluation
I did a short user study style evaluation (10 minutes per participant). If you are reading this before I ran the study, the method and questionnaire below are ready to use, and the results section can be filled in after.

## 5.1 Method
**Participants:** N = ____ (fill). Mix of VR experience (beginner / intermediate / experienced).  
**Setup:** Meta Quest with hand tracking enabled.

**Procedure:**
1. Explain controls (about 5 minutes).
2. Task A (locomotion): complete at least one parkour segment and collect coins.
3. Task B (interaction): complete one T-shape station (start with pinch, manipulate, finish with pinch-hold).
4. Answer quick questionnaire (1–5 ratings) + short open questions.

## 5.2 Questionnaire (1 = low / 5 = high)
- How easy was it to understand the locomotion (how to move)?
- How much control did you feel over direction and speed?
- How physically tiring was the hand/arm input during play?
- How “Airbender-like” / immersive did the movement feel?
- How much discomfort / motion sickness did you feel (dizziness, nausea)?

Open questions:
- What was the most confusing part?
- If you could change one thing, what would you change?

## 5.3 Results (fill in after the study)
Template:
- Average ease of learning: __ / 5  
- Average control: __ / 5  
- Average effort: __ / 5  
- Average immersion: __ / 5  
- Average discomfort: __ / 5  
- Average Task A time: __ seconds  
- Average Task B time: __ seconds  
- Key observations: (1) ___  (2) ___  (3) ___

---

# 6. Conclusion
Overall, the system mostly achieved what I wanted. The final left-hand tilt locomotion is predictable, and mapping it to HMD yaw made it usable on corners. The discrete right-hand jump was much more comfortable than continuous lift. The biggest technical challenge was not the gesture math, but integrating a third-person AvatarRoot back into a first-person parkour pipeline. Splitting the player into a physics body and a trigger sensor solved that cleanly.

Main takeaways:
- Comfort beats clever physics: “correct” motion can still feel bad in VR.
- Hand tracking needs gating + thresholds + smoothing to avoid accidental input.
- Integration problems (layers, collision matrices, repo assumptions) can be harder than the locomotion algorithm itself.

---

# 7. Video (required)
The submission requires a video of me using the techniques and completing the parkour at least once. I included multiple development and feature videos above. For the final submission, I will record one continuous full run and embed it here.

Embed line to use once the file exists in `static/videos/`:

{{< video src="final-run.mp4" autoplay="false" loop="false" muted="false" playsinline="true" >}}

---

# Appendix A: Controls cheat sheet

## Locomotion mode
- Left hand tilt (inside activation zone): move (direction mapped to HMD yaw).
- Right hand swirl: jump (one impulse, cooldown).
- Left fist / pinch gesture (if enabled): recalibrate neutral.

## Interaction mode (inside station)
- Index pinch: start task.
- Left pinch + rotate wrist: rotate object.
- Right pinch + tilt: move object in XZ.
- Right middle pinch + hand up/down: move object in Y.
- Hold both index pinches ~1s: finish task and return.

---

# Appendix B: Using AI (ChatGPT) during the project
I used ChatGPT mainly as a debugging and explanation partner when I got stuck on vector math, Unity update order, and organizing scripts (mode switching, collision relays).

Where AI helped:
- Explaining yaw-only mapping and camera-relative movement.
- Suggesting small utility scripts quickly (trigger relays, state checks).
- Helping me write clearer explanations for the blog when I was confused.

What did not work well:
- Some suggestions were technically correct but felt bad in VR (comfort can’t be predicted from code alone).
- Some proposed fixes did not match the repo’s layer/collision assumptions, so I still needed a lot of testing.

My takeaway: AI was best as a pair-programmer and rubber duck, not as autopilot. The final design choices came from testing in-headset.

---

# Appendix C: Media files used on this page

## Images (under `static/images/`)
- images/week-5--nobounce.png  
- images/week-5--image.png  
- images/winter-break-weeks--image.png  
- images/jan-first-half--raandom.png  
- images/jan-first-half--new_avatarRoot.png  
- images/jan-first-half--BannerHitBox.png  
- images/jan-first-half--goodCoin.png  
- images/jan-second-half--objintr1.png  
- images/jan-second-half--parkourcontrol.png  

## Videos (under `static/videos/`)
- videos/day-3--airscooter.mp4  
- videos/day-3--locomotion2.mp4  
- videos/week-6,7--zone.mp4  
- videos/week-6,7--week_6_yaw.mp4  
- videos/week-6,7--week_6_no_yaw.mp4  
- videos/winter-break-weeks--winterbreak_bouncebounce.mp4  
- videos/winter-break-weeks--jump.mp4  
- videos/winter-break-weeks--lefthandwithyaw.mp4  
- videos/jan-first-half--coinsWork.mp4  
- videos/jan-second-half--t-shape.mp4  