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
