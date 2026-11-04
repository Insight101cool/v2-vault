---
tags: [field-notes, studio, mcp, screenshots, look-dev, lighting, quality-level, testing, ai-agents, type/guide, area/ai, level/intermediate]
---
# Comparable Studio Look Shots - Quality Level, Settle Time and Camera Ownership

<!-- tldr:start -->
> [!tldr] TL;DR
> - Same view, different brightness between runs: Play renders at `Automatic` quality -> Pin `settings().Rendering.QualityLevel` from the Client and log it
> - First shot after Play washed out: Lighting and exposure still adapting -> Wait about 20 s after Play or a big brightness jump
> - One shot darker with no sun shadows: Render state differed, cause unknown -> Check the tells (shadows, logged level) before comparing
> - Quality level pinned from the Client and logged with every look shot.
> - About 20 s of settle time after Play and after big brightness changes.
> - Shadows and the logged level checked before two shots are compared.
>
> Cost: TL;DR ~200 tok · whole note ~1.9k tok. Read more: ## 1. Play starts at Automatic quality · ## 4. Own the camera during the shot · ## Common mistakes · ## Checklist
<!-- tldr:end -->

An agent's look loop works like this: change the lighting, take a Play screenshot, compare it with the last one, repeat. It only works if two shots of the same view differ only because of the change you made. In practice, shots of identical code and camera can differ by a third in brightness. The causes are the render quality level, how long the scene had to settle, and which script owned the camera. This note gives the rules that made shots comparable in a melee game built with agents driving Studio through the Studio MCP. The capture tool's own quirks (black frames when Studio is in the background, particles in Edit, missing AlwaysOnTop GUIs) are in [[Roblox Studio MCP Server]].

Source labels: **Roblox docs (as of October 2026)**, **Measured** (field-tested in a Roblox game in development, 2026-10-23 to 2026-11-04), **Rule of thumb**.

---

## 1. Play starts at Automatic quality

**Measured (2026-11-04).** Studio's Play session started rendering at `Automatic` quality even though Edit had been set to Level 10. Automatic picks a level from the frame rate, so two runs can render at different levels with no visible sign. Brightness readings (mean luma) of the same view were only comparable across runs once the level was pinned.

Fix: pin the level from the **Client** data model right after Play starts, and log it with the shot. The Studio MCP's `execute_luau` can run there; the command bar works too.

```lua
-- Run in the Client data model during Play (command bar or MCP execute_luau).
settings().Rendering.QualityLevel = Enum.QualityLevel.Level10
print("[shot] QualityLevel", settings().Rendering.QualityLevel)
```

- Put the level back to `Enum.QualityLevel.Automatic` when you are done, so a later performance run isn't read at Level 10 by accident.
- For a phone-like look, shoot the same view at two pinned levels (for example Level04 and Level08) and say they are a bracket, not a phone measurement.
- In-game code can only *read* the player's saved choice, through `UserGameSettings.SavedQualityLevel`. See [[Graphics and Performance Toggles for Low-End Devices]].

## 2. Let the scene settle

**Measured.** In a hall with Realistic lighting, a shot taken 10 s after Play came out washed white. A later shot of the same view didn't. Exposure and lighting adapt for a while after Play starts and after the camera jumps to a much brighter or darker view.

- Wait **about 20 s** after Play starts, or after a big jump in brightness, before a look shot (**Measured** in that hall; re-measure in yours).
- Moving between views of similar brightness needs much less time.
- To darken a scene in that hall, colour correction worked and lowering `ExposureCompensation` from -0.45 to -1 barely did (**Measured**, cause not established). Test both in your own scene before choosing one.

## 3. Check the shot is the scene you think it is

**Measured (2026-11-04).** The same code and camera read mean luma 28.7 and 37.7 in two runs. The darker one had no sun shadows, and Lighting's properties hadn't changed. The cause wasn't established (a quality drop with several Studio windows open was suspected).

- Before comparing two shots, check that the cheap tells match: sun shadows present, the same quality level logged, the same time of day.
- A shot that fails a tell gets retaken. It doesn't get scored.

## 4. Own the camera during the shot

A capture tool's own camera position is ignored whenever a game script rewrites the camera every frame (a fixed or over-the-shoulder camera does). Bind a test camera after the default camera step, drive it from a shared value, and release it afterwards. See [[Layering Camera Feedback on the Default Camera]] for the render-step order.

```lua
-- Client data model: a temporary test camera that wins over the game's camera.
local RunService = game:GetService("RunService")
local camera = workspace.CurrentCamera

_G.testCam = { position = Vector3.new(0, 12, -20), lookAt = Vector3.new(0, 4, 0) }

RunService:BindToRenderStep("TestCamera", Enum.RenderPriority.Camera.Value + 2, function()
	local shot = _G.testCam
	if shot then
		camera.CFrame = CFrame.lookAt(shot.position, shot.lookAt)
	end
end)

-- Afterwards: _G.testCam = nil, then RunService:UnbindFromRenderStep("TestCamera")
```

## 5. Freeze what you want to judge

- **Poses.** Don't race a capture against a 0.3 s swing. Play the track, set its `TimePosition`, then `AdjustSpeed(0)` to hold it, and capture the held frame.
- **Effects under a second.** Trigger the effect and capture in back-to-back calls, and expect to retry. A held frame shows colour and shape but not motion or fade, so say which one you judged.

## 6. Measure a feature's cost inside one run

**Measured.** With Studio in the background, the whole place ran at about 15 fps whatever was in it (15.0 fps without a set of new props, 14.9 with). Absolute numbers from a throttled window mean nothing. Toggle the feature on and off within the same run and compare, and take absolute numbers only with Studio in the foreground and the quality level pinned.

## Common mistakes

| Symptom | Cause | Fix |
|---|---|---|
| Same view, different brightness between runs | Play renders at `Automatic` quality | Pin `settings().Rendering.QualityLevel` from the Client and log it |
| First shot after Play washed out | Lighting and exposure still adapting | Wait about 20 s after Play or a big brightness jump |
| One shot darker with no sun shadows | Render state differed, cause unknown | Check the tells (shadows, logged level) before comparing |
| Capture ignores the camera you asked for | A game camera script rewrites the camera every frame | Bind a test camera after the camera step, then release it |
| A pose or effect missing from the shot | Raced a sub-second animation | Freeze the pose; trigger effects right before capture and retry |
| Performance numbers that never change | Studio throttled in the background | Toggle within one run; absolute numbers only in the foreground |

## Checklist

- [ ] Quality level pinned from the Client and logged with every look shot.
- [ ] About 20 s of settle time after Play and after big brightness changes.
- [ ] Shadows and the logged level checked before two shots are compared.
- [ ] Test camera bound after the camera step, released after.
- [ ] Poses frozen; effects triggered right before capture.
- [ ] Cost measured by toggling inside one run, with Studio in the foreground.
- [ ] Quality level put back to Automatic at the end.

## See also

[[Roblox Studio MCP Server]] · [[Field Notes Start Here]] · [[Blind Review Scores - Noise, Panels and Shipping the Best Version]] · [[Running Many AI Agents on One Roblox Place]] · [[Graphics and Performance Toggles for Low-End Devices]]

<!-- related:start -->
## Related
[[Studio Automation Without Freezing Studio]] · [[Moving Data Out of a Play Session]] · [[Held Weapons on Motor6D - Locking the Grip During Attacks]] · [[Springs That Survive Long Frames - dt Clamps and NaN Guards]] · [[Shift-Lock Sprint and Gamepad B for Menus]] · [[A Stuck Player Watchdog That Logs the Cause]] · [[Deferred Events - Decide From the Arguments, Not the Property]] · [[Streaming Joins - Frozen Players and Loading Screen Teardown]]
<!-- related:end -->
