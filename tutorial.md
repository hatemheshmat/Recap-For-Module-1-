## Part 2 — Add a Character body (for movement & collisions) 🏃

Now, we'll set up the CharacterController which acts as the player's physical body for smooth movement and physics.

**1. Create Player Parent:**
   - Create a new empty GameObject and name it `Player`. This object will be the root of your entire player rig.

**2. Parent the Rig:**
   - Drag your `OVRCameraRig` onto the `Player` object to make it a child.

**3. Reset Transforms:** This is critical!
   - Select the `Player` object and reset its Transform component to position `(0, 0, 0)`.
   - Select the child `OVRCameraRig` and reset its Transform as well.

**4. Add Player Controller:**
   - Select the parent `Player` object.
   - Click `Add Component` and add the `OVR Player Controller`.

**5. Configure Character Controller:**
   - Adding the `OVR Player Controller` also adds a `Character Controller`. This component represents your collision capsule. Set its values exactly like this to avoid falling or being too tall:
     - **Radius:** `0.1`
     - **Height:** `1.5`
     - **Center Y:** `0.75` (This must be half of the Height!)

At this point, you have a working continuous locomotion setup. The left thumbstick will move you, and the right will snap turn.

## Part 3 — Teleportation (Meta Interaction SDK way) ✨

Next, we'll add the components for teleportation. We will configure it to use only the right controller to avoid conflicts with the left stick's movement.

**1. Add Interactor:**
   - In the Project window, find the `LocomotionControllerInteractor` prefab.
   - Drag it into the `Controller Interactors` slot on the `RightController` object (found under `OVRCameraRig > OVRInteraction > OVRControllers`).

**2. Create Input Feature:**
   - Under the `RightController`, create an empty GameObject and name it `ControllerFeature`.
   - Add the `OVRAxis2D` component to it.
   - Set **Controller** to `Right Touch`.
   - Set **2D Axis** to `Primary Thumbstick`.

**3. Wire Up Input:**
   - Select the `RightController` again.
   - Expand the `LocomotionControllerInteractor` you added. You need to drag your new `ControllerFeature` object into several slots:
     - `TeleportationInteractor > TeleportationActive > ActiveState > Active`
     - `TeleportationInteractor > InputAxis`
     - `TeleportationInteractor > Selector`
     - `ControllerTurnInteractor > 2D Axis`

**4. Create Locomotion Handler:**
   - Right-click in the Hierarchy and create a new empty GameObject named `Locomotion`.
   - Add the `PlayerLocomotor` component to it.

**5. Making It All Work Together:**
   - This is the most important step where we connect the teleport system to our `CharacterController` body.

**6. Connect the Player Body:**
   - Select your `Locomotion` GameObject. Look at the `Player Locomotor` component.
     - For the **Player Origin** field, drag in your main `Player` parent object. This tells the teleport system to move your entire character controller, not just the camera.
     - For the **Player Head** field, find the `CenterEyeAnchor` (under `OVRCameraRig`) and drag it in.

**7. Connect the Handler:**
   - Select the `RightController` again.
   - Expand the `LocomotionControllerInteractorGroup`.
   - Drag your `Locomotion` GameObject into the `Handler` field.

**8. Make the Floor Teleportable:**
   - Select your `Large Room` (or floor object).
   - Add the `TeleportInteractable` component.
   - Add the `PlaneSurface` component.
   - Drag the `PlaneSurface` from the same object into the `Surface` slot on the `TeleportInteractable`.

**9. Final Test and Common Fixes:**
   - You're all set! Save your scene and press Play.
     - Your left thumbstick should provide smooth, continuous movement.
     - Your right thumbstick up/down will now aim the teleport ray. Releasing it will teleport you.
     - Your right thumbstick left/right will still provide snap turning.

**If you are still having problems:**
- **Falling Down?** Your floor's collider is the problem. Double-check that it exists, `Is Trigger` is off, and its green wireframe is positioned correctly, not halfway through the floor. A quick fix is to raise your `Player`'s Y-position to `0.1`.
- **Too Tall?** Your `CharacterController`'s `Center Y` value is wrong. It absolutely must be exactly half of the `Height`.
- **Rotation feels off?** Select the `Player` object and check the `Rotation Around Guardian` box on the `OVR Player Controller`.
