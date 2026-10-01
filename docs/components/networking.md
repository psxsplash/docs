# Networking

psxsplash games play together over **[SIO1](../reference/glossary.md#sio1)**,
the PlayStation's serial port - the link cable port, not the controller port
([SIO0](../reference/glossary.md#sio0)). Two consoles can share one cable
directly, or many can meet on a server.

This page is the reference. For a worked example of both topologies, see the
[Networked Multiplayer tutorial](../tutorials/networking.md).

## The two topologies

**Two consoles, one cable.** Nothing else needed. The handshake is symmetric: both
ends say hello, the higher random tiebreak becomes slot 0, and that console is the
authority for shared world objects.

```
PlayStation <---- link cable ----> PlayStation
```

**Many consoles, one server.** The server is slot 0 and has no avatar of its own,
so ten players occupy slots 1..10. Real consoles reach it through a bridge that
pumps bytes between the cable and TCP; emulators connect directly.

```
PlayStation --serial--> bridge --raw------> :6699 -+
                                                   +-> server
PCSX-Redux -------------protobuf----------> :6700 -+
```

Both use the same engine API. A game written for one works on the other.

## What the engine does for you

- **Framing and reliability.** Every packet is length-prefixed and CRC-checked.
  Corrupt frames are dropped and the stream resynchronises.
- **Avatar replication.** The local avatar's transform is sent on a clock; incoming
  ones are applied to the actors you map, and interpolated between snapshots so
  remote players move smoothly rather than teleporting. An avatar is any
  [actor](../lua/api-reference.md#actor), so nothing here assumes 2D or 3D, or any
  particular way of moving.
- **A reliable message channel** for anything discrete, retransmitted until
  acknowledged.
- **Session persistence** across a scene load.

## What it deliberately does not do

The engine has **no concept of a lobby, a room, a score or a match**. Those are
your game's, and they ride the reliable channel as opaque bytes
(`Net.SendData` / `onNetData`) that the engine never inspects.

That is a deliberate boundary. Putting `CreateRoom` into the wire protocol would
bake one game's model into the engine forever, and the next game would inherit a
lobby it never asked for.

## Getting connected

```lua
function onSceneCreationStart()
    Net.Connect()
end
```

Call it with no arguments. The baud comes from the engine's own constant and the
receive strategy resolves itself - interrupt-driven on real hardware, polled under
an emulator, because those two targets need opposite settings and the console can
tell them apart at runtime.

Then watch the state:

```lua
local NAMES = {
    [0] = "DISCONNECTED",
    [1] = "CONNECTING...",
    [2] = "CONNECTED",
    [3] = "VERSION MISMATCH - REBUILD THE DISC",
    [4] = "SCENE MISMATCH - RE-EXPORT THE SCENE",
}
```

!!! warning "States 3 and 4 are terminal and silent"
    A version or scene mismatch latches and the console stops talking. Nothing
    retries and nothing times out, so a screen that only knows how to say
    "connecting" will say it forever. Show the player which it was.

## Scene identity

Consoles only share a scene if they agree on its identity. Set **Scene Network Id**
on the Scene Exporter to a stable string:

```
mygame/lobby
```

Leave it empty and the engine derives a hash from the scene's contents - which
changes whenever you add an object, so two discs built a day apart will refuse to
talk. Pick a name and do not change it once players have the disc.

## Replicating players

```lua
local function bindAvatars()
    local mySlot = Net.LocalSlot()
    for slot = 1, MAX_PLAYERS do
        local actor = Actor.Find("player_" .. slot)
        if slot == mySlot then
            Net.SetLocalAvatar(actor)         -- ours to move; the engine sends it
        else
            Net.SetRemoteAvatar(slot, actor)  -- theirs; the engine moves it
        end
    end
end
```

Move your own avatar however you like - `Input.BindToActor` to hand it the engine's
own locomotion, `Tile.MoveActor` on a tilemap, or `Actor.SetPosition` from your own
controller. The other avatars are driven for you.

!!! warning "Bind after the handshake, not at scene creation"
    `Net.LocalSlot()` returns **255** until the handshake assigns a slot, and
    `Net.Connect()` returns long before that. Called from `onSceneCreationEnd` the
    loop above matches no branch, registers your own avatar as remote, and reports
    no error - your character simply never moves. Call `bindAvatars()` on the first
    frame `Net.IsConnected()` is true.

!!! tip "Turn replication off where there is no avatar"
    `Net.SetReplicationEnabled(false)` in menus and lobbies. A console in an
    avatar-less scene was measured spending 894 B/s broadcasting the position of a
    player that did not exist, while the message it was waiting for queued behind
    that traffic.

## Handing over between scenes

```lua
Net.SetPersistent(true)
Scene.Load(GAME_SCENE)
```

Without this, `Scene.Load` tears the session down and the console rejoins as a
brand new player with a new slot. With it, the slot and the link survive and the
console simply re-announces itself in the new scene.

## The hardware, and why it constrains the design

Worth knowing before you design a message, because none of it is visible on an
emulator.

**The receive FIFO is 8 bytes.** At 57600 a byte arrives every ~174us, so the FIFO
overruns about 1.4ms into a burst. A once-per-frame poll is 16.6ms apart and cannot
keep up, which is why receive is interrupt-driven on real hardware.

**The console's outbound path is capped at 48 bytes per frame.** That is roughly
1.4 KB/s regardless of what the cable can carry, and it is a deliberate bound on
how long the transmit path may busy-wait.

**The link carries 5.76 KB/s at 57600.** A ten-player position snapshot is about
127 bytes, so the fan-out alone is a substantial fraction of it.

The practical consequences for your game:

- **Send bytes, not strings.** A four-byte message and a forty-byte one differ by
  seven milliseconds of wire.
- **Do not send on a timer if you can send on a change.**
- **Check the return value of `Net.SendData`.** It refuses when the queue is full,
  and a dropped message is otherwise silent.
- **An emulator proves your logic, never your bandwidth.** It models bytes rather
  than bit timing and has no wire at all.

## Diagnostics

`Net.Stats()` returns the link counters. On a console the screen is the only
channel there is, so draw them somewhere reachable - the connecting screen is the
natural home, since that is where a link that never comes up leaves you.

The three that answer most questions:

| Reading | Means |
|---|---|
| `rxIrqs` 0 with `bytesRx` rising | the receive interrupt never fired |
| `rxFramingErrors` rising | the two ends disagree on bit timing: cable, baud, adapter |
| `rxOverrunErrors` rising | the console is too slow to drain the FIFO |

See the [`Net` API reference](../lua/api-reference.md#net) for the full table.

## See also

- [Networked Multiplayer tutorial](../tutorials/networking.md)
- [`Net` API reference](../lua/api-reference.md#net)
