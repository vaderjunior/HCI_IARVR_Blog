+++
date = '2025-12-10T09:38:16+01:00'
draft = false
title = 'Week 6 and 7: Teaching the Avatar to Follow Me or me to follow it, or him or them cause its a a cylinder and a sphere now'
+++

This week I finally started working on the `LocomotionTechnique` script itself instead of just moving GameObjects around like Lego. It was fun to test things out on the already existing repo, Must have taken a lot of hours to make this entir thing. Just thinking about it made me motion sick :scream:

In the original project the camera rig was the player. In my version the real body is `AvatarRoot`  a capsule collider rolling through the world with a cylinder (`avatar`) and a sphere (`airball`) as place holder. I thought I will put in design later, incase everythign gets done by February. 

I had to put in considerable amount of time and reading up to understand how to configure this change. In practice, AvatarRoot now has the rigidbody + capsule collider and is the thing that really moves and hits the world. The OVRCameraRig just follows it and looks at it instead of being the player itself.

> This part is open to change, since I thought id gameify it a bit. I thought the coin collection is much easier this way. Maybe I am wrong, but I will test things out and see from there

Also, I moved my third-person follow logic into `LocomotionTechnique`:

* `avatarRoot` and `avatarBody` became public fields on the script.
* I added a `cameraOffset` and `followLerp`.
* In `LateUpdate()` I started repositioning the `OVRCameraRig` behind the avatar instead of letting it move on its own.

**LateUpdate vs Update**
I wasn’t sure at first where this follow logic should be added, so I went down a small rabbit hole about Unitys update order and discovered `LateUpdate()`. Unity calls **`Update()` on everything first**, and then **`LateUpdate()` afterwards**. Since my avatar movement happens earlier in the frame, putting the camera follow code in `LateUpdate()` means the avatar has already finished moving, and then the `OVRCameraRig` simply snaps in behind it. That way the camera feels more like its reacting to the final position of the avatar each frame.

> **Note:**  
> This is also where I first learned the word *yaw*. Until then everything was just “tilt” in my head. I was reading a Unity forum post where someone casually wrote about “HMD yaw”, and that made more sense so I am using it everywhere now.

The very first version followed the HMD yaw: every time I turned my head, the whole rig tried to stay behind me. It looked fine when i tested it in my laptops unity game mode. But In VR it felt like my avatar was orbiting me whenever I looked to the side. Not great, and also a bit confusing because I was only trying to Look around, not drag my whole body with me.

After a bit of trial and error I realised the mistake: I was treating the **head direction** as the “true” forward direction for the whole rig. So any tiny head turn made the rig try to reposition itself again. That’s when I decided to change the mental model.

So I changed it to:

* The avatar is the “real” direction in the world.
* The camera rig is fixed behind the avatar. Like a game you can say.
* My head is just freely moving inside that rig.

Once I started thinking about it this way, the implementation also became clearer.

Technically that meant something like this:

```csharp
// Compute desired camera rig position behind the avatar
Vector3 targetPos =
    avatarRoot.position
    - avatarRoot.forward * Mathf.Abs(cameraOffset.z)
    + Vector3.up * cameraOffset.y;

// Smoothly move rig position
transform.position = Vector3.Lerp(
    transform.position,
    targetPos,
    1f - Mathf.Exp(-followLerp * Time.deltaTime)
);

// Smoothly align rig yaw with avatar yaw
Quaternion targetYaw = Quaternion.Euler(0f, avatarRoot.eulerAngles.y, 0f);
transform.rotation = Quaternion.Slerp(
    transform.rotation,
    targetYaw,
    1f - Mathf.Exp(-followLerp * Time.deltaTime)
);
```


So instead of following the HMD yaw, the rig now follows `AvatarRoot.forward`. The HMD is free to look around inside the rig without moving the body. Once this clicked, it suddenly felt more like a proper third-person game: the avatar is always in front of me, the world is stable, and my head isn’t secretly driving everything.

**Before**
{{< video src="week_6_yaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}


**After**
{{< video src="week_6_no_yaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}


With the camera behaviour in a decent spot for now, I finally started on the actual “airbender” part: **left-hand tilt movement**.

# Locomotion

> **Left Hand Idea**
> Hold your left controller straight and flat in front of you like you’re steering an invisible scooter, and tilt to move. I have never driven movement from a rotation like this before, so this part was pretty experimental and was doubtful to even work.

{{< video src="week_6_left.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

I added a calibration for a “neutral” pose:

* `CalibrateLeftNeutral()` stores the current left-hand rotation as neutral.
* **X** button triggers recalibration.
* For a short `neutralLockDuration`, movement is frozen. I might change it to pause till the x button is released.

The goal here was to let me hold the controller in whatever is comfortable and mostly for test purposes, since during testing i was rotating so much and i was getting dizzy. This helped a lot.

Then each frame I roughly do:

* Get the left controller rotation.
* Ask Unity for the controller’s current “up” vector from that rotation.
* Compare it to the neutral up vector in the XZ plane.
* Treat that difference as a tilt value and feed it into movement.

I don’t fully understand all the maths under the hood yet, but this was enough to get a first prototype where tilting my left controller actually nudged the avatar around. I had to get help from online forums and chatgpt for this calculations.

I Wasted around 2-3 days since even if I could not see my left hand my avatar was moving, it made no sense to me. Untill I saw by accident that my hands was still being seen at the edge f my vision, I was keeping my hands on my lap and this was still seen by the cameras even If i was not seeing it. I felt dumb and relived at the same time.

To fix it, I added “activation zones” around the head, because my left controller is almost always doing something random until i keep the whole arm behind me.

* The left controller has to be inside a small 3D box in front of the chest.
* Outside that box, tilt is ignored.

I also spawned a translucent box in the scene (`leftHandZoneVisual`) so I could see where this zone actually was. This helped a lot to debug why sometimes nothing was happening: most of the time my controller was  outside the zone.

By the end of Week 6 I had:

* A proper third-person camera integrated into the locomotion script.
* A calibrated left-hand controller steering concept with activation zone and reset. Needs a ton of work to calibrate tho. Now if i test i need to sit for a few minutes because i get so dizzy from the erratic movement.

Below you can see when my hand is in the zone which is very green(i made this so its very easy to understand in the video) the avatar moved, otherwise it wont.

{{< video src="zone.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

The movement still felt a bit wobbly. Sometimes a tiny tilt did more than I expected, sometimes less. But at least the basic idea of Aand on an airball that I can steer with my left controller was visible now. The plan for the next round is to make this feel more consistent and predictable. Also right hand controllers. 

Cheers,
-Ajay












