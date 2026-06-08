# SceneSequencer

## Requirements

### Binding

Entity references can be bound to existing entities in the level before playing the cutscene, or spawned in by the cutscene itself.

This is important for letting the cutscene animate existing entities. Binding can be done manually before playing the cutscene.

We should also provide a nice way to bind entities from levels/prefabs, via some sort of asset that maps entities from the prefab. This would allow designers to just play a cutscene without having to specify bindings, as the bindings are already defined in a separate asset, instructing the cutscene player which entities from the prefab to bind to what.

Also, binding doesn't only apply to entities. This concept is generalized to any object that the cutscene may want to animate, e.g., audio assets to play, anim assets to play, custom properties that can affect anything such as overall music volume or gameplay values, etc.

### Any-Scope Sequences

Not only should we support authoring cutscenes for entire levels, but for a single entity too.

### Cutscene Data Decoupled from Level

Cutscene data should be stored in its own file(s), separate from the level prefeb (unlike TrackView).

This is important for being able to play a cutscene in any level that relies on binding to entities or dynamically spawning its own entities.

### Multiple Cutscene Playback

Multiple cutscenes should be able to be played at a time.

### Interruptable (on a Per-Track Basis)

### Optional: Deterministic Support

### Optional: Being Able to be Used Outside of 3D Scenes
