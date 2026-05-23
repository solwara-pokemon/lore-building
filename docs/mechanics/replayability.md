# Replayability & Branching Narrative

## Philosophy
Solwara is not a play-and-done game. Replayability comes from **meaningful choices and branching narrative** rather than procedural generation.

## Branching Structure: The River Model
The game uses a river model — branches split at key decision points, converge at major story beats, then split again. This creates the feeling of meaningful choice without exponential content requirements.

Most selections change **minor aspects** of the experience rather than routing the player into an entirely different story. Examples:
- Starter choice may trigger a special item to become available
- Dialogue choices affect NPC relationships
- Key story decisions affect which gym leaders are sympathetic vs hostile

## The Role System
The players role — chosen invisibly through starter selection — is the primary branching axis. Each role experiences the same world but through a fundamentally different emotional lens. See `story/roles.md`.

## Gym Structure
- 8 gyms representing story arcs, not just badge collection
- Each gym leader embodies a different relationship with the flood
- The Elite Four equivalent exists but is **interrupted by story** — it is not possible to win
- This interruption is a narrative gut punch: you think you are at the finish line

## Unlock System
- Roles expose their full starter pools through gameplay
- Beating runs unlocks new starters (see `pokemon/starters.md`)
- The Defector role is unlocked by completing all three launch roles
- Discovery and secrets reward attentive players across multiple playthroughs

## No Multiplayer
Multiplayer features are excluded due to save integrity concerns (client-side saves are trivially editable). The game is designed as a rich single-player experience. This may be revisited in the future if a server-side validation solution is implemented.
