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

### Additive Timelines (or Additive Tracks?)

Timelines (or tracks) that store deltas rather than absolute values.

### Deterministic Support

The design and code should support this, although we don't have to implement determinism exactly in our MVP.

### Useable Beyond Animating 3D Scenes

The core design of this should support use cases outside of animating entities and such. We should be able to use this for UI animations, for example, and other things.

### Game-Engine-Agnostic

The timeline player (runtime) should ideally be implemented in raw C++ which we could take with us to any game engine. This separation also supports the design, as timeline tracks themselves should be unaware of the game engine effects they are having, and binding is what bridges this connection.

### Nested Timelines

E.g., a level timeline should be able to play timelines on entities in the level.

### Extensible Track Value Type Support

This is important because some games might have their own types that they have to use. E.g., fixed-point real numbers based on integer storage, instead of floating-point types.

A set of operations must be defined for us for track value types, such as an interpolation function. E.g., quaternions should be interpolated via a `slerp` rather than a `lerp`. This is something that the game only knows how to do (we don't want to reimplement 3D math into our timeline library). Another example is interpolating color values with an HSV-based interpolation rather than RGB. That is also something that the game must defined for us.

Serialization functions would probably have to be specified as well for custom value types. E.g., how should we store this in json? And how should we read from it?

## Timeline Data Format

The file format that stores the timeline data will be based on json.

We will avoid deep json object hierarchies of our tracks. We will keep the tracks stored as a flat hierarchy. Parent-child track relationships will be indicated by the track id (with a `.` character).

This wouldn't be very efficient to parse during timeline playback, but it's okay because we plan to parse it upfront (before starting playback), building an in-memory representation of the tracks for the timeline playback runtime to use. This in-memory data structure will have cached data such as parent-child relationships to make it faster to evaluate at runtime.

## Track Mapping (for Engine Integrations)

This timeline player runtime library alone is not enough to make things happen in the game engine that it's being used in. An integration plugin will have to be written which will hook up mappings from known track structures to actual results in the game engine.

E.g., certain tracks will have to be identified as an entity. And then the track id "EntityA.Transform.Translation.X" will be mapped to the X translation of whatever entity is bound to EntityA.
