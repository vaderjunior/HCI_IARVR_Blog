<!-- TEMP MERGE: 2026-01-31 15:12 -->
---

``text
SOURCE: D:\IARVR\Blog\HCI_IARVR_Blog\content\posts\day-3\index.md
SORTDATE: 2025-11-13 15:01
``


---

+++
date = '2025-11-13T14:03:36+01:00'
draft = false
title = 'Week 4: Thinking, thinking and more thinking'
+++

So,

This week was mostly spent on thinking and trying to work out what locomotion technique I want to use for the project.

Firstly I tried to remember and dig up all the things previous years’ students did, not just to be inspired but to do something that is different from what they have done.

Also I tried familiarising myself with the whole Unity and locomotion system in general.  
The video and playlist below were super useful. The guy is fun to listen to and overall I learned how to calibrate the controller feedback to actual input for movement and how to fine tune it. Even though everything is not clear I thought I would look up more when I actually start the project.

After a few days of planning and scrapping a ton of stuff I came to finalize the idea below for my locomotion technique.

### Air Scooter from Avatar: The Last Airbender

{{< video src="airscooter.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

I thought about how to translate this into a locomotion technique for 2 days and finally came down to the controls below.

- **Right hand:** rotate it in circles to lift the ball up. It will basically be like flappy bird where the rotation pushes the ball up but as soon as you stop it slowly starts coming down.
- **Left hand:** when it is held flat there is no movement. But if I tilt it to the front the player moves to the front, back means back, left and right as well. It is not just those 4 but combinations also work.

So the player needs to use both hands to guide Aang on top of the AirScooter to collect all the coins.

An idea of the movement is shown below.

{{< video src="locomotion2.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

The coming week I will start with implementation of this technique.

-Ajay


---

``text
SOURCE: D:\IARVR\Blog\HCI_IARVR_Blog\content\posts\week-5\index.md
SORTDATE: 2025-11-25 09:55
``


---

+++
date = '2025-11-25T09:38:16+01:00'
draft = false
title = 'Week 5: Getting started with Airbending or rather Unitybending'
+++


This week was finally “hands on” instead of just thinking.

I started by forking and cloning the VR locomotion parkour repo from our course. The instructions said “open the VRParkour folder as a Unity project”, but when I did that and hit Play, the scene was completely empty.

After poking around a bit I realised I had just opened a blank scene. The real content is in
Assets/Scenes/ParkourChallenge.unity. Once I opened that, suddenly the whole level, coins, banners, etc. showed up and actually ran on the Quest.

The next step was to prepare the scene for my Avatar air scooter idea. The original setup was first person, with the camera rig basically being the player. I wanted a third person view where the camera follows a character sitting on a ball of air.

So I:

- Created an AvatarRoot object to act as the real player body in the world.

- Made two children:

    - Airball – a white sphere that Aang will “sit” on.

    - avatar – a simple cylinder as a placeholder body.

- Added a Rigidbody and a CapsuleCollider to AvatarRoot and made this the only solid collider for the avatar. The ball and cylinder are just visuals.

- Gave the Capsule a NoBounce physics material so it doesn’t fly into the sky every time it touches the ground.

![NoBounce Code](nobounce.png)

- Wrote a small script so that OVRCameraRig no longer moves by itself but instead follows AvatarRoot from behind with an offset, like a third-person camera. I basically used the similar one as I did for the Roll a Ball game.

```csharp
public class ThirdPersonFollow : MonoBehaviour
{
    public Transform avatarRoot;   // the body we follow
    public Transform hmd;          // CenterEyeAnchor
    public Vector3 offset = new Vector3(0f, 1.6f, -3f);
    public float followLerp = 10f;

    void LateUpdate()
    {
        if (!avatarRoot || !hmd) return;

        // Use only the HMD's yaw so the camera stays behind the avatar
        Vector3 euler = hmd.rotation.eulerAngles;
        Quaternion yawOnly = Quaternion.Euler(0f, euler.y, 0f);

        // Desired camera position 
        Vector3 targetPos = avatarRoot.position + yawOnly * offset;

        // Smoothly move the rig there
        transform.position = Vector3.Lerp(
            transform.position,
            targetPos,
            1f - Mathf.Exp(-followLerp * Time.deltaTime)
        );

        // Look at the avatar from that position
        transform.rotation = Quaternion.LookRotation(
            avatarRoot.position - transform.position,
            Vector3.up
        );
    }
}
```

By the end of the week I had:

- The original parkour scene running,

- A separate physical body (AvatarRoot) rolling through the world,

- A third person camera that follows this body,

- And a basic “Aang on a ball” setup ready to connect to my air scooter locomotion script next.


![Setup Now](image.png)

The actual hand gesture locomotion still needs work, but the foundation (project structure, avatar, colliders, camera) is finally in place. I also have a good idea now of how and where I should start my work. I took me about 2 days to figure out everything in the project since after roll a ball this is my second unity project. Yaras guides were very helpful.


---

``text
SOURCE: D:\IARVR\Blog\HCI_IARVR_Blog\content\posts\week-6,7\index.md
SORTDATE: 2025-12-10 21:11
``


---

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

### <u><strong>LateUpdate vs Update</strong></u>

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

### <u><strong>Before</strong></u>

{{< video src="week_6_yaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}


### <u><strong>After</strong></u>

{{< video src="week_6_no_yaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}


With the camera behaviour in a decent spot for now, I finally started on the actual “airbender” part: **left-hand tilt movement**.

### <u><strong>Locomotion</strong></u>


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

I don’t fully understand all the maths under the hood yet, but this was enough to get a first prototype where tilting my left controller actually moved the avatar around. I had to get help from online forums and chatgpt for this calculations.

<div style="border-left: 4px solid #ef4444; background: #fee2e2; padding: 0.75rem 1rem; margin: 1rem 0;">

**Major roadblock**

I wasted around 2–3 days because even when I couldn’t see my left hand, my avatar was still moving. It made no sense to me.  
Only by accident I noticed that my hand was still just at the edge of my vision. I was keeping my hands on my lap and the Quest cameras could still see them, even though I couldn’t.  

I felt dumb and relieved at the same time.

</div>


To fix it, I added “activation zones” around the head, because my left controller is almost always doing something random until i keep the whole arm behind me.

* The left controller has to be inside a small 3D box in front of the chest.
* Outside that box, tilt is ignored.

I also spawned a translucent box in the scene (`leftHandZoneVisual`) so I could see where this zone actually was. This helped a lot to debug why sometimes nothing was happening: most of the time my controller was  outside the zone.

### <u><strong>So now below is what I have:</strong></u>

* A proper third-person camera integrated into the locomotion script.
* A calibrated left-hand controller steering concept with activation zone and reset. Needs a ton of work to calibrate tho. Now if i test i need to sit for a few minutes because i get so dizzy from the erratic movement.

Below you can see when my hand is in the zone which is very green(i made this so its very easy to understand in the video) the avatar moved, otherwise it wont.

{{< video src="zone.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

The movement still felt a bit wobbly. Sometimes a tiny tilt did more than I expected, sometimes less. But at least the basic idea of Aand on an airball that I can steer with my left controller was visible now. The plan for the next round is to make this feel more consistent and predictable. Also right hand controllers. 

Cheers,

-Ajay


---

``text
SOURCE: D:\IARVR\Blog\HCI_IARVR_Blog\content\posts\Winter Break Weeks\index.md
SORTDATE: 2026-01-06 22:50
``


---

+++
date = '2026-01-06T09:38:16+01:00'
draft = false
title = 'Winter Break and New year week: When Physics Starts Messing With My Hands'
+++

Last week (Week 6) I finally got the AvatarRoot + camera setup working and a very baby version of left-hand **controller** tilt movement. It moved, it kinda listened, and it also made me dizzy if I tested too long. So for the next weeks the goal was basically: make this feel less like a rollercoaster ride and more like something I can actually have fun with.

Also, very important change that happened here: I switched from controllers to hands.

### <u><strong>Goodbye controllers, hello hand tracking</strong></u>

![NoBounce Code](image.png)

Until now I was only using the Quest controllers. All the “tilt” logic was reading the controller rotation. That was easier for me to understand at first, and also it was closer to how the original repo works.

But then after pitching my idea in class Professor asked me plainly why I am not using just hands, it made me think cause honestly till then that idea did not even come to me. Maye its because of my lack of experience in VR but I thought that I should at least try full hand tracking instead of driving everything with two plastic sticks.

So I turned on hand tracking in the Oculus/OVR settings, and added `OVRHand leftHand` and `OVRHand rightHand` references to my `LocomotionTechnique` script. Then I changed the rotation code so that, if hand tracking is enabled and the hand is tracked with good confidence, I use `leftHand.transform.rotation` instead of the controller’s rotation. I kept the controller logic as a fallback, because hand tracking can randomly fail (lighting, me doing weird poses, etc.) and I didn’t want the avatar to act glitchy and weird while testing.

I also started using pinch strength to detect a gesture. If all four non-thumb fingers on the left hand have pinch strength above a threshold, I count that as a “left fist”. I hooked that into calibration:

left fist → call `CalibrateLeftNeutral()` → reset movement.

So now I don’t need to click the X button every time. I can just clench my hand and the system understands “ok neutral pose changed”.

The first time the VR skeleton hands showed up and the avatar responded to my actual hand tilt, it felt pretty magical. Also slightly cursed, because tracking is not always stable and honestly very janky.

From here on, when I say “left hand” / “right hand”, I mostly mean the real tracked hands. Controllers are still there as backup, but the main idea is now controller free airbending.

### <u><strong>Left hand: the drunk phase</strong></u>

The core idea stayed the same: hold left hand in front, tilt, avatar glides.

In practice this turned into: why does the avatar act like a typical person coming out of a bar at 10pm on a saturday??

I tried to make the movement feel more “analog”(I am still holding onto this cause i have grown up playing games and this is what is coming to me naturally) and smooth. 
So I added stuff like 

* deadzone for tiny tilts
* tiltGain to exaggerate angles
* maxSpeed so it doesn’t fly away, and also 
* acceleration/drag so it feels like it’s gliding instead of snapping. 

It did work, but it was super moody. Some days a tiny tilt would suddenly send the avatar off faster than expected. Other times I tilt a lot and it still felt lazy. The numbers were never stable enough for me to trust. :crying_cat_face:

This is the moment where I finally understood why people say “game feel matters”. The maths can be correct and still your brain goes “nope”.

Hand tracking also made the random movement problem worse, because my hands are always doing something. Scratching face, resting on lap, drifting out of view, coming back in, etc. Even if I don’t mean to steer, the system is like “ah yes, input”.

So I wrote to myself: I think I’m trying too hard to make it physically clever. I should first make it predictable and game-like. Only then I can add fancy stuff again.

### <u><strong>Right hand: We have a lift off (and why it sucked)</strong></u>

At the same time I started experimenting with right hand for vertical movement. The idea is simple: avatar sits on airball, so right hand should create “air lift”.

The first idea was: swirl your right hand like a cowboy spinning a rope and you build up lift. So I tracked right-hand position, computed velocity, measured how quickly the velocity direction changes (how “circular” it is), and combined that into a swirlStrength. That fed into a liftGauge which reduces over time. In physics I applied upward acceleration proportional to the gauge.
```csharp

Vector3 prevPos;
float liftGauge = 0f;
Vector3 prevVelDir = Vector3.forward;

void Update()
{
    // 1) estimate right-hand velocity from position change
    Vector3 pos = rightHand.transform.position;
    Vector3 vel = (pos - prevPos) / Mathf.Max(Time.deltaTime, 1e-4f);
    prevPos = pos;

    float speed = vel.magnitude;
    if (speed < 0.05f) return;

    // 2) how “circular” is the motion? (direction changing fast = more swirl)
    Vector3 velDir = vel.normalized;
    float ang = Mathf.Acos(Mathf.Clamp(Vector3.Dot(prevVelDir, velDir), -1f, 1f)); 
    float angPerSec = ang / Mathf.Max(Time.deltaTime, 1e-4f);
    prevVelDir = velDir;

    // 3) swirl strength + gauge (build up + decay)
    float swirlStrength = Mathf.Clamp01((speed * angPerSec) / 5f); // 5f = random scale
    liftGauge = Mathf.Clamp01(liftGauge + swirlStrength * Time.deltaTime);
    liftGauge = Mathf.MoveTowards(liftGauge, 0f, 0.6f * Time.deltaTime); // decay
}

void FixedUpdate()
{
    // 4) apply smooth upward acceleration based on gauge
    float upAccel = liftGauge * maxLiftAccel;
    avatarBody.AddForce(Vector3.up * upAccel, ForceMode.Acceleration);
}
```

In my head and in theory this was very cool: you charge air in a circle and the ball softly rises.

In reality it turned into a tiny trampoline. Even small accidental right-hand motion kept the gauge slightly above zero, which meant the avatar was always bouncing a little bit. When combined with left-hand movement it became: forward – boing – forward – boing. Like climbing an invisible staircase. With a headset on, this is not funny after 10 seconds.

{{< video src="winterbreak_bouncebounce.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}


Somewhere in the middle I also tried a side experiment: right fist to charge and open to release an upward impulse. It worked but felt wrong. It was tiring and it looked strange. So I deleted it.

> **Note to future me:**  
> It’s ok to throw away ideas. The code is not sacred. Maybe its not the 1 grade point that I will get but the real treasure was knowing what doesnt work for sure.

After a few rounds of this I basically had a comfort  check. In VR, weird physics + head movement + camera movement = nausea. So I stopped trying to be clever and decided to make things simple. Long time coming if you ask me.

### <u><strong>Pre-Christmas cleanup: making it feel like a game</strong></u>

This week I didn’t want to add ten new features. I just wanted the existing controls to feel less chaotic. So I decided to stop being fancy and simplify everything.

### <u><strong>Left hand: from analog mess to two clear states</strong></u>

The analog speed idea was cool in theory, but in practice it always felt inconsistent. So I simplified it into basically a switch:

If tilt is below threshold → no movement.  
If tilt is above threshold → move at a fixed speed.

Now it behaves more like “I am moving” vs “I am not moving”. Instantly more easy to use.

I also changed how I decide the direction. Earlier I was thinking too much in world axes. Now I map movement relative to where I’m facing (and later also head yaw, see below). So tilt forward moves forward, tilt right gives clean strafe right, diagonals actually feel like diagonals.

I still kept a small physics-y part for feel, so it accelerates into motion and glides a bit instead of snapping instantly. But overall the important part is: it’s predictable now.
```csharp
// LEFT HAND 

// 1) measure tilt (compare current hand "up" vs neutral "up" on XZ plane)
Vector3 upNow = leftHandRotation * Vector3.up;
Vector3 tilt = new Vector3(upNow.x - upNeutral.x, 0f, upNow.z - upNeutral.z);
float tiltMag = tilt.magnitude;

if (tiltMag < moveTiltThreshold)
{
    targetHorizVel = Vector3.zero;          // OFF
}
else
{
    Vector3 tiltDir = tilt.normalized;

    // 2) get forward/right based on where I’m facing (yaw only)
    Quaternion yawOnly = HeadYawOnly();     // from HMD 
    Vector3 fwd   = (yawOnly * Vector3.forward).XZ().normalized;
    Vector3 right = (yawOnly * Vector3.right).XZ().normalized;

    // 3) convert tilt direction into move direction 
    float f = Vector3.Dot(tiltDir, fwd);
    float r = Vector3.Dot(tiltDir, right);
    Vector3 moveDir = (fwd * f + right * r).normalized;

    targetHorizVel = moveDir * moveSpeed;  // ON (fixed speed)
}

// 4) smooth acceleration
horizVel = MoveTowards(horizVel, targetHorizVel, moveAccel * dt);
avatarBody.linearVelocity = new Vector3(horizVel.x, avatarBody.linearVelocity.y, horizVel.z);
```

### <u><strong>Backwards walking = discomfort, so I nerfed it</strong></u>

One thing I noticed quickly: fast backwards movement in VR feels horrible. So I added a small comfort rule: if the intended direction is mostly backwards, multiply speed by a backwardSpeedMultiplier (like 0.4). So straight back is intentionally slow. Diagonal back can still be ok.

It sounds like a tiny detail, but it reduced a lot of weird movements that are not even intended.

### <u><strong>Right hand: air jumps instead of continuous lift</strong></u>

Big change on the right hand: I scrapped the lift gauge and made it discrete.

New idea: one strong swirl / whip motion → one clean upward pop. Then a cooldown so you can’t spam it.

Under the hood I still track hand velocity and check if it’s fast enough, direction changes fast enough, and mostly horizontal (because I want rope motion, not up-down flapping). If it qualifies, I apply an impulse:

```csharp
avatarBody.AddForce(Vector3.up * swirlLiftImpulse, ForceMode.VelocityChange);
```
Then I clamp max upward velocity so I don’t launch into VR space. The result feels much nicer. Nothing happens when I casually move my right hand. One deliberate swirl gives one jump. It feels like an actual mechanic.

{{< video src="jump.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

### <u><strong>Turning corners without fighting the track</strong></u>
One sneaky problem with third-person movement on the parkour track is corners. If the road turns 90 degrees and my movement reference is only avatar forward, then I end up kind of strafing down the new road instead of moving forward, because the avatar forward is still the old direction.

So I let head yaw help here. When I look into the new road segment, head yaw changes. I use that yaw to decide what “forward” should be for the tilt mapping. So I can look into the next segment, tilt forward again, and the avatar moves into the turn naturally.

It’s basically: “where I look is where I mean to go next”. The camera still stays behind the avatar, but the direction mapping respects my attention, which made 90° and 180° turns feel way better.

{{< video src="lefthandwithyaw.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

## <u><strong>What’s next</strong></u>
So right now this phase ended with:

Left hand = predictable movement (two states).
Right hand = clicky air jumps .
Turning corners 

Next up is the other half of the problem: the parkour game still doesn’t fully see my air bender boy on an air scooter. The banners, coins and mini object-interaction tasks were written for the original first-person rig. Now that my capsule avatar is the “real” body, I need to make sure the rest of the level recognizes it again. For some reason the coins does not trigger when i enter the start pole. Well a problem for another day.

Cheers,
– Ajay


---

``text
SOURCE: D:\IARVR\Blog\HCI_IARVR_Blog\content\posts\Jan-First-Half\index.md
SORTDATE: 2026-01-31 14:17
``


---

+++
date = '2026-01-17T11:10:19+01:00'
draft = false
title = 'Jan First Half: Finishing up Locomotion'
+++

After all the work on left-hand and right-hand gestures, I realised something slightly embarrassing: the parkour course didn’t care at all about my `AvatarRoot`. It still thought the **OVRCameraRig** was the player.

So even though I could move around and do my airbending jumps, the game logic was like: “cool story bro, who are you?”
No timer, no coin pickups, banners didn’t react, and those mini interaction tasks never started.

This week was basically me trying to make the level *recognise* my avatar again after i panicked when I realized this was happening in the first place. So I went back to the original game in the github and tried playing it, for some reason I never played it till now and hence why it took me so long to realize that nothing was recognizing me.

---

### <u><strong>What the original project expected</strong></u>

First I tried to understand the original flow without changing anything.

There’s a script called `ParkourCounter` that reacts when the player hits stuff:

* banners  → start / change stages
* coins  → count them
* task triggers (tagged `objectInteractionTask`) → show the UI and start the mini task

And inside my `LocomotionTechnique` there’s already a function for this:

`AvatarTriggerEnter(Collider other)`

In the original repo, the camera rig itself was moving and touching those colliders. So `AvatarTriggerEnter()` would get called naturally.

But in my setup:

* the thing that *actually moves and collides* is `AvatarRoot` (capsule + rigidbody)
* the `OVRCameraRig` is basically a follower camera now
* which means the rig never touches banners/coins/tasks

So the parkour logic didn’t “break”… it just never got triggered till now :(.

---

### <u><strong>TriggerRelay: the tiny script that fixes everything</strong></u>

I didn’t want to rewrite the whole parkour system because it was already taking me a lot of time to figure out different things in the project and unity coding itself. I just wanted to forward collisions from my physical body to the script that already knows what to do. That looked more simpler even though its like a work-around.

So I added a small relay script on the moving collider (AvatarRoot / hitbox object):

```csharp
public class TriggerRelay : MonoBehaviour
{
    public LocomotionTechnique locomotion; 

    void OnTriggerEnter(Collider other)
    {
        if (locomotion) locomotion.AvatarTriggerEnter(other);
    }

    void OnCollisionEnter(Collision collision)
    {
        if (locomotion) locomotion.AvatarTriggerEnter(collision.collider);
    }
}
```

And I also added a debug line inside `AvatarTriggerEnter()` so I could see what I’m hitting:

```csharp
Debug.Log("AvatarTriggerEnter with: " + other.name + " tag: " + other.tag);
```

This was super useful because it told me: *collisions are happening*, but not with the things I care about.

At first my console spam looked like:

* `LeftHandZoneVisual` 
* random floor tiles 
* controller prefab stuff 

![BannerHitBox](raandom.png)

But never:

* `StartBanner`
* `Coin_xx`
* `ObjectInteractionInitiator...`

That’s when I realised this wasn’t a “trigger relay” problem. This was a **layers** problem.

---

### <u><strong>Layers: parkour sees me, ground doesn’t (and vice versa)</strong></u>

The parkour objects (banners/coins/tasks) live on a custom layer called `locomotion`.
The ground is on a different layer.

My `AvatarRoot` was on **Default**. So:

* it collided with the ground 
* but based on the project’s physics collision matrix, it didn’t interact with the `locomotion` layer.

Then I tried the obvious thing: put `AvatarRoot` on the `locomotion` layer.

It worked… for 2 seconds.
Banners started triggering, and then my avatar immediately **fell through the ground** because now it wasn’t colliding with the floor layer anymore.

So the situation was:

* Default layer → ground works, parkour ignores me
* Locomotion layer → parkour works, ground ignores me

Lovely.

---

### <u><strong>Two bodies: one for physics, one for talking</strong></u>

The solution that finally made everything sane was: don’t force one collider to do two jobs.

So I split it:

**1. `AvatarRoot` (main body)**

* Layer: `Default`
* CapsuleCollider + Rigidbody
* Handles gravity, floor, walls, normal physics

![First steps](new_avatarRoot.png)

**2. `BannerHitbox` (new child under AvatarRoot)**

* Layer: `locomotion`
* CapsuleCollider set to **Is Trigger**
* Has the `TriggerRelay` script

![BannerHitBox](BannerHitBox.png)

So:

* `AvatarRoot` keeps behaving like a normal physical player
* `BannerHitbox` just follows along as a “sensor” that talks to banners/coins/tasks

Once I did this and hit Play, I finally saw the console print something satisfying:

* `AvatarTriggerEnter with: StartBanner tag: banner`

And the scene actually responded:

* start banner disappears
* next banner appears
* coins spawn
* timer starts ticking

That moment felt *way* too good for something this basic 😭

![BannerHitBox](goodCoin.png)

---

### <u><strong>Bonus: Unity froze on Play Mode</strong></u>

While doing all this trial-and-error, Unity randomly decided to freeze sometimes on:

`Application.EnterPlaymode`

Only fix was to kill Unity from Task Manager. Not fun at alllllll.

A friend told me a hack that helped a lot:

* Edit → Project Settings → Editor
* enable “Enter Play Mode Options”
* disable:

  * Reload Domain
  * Reload Scene

After this, Play Mode became almost instant and the freeze stopped (for a bit).  
I happily assumed: “Cool, that was the problem.”

Later I realised two things:

1. This wasn’t the real fix, i am still to figure out why it happens.
2. Those settings come with side effects: static variables and some states don’t reset properly between plays, which started to be problematic.

So in the end I **switched those options back off** and returned to the default behaviour.  
Still, if your project is big and you know what you’re doing with static state, this trick can be worth it. Just don’t let it distract you from actually fixing your code like it did for me. xD


---

### <u><strong>So where things stand now</strong></u>

Right now the full loop finally feels connected:

* third-person avatar + camera works
* left hand tilt gives predictable movement
* right hand swirl gives clicky jumps
* **and** the course actually reacts to my avatar again:

  * banners trigger
  * coins count

{{< video src="coinsWork.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

There’s still polish left , and also next week I will try to figure out the interaction task.
I have not started that at all and it is kind of making me anxious. But lets see. 

Cheers,
– Ajay


---

``text
SOURCE: D:\IARVR\Blog\HCI_IARVR_Blog\content\posts\Jan-Second-Half\index.md
SORTDATE: 2026-01-31 14:43
``


---

+++
date = '2026-01-31T09:57:00+01:00'
draft = false
title = 'Jan Second Half: Interaction Task'
+++

Last week I ended with: “I haven’t started the interaction task at all and it’s making me anxious.”
Yeah… that anxiety aged like milk. 
Because the interaction task turned out to be *two separate problems* hiding behind one :

1. **How do I even start the task?**
2. **How do I disable locomotion so I don’t drift away while trying to fit a T inside a hole like a stressed-out toddler?**

This post is basically how I made the “T shape puzzle” work end-to-end: 
    1. enter checkpoint 
    2. pinch to start 
    3. manipulate object 
    4. pinch hold to finish 
    5. return to locomotion.

---

## The “weird start block” was a trigger station (not a button)

I kept thinking there’s a physical start button I’m supposed to grab (because there *is* that tiny block sitting there). But in my build, grabbing it wasn’t realistic since my hands were already mapped to locomotion gestures.

So instead of fighting the tiny button, I made the checkpoint behave like a proper “VR station”:

* the checkpoint object is still a trigger volume (so it knows when I arrived)
* entering it only shows the UI (“Pinch to start”)
* **I added “pinch to start” as the actual trigger** (so I can start reliably without UI clicking)

That small change basically made the whole task usable. I tried a whole set f different gestures t trigger it but the pinch was the most reliable ne. So i sticked with it.


![ObjectInteraction1](objintr1.png)

**Important detail:** I learned it the hard way that triggers in Unity can be annoying if no Rigidbody is involved.
So each initiator got:

* **CapsuleCollider** → `Is Trigger = true`
* **Rigidbody** → `Use Gravity = false`, `Is Kinematic = true`

This way the station better detects the player entering.

---

## Starting the task: pinch to make the two T objects appear

I created a tiny starter script and slapped it on each trigger station (`ObjectInteractionInitiator1/2/3`).

Logic:

* if I’m inside the station
* and I do an index pinch
* then start the task + switch mode to interaction

Also: I fixed a subtle bug from my older logic. My `AvatarTriggerEnter()` was setting `isTaskStart = true` immediately on entering the zone. That meant timing could start before the task actually starts (while I’m still reading “start”). I changed it to **only start timing on pinch**.

This is the core idea of the pinch start :

```csharp
modeManager.SetMode(PlayerMode.ObjectInteraction);
selectionTask.isTaskStart = true;
selectionTask.isTaskEnd = false;
selectionTask.StartOneTask();
```

So now entering the zone is just “ready state”, pinch is the real “go”.

![ParkourControl](parkourcontrol.png)

---

## The most annoying part: locomotion wouldn’t stop

Even after starting interaction, my avatar was still moving because… of course it was.

I originally disabled “Player Locomotor”, but my actual movement logic is in my big custom script `LocomotionTechnique` attached to **OVRCameraRig**. So locomotion kept reading hand input and pushing the Rigidbody.

The fix was very simple in hindsight:

* put **LocomotionTechnique** inside `PlayerModeManager → locomotionBehaviours[]`
* put `ObjectTInteractionController` inside `interactionBehaviours[]`

Now switching mode actually disables the right components. This took the majority of my time during the third week since fugure out th logic was not easy. :D

Also, I added a small “stop momentum” so the avatar doesn’t keep sliding after the switch.

---

## Interaction controls: make it feel POV-based, not world-based

The original swirl translation idea worked… but it was not precise and it felt one-direction-y.

So I changed the control mapping to something that makes more sense to us mere mortals:

### Left hand (rotation)

* **Left index pinch + rotate wrist**
* rotation is slightly adjusted in camera yaw space so it feels POV aligned (less “why is it rotating like that?”)

### Right hand (translation)

* **Right index pinch + tilt** moves the object in XZ plane relative to my POV

  * tilt forward = push away 
  * tilt back = pull towards me 
  * tilt right = move right 
* **Right middle pinch + lift/drop hand** moves the object up/down (Y axis)

And because wobble was a thing, I added smoothing + deadzone, so tiny wrist noise doesn’t jiggle the object constantly.

---

## Finishing the task

I also needed a clean way to end the task and go back to locomotion.

I didn’t want a button UI, because:

* I’m already using hands for movement
* UI clicking in VR is pain
* I just want something reliable

So I added:
  **hold BOTH index pinches for approx 1s** to finish

This calls:

* `selectionTask.EndOneTask()`
* switches mode back to locomotion

The essential part :

```csharp
if (selectionTask) selectionTask.EndOneTask();
if (modeManager) modeManager.SetMode(PlayerMode.Locomotion);
```

Now the task loop feels complete:
    1. start 
    2. manipulate 
    3. finish 
    4. return

---

## One tiny change that made it so much better: hide the avatar

My avatar body was blocking the view while doing the puzzle. Like, my own shoulders were the final boss like in dark souls where the camera is sometimes the villain.

So during interaction mode I hide the avatar renderers (not the collider / rigidbody, just the visuals).
That alone made the puzzle feel *way* less claustrophobic.

{{< video src="t-shape.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

## Tiny bug fix but took me a day to figure out

my avatar didn’t come back after exiting the puzzle, it was as if it completely disappeared even tho I could move normally with my two hand gestures. This was because the interaction script got disabled before it could re-enable the renderers. 
Adding 
```csharp
OnDisable() 
{ 
    SetAvatarVisible(true); 
} 
```
fixed it instantly.

---

## Where things stand now

The interaction task is finally real and playable:

* enter checkpoint → UI appears
* pinch → spawns object + target
* locomotion disables cleanly
* I can rotate + translate the T piece precisely
* hold both pinches → task ends and locomotion returns
* avatar doesn’t block my view anymore

This was one of those hard weeks, but it’s also the first time the project feels like kinda close to the finish line.

Next up: making the interaction feel even more natural, and cleaning up the code.

Cheers, 
Ajay


---

