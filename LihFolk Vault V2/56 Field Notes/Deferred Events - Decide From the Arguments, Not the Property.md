---
tags: [field-notes, luau, events, deferred, signalbehavior, ancestrychanged, tools, race-conditions, type/guide, area/scripting, level/intermediate, area/liveops]
---
# Deferred Events - Decide From the Arguments, Not the Property

<!-- tldr:start -->
> [!tldr] TL;DR
> - A quick unequip and re-equip isn't detected: Handler rereads `tool.Parent`, which is back to the start -> Use the `parent` argument of `AncestryChanged`
> - A state machine skips a state: Two changes in one frame; the handler sees only the last -> Record each transition from the event arguments
> - A bug that only shows up under Deferred: Logic assumes handlers run inside the change -> Treat handlers as running later; never assume ordering
> - Handlers decide from their arguments, not from rereading the property.
> - Quick out-and-back changes (unequip and re-equip, reparent and restore) tested in one frame.
> - The place runs under Deferred signal behaviour in every test.
>
> Cost: TL;DR ~210 tok · whole note ~1.0k tok. Read more: ## What happened · ## The fix · ## Common mistakes · ## Checklist
<!-- tldr:end -->

With deferred signal behaviour, an event handler doesn't run when the change happens. It runs at the next resumption point. By then the property it was told about may have changed again. A handler that rereads the property sees only the final state and misses everything in between. This note shows the trap that let a cancel-on-unequip check miss a real unequip, and the one-line fix. Background on deferred signals is in [[Debugging Race Conditions and Yield Order]] and [[Frame Order and RunService Timing]].

Source labels: **Roblox docs (as of October 2026)**, **Measured** (field-tested in a Roblox game in development, 2026-10-23), **Rule of thumb**.

---

## What happened

**Measured.** A heal item had to cancel if the player put it away. The check watched the tool's `AncestryChanged` and, in the handler, read `tool.Parent` to see whether it was still in the character. A player who put the heal away and took it out again within one frame kept healing.

Why: the place delivered engine events deferred (**Roblox docs**: deferred is a supported `Workspace.SignalBehavior` mode, and new places default to it). The tool moved character → Backpack → character with no yield in between. No handler ran until the next resumption point. Then both handlers ran, and in each one `tool.Parent` was already the character again. The handlers' own **arguments** still said Backpack, then character.

## The fix

Decide from the arguments the event passes. Never reread the property inside the handler.

```lua
-- Cancel a channelled use if the tool ever leaves the character, even for one frame.
local function watchTool(tool: Tool, character: Model, cancel: () -> ())
	local connection
	connection = tool.AncestryChanged:Connect(function(_child: Instance, parent: Instance?)
		-- 'parent' is where the tool went at the time of this change, not where it is now.
		if parent ~= character then
			connection:Disconnect()
			cancel()
		end
	end)
	return connection
end
```

The same rule applies to `Changed`, `GetPropertyChangedSignal`, `ChildAdded` and `ChildRemoved`. Use the value the signal hands you, or keep your own record of each transition, but don't read the live property and assume nothing happened since.

**Don't** switch the place to `Immediate` to make the bug go away. Roblox has said the default will move to Deferred ([[Engine and API Changes 2025-2026]]). Code that only works under Immediate will break later.

## Common mistakes

| Symptom | Cause | Fix |
|---|---|---|
| A quick unequip and re-equip isn't detected | Handler rereads `tool.Parent`, which is back to the start | Use the `parent` argument of `AncestryChanged` |
| A state machine skips a state | Two changes in one frame; the handler sees only the last | Record each transition from the event arguments |
| A bug that only shows up under Deferred | Logic assumes handlers run inside the change | Treat handlers as running later; never assume ordering |
| The bug gets "fixed" by setting Immediate | Hides the race; the default is moving to Deferred | Fix the handler, keep Deferred |

## Checklist

- [ ] Handlers decide from their arguments, not from rereading the property.
- [ ] Quick out-and-back changes (unequip and re-equip, reparent and restore) tested in one frame.
- [ ] The place runs under Deferred signal behaviour in every test.
- [ ] No logic depends on a handler running before the next line of the code that fired it.

## See also

[[Debugging Race Conditions and Yield Order]] · [[Frame Order and RunService Timing]] · [[Tools and Inventory Systems]] · [[ipairs Stops at the First nil - Cleanup Tables That Leak]] · [[Field Notes Start Here]]

<!-- related:start -->
## Related
[[Events Signals and Connections]] · [[Building a Signal Library]] · [[Task Library Coroutines and Yielding]] · [[Replication Rules]] · [[Cookbook - Remotes and Networking]] · [[Streaming Joins - Frozen Players and Loading Screen Teardown]]
<!-- related:end -->
