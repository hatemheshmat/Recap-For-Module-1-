# Recap-For-Module-1

# Session-3 — **Locomotion: Teleport + Snap Turn**

*(Unity 6.2 • URP • Meta XR AIO v65+, Meta/Unity prefabs & components only)*

**Objective**
Add movement to the player rig using Meta’s modern Interaction/Movement stack: set up **teleportation** and **snap-turn**, confirm **guardian** behavior, and introduce **AI Navigation** (NavMesh) for a basic patrolling agent.

---

### ✅ **To-Do Checklist (with why)**

1. **Create the project**

* ⬜ **Unity Hub → New → 3D (URP)**

  * • URP is the most stable + performant path on Quest.

2. **Switch to Android**

* ⬜ **File → Build Settings → Android → Switch Platform**

  * • Quest is Android; switching now avoids later re-imports.
* ⬜ **Texture Compression = ASTC**

  * • Best memory/quality tradeoff on Quest.

3. **Player Settings** *(Project Settings → Player → Other Settings)*

* ⬜ **Package Name** `com.company.app`

  * • Required for Android build/installation.
* ⬜ **Min API Level ≥ 29 (Android 10)**

  * • Matches current Meta requirements.
* ⬜ **Scripting Backend = IL2CPP**, **ARM64 only**

  * • Required for store + best perf.
* ⬜ **Graphics API = Vulkan** *(uncheck GLES3 unless you have a reason)*

  * • Usually faster on Quest with modern SDKs.

4. **Install Meta XR packages (no OpenXR)**

* ⬜ **Window → Package Manager → Unity Registry**

  * ✅ **XR Plug-in Management**
  * ✅ **Meta XR (a.k.a. Oculus) plugin provider**
  * ✅ **Meta XR All-in-One SDK** (**v65+**)
* ⬜ **Project Settings → XR Plug-in Management → Android**

  * ✅ **Enable Meta XR**
  * ❌ **Do NOT enable OpenXR** (we’re Meta-only here)
  * • Keeps the stack simple and avoids mixed-runtime warnings.

5. **Let Meta’s “Project Validation” auto-fix**

* ⬜ **Edit → Project Settings → XR Plug-in Management → Project Validation → Fix All**

  * • Applies recommended flags (permissions, input backends, etc.).

6. **URP asset tuning** *(Project: select your **Mobile_RPAsset**)*

* ⬜ **HDR = OFF**
* ⬜ **MSAA = 4x**
* ⬜ **Shadows: 512 (Medium), Distance 20–25m, Cascades = 2**
* ⬜ **Disable** Bloom / Motion Blur / Vignette

  * • Solid 72–90 FPS baseline on Quest.

7. **URP renderer tuning** *(select **Mobile_Renderer**)*

* ⬜ **Post-processing = OFF**
* ⬜ **Transparent Receive Shadows = OFF**
* ⬜ **Renderer Features = (empty)**

  * • Removes heavy passes that hurt mobile VR.

8. **Meta runtime tuning** *(Meta XR provider / OVRManager)*

* ⬜ **Stereo = Single-Pass Instanced**

  * • Renders both eyes in one pass (large GPU win).
* ⬜ **Foveated Rendering = Enabled (Low/Medium)** *(optional)*

  * • Big perf gain; watch peripheral blur.
* ⬜ **Phase Sync / ASW** *(optional)*

  * • Smooth frame pacing; test with your content.

9. **Silence common OVR warnings** *(optional)*

* ⬜ If you see micro-gesture logs → **disable Microgestures** (or update SDK 65+)

  * • “ActionSet not attached” warnings are harmless but noisy if unused.

---

---

## Part 1 — Rig & Floor (one-minute scaffold)

1. **Add the rig**
   **Menu:** *Meta Tools → Building Blocks* → add **Camera** (creates `OVRCameraRig`), then **Controller Tracking**, **Hand Tracking**, **Virtual Hands**.
   This yields a correct Meta **Interaction Rig** (tracking space + anchors + interaction roots). ([Meta Developers][1])

2. **Add a floor**
   **Hierarchy:** 3D Object → **Plane** → rename **Ground** → Position *(0,0,0)* → **Add Component: Box Collider** (Is Trigger **OFF**).
   *(You can substitute the “Large Room” prefab if you have it.)*

---

## Part 2 — Add a **Character body** (for movement & collisions)

> Meta’s locomotion handler moves a **CharacterController** capsule that represents the player’s body. In newer docs it’s the **FirstPersonLocomotor** moving a **CharacterController** while keeping HMD/hand alignment in sync.

1. **Choose the host**

* **Recommended:** Create **Empty** `Player` at *(0,0,0)* → **make `OVRCameraRig` a child** of `Player`.
* *(If your template expects it, you can place the components directly on `OVRCameraRig` instead. Keep transforms reset.)*

2. **Add CharacterController**

* Select **Player** (or **OVRCameraRig** if you’re skipping a parent) → **Add Component → OVR Player Controller**

  * **Radius = 0.10**, **Height = 1.50**, **Center.Y = 0.75** *(must be Height ÷ 2 to sit on the floor)*.

3. **Add the locomotion handler**

* On the same object as the **CharacterController**, **Add Component → FirstPersonLocomotor** *(or “Player/Character Movement” in some packages; it’s the same role)*.

  * This component is the **movement provider** that applies rotation/movement/teleport to the capsule.

> **Tip (comfort):** If you previously saw “rotate around offset,” ensure your **capsule is centered** and transforms are **reset** (0/0/0). That removes the feel of pivoting around a weird point.

---

## Part 3 — **Teleportation** (Meta Interaction SDK way)

> The Interaction SDK’s locomotion stack pairs **controller-side interactors** with a **locomotor** handler. You’ll wire a **TeleportationInteractor** on the controller to the **FirstPersonLocomotor**, and make the floor teleportable.

### A) Add the controller interactor (Right hand)

1. **LocomotionControllerInteractor**

* Expand: `OVRCameraRig/TrackingSpace/RightHandAnchor/ControllerInteractors`
* **Project** search **LocomotionControllerInteractor** prefab → **drag** into **ControllerInteractors** (as a child).

2. **Create an input feature (OVR Axis)**

* Under **RightHandAnchor**, **Create Empty** → **ControllerFeature (R)** → **Add Component → OVRAxis2D**

  * **Controller = Right Touch**
  * **2D Axis = Primary Thumbstick**
    *(This object is referenced by teleport & turn modules as the input source.)*

3. **Wire inputs**

* Select the **LocomotionControllerInteractor (R)** → expand **TeleportationInteractor**:

  * **TeleportationActive → ActiveState → Active** = **ControllerFeature (R)**
  * **StateActive → Deactivate** = **ControllerFeature (R)**
  * **Input Axis** = **ControllerFeature (R)**
  * **Selector** = **ControllerFeature (R)**
* Still on the same interactor, expand **ControllerTurnInteractor**:

  * **2D Axis** = **ControllerFeature (R)**.

### B) Add the locomotion handler object

* **Hierarchy → Create Empty** → **Locomotion** → **Add Component → FirstPersonLocomotor**

  * **Player Origin** = your **Player** (or host of CharacterController)
  * **Player Head** = `OVRCameraRig/TrackingSpace/CenterEyeAnchor`
* On **RightHandAnchor/ControllerInteractors/...InteractorGroup**, set **Handler = Locomotion**.

### C) Make the ground teleportable

* Select **Ground** → **Add Component → TeleportInteractable**
* **Add Component → PlaneSurface** → in **TeleportInteractable**, set **Surface = this PlaneSurface**.
  *(Meta’s docs call out `PlaneSurface`/`TeleportInteractable` for flat floors.)*

**Playtest:**

* Press/tilt **Right thumbstick** to cast the teleport arc; **release** to teleport onto **Ground**.
* Locomotor applies the move to your **CharacterController** capsule (not just the camera), preserving head alignment.

---

## Part 4 — **Snap Turn** (comfort rotation)

> Snap-turn comes from the **ControllerTurnInteractor** using your **OVRAxis2D** input and rotating the **FirstPersonLocomotor**/**CharacterController**.

1. **Open ControllerTurnInteractor (R)**

* **Turn Mode**: **Snap**
* **Snap Angle**: **45°** *(comfort default)*
* **Deadzone**: **0.2–0.3** *(avoid accidental turns)*
* **Cooldown / Debounce**: **~0.25 s** *(prevents rapid repeated snaps)*

2. **(Optional) Left hand**

* Repeat **A–C** for **LeftHandAnchor** if you want both sticks to support teleport/turn.

**Playtest:**

* **Right stick left/right** snaps by **45°**.
* **Right stick up/front** shows teleport arc; **release** to teleport.

---

## Part 5 — Guardian & comfort

* **OVRManager → Boundary / Guardian options**: confirm boundary behavior as desired (visibility/suppression depends on runtime & device settings). Meta exposes boundary controls through **OVRManager**.
* **Floor-level origin**: Ensure tracking origin is floor-relative (typical for room-scale).
* **Teleport comfort**: Use teleport (rather than smooth strafe) as the primary move; keep **snap** (not smooth) turn for first-time users.

---

## Part 6 — AI Navigation (no-code intro)

> You’ll **bake a NavMesh** on the floor and drop a **NavMeshAgent** cube. For a true *patrol* loop without custom code, import the **AI Navigation Samples** and use their patrol example content; otherwise, you can still demonstrate pathing by assigning a single target in-editor during Play.

1. **Install AI Navigation**

* **Window → Package Manager → Unity Registry → AI Navigation** *(if not already installed)*.

2. **Bake a NavMesh**

* Select **Ground** → **Add Component → NavMeshSurface** → **Collect Objects = All** → **Bake**.
  *(NavMesh **Surface/Agent/Obstacle/Link** are the standard authoring components.)*

3. **Create an agent**

* **Hierarchy:** 3D Object → **Cube** → rename **AgentCube** → **Add Component → NavMeshAgent**.
* **Speed = 1.5–2.0**, **Angular Speed = 360**, **Stopping Distance = 0.1**.

4. **Patrol (zero-code options)**

* **Best:** In **Package Manager → AI Navigation → Samples**, import the **Examples** and drop the sample **patrol agent**/waypoint setup into your scene; replace the model with **AgentCube**. *(Samples vary by version but include working agent behaviors.)*
* **Fallback demo:** Create two **Empty** transforms **WayA / WayB** on the baked floor. In Play mode, set the agent’s **Destination** from the Inspector to WayA, then to WayB to demonstrate pathfinding and obstacle avoidance.

---

## Quick Test (2 minutes)

* **Teleport:** Right stick → aim → release → land on **Ground**.
* **Snap turn:** Right stick **left/right** → 45° increments.
* **Guardian:** Boundary behaves as expected (visible/suppressed per OVRManager/runtime).
* **NavMesh:** AgentCube moves across the baked floor toward the chosen waypoint (or patrols if you used the sample).

---

## Student Task (Build)

1. Select **OVRCameraRig** *(or `Player` root if you created one)* → **Add Component: Character Controller**; set **Radius 0.10 / Height 1.50 / Center.Y 0.75**.
2. Add **FirstPersonLocomotor** *(a.k.a. “CharacterMovement” in Movement SDK templates)* to the same object; it coordinates movement & teleport/turn modules.
3. On **RightHandAnchor**: add **LocomotionControllerInteractor** and **ControllerFeature (OVRAxis2D)**; wire **TeleportationInteractor** and **ControllerTurnInteractor** to the feature. Set **Turn Mode = Snap**, **Snap Angle = 45°**.
4. On **Ground**: add **TeleportInteractable + PlaneSurface**; set **Surface** reference.
5. **AI Navigation**: Add **NavMeshSurface** to Ground → **Bake**. Create **AgentCube** with **NavMeshAgent**. Use **AI Navigation Samples** patrol content (no code) or demonstrate by switching **Destination** between two waypoints in Play.

---

### What your Hierarchy should look like now (essentials)

```
Player  (Character Controller + FirstPersonLocomotor)
└─ OVRCameraRig
   └─ TrackingSpace
      ├─ LeftHandAnchor
      │  └─ ControllerInteractors
      │     ├─ Controller Ray Interactor
      │     └─ Grab Interactor
      └─ RightHandAnchor
         └─ ControllerInteractors
            ├─ Controller Ray Interactor
            ├─ Grab Interactor
            └─ LocomotionControllerInteractor  (↔ ControllerFeature (R): OVRAxis2D)
Locomotion  (FirstPersonLocomotor handler, if separate)
Ground  (Box Collider + NavMeshSurface + TeleportInteractable + PlaneSurface)
AgentCube  (NavMeshAgent)
Directional Light
EventSystem
```

https://developers.meta.com/horizon/documentation/unity/unity-isdk-interaction-sdk-overview/ "Meta for Developers"
https://developers.meta.com/horizon/documentation/unity/unity-isdk-create-teleport-hotspot
https://developers.meta.com/horizon/documentation/unity/unity-isdk-create-teleport-physics-layer
https://developers.meta.com/horizon/documentation/unity/unity-isdk-create-teleport-navmesh
[https://developers.meta.com/horizon/documentation/unity/unity-isdk-create-teleport-plane
