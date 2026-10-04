# Catatan-dan-Update-PKDN

# Unity Project Context — Desa Wisata Pete Golf Cart Auto Tour
## Master Context — Updated 4 October 2026

---

# 1. Project Overview

I am working on a Unity project for **Desa Wisata Pete**, a tourist village simulation with an automatic golf-cart tour system.

### Unity

- **Unity version:** 2022.3.62f1
- **Target platforms:**
  1. Desktop
  2. VR / WebXR

### Original development priority

1. Functional system
2. Reliable Editor/Desktop testing
3. Integration with teammate UI
4. WebXR/browser testing
5. Visual polish

I am responsible primarily for the **system/functional logic**, while teammates are handling other aspects such as POV/UI.

### Important development rule

> **The existing golf-cart and map-board systems are already functional. Do not unnecessarily rewrite/rebuild them unless a new problem actually requires it.**

More generally:

> **Preserve working systems. Make the smallest targeted change necessary to solve the current problem.**

---

# 2. Golf Cart Route System

The golf cart follows a predefined route using a **doubly linked list** concept.

Each route node has:

```text
previous
next
```

There is **no branching** in the route.

### Node types

The route-node script uses:

```csharp
public enum NodeType
{
    Border,
    Node,
    Waypoint
}
```

The current actual node component is represented by `TourRouteNode`, containing approximately:

```csharp
public class TourRouteNode : MonoBehaviour
{
    public NodeType nodeType;
    public int WaypointID = -1;

    public TourRouteNode previous;
    public TourRouteNode next;
}
```

Older project notes sometimes referred to this generally as the `RouteNode` system.

### Current waypoint destinations

| ID | Destination |
|---|---|
| 1 | Masjid Syekh Mubarok |
| 2 | Kebun Edukasi |
| 3 | Parkir & UMKM |
| 4 | BUMDes |

The route contains multiple nodes along the road, but the old **Sumur Keramat** waypoint was skipped.

---

# 3. GolfCartController

The golf-cart controller manages:

- Current node
- Target node
- Movement
- Destination
- Forward/backward movement through the linked route

Important fields include:

```csharp
public TourRouteNode startNode;

public int destinationWaypointID = 2;
public int selectedDestination = -1;

public float moveSpeed = 5f;
public float rotationSpeed = 5f;
public float reachDistance = 0.3f;
```

The controller tracks:

```csharp
private TourRouteNode currentNode;
private TourRouteNode targetNode;

private bool isMoving = false;
private bool movingForward = true;
```

Movement state is exposed through:

```csharp
public bool IsMoving
{
    get { return isMoving; }
}
```

The tour starts using:

```csharp
StartTour(destinationID);
```

### Important behavior

The golf cart does **not** automatically start moving just because a destination was selected.

Destination selection is stored in:

```csharp
golfCart.selectedDestination
```

The cart starts when the player actually boards/interacts with it.

The system also supports continuing from the cart's current location rather than resetting to the beginning of the route.

---

# 4. Golf Cart Startup Behaviour

At startup:

```csharp
currentNode = startNode;

if (currentNode != null)
{
    transform.position = currentNode.transform.position;
}
```

The previous automatic:

```csharp
StartTour(...)
```

startup behavior was removed/commented out.

Therefore:

```text
Scene starts
    ↓
Golf cart placed at startNode
    ↓
Golf cart waits
```

It does not begin a tour until requested through the interaction system.

---

# 5. Golf Cart Route Selection

When:

```csharp
StartTour(int waypointID)
```

is called:

1. `destinationWaypointID` is updated.
2. The controller checks whether the cart is already at the destination.
3. `IsDestinationAhead()` searches through `.next`.
4. If the destination exists ahead:
   ```text
   movingForward = true
   ```
5. Otherwise:
   ```text
   movingForward = false
   ```
6. `targetNode` becomes either:
   ```text
   currentNode.next
   ```
   or:
   ```text
   currentNode.previous
   ```
7. Movement begins.

This allows the golf cart to travel both forward and backward through the doubly linked route.

---

# 6. Golf Cart Movement

While moving:

```csharp
Vector3 direction =
    targetNode.transform.position -
    transform.position;
```

Vertical direction is removed:

```csharp
direction.y = 0f;
```

The cart rotates toward the target using:

```csharp
Quaternion.LookRotation(...)
Quaternion.Slerp(...)
```

Movement uses:

```csharp
Vector3.MoveTowards(...)
```

When sufficiently close to the target:

```csharp
currentNode = targetNode;
```

If the current node is the requested waypoint:

```csharp
isMoving = false;
targetNode = null;
```

Otherwise, movement continues to:

```text
currentNode.next
```

or:

```text
currentNode.previous
```

depending on travel direction.

---

# 7. Destination Selection Flow

The intended gameplay flow is:

```text
Player walks around
        ↓
Player approaches physical map board
        ↓
Press E
        ↓
Destination UI appears
        ↓
Player selects destination
        ↓
Destination is stored in GolfCartController
        ↓
Destination UI closes
        ↓
Player walks to golf cart
        ↓
Press E
        ↓
Player boards cart
        ↓
Golf cart starts toward selected destination
        ↓
Cart reaches destination
        ↓
Player can press F to exit
        ↓
Player can return to map
        ↓
Select another destination
        ↓
Repeat
```

### Important behavior

The destination UI does **not** appear when boarding the cart.

The destination is chosen beforehand using the physical map board.

### Intended complete gameplay loop

```text
Walk to map
    ↓
Pick destination
    ↓
Go to golf cart
    ↓
Tour starts
    ↓
Reach destination
    ↓
Get off
    ↓
Return to map
    ↓
Pick new destination
    ↓
OR finish
```

---

# 8. DestinationUI

`DestinationUI.cs` controls the destination-selection panel.

Important references include:

```csharp
destinationPanel
GolfCartController golfCart
```

Important methods include:

```csharp
ShowMenu()
HideMenu()
```

Destination buttons currently map to:

```text
GoToMasjid()
→ selectedDestination = 1

GoToKebun()
→ selectedDestination = 2

GoToUMKM()
→ selectedDestination = 3

GoToBUMDes()
→ selectedDestination = 4
```

Selecting a destination closes the menu.

`Cancel()` also closes the destination menu.

---

# 9. Map Board System

The physical map board and `MapBoardInteraction` are already complete and functional.

Current behavior:

- Player approaches map board
- Presses E
- Destination menu opens
- Destination can be selected
- Selected destination is stored
- Player then walks to the golf cart

Important fields include approximately:

```csharp
DestinationUI destination;
Transform player;
float interactionDistance = 3f;
```

If the destination panel is already active, interaction processing returns early.

When the player is within range and presses E:

```csharp
destination.ShowMenu();
```

### Important

> **Do not restart or unnecessarily rewrite this system.**

---

# 10. GolfCartInteraction

Current `GolfCartInteraction.cs` is responsible for:

- Detecting player near golf cart
- Boarding with E
- Moving player to `seatPoint`
- Disabling `DesktopMovement` while riding
- Starting the selected tour
- Keeping player at the seat while riding
- Allowing exit only after the cart stops
- Exiting with F
- Moving player to `exitPoint`
- Re-enabling `DesktopMovement`

Current important structure:

```csharp
public GolfCartController golfCart;
public DestinationUI destinationUI;

public Transform player;
public float interactionDistance = 3f;

public Transform seatPoint;
public Transform exitPoint;
```

It obtains/references:

```csharp
DesktopMovement desktopMovement;
```

It checks:

```csharp
golfCart.selectedDestination
```

before starting the tour.

If no destination has been selected:

```text
No destination selected!
```

is logged instead.

---

# 11. Golf Cart Boarding

The current intended sequence is:

```text
Destination selected at map
    ↓
selectedDestination stored
    ↓
Player walks to cart
    ↓
Press E
    ↓
Player moves to seatPoint
    ↓
isRiding = true
    ↓
DesktopMovement disabled
    ↓
StartTour(selectedDestination)
```

While riding:

```csharp
player.position = seatPoint.position;
player.rotation = seatPoint.rotation;
```

The player therefore remains attached to the cart seat while the cart moves.

Older trigger/menu logic in `GolfCartInteraction` is commented out and is not part of the current working flow.

---

# 12. Golf Cart Exit

Exit is only allowed when:

```csharp
!golfCart.IsMoving
```

and the player presses:

```text
F
```

Then:

```text
isRiding = false
player → exitPoint
DesktopMovement → enabled
```

This behavior is currently functional.

---

# 13. Current Player / Camera Hierarchy

The important First Person hierarchy is approximately:

```text
WebXRCameraSet
│
└── Main Camera
```

## WebXRCameraSet

Current Transform:

### Position

```text
X = 81.7
Y = 0
Z = -48
```

### Rotation

```text
X = 0
Y = 30.3
Z = 0
```

### Scale

```text
X = 1
Y = 1
Z = 1
```

It has:

- Transform
- `LinkedAliasAssociationCollection`

The Linked Alias Association Collection contains WebXR references such as:

```text
Play Area:
CameraRigs.TrackedAlias

Headset:
Main Camera

Headset Camera:
Main Camera (Camera)
```

There is no additional movement/camera controller on `WebXRCameraSet`.

Therefore, the **30.3° Y rotation** of `WebXRCameraSet` must be accounted for when calculating First Person movement.

---

# 14. Main Camera

Main Camera currently has approximately:

### Position

```text
X = 0
Y = 1.8
Z = -2
```

### Rotation

```text
X = 0
Y = 0
Z = 0
```

### Scale

```text
1, 1, 1
```

It is directly under:

```text
WebXRCameraSet
```

Main Camera is tagged:

```text
Player
```

This was important because the original interaction system was referencing the wrong player/camera object.

Main Camera has:

- Camera
- Tracked Pose Driver
- Web XR Camera Settings
- Audio Listener
- `DesktopMovement`

### CharacterController

A `CharacterController` is currently created dynamically by `DesktopMovement` if one does not already exist.

---

# 15. Previous DesktopMovement Issue — SOLVED

The previous major issue was First Person camera control.

Originally:

- Horizontal mouse movement worked
- Vertical mouse movement was added
- W/S movement then became strange/diagonal
- Movement sometimes felt like it was fighting against walls
- Camera movement felt unusual

A temporary test using:

```csharp
controller.detectCollisions = false;
```

did **not** solve the problem.

Therefore, collision was not the primary cause.

The real issue was a **coordinate-space / rotation-variable problem**, especially because Main Camera is a child of `WebXRCameraSet`, whose Y rotation is `30.3°`.

The final corrected version solved the issue.

---

# 16. CURRENT WORKING `DesktopMovement.cs`

This is the latest confirmed-working First Person implementation:

```csharp
using UnityEngine;

public class DesktopMovement : MonoBehaviour
{
    public float speed = 3.0f;
    public float sensitivity = 2.0f;

    // limit how far user can look up or down
    public float minLookAngle = -80f;
    public float maxLookAngle = 80f;

    private CharacterController controller;

    private float rotationX = 0; // vertical camera rotation
    private float rotationY = 0; // horizontal camera rotation

    void Start()
    {
        controller = GetComponent<CharacterController>();

        if (controller == null)
        {
            controller = gameObject.AddComponent<CharacterController>();
            controller.height = 2f;
            controller.center = new Vector3(0, 1f, 0);
        }

        // start from camera's current local y rotation
        rotationY = transform.localEulerAngles.y;

        // convert 0-360 range into -180 to 180
        if (rotationY > 180f)
        {
            rotationY -= 360f;
        }
    }

    void Update()
    {
        // =========================
        // MOUSE LOOK
        // =========================

        float mouseX = Input.GetAxis("Mouse X") * sensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * sensitivity;

        // look left / right
        rotationY += mouseX;

        // look up / down
        rotationX -= mouseY;

        rotationX = Mathf.Clamp(
            rotationX,
            minLookAngle,
            maxLookAngle
        );

        // apply camera rotation
        transform.localRotation = Quaternion.Euler(
            rotationX,
            rotationY,
            0f
        );

        // =========================
        // WASD MOVEMENT
        // =========================

        float moveX = Input.GetAxis("Horizontal");
        float moveZ = Input.GetAxis("Vertical");

        Quaternion yawRotation = Quaternion.Euler(
            0f,
            rotationY,
            0f
        );

        Vector3 forward;
        Vector3 right;

        if (transform.parent != null)
        {
            // follow WebXRCameraSet's 30.3 rotation
            forward = transform.parent.TransformDirection(
                yawRotation * Vector3.forward
            );

            right = transform.parent.TransformDirection(
                yawRotation * Vector3.right
            );
        }
        else
        {
            forward = yawRotation * Vector3.forward;
            right = yawRotation * Vector3.right;
        }

        // keep movement horizontal
        forward.y = 0f;
        right.y = 0f;

        forward.Normalize();
        right.Normalize();

        Vector3 move =
            right * moveX +
            forward * moveZ;

        // prevent diagonal movement from being faster
        if (move.magnitude > 1f)
        {
            move.Normalize();
        }

        controller.Move(
            move *
            speed *
            Time.deltaTime
        );
    }
}
```

### Current status

This version has been tested and confirmed to work perfectly.

Current controls:

```text
Mouse X → Look left/right
Mouse Y → Look up/down

W → Forward
S → Backward
A → Left
D → Right
```

Looking up/down does not interfere with horizontal WASD movement.

Diagonal movement is normalized so:

```text
W + A
W + D
S + A
S + D
```

does not move faster than a single direction.

---

# 17. Important Rotation Logic

This is critical if modifying First Person movement later.

The correct relationship is:

```text
Mouse X
   ↓
rotationY
   ↓
Horizontal / Yaw rotation
   ↓
WASD direction
```

and:

```text
Mouse Y
   ↓
rotationX
   ↓
Vertical / Pitch rotation
   ↓
Camera look up/down ONLY
```

Therefore:

```csharp
rotationY += mouseX;
rotationX -= mouseY;
```

is intentional.

### Do not swap them.

Movement uses:

```csharp
Quaternion yawRotation =
    Quaternion.Euler(
        0f,
        rotationY,
        0f
    );
```

rather than the camera's complete rotation.

This prevents looking up/down from affecting movement.

Because Main Camera is under a rotated parent, movement additionally uses:

```csharp
transform.parent.TransformDirection(...)
```

to respect `WebXRCameraSet`'s 30.3° rotation.

---

# 18. CharacterController

Current runtime CharacterController settings observed:

```text
Slope Limit: 45
Step Offset: 0.3
Skin Width: 0.08
Min Move Distance: 0.001
```

### Center

```text
X = 0
Y = 1
Z = 0
```

### Dimensions

```text
Radius = 0.5
Height = 2
```

The CharacterController is currently added at runtime by:

```csharp
GetComponent<CharacterController>();
```

followed by:

```csharp
if (controller == null)
{
    controller =
        gameObject.AddComponent<CharacterController>();

    ...
}
```

### Important

The CharacterController does not appear in the normal Inspector before Play Mode because it is created dynamically.

When testing, disabling it causes all WASD movement to stop because movement relies on:

```csharp
controller.Move(...)
```

That alone does not indicate a collision problem.

The previous:

```csharp
controller.detectCollisions = false;
```

test did not resolve the original W/S problem, so that diagnostic has no reason to remain in the final code.

---

# 19. `CameraController.cs`

There is also a:

```text
CameraController.cs
```

file under the:

```text
TMPro.Examples
```

namespace.

It appears to be a separate camera system with:

```text
CameraModes
- Follow
- Isometric
- Free

CameraTarget
FollowDistance
ElevationAngle
OrbitalAngle
```

It performs camera positioning/rotation in `LateUpdate()`.

It also has right-mouse controls for elevation/orbit and middle-mouse panning.

However, based on inspection, it is **not attached to Main Camera** in the current setup.

Therefore, it is not considered responsible for the current `DesktopMovement` system.

> Do not modify it unless future investigation proves another camera in the scene uses it.

---

# 20. WebXR Considerations

The project uses WebXR, so changes to Main Camera must be made carefully.

Main Camera has:

- Tracked Pose Driver
- Web XR Camera Settings

Therefore, if future camera behavior becomes strange again, check whether two systems are attempting to manipulate the camera's rotation simultaneously.

Current `DesktopMovement` uses:

```csharp
transform.localRotation
```

rather than world-space rotation because Main Camera is a child of `WebXRCameraSet`.

Do not blindly replace this with:

```csharp
transform.rotation = ...
```

because that can interfere with the parent/WebXR coordinate setup.

---

# 21. Previous Problem — E Key Didn't Work

Cause was related to the player/camera reference.

The player position was originally referencing the wrong object/camera setup.

Current solution:

- Main Camera is tagged `Player`
- Interaction uses the correct player Transform
- Distance checking is used

At one point:

```text
interactionDistance = 100f
```

was temporarily used for debugging.

This helped prove that the interaction logic itself worked.

---

# 22. Previous Problem — Cart Moved Before Player Boarded

Solution:

Destination selection only stores:

```csharp
golfCart.selectedDestination
```

The actual:

```csharp
StartTour()
```

happens when the player boards.

This behavior should be preserved.

---

# 23. Previous Problem — Player Could Not Exit Properly

Solution:

Exit is only available when:

```csharp
!golfCart.IsMoving
```

and the player presses:

```text
F
```

Then:

```text
isRiding = false
player → exitPoint
DesktopMovement → enabled
```

---

# 24. Previous Problem — W/S Movement Became Strange

This happened after vertical camera look was introduced.

CharacterController collision was initially suspected.

Testing:

```csharp
controller.detectCollisions = false;
```

did not solve it.

The actual problem was rotation-space / yaw-pitch mapping.

Final solution:

```text
Mouse X → rotationY
Mouse Y → rotationX
```

and movement uses only horizontal yaw while respecting the parent's 30.3° rotation.

This is confirmed working.

---

# 25. Third Person Integration — Original Diagnostic

A teammate is responsible for the Third Person POV system.

When Third Person `PlayerController` was enabled, the Console produced large amounts of:

```text
UnassignedReferenceException:
The variable Orientation of PlayerController
has not been assigned.
```

Stack trace:

```text
PlayerController.Update()
Assets/Third Person/PlayerController.cs:43
```

### Previous diagnostic test

With `PlayerController` enabled:

```text
Third Person PlayerController
        ↓
Orientation reference missing
        ↓
UnassignedReferenceException spam
        ↓
Map destination buttons don't respond
        ↓
Destination cannot be selected
        ↓
Golf-cart flow appears unresponsive
```

With `PlayerController` disabled:

```text
PlayerController disabled
        ↓
Console spam stops
        ↓
Map opens normally
        ↓
Destination buttons respond normally
        ↓
Destination selection works
        ↓
Golf-cart flow works normally
```

### Original conclusion

This strongly indicated that the Third Person `PlayerController` was interfering with the existing interaction system.

The existing:

- Golf-cart system
- Map system
- Destination system
- Desktop system

were not considered broken.

The teammate responsible for Third Person should fix its own missing reference rather than modifying the working map/cart systems.

---

# 26. Confirmed Working State Before New POV Work

Before implementing the new POV manager, the following were confirmed working:

### Golf cart

- Golf cart route system
- Doubly linked route nodes
- Four destination waypoints
- Destination storage
- Boarding
- Tour starts after boarding
- Player locked to seat
- Cart continues along route
- Cart stops at destination
- F exit

### Map

- Physical map board
- Map interaction
- Destination UI
- Destination buttons
- Destination selection

### Desktop First Person

- WASD
- Mouse horizontal look
- Mouse vertical look
- Horizontal movement unaffected by pitch
- Parent WebXR rotation accounted for
- Diagonal movement normalized

### Third Person diagnostic

With teammate `PlayerController` disabled:

- Map opens
- Destination buttons respond
- Destination can be selected
- UI works
- Golf-cart interaction works

---

# 27. Third Person Scripts — Direct Inspection

The teammate's actual Third Person scripts were later inspected directly.

The two relevant scripts are:

```text
PlayerMovement.cs
PlayerController.cs
```

The existing First Person system remains:

```text
DesktopMovement.cs
```

The two POV implementations should remain logically separate.

---

# 28. Third Person `PlayerMovement.cs`

The teammate's `PlayerMovement` uses Rigidbody-based movement.

Important fields include:

```csharp
[Header("Movement")]
public float moveSpeed;
public float sprintSpeed;
public float groundDrag;
public float jumpForce;
public float jumpCooldown;
public float airMultiplier;

[Header("Keybinds")]
public KeyCode jumpKey = KeyCode.Space;
public KeyCode sprintKey = KeyCode.LeftShift;

[Header("Ground Check")]
public float playerHeight;
public LayerMask whatIsGround;

public Transform orientation;
```

Internal references include:

```csharp
Rigidbody rb;
Transform cameraTransform;
```

During `Start()`:

```csharp
rb = GetComponent<Rigidbody>();
rb.freezeRotation = true;

cameraTransform = Camera.main.transform;
```

The script therefore uses whichever camera Unity resolves as:

```text
Camera.main
```

---

# 29. Third Person Movement

The script performs ground detection using:

```csharp
Physics.Raycast(...)
```

and changes Rigidbody drag depending on whether the player is grounded.

Input:

```csharp
horizontalInput =
    Input.GetAxisRaw("Horizontal");

verticalInput =
    Input.GetAxisRaw("Vertical");
```

Movement is camera-relative.

It obtains:

```csharp
Vector3 cameraForward =
    cameraTransform.forward;

Vector3 cameraRight =
    cameraTransform.right;
```

then:

```csharp
cameraForward.y = 0;
cameraRight.y = 0;
```

and normalizes them.

Movement direction:

```csharp
moveDirection =
    cameraForward * verticalInput +
    cameraRight * horizontalInput;
```

The script supports:

- Walking
- Sprinting
- Jumping
- Ground movement
- Air movement
- Ground drag
- Velocity limiting

Movement uses Rigidbody forces.

---

# 30. `PlayerMovement.orientation`

`PlayerMovement.cs` contains:

```csharp
public Transform orientation;
```

However, based on the inspected version, this variable is not actually used by its current movement logic.

Movement instead directly uses:

```text
cameraTransform.forward
cameraTransform.right
```

Therefore, this lowercase:

```text
orientation
```

is **not** the source of the previously observed `UnassignedReferenceException`.

---

# 31. Third Person `PlayerController.cs`

The teammate's `PlayerController` contains:

```csharp
[Header("References")]
public Transform Orientation;
public Transform Player;
public Transform PlayerObj;
public Rigidbody rb;

public float rotationSpeed;
```

Internal references:

```csharp
private Transform cameraTransform;
private Animator animator;
```

During `Start()`:

```csharp
Cursor.lockState = CursorLockMode.Locked;
Cursor.visible = false;

cameraTransform = Camera.main.transform;

animator = GetComponentInChildren<Animator>();
```

This script therefore handles aspects including:

- Cursor locking
- Camera-relative orientation
- Character/model rotation
- Walking animation

---

# 32. Confirmed Cause of `UnassignedReferenceException`

Inspection of the actual script confirmed the previous error.

The relevant line is:

```csharp
Orientation.forward = cameraForward;
```

The field is:

```csharp
public Transform Orientation;
```

Therefore:

```text
PlayerController.Orientation
        ↓
not assigned
        ↓
Orientation.forward = cameraForward
        ↓
UnassignedReferenceException
```

The previous diagnosis is confirmed.

---

# 33. `orientation` vs `Orientation`

There are two similarly named fields.

### PlayerMovement

```csharp
public Transform orientation;
```

### PlayerController

```csharp
public Transform Orientation;
```

These are separate variables.

C# is case-sensitive.

Therefore:

```text
orientation
```

and:

```text
Orientation
```

are not automatically connected.

Capitalizing the lowercase `orientation` in `PlayerMovement` is **not** the solution.

The actual problem is:

```text
PlayerController → Orientation
```

has not been assigned.

The intended Transform should be confirmed/configured by the teammate responsible for Third Person rather than guessed.

---

# 34. Third Person `PlayerController.Update()`

The controller obtains:

```csharp
Vector3 cameraForward =
    cameraTransform.forward;

Vector3 cameraRight =
    cameraTransform.right;
```

Then:

```csharp
cameraForward.y = 0;
cameraRight.y = 0;

cameraForward.Normalize();
cameraRight.Normalize();
```

It then executes:

```csharp
Orientation.forward = cameraForward;
```

This is the exact operation that fails when `Orientation` is null.

The controller reads:

```csharp
float horizontalInput =
    Input.GetAxis("Horizontal");

float verticalInput =
    Input.GetAxis("Vertical");
```

It determines:

```csharp
bool isWalking =
    horizontalInput != 0 ||
    verticalInput != 0;
```

and, if an Animator exists:

```csharp
animator.SetBool(
    "isWalking",
    isWalking
);
```

Movement direction for model orientation:

```csharp
Vector3 inputDir =
    cameraForward * verticalInput +
    cameraRight * horizontalInput;
```

The model then rotates approximately using:

```csharp
PlayerObj.forward =
    Vector3.Slerp(
        PlayerObj.forward,
        inputDir.normalized,
        Time.deltaTime * rotationSpeed
    );
```

---

# 35. First Person / Third Person Input Conflict

Both systems use standard Unity movement input.

First Person:

```text
W/A/S/D
    ↓
DesktopMovement
```

Third Person:

```text
W/A/S/D
    ↓
PlayerMovement

AND

W/A/S/D
    ↓
PlayerController
```

Therefore, if all controllers are active simultaneously:

```text
W/A/S/D
   ├── DesktopMovement
   ├── PlayerMovement
   └── PlayerController
```

multiple systems respond to the same input.

This is undesirable.

The First Person and Third Person systems should therefore be **mutually exclusive during normal gameplay**.

This became a primary reason for introducing `POVManager`.

---

# 36. New POV Coordination System

A dedicated POV folder/system was created.

Current structure is approximately:

```text
POV/
└── POVManager.cs
```

Only one new coordination script was created:

```text
POVManager.cs
```

Separate:

```text
FirstPerson.cs
ThirdPerson.cs
```

wrapper scripts were deliberately not created.

Reason:

The actual POV implementations already exist:

```text
First Person
→ DesktopMovement

Third Person
→ PlayerMovement
→ PlayerController
```

`POVManager` should coordinate them rather than duplicate their functionality.

---

# 37. POV State Design

The manager uses:

```csharp
public enum POVMode
{
    FirstPerson,
    ThirdPerson
}
```

rather than two independent booleans.

The selected state is stored in:

```csharp
public POVMode currentPOV;
```

This is preferable to:

```csharp
bool firstPersonActive;
bool thirdPersonActive;
```

because two independent booleans could produce contradictory states.

---

# 38. POVManager References

Current fields:

```csharp
[Header("First Person")]
public DesktopMovement firstPersonMovement;

[Header("Third Person")]
public PlayerMovement thirdPersonMovement;
public PlayerController thirdPersonController;

[Header("POV Menu")]
public GameObject povMenuPanel;
```

The actual class types are used rather than generic:

```csharp
MonoBehaviour
```

references.

This makes the Inspector clearer and restricts each reference to the intended component type.

---

# 39. Current POVManager Inspector Assignments

The current references have been assigned as:

```text
First Person Movement
→ Main Camera (Desktop Movement)

Third Person Movement
→ Player (Player Movement)

Third Person Controller
→ Player (Player Controller)

POV Menu Panel
→ POVMenuPanel
```

### Why First Person shows Main Camera

`DesktopMovement` is attached to:

```text
Main Camera
```

Therefore, this is correct.

### Why Third Person shows Player

The teammate's:

```text
PlayerMovement
PlayerController
```

components are attached to:

```text
Player
```

Therefore, those references are also correct.

---

# 40. Current `POVManager.cs`

Current functional version:

```csharp
using UnityEngine;

public class POVManager : MonoBehaviour
{
    public enum POVMode
    {
        FirstPerson,
        ThirdPerson
    }

    [Header("Current POV")]
    public POVMode currentPOV;

    [Header("First Person")]
    public DesktopMovement firstPersonMovement;

    [Header("Third Person")]
    public PlayerMovement thirdPersonMovement;
    public PlayerController thirdPersonController;

    [Header("POV Menu")]
    public GameObject povMenuPanel;

    void Start()
    {
        // disable both movement systems while choosing a pov
        SetControls(false, false);

        // show pov menu
        if (povMenuPanel != null)
            povMenuPanel.SetActive(true);

        // enable cursor to click button
        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
    }

    public void SelectFirstPerson()
    {
        currentPOV = POVMode.FirstPerson;

        SetControls(true, false);

        ClosePOVMenu();
    }

    public void SelectThirdPerson()
    {
        currentPOV = POVMode.ThirdPerson;

        SetControls(false, true);

        ClosePOVMenu();
    }

    private void SetControls(
        bool isFirstPerson,
        bool isThirdPerson
    )
    {
        if (firstPersonMovement != null)
            firstPersonMovement.enabled =
                isFirstPerson;

        if (thirdPersonMovement != null)
            thirdPersonMovement.enabled =
                isThirdPerson;

        if (thirdPersonController != null)
            thirdPersonController.enabled =
                isThirdPerson;
    }

    private void ClosePOVMenu()
    {
        if (povMenuPanel != null)
            povMenuPanel.SetActive(false);

        Cursor.lockState =
            CursorLockMode.Locked;

        Cursor.visible = false;
    }
}
```

### Cleanup

Earlier local code contained:

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEditor.Timeline.Actions;
```

These are unnecessary for the current manager.

In particular:

```csharp
using UnityEditor.Timeline.Actions;
```

is not needed by this runtime script.

An empty:

```csharp
void Update()
{
}
```

is also unnecessary.

These are cleanup items, not functional requirements.

---

# 41. POV Startup Behaviour

When gameplay starts:

```text
Scene starts
    ↓
POVManager.Start()
    ↓
SetControls(false, false)
```

Result:

```text
DesktopMovement
→ DISABLED

PlayerMovement
→ DISABLED

PlayerController
→ DISABLED
```

The POV menu is shown:

```csharp
povMenuPanel.SetActive(true);
```

The cursor becomes available:

```csharp
Cursor.lockState = CursorLockMode.None;
Cursor.visible = true;
```

Therefore:

```text
Game starts
    ↓
POV menu visible
    ↓
Both movement systems disabled
    ↓
Cursor available
    ↓
Wait for POV selection
```

---

# 42. POV Main Menu UI

A POV menu has been created.

Current hierarchy:

```text
POVMenuCanvas
└── POVMenuPanel
    ├── Title
    ├── First Person
    │   └── Text (TMP)
    └── Third Person
        └── Text (TMP)
```

Title:

```text
Choose Your POV
```

Buttons:

```text
First Person
Third Person
```

The current layout is intentionally simple and functional.

Visual polish is not currently the priority.

---

# 43. EventSystem

The scene already contained:

```text
EventSystem
```

before the POV menu was created.

Therefore, if Unity automatically creates another EventSystem while creating UI components, an unnecessary duplicate should not be kept.

The existing functional EventSystem should be used.

---

# 44. POV Menu Panel Reference

`POVManager` contains:

```csharp
public GameObject povMenuPanel;
```

The actual:

```text
POVMenuPanel
```

under:

```text
POVMenuCanvas
```

has been assigned to this field.

The panel does **not** need to physically be a child of:

```text
POVManager
```

The relationship is simply:

```text
POVManager
    │
    └── reference → POVMenuPanel
```

---

# 45. POV Button `OnClick()` Configuration

An important UI configuration issue occurred during initial testing.

Initially, the `POVManager` GameObject had been assigned to the button's:

```text
On Click()
```

event, but no method had been selected.

Merely assigning the GameObject does **not** automatically invoke a function.

### Correct First Person button

```text
First Person Button
    ↓
Button → On Click()
    ↓
POVManager
    ↓
POVManager.SelectFirstPerson()
```

### Correct Third Person button

```text
Third Person Button
    ↓
Button → On Click()
    ↓
POVManager
    ↓
POVManager.SelectThirdPerson()
```

Once these methods were explicitly selected, the button functionality worked.

---

# 46. First Person Selection Behaviour

Clicking:

```text
First Person
```

calls:

```csharp
SelectFirstPerson();
```

which executes:

```csharp
currentPOV = POVMode.FirstPerson;

SetControls(true, false);

ClosePOVMenu();
```

Result:

```text
DesktopMovement
→ ENABLED

PlayerMovement
→ DISABLED

PlayerController
→ DISABLED

POVMenuPanel
→ DISABLED

Cursor
→ LOCKED

Cursor Visible
→ FALSE
```

The player then returns to normal First Person gameplay.

---

# 47. First Person POV Test — CONFIRMED WORKING

The complete First Person selection flow has been tested successfully.

Confirmed sequence:

```text
Game starts
    ↓
POV menu appears
    ↓
Movement initially disabled
    ↓
Player clicks First Person
    ↓
SelectFirstPerson()
    ↓
DesktopMovement enabled
    ↓
Third Person components remain disabled
    ↓
POV menu closes
    ↓
Cursor locks/hides
    ↓
First Person gameplay works normally
```

Therefore:

> **The POV system has not broken the existing First Person implementation.**

The existing `DesktopMovement` remains functional after activation through `POVManager`.

---

# 48. Third Person Selection Behaviour

Clicking:

```text
Third Person
```

calls:

```csharp
SelectThirdPerson();
```

which executes:

```csharp
currentPOV = POVMode.ThirdPerson;

SetControls(false, true);

ClosePOVMenu();
```

Intended state:

```text
DesktopMovement
→ DISABLED

PlayerMovement
→ ENABLED

PlayerController
→ ENABLED
```

The manager successfully reaches the Third Person components.

---

# 49. Third Person Selection Test

During testing, the previous Third Person error starts after clicking the Third Person button.

This is expected.

Sequence:

```text
Third Person selected
    ↓
POVManager enables PlayerMovement
    ↓
POVManager enables PlayerController
    ↓
PlayerController.Update()
    ↓
Orientation.forward = cameraForward
    ↓
Orientation is unassigned
    ↓
UnassignedReferenceException
```

This is useful diagnostic evidence because it confirms that `POVManager` is actually activating the Third Person controller.

The error is **not** generated while Third Person remains disabled.

---

# 50. Responsibility Boundary for Third Person

The current Third Person failure is **not considered a POVManager problem**.

`POVManager` successfully performs:

```text
Third Person button
    ↓
SelectThirdPerson()
    ↓
Enable PlayerMovement
Enable PlayerController
```

The failure occurs afterward because:

```text
PlayerController.Orientation
```

is missing.

Therefore:

- Do not modify `POVManager` merely to hide this error.
- Do not modify `DesktopMovement`.
- Do not modify the map system.
- Do not modify the golf-cart system.
- Do not guess the intended `Orientation`.
- The teammate responsible for Third Person should configure/fix the reference.
- Retest Third Person after the teammate fixes it.

---

# 51. Current POV System Status

## Confirmed working

- `POVManager` exists
- First Person reference assigned
- Third Person references assigned
- POV menu appears
- Cursor available during selection
- Movement disabled before selection
- First Person button calls `SelectFirstPerson()`
- Third Person button calls `SelectThirdPerson()`
- POV menu closes after selection
- First Person activates correctly
- Third Person components can be activated
- First Person remains functional after integration

## External unresolved problem

```text
PlayerController.Orientation
```

remains unassigned.

Therefore, Third Person itself cannot yet be considered fully functional.

---

# 52. Updated High-Level Architecture

The project is moving toward a central **coordination** architecture rather than one giant manager.

Current startup:

```text
Game Starts
    ↓
POVManager
    ↓
POV Selection
    ↓
┌─────────────────────┐
│                     │
First Person      Third Person
│                     │
DesktopMovement   PlayerMovement
                  PlayerController
│                     │
└──────────┬──────────┘
           ↓
       Gameplay
```

The manager decides which existing system is active.

It should **not** absorb:

- Golf-cart movement
- Map interaction
- Destination selection
- Route traversal
- First Person movement implementation
- Third Person movement implementation

Those responsibilities remain in their dedicated scripts.

---

# 53. Planned Spawn Selection System

A future startup stage is planned after POV selection.

Intended high-level flow:

```text
Game Starts
    ↓
Choose POV
    ↓
Choose Spawn Location
    ↓
Place selected player/POV
    ↓
Gameplay begins
```

The current intention is to investigate whether the existing:

```text
TourRouteNode
```

/ route-node infrastructure can be reused for spawn locations.

This system has **not** been implemented yet.

Its exact implementation should not be assumed until the existing systems have been reviewed.

---

# 54. Planned Fast Travel System

A future fast-travel system is also planned.

The intention is to reuse the existing node/location infrastructure where appropriate.

Potential conceptual architecture:

```text
Existing Route / Node System
          ↓
    Location References
       ↙       ↘
Spawn System   Fast Travel
```

However, exact fast-travel behavior has **not yet been defined**.

Do not redesign the route system for fast travel until those requirements are discussed.

---

# 55. Central Architecture Principle

Avoid creating one enormous manager containing every feature.

Preferred conceptual structure:

```text
Central Coordination
        │
        ├── POVManager
        │      ↓
        │   POV selection
        │
        ├── Spawn system [future]
        │
        └── Fast travel coordinator [future]

Existing Gameplay Systems
        │
        ├── DesktopMovement
        ├── PlayerMovement
        ├── PlayerController
        ├── MapBoardInteraction
        ├── DestinationUI
        ├── GolfCartInteraction
        ├── GolfCartController
        └── TourRouteNode
```

Managers should coordinate systems rather than duplicate their logic.

---

# 56. Development Rules Going Forward

1. Preserve confirmed-working systems.
2. Diagnose before rewriting.
3. Make the smallest targeted change necessary.
4. Do not unnecessarily rewrite `DesktopMovement`.
5. Do not unnecessarily rewrite `GolfCartController`.
6. Do not unnecessarily rewrite `GolfCartInteraction`.
7. Do not unnecessarily rewrite `MapBoardInteraction`.
8. Do not unnecessarily rewrite `DestinationUI`.
9. Do not unnecessarily rewrite the route/node system.
10. Keep First Person and Third Person implementations separate.
11. Do not normally enable First Person and Third Person controllers simultaneously.
12. Use `POVManager` to coordinate controller activation.
13. Do not modify working systems to compensate for the teammate's missing `Orientation`.
14. Do not guess the intended Third Person `Orientation` Transform.
15. Reuse existing route/node infrastructure where sensible.
16. Avoid unnecessary wrapper scripts.
17. Keep central managers focused on coordination.
18. Test new systems independently before integrating the next one.
19. Remember the WebXR camera hierarchy before changing camera-space movement.
20. Preserve the current confirmed-working `DesktopMovement` rotation logic.

---

# 57. Current Confirmed Working Systems

## Route / Golf Cart

- Doubly linked route
- No route branching
- Four destination waypoints
- Forward/backward route traversal
- Destination storage
- Cart waits for player boarding
- Cart starts after boarding
- Cart follows selected destination
- Cart stops at destination
- Player stays at seat while riding
- F exit after cart stops

## Map / Destination

- Physical map board
- E interaction
- Destination menu
- Destination buttons
- Destination selection
- Destination stored in `GolfCartController`
- Menu closes after selection

## First Person

- Desktop WASD movement
- Mouse X horizontal look
- Mouse Y vertical look
- Pitch does not affect horizontal movement
- Parent WebXR rotation accounted for
- Diagonal movement normalized
- Correct interaction player reference
- Dynamic CharacterController

## POV Coordination

- POV menu appears at startup
- Both movement systems initially disabled
- Cursor available during selection
- First Person button works
- First Person activates
- Menu closes
- Cursor locks
- Existing First Person gameplay remains functional
- Third Person activation path works at manager level

---

# 58. Currently Unresolved

## Third Person

Still unresolved:

```text
PlayerController.Orientation
```

causes:

```text
UnassignedReferenceException
```

when `PlayerController` becomes enabled.

This is considered a teammate-side Third Person configuration issue.

## Spawn Selection

Not implemented.

## Fast Travel

Not implemented.

## Full Third Person Integration

Not verified.

Third Person must be retested after its missing `Orientation` reference is fixed.

## WebXR Final Integration

Desktop First Person behavior is confirmed, but complete final browser/WebXR integration remains part of later project testing.

---

# 59. Current Development Priority

The original broader priority remains:

1. Functional system
2. Reliable Editor/Desktop testing
3. Integration with teammate UI
4. WebXR/browser testing
5. Visual polish

However, at the current checkpoint, before implementing:

```text
Spawn Selection
```

or:

```text
Fast Travel
```

the user wants to:

> **Revisit and review older systems that have already been created.**

Therefore, do **not** automatically proceed to spawn selection.

The next task should depend on which existing system the user chooses to revisit.

---

# 60. CURRENT CHECKPOINT — 4 October 2026

The previously established:

- First Person system
- Map-board system
- Destination system
- Route system
- Golf-cart system

remain the baseline working systems.

The working First Person implementation still uses:

```text
Main Camera
    ↓
DesktopMovement
```

under:

```text
WebXRCameraSet
```

with the important parent Y rotation:

```text
30.3°
```

The teammate's Third Person scripts have now been inspected directly.

The previous:

```text
UnassignedReferenceException
```

is confirmed to originate from:

```csharp
PlayerController.Orientation
```

being unassigned when:

```csharp
Orientation.forward = cameraForward;
```

executes.

A new:

```text
POVManager
```

has been created.

Its purpose is to coordinate First Person and Third Person rather than implement either movement system itself.

Current startup flow:

```text
Game Starts
    ↓
POV Menu
    ↓
Choose POV
```

Confirmed First Person path:

```text
Choose First Person
    ↓
SelectFirstPerson()
    ↓
DesktopMovement enabled
    ↓
PlayerMovement disabled
PlayerController disabled
    ↓
POV menu closes
    ↓
Cursor locks
    ↓
First Person gameplay works
```

This has been tested successfully.

Current Third Person path:

```text
Choose Third Person
    ↓
SelectThirdPerson()
    ↓
DesktopMovement disabled
    ↓
PlayerMovement enabled
PlayerController enabled
    ↓
PlayerController reaches missing Orientation
    ↓
UnassignedReferenceException
```

Therefore:

> **The POV-selection infrastructure is functional. First Person through POVManager is confirmed working. Third Person remains blocked by the teammate's existing `PlayerController.Orientation` configuration issue, not by POVManager.**

The next development task is:

> **Revisit/review existing older systems before continuing with spawn selection or fast travel.**
