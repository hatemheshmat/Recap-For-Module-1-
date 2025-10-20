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


# **Lab-Guide — Grabbing + Ray Interaction (Meta XR AIO v65+ • Unity 6.x • URP)**

**Stack:** Meta XR **only** (no OpenXR), **no custom scripts**, **Interaction SDK** prefabs/wizards/components

---
---

## 🔹  Grabbing (Meta Interaction SDK “Building Blocks”)**

Everything below uses **prefabs/wizards/components only** — no scripts.

### ✅ **To-Do Checklist (with why)**

### 1) **Add the ready camera rig & interactions (one click)**

* ⬜ **Oculus/Meta → Tools → Building Blocks → Grab Interaction**
  *(or: **Assets/Samples/Meta XR…/Interaction SDK/Prefabs →** Camera Rig / Interactions)*

  * • Creates **OVRCameraRig + OVRManager**, **TrackingSpace**, **Hand Interactions (L/R)**, **Controller Interactions (L/R)**.
  * • Hands & controllers include the **correct Interactors** wired for grab.
  * *(You can delete any demo cube they add — it’s just for quick testing.)*

---

### 2) **Create your own Grabbable (from scratch)**

* ⬜ **GameObject → 3D Object → Cube** → **Reset** → **Scale = (0.1, 0.1, 0.1)** → lift slightly above floor

  * • Tiny = easy to test, won’t clip the ground.
* ⬜ **Add Component → Rigidbody**

  * **Use Gravity = OFF** *(common for props that shouldn’t drop before first grab)*
  * **Is Kinematic = ON** *(stable while held; release behavior is configurable)*
* ⬜ **Right-click the cube → Interaction SDK → Add Grab Interaction** *(Wizard)*

  * Choose **Interactors**: **Hands**, **Controllers**, or **Both**
  * Choose **Grab Types**: **Pinch**, **Palm**, or **Both**
  * Let the wizard **Generate Collider** / **Fix Rigidbody** if needed
  * **Create**
  * **Wizard adds (and links):**

    * **Grabbable** (core selection/ownership state)
    * **Hand Grab Interactable** (for **hands**)
    * **Grab Interactable** (for **controllers**)

**Why two interactables?**

* **Hand Grab Interactable** → hand poses, poke/touch affordances.
* **Grab Interactable** → controller inputs (grip/trigger), ray distance logic.
* Having **both** = grab with **either** hands or controllers.

---

### 3) **One-hand grab (baseline)**

* ⬜ On the object’s **Grabbable** → **Add Component → Grab Free Transformer**
* ⬜ **Grabbable → Optionals → One Grab Transformer = Grab Free Transformer**
* ⬜ *(Optional)* **Uncheck “Transfer On Second Selection”**

  * • Prevents auto-handoff if the other hand touches it.

**Concept:** **Transformer** decides how the object **moves/rotates/scales** while selected.

---

### 4) **Two-hand grab**

* ⬜ Keep **Grab Free Transformer** on the object
* ⬜ **Grabbable → Optionals → Two Grab Transformer = Grab Free Transformer**
* ⬜ **Uncheck “Transfer On Second Selection”**

  * • Requires both hands to stay on it (nice for “heavy” props).

---

### 5) **One-hand grab + two-hand scaling**

* ⬜ **Grab Free Transformer** in **both** fields (One & Two Grab Transformer)
* ⬜ In the transformer:

  * **Min Scale / Max Scale** *(e.g., 1.0 → 2.2)*
  * **Constraints Are Relative = ON**
  * • “Relative” keeps scaling intuitive when your model isn’t at scale 1.

---

### 6) **Constrain to a plane (slide on a surface)**

* ⬜ **Grab Free Transformer**:

  * **Lock Rotation (X/Y/Z)** as needed
  * **Allow Position** only on axes you want (e.g., **X & Z**)
  * **Constraints Are Relative = ON**
  * • Great for sliders/table-top items; won’t drift off plane.

---

### 7) **Constrain to an axis (door hinge)**

* ⬜ **Create parent** *Door* GO; make mesh a **child**
* ⬜ **Create child** *Hinge* at **exact pivot** (use **V** vertex-snap)
* ⬜ **Add Component → One Grab Rotate Transformer** *(on interactable)*
* ⬜ **Grabbable → Optionals → One Grab Transformer = One Grab Rotate Transformer**
* ⬜ **Axis = Up** *(typical for a vertical hinge)*
  **Pivot = Hinge**, **Min/Max Angle** *(e.g., −90° → +90°)*

  * • Locks rotation to your hinge for realistic doors/lids.

---

### 8) **Distance grab (three styles)**

* ⬜ **Interaction SDK → Add Distance Grab Interaction** *(Wizard)*

  * **Hands**, **Controllers**, or **Both**
  * **Mode**:

    * **Hand-relative** (follows with offset)
    * **Pull to hand** (snaps to grip)
    * **Manipulate in place** (stays put; local move near hit point)
  * *(Optional)* **Timeout Snap Zone** (auto-return after release)
  * **Create**
  * • Wizard adds **Distance Hand/Controller Interactors** and the **ISDK Distance…** component on your object if missing.

---

### 9) **Touch grab (surface touch → grab)**

* ⬜ **Project search:** `OVR Touch Hand Grab Interactor` *(prefab)*
* ⬜ Drop one under each **Hand Interactions (L/R)** in your rig
* ⬜ On your **object**:

  * **Rigidbody** *(Kinematic / no gravity recommended while held)*
  * **Grabbable** *(link Rigidbody; usually keep **Transfer On Second Selection = ON** for natural pass)*
  * **Touch Hand Grab Interactable**
  * **Create child GO → “Bounds”** → copy your collider(s) here → **Is Trigger = ON**
  * In **Touch Hand Grab Interactable**:

    * **Pointable Element = Grabbable**
    * Add the **Bounds** triggers to its **Colliders** list
  * • Touch-based grabbing is perfect for knobs, sliders, small props.

---

### 10) **Hand poses (realism)**

* ⬜ Record **Grab Poses** per object/hand if fingers clip or look awkward

  * • Makes contact look natural (no fingers through mesh).

---

## 🛠️ **Ray Interaction (3D objects + World-Space UI)**

> **Interactor** = on hand/controller (source)
> **Interactable** = on object/UI (target)

### A) **Controller Ray Interactors**

* ⬜ **Project search → Controller Ray Interactor**
* ⬜ Add to both:

  * `OVRCameraRig/TrackingSpace/LeftHandAnchor/ControllerInteractors`
  * `OVRCameraRig/TrackingSpace/RightHandAnchor/ControllerInteractors`
* ⬜ If your rig uses **Best Hover / Interactor Group**, add each Ray Interactor there.
* ⬜ **Visual polish on each ray**

  * **Max Ray Length ≈ 6–8m**
  * **Hide When No Interactable = ON** *(cleaner UX)*

---

### B) **Ray-grabbable 3D object**

* ⬜ **Create** Cube `RayObject` → **Scale = 0.1** → lift to y ~ 0.5
* ⬜ **Right-click → Interaction SDK → Add Ray Grab Interaction** *(Wizard)*

  * **Fix All** to add **Rigidbody** (often **Use Gravity OFF**, **Kinematic ON**)
  * **Create** → adds an **ISDK Ray Grab** host with:

    * **Ray Interactable**
    * Default **Movement Provider** = **Move From Target Provider**
* ⬜ *(Optionally change movement)*:

  * Remove **Move From Target Provider**
  * **Add Component → Move Towards Target Provider**
  * On **Ray Interactable → Optionals → Movement Provider = Move Towards Target Provider**

**Movement Provider tip:**

* **Move From Target** → keeps offset at the hit point (feels like “drag from hit”).
* **Move Towards Target** → object travels to controller (nice for “bring to hand”).
* *(Other providers exist in the package for specialized feels.)*

---

### C) **Ray-interactable World-Space UI**

* ⬜ **GameObject → UI → Canvas** → rename `UI_Canvas`

  * **Render Mode = World Space**
  * **Scale = (0.001, 0.001, 0.001)**
  * **Position ≈ (0, 1.5, 1.6)**, **Size (W=100, H=50)** (adjust to taste)
* ⬜ Add a **Panel** and a **Button (TextMeshPro)** (e.g., “Press”)
* ⬜ **Right-click `UI_Canvas` → Interaction SDK → Add Ray Interaction to Canvas**

  * Wizard will warn **“No Pointable Canvas Module”** → click **Fix**

    * EventSystem gets the correct XR UI module
  * Click **Create** (adds the helper under canvas)
* ⬜ *(Optional)* Put canvas on **UI** layer and include **UI** in Ray Interactor masks.

---

### D) **Mask rays to different targets (Tag Set Filter)**

*(Example: Left ray = 3D only, Right ray = UI only)*

* ⬜ **On Left Controller Ray Interactor**:

  * **Add Component → Tag Set Filter**
  * **Required Tag =** `3DObject` • **Exclude Tag =** `UICanvas`
* ⬜ **On Right Controller Ray Interactor**:

  * **Add Component → Tag Set Filter**
  * **Required Tag =** `UICanvas` • **Exclude Tag =** `3DObject`
* ⬜ **On the 3D object’s ISDK Ray child**:

  * **Add Component → Tag Set** → **Tag =** `3DObject`
* ⬜ **On the UI Canvas’s ISDK Ray child**:

  * **Add Component → Tag Set** → **Tag =** `UICanvas`
* ⬜ **IMPORTANT:** On each **Ray Interactor → Optionals → Interactable Filters (+)**
  Drag its **own Tag Set Filter** into this list.

  * • Without this reference, the filter won’t apply.

---

## 🎮 **Playtest (quick paths)**

**Grabbing**

* Press **Play** (Quest via Link/Air Link).
* Reach and **grab** your cube with **hand** or **controller**; release/rehab; try **two-hand scale** if enabled; slide across **plane constraint** objects; rotate **door/hinge**.

**Ray → 3D**

* Aim controller **ray** at the `RayObject`; **select**.
* **Move From Target** feels like dragging at hit point; **Move Towards Target** flies to you.

**Ray → UI**

* Aim **Right** ray at **UI_Canvas** → click **Button**.
* **Left** ray should **ignore** UI if masked, and vice-versa.

**Hide When No Interactable**

* Rays should appear **only** when pointed at a valid target.

---

## 🚧 **Troubleshooting (fast fixes)**

**Nothing grabs**

* ⬜ **Grabbable** exists and **Rigidbody** is linked
* ⬜ At least one of **Hand Grab Interactable / Grab Interactable** is present
* ⬜ Appropriate **Interactor** exists on the hand/controller

**Object jitter / unstable while held**

* ⬜ Keep **Rigidbody Is Kinematic = ON while held** (default via wizard)
* ⬜ Avoid heavy physics on small props

**Two-hand scaling not working**

* ⬜ **Grab Free Transformer** is assigned to **both** One & Two Grab Transformer
* ⬜ **Transfer On Second Selection = OFF**

**Door won’t pivot correctly**

* ⬜ **Hinge pivot** exactly at the desired axis (use **V** snap)
* ⬜ **One Grab Rotate Transformer**: Axis/Pivot/Angles are set

**Ray doesn’t interact with object**

* ⬜ **Ray Interactable** present on the object
* ⬜ **ColliderSurface**/**Movement Provider** assigned if your prefab expects them
* ⬜ Layer mask includes the object’s layer

**Ray doesn’t interact with UI**

* ⬜ **Canvas = World Space**
* ⬜ EventSystem has **XR/Pointable Canvas** module (wizard “Fix”)
* ⬜ Canvas not microscopically scaled (≈ **0.001** is good)
* ⬜ Ray’s mask includes the **UI** layer (if used)

**Ray masking not working**

* ⬜ **Tag Set** on the interactable child matches the exact **Required Tag**
* ⬜ Ray Interactor → **Interactable Filters** includes the **Tag Set Filter**
* ⬜ Spelling/case match for tag strings

**Ray always visible**

* ⬜ Enable **Hide When No Interactable** on the ray’s **Line Visual**

---

## 🗂️ **What your Hierarchy should look like (typical)**

```
OVRCameraRig
└─ TrackingSpace
   ├─ LeftHandAnchor
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor   (Tag Set Filter: 3D only, optional)
   │     └─ Grab Interactor
   └─ RightHandAnchor
      └─ ControllerInteractors
         ├─ Controller Ray Interactor   (Tag Set Filter: UI only, optional)
         └─ Grab Interactor
Interactions
└─ Hand Interaction
   ├─ Left Hand Synthetic   (Touch Hand Grab Interactor, optional)
   └─ Right Hand Synthetic  (Touch Hand Grab Interactor, optional)

Ground (Collider)
Large Room (optional)
Directional Light
EventSystem (with XR/Pointable Canvas module)
```

**Objects**

```
ISDK Hand Grab Interaction (wizard host for your cube)
├─ Cube (mesh + collider)
└─ [components on host]  Grabbable + Hand Grab Interactable + Grab Interactable
                         Grab Free Transformer (assigned in One/Two Grab)
                         (Distance Grab components if added)
                         (One Grab Rotate Transformer for door/hinge cases)
```

**Ray object**

```
RayObject (Cube)
└─ ISDK Ray Grab Interaction (wizard host)
   └─ Ray Interactable + Movement Provider (From Target / Towards Target)
```

**UI**

```
UI_Canvas (World Space)
├─ Panel
└─ Button (TMP)
└─ [child helper from wizard] ISDK Ray Interaction (canvas)
```

---

## ✅ **End-of-Session Validation**

* ⬜ **Hands & controllers** visible; grab works with **either** input (if both added)
* ⬜ **One-hand** grab moves cleanly; **Two-hand** grab/scaling works (if configured)
* ⬜ **Plane constraint** object slides only on allowed axes
* ⬜ **Door/hinge** rotates only around its pivot within Min/Max angles
* ⬜ **Ray → 3D**: object responds; movement provider behavior matches your choice
* ⬜ **Ray → UI**: world-space button is clickable
* ⬜ **Masking**: Left ray hits **3D only**, Right ray hits **UI only** (if set)
* ⬜ **Hide When No Interactable**: rays appear only over valid targets
* ⬜ **Perf**: SP-Instanced, URP tuned; smooth 72–90 FPS scene

---

## 📋 **Copy-paste Micro-Checklist**

**Scene**

* [ ] Large Room or Plane ground (with Collider)
* [ ] Directional Light

**Rig & Managers**

* [ ] OVRCameraRig + OVRManager
* [ ] Interactions: Hand + Controller blocks (from Building Blocks)
* [ ] EventSystem (with XR/Pointable Canvas module if using UI)

**Grabbable**

* [ ] Rigidbody (Kinematic, Gravity OFF for props)
* [ ] Grabbable (Rigidbody linked)
* [ ] Hand Grab Interactable + Grab Interactable
* [ ] Grab Free Transformer (One & Two Grab)
* [ ] (Optional) One Grab Rotate Transformer + Hinge pivot

**Distance/Touch**

* [ ] Distance Grab Interaction (wizard) if needed
* [ ] Touch Hand Grab Interactor (hands) + Touch Hand Grab Interactable (object)
* [ ] Bounds triggers for touch

**Rays**

* [ ] Controller Ray Interactor (L/R), MaxLen ~6–8m
* [ ] Hide When No Interactable = ON
* [ ] Ray Interactable on 3D targets + Movement Provider

**UI**

* [ ] Canvas = World Space, Scale ≈ 0.001
* [ ] Button to test
* [ ] Add Ray Interaction to Canvas (wizard “Fix” done)

**Masking (optional)**

* [ ] Tag Set Filter on each Ray Interactor
* [ ] Tag Set on each target (3DObject / UICanvas)
* [ ] Interactable Filters list includes Tag Set Filter

**Perf**

* [ ] Single-Pass Instanced, Foveation Low/Med
* [ ] URP: HDR OFF, MSAA 4x, Shadows 512/20–25m/2 casc., PostFX OFF

---

