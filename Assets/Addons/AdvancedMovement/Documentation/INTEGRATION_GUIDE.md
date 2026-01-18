# MFPS Advanced Movement Addon - Integration Guide

## Overview

The Advanced Movement Addon enhances MFPS with advanced movement mechanics including parkour actions (vaulting, climbing, wall running) and slope skiing physics. The addon is modular, allowing you to enable/disable parkour and slope features independently.

## Features

- **Parkour System**: Vaulting, climbing, wall running, and ledge grabbing
- **Slope Skiing**: Tribes-style skiing physics with momentum-based movement
- **Network Synchronization**: Full multiplayer support with state synchronization
- **Modular Design**: Enable/disable parkour and slope modules independently
- **Easy Integration**: One-click integration with MFPS player controller

## Installation

### Step 1: Import the Addon

1. Copy the `Assets/Addons/AdvancedMovement` folder into your MFPS project's `Assets/Addons/` directory

### Step 2: Enable Required Modules

1. Open Unity and go to `MFPS > Addons > Advanced Movement`
2. Choose which modules to enable:
   - **Enable Parkour Module**: Adds vaulting, climbing, and wall running
   - **Enable Slope Module**: Adds skiing physics
3. Wait for recompilation after enabling each module

### Step 3: Integrate with MFPS

1. Open your gameplay scene (where the player prefab is instantiated)
2. Go to `MFPS > Addons > Advanced Movement > Integrate Into Scene`
3. This will automatically add the `MFPS_IntegrationHelper` component to your player

## Required Components & Prefabs

### Player Prefab Setup

The addon requires your player prefab to have these MFPS components:

#### Required MFPS Components:
- `bl_FirstPersonController` - Main player controller
- `bl_PlayerReferences` - Component references
- `bl_PlayerAnimationsBase` - Animation controller
- `bl_PlayerNetwork` - Network synchronization
- `PhotonView` - Photon networking
- `CharacterController` - Unity character controller

#### Automatically Added Components:
When you use "Integrate Into Scene", these components are added automatically:

**Parkour Module (if enabled):**
- `ParkourController` - Handles parkour actions
- `ClimbController` - Manages climbing mechanics
- `EnvironmentScanner` - Detects parkour surfaces

**Slope Module (if enabled):**
- `SlopeMovement` - Tribes-style skiing physics
- `SlopeJetpack` - Additional slope controls

**Always Added:**
- `MFPS_IntegrationHelper` - Main integration component

### Manual Component Setup

If automatic integration doesn't work, manually add these components:

#### Basic Setup:
```csharp
// Attach to your player prefab's bl_FirstPersonController GameObject
gameObject.AddComponent<FC_AdvancedMovement.MFPS_IntegrationHelper>();
```

#### Parkour Components (if Parkour module enabled):
```csharp
// Add parkour controllers
gameObject.AddComponent<FC_ParkourSystem.ParkourController>();
gameObject.AddComponent<FC_ParkourSystem.ClimbController>();

// Configure ParkourController
var parkourController = GetComponent<FC_ParkourSystem.ParkourController>();
parkourController.obstacleLayer = LayerMask.GetMask("Default", "Environment");
parkourController.forwardRayLength = 1.5f;
parkourController.heightRayLength = 3f;
```

#### Slope Components (if Slope module enabled):
```csharp
// Add slope movement
gameObject.AddComponent<FC_AdvancedMovement.SlopeMovement>();
gameObject.AddComponent<FC_AdvancedMovement.SlopeJetpack>();

// Configure SlopeMovement
var slopeMovement = GetComponent<FC_AdvancedMovement.SlopeMovement>();
slopeMovement.minSpeedToSki = 8f;
slopeMovement.maxSpeed = 35f;
slopeMovement.gravity = 18f;
```

## Configuration

### MFPS_IntegrationHelper Settings

The main integration component has these key settings:

```csharp
// Root motion for parkour animations
UseRootMotion = true;

// Gravity settings
Gravity = Physics.gravity.magnitude;

// States where parkour is disabled
PreventParkourDuring = PlayerState.Dead | PlayerState.Spectating;
```

### Parkour Configuration

#### ParkourController Settings:
```csharp
// Detection ranges
forwardRayLength = 1.5f;      // How far ahead to detect obstacles
heightRayLength = 3f;         // How high to scan for surfaces
forwardRayOffset = 0.5f;      // Offset from player center

// Height thresholds
vaultHeight = 1.0f;           // Max height for vaulting
stepUpHeight = 0.6f;          // Max height for stepping up
climbUpHeight = 2.0f;         // Max height for climbing

// Speed requirements
minWallRunSpeed = 5f;         // Minimum speed for wall running

// Wall run settings
wallRunDuration = 2f;         // How long wall run lasts
wallRunSpeed = 6f;            // Speed during wall run
wallRunGravity = 2f;          // Reduced gravity during wall run
```

#### Environment Setup:
Parkour works best with properly configured collision layers:

1. Create a layer called "Parkour" or "Environment"
2. Assign this layer to walls, obstacles, and climbable surfaces
3. Set the `obstacleLayer` in ParkourController to include this layer

### Slope Configuration

#### SlopeMovement Settings:
```csharp
// Speed limits
minSpeedToSki = 8f;           // Minimum speed to enter ski mode
maxSpeed = 35f;               // Maximum horizontal speed

// Physics
gravity = 18f;                // Gravity force
groundFriction = 4f;          // Friction on ground
skiFriction = 0.3f;           // Friction while skiing

// Control
groundControl = 25f;          // Ground movement control
skiControl = 20f;             // Ski movement control
airControl = 1.2f;            // Air movement control

// Balance
downhillSpeedGain = 0.6f;     // Speed gained going downhill
uphillMomentumRetention = 0.75f; // Momentum kept going uphill
```

## Controls

### Parkour Controls (when Parkour module enabled):

- **Jump + Forward**: Vault over obstacles
- **Interact (E) + Forward**: Drop to hang from ledges
- **Crouch + Sprint**: Slide (if on suitable surface)
- **Automatic Wall Run**: Occurs when running toward walls at sufficient speed

### Slope Controls (when Slope module enabled):

- **Automatic**: Slope skiing activates when sliding down steep surfaces
- **Momentum-based**: Speed increases with slope steepness
- **Directional Control**: Use movement keys to steer while skiing

## Animation Setup

### Required Animator Setup

The addon requires specific animation states and parameters:

#### Required Parameters:
- `ParkourAction` (int) - Current parkour action type
- `IsParkour` (bool) - Whether player is in parkour action
- `WallRun` (bool) - Wall running state
- `Climbing` (bool) - Climbing state

#### Required Animation States:
Create these states in your animator controller:

- `Vault` - Vaulting animation
- `StepUp` - Stepping up animation
- `Climb` - Climbing animation
- `WallRun` - Wall running animation
- `LedgeGrab` - Ledge grabbing animation

#### Animation Events:
Add these events to your animations:

```csharp
// In vault animation
AnimationEvent("OnParkourComplete", 0.8f); // When vault completes

// In climb animation
AnimationEvent("OnClimbComplete", 0.9f);   // When climb completes
```

### Root Motion Setup

1. Enable **Root Motion** in your Animator component
2. Set **Update Mode** to **Normal** in Animator
3. Ensure animations have proper root motion curves
4. The `MFPS_IntegrationHelper` will manage root motion automatically

## Network Synchronization

### Automatic Synchronization

The addon automatically synchronizes:

- **Parkour Actions**: Vaulting, climbing, wall running across all clients
- **Player State**: Current movement state and position
- **Animation States**: Parkour animations and parameters

### Network Components

The addon includes specialized network components:

- `AdvancedMovement_AINetworkSync` - Synchronizes AI movement
- `AdvancedMovement_AIShooterAgent` - AI parkour behavior
- `AdvancedMovement_AISlopeMovement` - AI slope skiing

### Photon Integration

Ensure your player prefab has:
- `PhotonView` component
- Proper observation settings for movement synchronization

## Scene Setup

### Environment Requirements

#### For Parkour:
- **Walls**: Must have colliders and be on appropriate layer
- **Ledges**: Need proper geometry for ledge grabbing
- **Obstacles**: Vaultable objects should be appropriately sized

#### For Slope Skiing:
- **Slopes**: Terrain or mesh colliders with proper normals
- **Surface Types**: Different friction values for different surfaces
- **Boundaries**: Invisible walls to prevent falling off edges

### Testing Environment

Create a test scene with:

1. **Flat Ground**: For basic movement testing
2. **Walls**: Various heights for parkour testing
3. **Slopes**: Different steepness for skiing testing
4. **Obstacles**: Boxes, crates for vaulting practice

## Troubleshooting

### Common Issues

#### "Parkour not working"
- Check that Parkour module is enabled via scripting define
- Verify `ParkourController` and `ClimbController` are attached
- Ensure environment has proper collision layers

#### "Slope skiing not activating"
- Check that Slope module is enabled
- Verify `SlopeMovement` component is attached
- Ensure player reaches minimum speed (`minSpeedToSki`)

#### "Animation issues"
- Check animator has required parameters and states
- Verify root motion is enabled
- Ensure animation events are properly set up

#### "Network desync"
- Check PhotonView observation settings
- Verify network synchronization components are attached
- Ensure proper authority settings

### Debug Information

Enable debug logging:

```csharp
// In MFPS_IntegrationHelper
public bool enableDebugLogs = true;
```

This will output detailed information about:
- Parkour detection
- Slope calculations
- Network synchronization
- Animation state changes

### Performance Considerations

- **Parkour Raycasts**: Adjust raycast lengths for performance
- **Slope Calculations**: Monitor physics update frequency
- **Network Traffic**: Balance sync frequency vs. bandwidth

## Advanced Usage

### Custom Parkour Actions

Extend the parkour system:

```csharp
public class CustomParkourController : ParkourController
{
    protected override void DetectParkourAction()
    {
        base.DetectParkourAction();

        // Add custom parkour detection logic
        if (CustomCondition())
        {
            StartCustomAction();
        }
    }

    private void StartCustomAction()
    {
        // Implement custom parkour action
    }
}
```

### Custom Slope Physics

Modify slope behavior:

```csharp
public class CustomSlopeMovement : SlopeMovement
{
    protected override void UpdateSlopePhysics()
    {
        base.UpdateSlopePhysics();

        // Add custom slope modifications
        if (specialSurface)
        {
            ApplyCustomForces();
        }
    }
}
```

### Event Integration

Hook into MFPS events:

```csharp
public class AdvancedMovementEvents : MonoBehaviour
{
    void Start()
    {
        // Subscribe to MFPS events
        bl_EventHandler.onPlayerSpawn += OnPlayerSpawn;
        bl_EventHandler.onMatchStart += OnMatchStart;
    }

    void OnPlayerSpawn(bl_PlayerReferences player)
    {
        // Setup advanced movement for new player
        var integration = player.GetComponent<MFPS_IntegrationHelper>();
        if (integration != null)
        {
            ConfigurePlayerMovement(integration);
        }
    }
}
```

## AI Integration

### AI Components

The addon includes AI-specific components:

- `AdvancedMovement_AIAnimation` - AI animation handling
- `AdvancedMovement_AIShooterAgent` - AI parkour behavior
- `AdvancedMovement_AISlopeMovement` - AI slope navigation

### AI Setup

For AI bots to use advanced movement:

1. Attach AI components to bot prefabs
2. Configure AI movement parameters
3. Ensure AI has proper navigation mesh setup

## Version History

- **v1.0.0**: Initial release with parkour and slope systems
- Modular design with independent enable/disable
- Full MFPS integration and network synchronization

## Support

For issues with the Advanced Movement addon:

1. Check Unity console for `[Advanced Movement]` logs
2. Verify all required components are attached
3. Ensure scripting defines are properly set
4. Test in a clean scene with minimal setup

## License

This addon is provided for use with MFPS. Ensure compliance with Unity Asset Store terms and MFPS licensing.