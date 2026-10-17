# Failure-Closed Java Host Integration

How my Java/Swing extension for a third-party desktop host decides when it may act, and stops cleanly when that permission goes away.

- **Source:** [G-Earth Facts](https://github.com/wouthh/g-earth-facts), a Java 21 and Swing extension for the G-Earth proxy that I designed and built. It posts facts from the API Ninjas service into a Habbo Origins room as shouts.

## Summary

The extension runs beside a host it does not own. G-Earth owns the connection, the game client owns the room, and a remote API owns the facts. None of that state is guaranteed to stay true between a button press and the moment a message goes out.

I treat that as an authority problem. Pressing Start expresses intent; it does not prove the host is connected or that the room is still the one the user picked. Every send rechecks that state, and every lifecycle event that removes it cancels the work it would have authorized.

## Decisions and trade-offs

### One generation-bound loop

[`FactScheduler`](https://github.com/wouthh/g-earth-facts/blob/main/src/main/java/io/github/wouthh/gearthfacts/runtime/FactScheduler.java) owns the whole publishing loop: one scheduled tick, one in-flight API request and one multipart message sequence. A generation counter guards all of it. Stop, disconnect, a room change, a rejected API key and a permanent API failure each bump the generation and cancel the scheduled tick, the in-flight request and any remaining message parts.

Every callback carries the generation it was scheduled under. Before each send, the scheduler checks under one lock that it is still running, that a known room is still set and that the generation has not moved. A late API response or a delayed message part from an older generation is dropped rather than replayed.

A room change while running cancels the current cycle and arms a fresh interval for the new room, so nothing queued for one room reaches another. Temporary API failures keep the next interval scheduled; an authentication or permanent failure stops publishing and says why.

### Stopped on every boot

The extension always starts stopped, even when an API key and prefix are saved. Starting needs a connected Origins session, a known room and a saved key, and the start call re-reads the room under the scheduler lock. If navigation cleared the room while the user clicked, start refuses instead of arming against stale context. Saving settings never starts publishing.

### Atomic settings

[`SettingsStore`](https://github.com/wouthh/g-earth-facts/blob/main/src/main/java/io/github/wouthh/gearthfacts/runtime/SettingsStore.java) writes through a temporary file in the same directory and moves it into place atomically, so a crash leaves either the old file or the new one. It refuses symlinks and oversized files, keeps owner-only permissions where the platform supports them, and holds a file lock so a second instance cannot share the same profile.

### Byte-aware splitting and pacing

Origins messages are ISO-8859-1 and length-limited. [`ShoutComposer`](https://github.com/wouthh/g-earth-facts/blob/main/src/main/java/io/github/wouthh/gearthfacts/protocol/ShoutComposer.java) validates that the prefix and fact are representable in Latin-1, splits the text at whitespace into parts of at most 100 bytes, caps a fact at 16 parts, and rejects a prefix that leaves no room for a fact. The scheduler sends the parts five seconds apart, so a long fact never floods the room, and a cancelled sequence stops at the next part.

### Keep the host at the edge

Only the `protocol/` package touches native G-Earth packets. The scheduler and UI work with project-owned types, which lets most tests run without starting G-Earth or the game. A successful send means G-Earth accepted the packet, not that the game server accepted the shout.

## Verification

The [CI gate](https://github.com/wouthh/g-earth-facts/blob/main/.github/workflows/ci.yml) builds the extension and runs the unit tests, a public-hygiene check, synthetic checks of the Steam installer, and a smoke test of the packaged extension against a synthetic G-Earth host. Tests use temporary settings, local HTTP servers and synthetic packets; none of them contact the game, the API or a live G-Earth process.

## What I would add

- Keep one backup of the last valid settings file, and preserve a malformed one for recovery before loading defaults.
- A schema version check that blocks activation on settings written by a newer release.
