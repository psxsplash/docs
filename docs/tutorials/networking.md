# Networked Multiplayer

Build a networked game twice: first two consoles on a single link cable, then many
consoles through a server. The engine API is identical; only the setup differs.

By the end you will have players who can see each other move, and a scoring rule
that every console agrees on.

Nothing here is specific to 2D or 3D. The engine replicates an **actor transform**,
so the same six calls work for a first-person shooter, a top-down
[tilemap](../components/tilemaps.md) game, or a racing game - the only thing that
changes is how your game already moves the actor.

!!! info "What you need"
    **For the cable version:** two consoles (or two PCSX-Redux instances) and a
    link cable. Nothing else.

    **For the server version:** a checkout of
    [psnl-server](https://github.com/psxsplash/psnl_server), Python 3.12, and a
    USB-to-serial adaptor if you are using real hardware.

---

## Part 1 - Two consoles, one cable

### 1.1 Connect

The whole handshake is one call. Put it in `onSceneCreationStart`, so the link is
coming up while the rest of the scene loads:

```lua
function onSceneCreationStart()
    Net.Connect()
end
```

Call it with **no arguments**. The baud comes from the engine's own constant, and
the receive strategy resolves itself: interrupt-driven on real hardware, polled
under an emulator. Passing a literal baud here is how the two ends end up at
different rates, and a baud mismatch looks exactly like an unplugged cable.

### 1.2 Show the player what is happening

A serial link that does not come up is otherwise completely silent, so spend five
lines on this now rather than debugging blind later:

```lua
local STATE = {
    [0] = "DISCONNECTED",
    [1] = "CONNECTING...",
    [2] = "CONNECTED",
    [3] = "VERSION MISMATCH - REBUILD THE DISC",
    [4] = "SCENE MISMATCH - RE-EXPORT THE SCENE",
}

function onUpdate(self, dt)
    UI.SetText(statusLabel, STATE[Net.State()] or "?")
end
```

States 3 and 4 are **terminal**: the console latches them and stops talking
entirely. A screen that only knows how to say "connecting" will say it forever.

### 1.3 Agree on the scene

Two consoles will only share a scene if they agree on its identity. On the
**Scene Exporter**, set **Scene Network Id** to a stable string:

```
mygame/arena
```

Leave it blank and the engine derives a hash from the scene's contents. That hash
changes whenever you add an object, so two discs built a day apart refuse to talk
and report state 4. Pick a name and never change it once players have the disc.

### 1.4 Give each console an avatar

An **avatar** is any [actor](../lua/api-reference.md#actor): a character in a 3D
scene, a sprite on a tilemap, a ship, a cursor. The engine replicates its
**transform**, so nothing here assumes a dimension or a movement style.

Author one actor per player in the scene, named `player_1` and `player_2`. Then
tell the engine which is yours:

```lua
local MAX_PLAYERS = 2
local avatars = {}
local bound = false

function onSceneCreationEnd()
    for slot = 1, MAX_PLAYERS do
        avatars[slot] = Actor.Find("player_" .. slot)
    end
end

local function bindAvatars()
    local mySlot = Net.LocalSlot()
    for slot = 1, MAX_PLAYERS do
        if slot == mySlot then
            Net.SetLocalAvatar(avatars[slot])         -- we move this one
        else
            Net.SetRemoteAvatar(slot, avatars[slot])  -- the engine moves this one
        end
    end
    bound = true
end

function onUpdate(self, dt)
    if not bound and Net.IsConnected() then bindAvatars() end
end
```

That is the whole of avatar replication. Move your own avatar however you like and
the engine sends it; incoming transforms are applied to the actors you mapped, and
interpolated between snapshots so remote players glide rather than teleport.

!!! warning "Do not read LocalSlot before you are connected"
    The slot is handed out by the handshake, and `Net.Connect()` returns long
    before that finishes. Called too early, `Net.LocalSlot()` returns **255**, no
    branch of the loop matches, and every avatar including your own is registered
    as remote. Nothing errors; your character simply never moves. Bind on the frame
    `Net.IsConnected()` first turns true, as above.

!!! tip "Slots start at 1, not 0"
    Slot 0 is the authority. On a cable that is one of the two consoles; with a
    server it is the server, which has no avatar. Either way, player avatars are
    slots 1 upward.

### 1.5 Move

**There is no networked movement API, and that is the point.** Whatever already
moves an actor in your game keeps moving it; the engine reads the local avatar's
transform each tick and sends it. Pick whichever of these your game already does -
none of it is networking code:

=== "3D, engine locomotion"

    The shortest path in a 3D scene. Hand the pad to the avatar and the engine's
    own movement, collision, nav mesh and camera all apply:

    ```lua
    local function bindAvatars()
        -- ... the loop from 1.4 ...
        local me = avatars[Net.LocalSlot()]
        Input.BindToActor(1, me)        -- pad 1 now drives this actor
        Camera.SetFollowTarget(me)      -- and the camera follows it
        bound = true
    end
    ```

    No movement code at all. The player walks the scene exactly as in a
    single-player game - collision, nav mesh, camera and all - and the far console
    sees it.

=== "3D, your own controller"

    When you drive the transform yourself, keep doing that. The engine does not
    care where the numbers came from:

    ```lua
    local STEP = FixedPoint.new(1) / 8    -- world units per frame

    function onUpdate(self, dt)
        if not Net.IsConnected() then return end
        local me = avatars[Net.LocalSlot()]
        if not me then return end

        local move = Vec3.new(0, 0, 0)
        if Input.IsHeldPlayer1(Input.UP)   then move.z =  STEP end
        if Input.IsHeldPlayer1(Input.DOWN) then move.z = -STEP end

        Actor.SetPosition(me, Vec3.add(Actor.GetPosition(me), move))
    end
    ```

    Rotation replicates too, so `Actor.SetRotation(me, Vec3.new(0, heading, 0))`
    turns the remote copy as well.

    !!! warning "You own collision on this path"
        Writing a position directly bypasses the nav mesh. The engine's collision
        pass runs *before* `onUpdate`, so a position you write here is not checked
        until the next frame. The **engine locomotion** tab avoids this entirely.

=== "2D on a tilemap"

    A [tilemap](../components/tilemaps.md) scene moves actors against the grid:

    ```lua
    local SPEED = 2

    function onUpdate(self, dt)
        if not Net.IsConnected() then return end
        local me = avatars[Net.LocalSlot()]
        if not me then return end

        local dx, dz = 0, 0
        if Input.IsHeldPlayer1(Input.LEFT)  then dx = -SPEED end
        if Input.IsHeldPlayer1(Input.RIGHT) then dx =  SPEED end
        if Input.IsHeldPlayer1(Input.UP)    then dz = -SPEED end
        if Input.IsHeldPlayer1(Input.DOWN)  then dz =  SPEED end

        Tile.MoveActor(me, dx, dz)
    end
    ```

Build any of these onto two discs, connect the cable, and the two players can see
each other move. **That is a complete networked game in about thirty lines**, and
the only part that mentions the network is the six calls in 1.1 and 1.4.

!!! warning "Only move the avatar that is yours"
    The remote avatars are driven by the engine every time a snapshot lands. Move
    one from Lua and the two fight: the actor stutters between where you put it and
    where the far console says it is. Guard on `Net.LocalSlot()`, as every example
    above does.

### 1.6 Send a message

Positions are continuous state and the engine handles them. Discrete events - a
door opening, a point scored - are yours, and they go on the reliable channel:

```lua
local MSG_SCORED = 1

-- sending
Net.SendData(string.char(MSG_SCORED, Net.LocalSlot()))

-- receiving, on every other console
function onNetData(payload)
    local op = string.byte(payload, 1)
    if op == MSG_SCORED then
        local slot = string.byte(payload, 2)
        scores[slot] = (scores[slot] or 0) + 1
    end
end
```

The payload is **bytes**, not a string of text: build it with `string.char` and
read it with `string.byte`. The engine never looks inside it, so the format is
entirely yours. A leading opcode byte costs nothing and gives you something to
dispatch on.

!!! warning "Always check the return value"
    `Net.SendData` returns `false` when the reliable queue is full. Ignore it and
    the message is gone with no error anywhere - and the symptom appears much later
    as the two consoles disagreeing about what happened.

    ```lua
    local outbox = {}

    local function send(payload)
        if #outbox == 0 and Net.SendData(payload) then return true end
        outbox[#outbox + 1] = payload   -- keep order: a vote must not
        return false                    -- overtake the meeting it belongs to
    end

    local function flush()
        while #outbox > 0 do
            if not Net.SendData(outbox[1]) then return end
            table.remove(outbox, 1)
        end
    end
    ```

    Call `flush()` once per frame.

### 1.7 Who decides?

With two consoles there is no referee, so one of them has to be. The engine elects
slot 0 during the handshake:

```lua
if Net.IsHost() then
    -- decide here, then tell the other console
end
```

This works, and it is the right answer for a co-operative game. It is the wrong
answer for a competitive one, because the host's console is deciding the rules and
a modified host decides them differently. That is what Part 2 is for.

---

## Part 2 - Many consoles, one server

Everything from Part 1 still applies. What changes: there can be up to ten
players, and the rules live somewhere neither player controls.

### 2.1 Run the server

```sh
git clone https://github.com/psxsplash/psnl_server
cd psnl_server
pip install -e ".[dev]"
psnl-server
```

With no game plug-in it is a pure relay: it fans out positions and passes your
messages between consoles. That alone is enough for Part 1's game with ten players
instead of two.

Or in a container:

```sh
docker compose up --build
```

### 2.2 Connect a console

**PCSX-Redux** needs no extra software. Open the SIO1 settings, choose **Client**
mode and **Protobuf**, and point it at your server's IP on port **6700**.

!!! warning "Redux's client cannot use Raw"
    It throws on Raw mode, so the protobuf port is the only one that works from the
    emulator. Its host field also accepts digits and dots only, so type a bare IP
    address rather than a hostname.

**Real hardware** goes through the bridge, which pumps bytes between the serial
cable and the server:

```sh
psnl-bridge
```

Pick the COM port and enter the server address. It connects to port **6699**, the
raw port, because a serial cable carries raw bytes with no protobuf involved.

```
PlayStation --serial--> psnl-bridge --raw------> :6699 -+
                                                        +-> psnl-server
PCSX-Redux -------------protobuf---------------> :6700 -+
```

Both kinds of player share the same rooms. The wire below the adapter is identical.

### 2.3 Ten players instead of two

Author ten actors, `player_1` through `player_10`, and change one constant:

```lua
local MAX_PLAYERS = 10
```

`bindAvatars` from 1.4 is unchanged, and so is your movement code. Slot 0 is now
the server, which has no avatar, which is why players are slots 1..10 and the
engine's slot space is eleven.

### 2.4 Move the rules onto the server

This is the part that is worth the server. Write a plug-in:

```python
# games/mygame/game.py
from psnl_server.game import Game

SCORED = 1          # console -> server
TOTALS = 0x80       # server -> console


class MyGame(Game):
    name = "mygame"

    def __init__(self) -> None:
        self._scores: dict[int, int] = {}

    def on_app_data(self, session, payload: bytes) -> bool:
        if not payload or payload[0] != SCORED:
            return False        # not ours; the relay passes it on untouched
        self._scores[session.slot] = self._scores.get(session.slot, 0) + 1
        self._broadcast(session.room)
        return True             # handled

    def _broadcast(self, room) -> None:
        body = bytes([TOTALS, len(self._scores)])
        for slot, score in sorted(self._scores.items()):
            body += bytes([slot, min(score, 255)])
        for member in room.members.values():
            member.session.send_app_data(body)
```

```sh
psnl-server --game games.mygame.game:MyGame
```

Now the console **asks** and **draws**; it never decides. Pressing the score button
sends a request and waits. A modified client can send whatever it likes and the
server still counts what it counts.

!!! tip "Serialise once, send many"
    `_broadcast` builds the payload once and hands the same bytes to every member.
    At ten players on a 5.76 KB/s link that is not a micro-optimisation.

### 2.5 Test it without hardware

`FakeConsole` speaks the byte-exact wire the engine emits:

```python
from psnl_server.testing import FakeConsole

console = FakeConsole()
await console.connect("127.0.0.1", port)
await console.handshake()
console.send_app_data(bytes([SCORED]))
reply = await console.wait_for_app_data(TOTALS)
assert reply[2] == console.slot
```

A ten-player room can be proven with nothing plugged in. Do this before you burn a
disc.

### 2.6 Handing over between scenes

A lobby that starts a match needs the session to survive the scene load:

```lua
Net.SetPersistent(true)
Scene.Load(GAME_SCENE)
```

Without it, `Scene.Load` tears the session down and every console rejoins as a
brand new player with a new slot - which, in a room that is already full of the
very players trying to move, fails.

---

## Designing for the wire

Everything below is invisible on an emulator, which models bytes rather than bit
timing and has no cable at all. It matters on hardware.

**The link carries 5.76 KB/s at 57600, and the console can send about 1.4 KB/s.**
That asymmetry is deliberate: the transmit path is bounded so it cannot busy-wait
away the frame.

**The receive FIFO is 8 bytes** and overruns about 1.4ms into a burst, which is why
receive is interrupt-driven on real hardware.

So:

- **Send on a change, not on a timer.** The engine already sends positions on a
  clock; almost nothing else needs to be.
- **Send bytes, not text.** A four-byte message and a forty-byte one differ by
  seven milliseconds of wire.
- **Do not send what the receiver can derive.** Send the event, not the
  consequences.
- **Let the server pace itself.** It measures the link and throttles position
  updates before it throttles anything that matters.

## When it does not work

Draw `Net.Stats()` somewhere reachable. The connecting screen is the natural home,
since that is exactly where a link that never comes up leaves you.

| Reading | Means |
|---|---|
| `bytesRx` 0, `bytesTx` rising | nothing is coming back: wrong port, wrong address, or the far end is not running |
| `bytesRx` rising, `rxIrqs` 0 | the receive interrupt never fired |
| `frames` 0 with `bytesRx` rising | bytes arrive but no frame completes: almost always a baud mismatch |
| `rxFramingErrors` rising | the two ends disagree on bit timing: cable, adapter, grounding |
| `rxOverrunErrors` rising | the console cannot drain the FIFO fast enough |
| `crcErrors` and `resyncs` rising | corruption on the wire; check the same things as framing |
| `rttSamples` 0 after a while | nothing has ever replied |
| `baud` | compare it with the bridge's setting; they must match exactly |

## See also

- [Networking reference](../components/networking.md)
- [`Net` API reference](../lua/api-reference.md#net)
