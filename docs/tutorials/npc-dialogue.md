# Tutorial: NPC Dialogue

Create an interactive NPC that shows multi-line dialogue when the player approaches and presses a button.

## What You'll Learn

- `onInteract` callback with `PSXInteractable`
- `onButtonPress` for advancing dialogue
- UI canvas show/hide for dialogue boxes
- `Controls.SetEnabled` to freeze the player during dialogue
- `Interact.SetEnabled` to manage prompt visibility

## Step 1: Create the Dialogue Canvas

1. Create a Canvas, add `PSXCanvas` with name `"Dialogue"`, **startVisible = false**, sort order 20
2. Create a child Text element, add `PSXUIText` with name `"DialogueText"`, default text empty

## Step 2: Add Dialogue System to Scene Script

Add these functions to your `scene.lua`:

```lua
local dialogueCanvas = -1
local dialogueText = -1
local inDialogue = false
local dialogueLines = {}
local dialogueLine = 0

-- Add this to your existing onSceneCreationEnd:
-- dialogueCanvas = UI.FindCanvas("Dialogue")
-- if dialogueCanvas >= 0 then
--     dialogueText = UI.FindElement(dialogueCanvas, "DialogueText")
--     UI.SetCanvasVisible(dialogueCanvas, false)
-- end

function startDialogue(lines)
    if inDialogue then return end
    inDialogue = true
    dialogueLines = lines
    dialogueLine = 1
    Controls.SetEnabled(false)
    UI.SetCanvasVisible(dialogueCanvas, true)
    UI.SetText(dialogueText, dialogueLines[1])
end

function advanceDialogue()
    if not inDialogue then return end
    dialogueLine = dialogueLine + 1
    if dialogueLine > #dialogueLines then
        endDialogue()
    else
        UI.SetText(dialogueText, dialogueLines[dialogueLine])
    end
end

function endDialogue()
    inDialogue = false
    Controls.SetEnabled(true)
    UI.SetCanvasVisible(dialogueCanvas, false)
end

function isInDialogue()
    return inDialogue
end

-- Publish
_G.startDialogue = startDialogue
_G.advanceDialogue = advanceDialogue
_G.endDialogue = endDialogue
_G.isInDialogue = isInDialogue
```

## Step 3: NPC Script

Create `npc.lua`:

```lua
local talked = false

function onCreate(self)
    talked = false
end

function onInteract(self)
    if isInDialogue() then return end

    -- Hide the interact prompt during dialogue
    Interact.SetEnabled(self, false)

    if not talked then
        talked = true
        startDialogue({
            "Hello, traveler!",
            "Welcome to the test scene.",
            "There are collectibles, a door,",
            "triggers, and a portal here.",
            "Press CROSS to advance dialogue.",
            "Good luck exploring!"
        })
    else
        startDialogue({
            "You again?",
            "Go find the collectibles!",
            "There should be three of them."
        })
    end
end

function onButtonPress(self, button)
    if not isInDialogue() then return end
    if button == Input.CROSS then
        advanceDialogue()
        -- Re-enable interaction when dialogue ends
        if not isInDialogue() then
            Interact.SetEnabled(self, true)
        end
    end
end
```

## Step 4: Create the NPC

1. Create a mesh GameObject for the NPC (a character model, or a simple shape)
2. Add `PSXObjectExporter`, assign `npc.lua`
3. Add `PSXInteractable` with default settings (Cross button, radius 2.0)

## Step 5: Build and Test

Walk up to the NPC and press Cross. Dialogue should appear line by line. Press Cross to advance. Different dialogue on repeat visits.

## How It Works

1. Player enters interaction radius -> prompt shows (if configured)
2. Player presses Cross -> `onInteract` fires
3. `startDialogue` freezes the player, shows the dialogue canvas, sets the first line
4. Each Cross press calls `advanceDialogue`, updating the text
5. After the last line, `endDialogue` unfreezes the player and hides the canvas
6. `Interact.SetEnabled(self, false)` hides the interact prompt during dialogue

## Customization Ideas

- Add different dialogue based on game state (score, items collected)
- Use `Persist.Get` to remember NPC conversations across scenes
- Add branching dialogue with different responses based on game state
