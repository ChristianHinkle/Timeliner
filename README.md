# Timeliner

## Requirements

### Binding

Entity references can be bound to existing entities in the level before playing the timeline.

This is important for letting the timeline animate existing entities. Binding can be done manually before playing the timeline.

We should also provide a nice way to bind entities from levels/prefabs, via some sort of asset that maps entities from the prefab. This would allow designers to just play a timeline without having to specify bindings, as the bindings are already defined in a separate asset, instructing the timeline player which entities from the prefab to bind to what.

Also, binding doesn't only apply to entities. This concept is generalized to any object that the timeline may want to animate, e.g., audio assets to play, anim assets to play, custom properties that can affect anything such as overall music volume or gameplay values, etc.

### Entity Spawning

Timelines must be able to spawn entities.

### Any-Scope Timelines

Not only should we support authoring timelines for entire levels, but for a single entity too.

### Timeline Data Decoupled from Level

Timeline data should be stored in its own file(s), separate from the level prefeb (unlike TrackView).

This is important for being able to play a timeline in any level that relies on binding to entities or dynamically spawning its own entities.

### Multiple Concurrent Timeline Playback

Multiple timelines should be able to be played at a time.

This gets complicated if two timelines have conflicting tracks (trying to animate the same thing). This could be handled with:
- Warnings/errors to raise awareness (this is enough for the MVP).
- A priority system to allow one timeline to trump the other.
- A blending system? With an alpha?

### Partial Interruptibility

The timeline playback must be interruptable, on a per-track basis.

This implies the idea of "ownership". At any moment, a property is either in control by the gameplay, or the timeline. This ownership of properties should be able to be transferred back and forth.

### Additive Timelines (or Additive Tracks?)

Timelines (or tracks) that store deltas rather than absolute values.

### Frame-Based Timing

The timeline should store timing information:
- A frames per seconds value.
- An end frame value to determine the duration.

Although milliseconds is a nice objective measurement, frames are the language that animators understand. E.g., in 2D animation at 24 FPS, people know what the timing of 1's, on 2's, and on 3's feel like because that's part of the convention they work by.

Timeline start frame: `1`. Animators think of frames as duration of time rather than positional indicies. E.g., a full second of time has passed when the end of frame 24 is reached (at 24 fps). The source timeline file should store these frames in the same way, so that developers can easily navigate the a timeline's file contents.

Frame `n` represents the `n`th frame in the timeline.

### Deterministic Support

The design and code should support this, although we don't have to implement determinism exactly in our MVP.

The timeliner runtime should produce deterministic results when simulation tick rate mismatches the timeline's frame rate. E.g., a lockstep multiplayer game with a fixed tick rate of 30 FPS should be able to play 24-FPS-based timelines (timelines that affect gameplay).

| Tick Rate                       |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| ------------------------------- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| 240 Hz (for reference)          | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o | o |
| 120 Hz (for reference)          | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o | - | o |
| 60 Hz (for reference)           | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o | - | - | - | o |
| 30 FPS (game's fixed tick rate) | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o |
| 24 FPS (timeline frame rate)    | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o |

### Multiplayer Compatibility (Rollback and Replays)

Timeline playback needs to be rewindable and replayable. This seems trivial for value track types, but event track types - such as playing audio, spawning entity, dealing damage - may need special handling in the game engine's integration plugin. You'd probably want to undo the side effects these events before resimulating. Or, in a less deterministic fashion, avoid re-firing the events during resimulation.

Replay system support is a similar kinda deal. You may provide playback controls for the user to scrub through a replay (back and forth). In this case, that is similar to a rewind in rollback netcode. You would have to undo all the possible side effects of event tracks.

### Save/Load Support

Timeline playback state should be serializable into a game's save, and able to be restored when loading the save.

E.g., in an open world game, you might have periodic auto-saves and auto-save on quit. At any moment a timeline may be playing and an auto-save may occur.

Timeline playback data that must be saved and restored:
- Bindings - If the gameplay code assigned dynamic bindings before playing the timeline, these have to be saved and restored.
- Playhead Position

What's a little weird about this is that, the game will already be saving and restoring any important state. So, when the timeline is restored during a load, it would be unnecessary for the game engine integration plugin to apply the effects of track bindings on this frame, as it would just be the same state as what the game has already loaded. In this case, it would be nice to add some assertions or something like that to validate that this is the case.

### Useable Beyond Animating 3D Scenes

The core design of this should support use cases outside of animating entities and such. We should be able to use this for UI animations, for example, and other things.

### Game-Engine-Agnostic

The timeline player (runtime) should ideally be implemented in raw C++ which we could take with us to any game engine. This separation also supports the design, as timeline tracks themselves should be unaware of the game engine effects they are having, and binding is what bridges this connection.

### Nested Timelines

E.g., a level-scoped timeline should be able to play timelines on entities in the level.

### Extensible Track Value Type Support

This is important because some games might have their own types that they have to use. E.g., fixed-point real numbers based on integer storage, instead of floating-point types.

A set of operations must be defined for us for track value types, such as an interpolation function. E.g., quaternions should be interpolated via spherical interpolation (`slerp`) rather than linear (`lerp`). This is something that the game only knows how to do (we don't want to reimplement 3D math into our timeline library). Another example is interpolating color values with an HSV-based interpolation rather than RGB. That is also something that the game must defined for us.

Serialization functions would probably have to be specified as well for custom value types. E.g., how should we store this in json? And how should we read from it?

### Extensible at Runtime

Compile-time extensibility is cool, but that means the library couldn't be distributed as its own binary. So we should make sure that we are extensible at runtime to avoid imposing the requirement of recompiling.

This is important for third-party users of our library to be able to package this into their own 3D engines if they wanted to. E.g., if this became the official the TrackView replacement for O3DE, it would need to be distributed with the engine without requiring developers to compile from source.

This also unlocks new capabilities technically. E.g., there may be runtime-loaded plugins in a game that introduce their own timelines with their own custom track value types. Those custom track value types need to be registered with the game's timeliner library in that moment (you can't just recompile during the game lol). A similar example is a game loading mods that may have custom timeline assets and such.

### Cache-Friendly / Data-Oriented Runtime Playback

We will aim to implement the timeline player in a cache-friendly manner.
- Storing common track values in contiguous arrays grouped together by their type.
- Iterating on each group of track values and processing them together by the same logic at a time.

### Track Curves and Interpolation

Keys define the state of a track at certain timestamps/frames, but we also need curves to tell us what should be happening in between two keys.

Animators should be able to author a curve for their track, as well as specify an interpolation function.

#### Curves

The purpose of curves concerns blend alpha between two keys.

A customizable math function that determines the blend alpha throughout the range of time from one key to the next.

Animators choose a curve type, a default curve of that type is generated, and the curve is able to be customized.

Example curve types:
- Linear (no customization available)
- Discrete (no customization available)
- Bezier (customizable via control handles)

Curve types should be an extensible feature. Library users can write their own custom curve types. They must specify an alpha blending function, a serialization function, etc.

#### Interpolation Functions

The purpose of interpolation functions concerns how exactly do we blend two different values for a specific track value type when provided an alpha.

A function that determines how to blend between two values given a blend alpha.

Example interpolation functions:
- Value Type: Quaternion
    - Spherical
- Value Type: Color
    - HSV Interp
- Value Type: Integer
    - Floor/Truncate
    - Round
    - Ciel

Interpolation functions should be an extensible feature. Library users can write their own interpolation functions for any track value type. They must specify what track value type they are providing an interpolation function for, they must specify the interpolation function itself, etc. Maybe also a serialization function in case we want to support being customizable by the animator.

## Timeline Data Format

The file format that stores the timeline data will be based on json.

We will avoid deep json object hierarchies of our tracks. We will keep the tracks stored as flat collections. Parent-child track relationships will be indicated by the track id (with a `.` character).

This wouldn't be very efficient to parse during timeline playback, but it's okay because we plan to parse it upfront (before starting playback), building an in-memory representation of the tracks for the timeline playback runtime to use. This in-memory data structure will have cached data such as parent-child relationships to make it faster to evaluate at runtime.

## Track Mapping (for Engine Integrations)

This timeline player runtime library alone is not enough to make things happen in the game engine that it's being used in. An integration plugin will have to be written which will hook up mappings from known track structures to actual results in the game engine.

E.g., certain tracks will have to be identified as an entity. And then the track id "EntityA.Transform.Translation.X" will be mapped to the X translation of whatever entity is bound to EntityA.

## Tick Matching Methods

The timeline's frame rate will likely not match with the tick rate of the simulation it is being played in.

Each track should be able to specify a "tick matching method" (name subject to change) to determine how its keys will "snap" to the simulation's actual tick timing.

This problem can be divided into two main situations: discrete and continuous track types. Track types that are continuous can elegantly be interpolated to the most natural value at the current tick. Discrete track types, however, cannot simply be interpolated. Instead, one of the discrete states must be chosen at a time.

### Track Type: Event

This is discrete.

|                                 |   |   |   |   |   |   |   |   |   |   |        |   |   |   |   |   |        |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| ------------------------------- | - | - | - | - | - | - | - | - | - | - | ------ | - | - | - | - | - | ------ | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| 24 FPS (timeline)               | o | - | - | - | - | - | - | - | - | - | o      | - | - | - | - | - | -      | - | - | - | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o |
| Track Keys                      |   |   |   |   |   |   |   |   |   |   | SIGNAL |   |   |   |   |   |        |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
|                                 |   |   |   |   |   |   |   |   |   |   |        |   |   |   |   |   |        |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| 30 FPS (simulation)             | o | - | - | - | - | - | - | - | o | - | -      | - | - | - | - | - | o      | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o |
| Track Playback Results          |   |   |   |   |   |   |   |   |   |   |        |   |   |   |   |   | SIGNAL |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

### Track Type: Value - Bool

This is discrete.

|                                 |     |   |   |   |   |   |   |   |   |   |    |   |   |   |   |   |    |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |     |
| ------------------------------- | --- | - | - | - | - | - | - | - | - | - | -- | - | - | - | - | - | -- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | -   |
| 24 FPS (timeline)               | o   | - | - | - | - | - | - | - | - | - | o  | - | - | - | - | - | -  | - | - | - | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o   |
| Track Keys                      | OFF |   |   |   |   |   |   |   |   |   | ON |   |   |   |   |   |    |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | OFF |
|                                 |     |   |   |   |   |   |   |   |   |   |    |   |   |   |   |   |    |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |     |
| 30 FPS (simulation)             | o   | - | - | - | - | - | - | - | o | - | -  | - | - | - | - | - | o  | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o   |
| Track Playback Results          | OFF |   |   |   |   |   |   |   |   |   |    |   |   |   |   |   | ON |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | OFF |

### Track Type: Value - Enum

This is discrete.

|                                 |         |   |   |   |   |   |   |   |   |   |       |   |   |   |   |   |       |   |   |   |       |   |   |   |       |   |   |   |   |   |      |   |      |   |   |   |   |   |   |   |         |
| ------------------------------- | ------- | - | - | - | - | - | - | - | - | - | ----- | - | - | - | - | - | ----- | - | - | - | ----- | - | - | - | ----- | - | - | - | - | - | ---- | - | ---- | - | - | - | - | - | - | - | ------- |
| 24 FPS (timeline)               | o       | - | - | - | - | - | - | - | - | - | o     | - | - | - | - | - | -     | - | - | - | o     | - | - | - | -     | - | - | - | - | - | o    | - | -    | - | - | - | - | - | - | - | o       |
| Track Keys                      | neutral |   |   |   |   |   |   |   |   |   | angry |   |   |   |   |   |       |   |   |   | tired |   |   |   |       |   |   |   |   |   | calm |   |      |   |   |   |   |   |   |   | neutral |
|                                 |         |   |   |   |   |   |   |   |   |   |       |   |   |   |   |   |       |   |   |   |       |   |   |   |       |   |   |   |   |   |      |   |      |   |   |   |   |   |   |   |         |
| 30 FPS (simulation)             | o       | - | - | - | - | - | - | - | o | - | -     | - | - | - | - | - | o     | - | - | - | -     | - | - | - | o     | - | - | - | - | - | -    | - | o    | - | - | - | - | - | - | - | o       |
| Track Playback Results          | neutral |   |   |   |   |   |   |   |   |   |       |   |   |   |   |   | angry |   |   |   |       |   |   |   | tired |   |   |   |   |   |      |   | calm |   |   |   |   |   |   |   | neutral |

### Track Type: Value - Integer

This is discrete.

|                                 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| ------------------------------- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| 24 FPS (timeline)               | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | - | - | o |
| Track Keys                      | 7 |   |   |   |   |   |   |   |   |   | 8 |   |   |   |   |   |   |   |   |   | 9 |   |   |   |   |   |   |   |   |   | 4 |   |   |   |   |   |   |   |   |   | 3 |
|                                 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| 30 FPS (simulation)             | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o | - | - | - | - | - | - | - | o |
| Track Playback Results          | 7 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | 8 |   |   |   |   |   |   |   | 9 |   |   |   |   |   |   |   | 4 |   |   |   |   |   |   |   | 3 |

### Track Type: Value - Float

This is continuous.

|                                 |     |   |   |   |   |   |   |   |     |   |     |   |   |   |   |   |     |   |   |   |     |   |   |   |     |   |   |   |   |   |     |   |     |   |   |   |   |   |   |   |     |
| ------------------------------- | --- | - | - | - | - | - | - | - | --- | - | --- | - | - | - | - | - | --- | - | - | - | --- | - | - | - | --- | - | - | - | - | - | --- | - | --- | - | - | - | - | - | - | - | --- |
| 24 FPS (timeline)               | o   | - | - | - | - | - | - | - | -   | - | o   | - | - | - | - | - | -   | - | - | - | o   | - | - | - | -   | - | - | - | - | - | o   | - | -   | - | - | - | - | - | - | - | o   |
| Track Keys                      | 0.0 |   |   |   |   |   |   |   |     |   | 6.0 |   |   |   |   |   |     |   |   |   | 4.0 |   |   |   |     |   |   |   |   |   | 3.0 |   |     |   |   |   |   |   |   |   | 0.0 |
|                                 |     |   |   |   |   |   |   |   |     |   |     |   |   |   |   |   |     |   |   |   |     |   |   |   |     |   |   |   |   |   |     |   |     |   |   |   |   |   |   |   |     |
| 30 FPS (simulation)             | o   | - | - | - | - | - | - | - | o   | - | -   | - | - | - | - | - | o   | - | - | - | -   | - | - | - | o   | - | - | - | - | - | -   | - | o   | - | - | - | - | - | - | - | o   |
| Track Playback Results          | 0.0 |   |   |   |   |   |   |   | 4.8 |   |     |   |   |   |   |   | 7.2 |   |   |   |     |   |   |   | 3.6 |   |   |   |   |   |     |   | 2.4 |   |   |   |   |   |   |   | 0.0 |

Note: This example assumes a linear interpolation curve between each key.
