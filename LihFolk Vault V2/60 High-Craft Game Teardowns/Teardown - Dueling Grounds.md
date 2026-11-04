---
tags: [teardown, high-craft, pvp, melee, duel, parry, posture, timing, frame-data, type/case-study, area/design, level/advanced]
verified: 2026-11-04
---
# Teardown - Dueling Grounds

<!-- tldr:start -->
> [!tldr] TL;DR
> - Avoid: Text pop-ups for hit, block and parry -> drop them; give each outcome its own look and colour.
> - Avoid: A burst on every light hit -> keep bursts for parries, blocks and breaks, and give a plain hit only the slash line.
> - Avoid: A heavy that shares the light's button with a hold -> put it on its own button, so it lands about 0.3 s after the press.
> - Light chain gap near 0.42 s; heavy contact about 0.3-0.35 s after the press.
> - One distinct look per outcome; no text pop-ups.
> - Bursts only on parry, block and break.
>
> Cost: TL;DR ~170 tok · whole note ~3.0k tok. Read more: ## The craft that sets it apart · ## How this was measured (reuse it on any reference clip) · ## Common mistakes when copying this style · ## Checklist
<!-- tldr:end -->

Dueling Grounds is a 1v1 sword-duel game: two players in a small ring, short rounds, and combat built on a light chain, a separate heavy, block, parry and a posture (guard) bar that breaks into a punish. This note is different from the other teardowns. Its numbers come from frame-stepping real duels, not from store pages, so it is mostly frame data you can build against.

Trust labels: **Measured** here means read frame by frame from screen recordings of real duels: a 120 s Classic 1v1 on PC in third person with lock-on, and a 20.65 s clip of heavies swung alone in the lobby, recorded and measured 2026-11-03. Precision is ±1-2 frames (about ±17-33 ms) because the recordings have a variable frame rate that repeats frames now and then. **Rule of thumb** is our design reading, and **Unverified** is a community guide we could not check. Distances are in body heights (an R15 character is about 5 studs tall) and are estimates, because the footage has no scale.

## Snapshot

| Field | Value | Label and source |
|---|---|---|
| Genre | 1v1 melee duels, best-of rounds, several weapon types (katana, sickle-type and others seen) | Measured, recordings 2026-11-03 |
| Camera | Third person with lock-on | Measured |
| Visits, rating, CCU | Not checked: the Roblox games API was unreachable when this note was written | Fill in on the next refresh |

## Core loop

Queue, enter a ring, fight to a round win, repeat. Each round is a short exchange of chains, parries and guard breaks; a typical round in the recording lasts about 40 s (round 1: FIGHT at 27.5 s, death at 71.3 s).

## First 60 seconds

- The only words on screen are ROUND n, FIGHT, DEATH and VICTORY (**Measured**). There are no text pop-ups for hit, block, parry or break. Every outcome is taught by its look and sound.
- **Rule of thumb:** a duel game can skip a tutorial if every outcome has its own unmistakable effect and colour. That is what the feedback vocabulary below does.

## The craft that sets it apart

**1. A feedback vocabulary with one look per outcome (Measured).**

| Outcome | Look | Duration |
|---|---|---|
| Hit lands | A thin red slash line across the victim (wider and glowing for a heavy), blood spatter, the overhead bar flashes red, the health bar drops with a pale chunk that drains after | line 0.08-0.12 s (5-7 frames); overhead red about 0.25 s |
| Parry | Gold-white flash on the parrier (white-hot on the body, gold around it), then orange sparks that hang and fall; the attacker's overhead bar turns red | flash 0.08 s (5 frames); sparks about 0.5 s; attacker's bar red about 0.38 s |
| Block | Smaller white-pink burst and orange sparks on the blocker, who slides back a little; the guard bar drops | burst about 0.1 s, sparks about 0.25 s |
| Guard (posture) break | White shard burst with streaks, the screen edges darken red, a red X on the victim's chest | burst about 0.5 s; red X about 0.73 s |
| Knockdown | After the post-break heavy: a stumble, then flat on the ground with a dust puff | stumble about 0.8 s, down about 1.2 s |
| Dodge | Translucent ghost body during a short dash, faint smear at the feet | about 0.35 s |
| Heavy tell | Small white glint on the blade at the start of the swing | glint to contact 0.42 s |

A light hit has **no burst** at all, only the red line. Bursts are kept for parries, blocks and breaks, so the flashiest effect always means "something changed" (**Measured**, frame crops at 28.65, 29.22 and 30.85 s).

**2. A fast, rhythmic light chain.** Hit-to-hit gaps inside a chain are 0.33-0.52 s (median 0.42 s over 12 gaps), and the startup is estimated at 0.15-0.2 s (**Measured** for gaps; startup **Est.**, since the press isn't visible). That is faster than the 8-frame (0.27 s) PvP reaction rule in [[Locomotion and Melee Reference Numbers]]. **Rule of thumb:** DG doesn't expect you to react to a single light. You defend against the chain's rhythm and the opponent's commitment, and you react to the heavy, which has a tell.

**3. A heavy that commits.** In the heavy clip every swing is a heavy: a two-handed overhead chop (or a low horizontal draw-cut) from the first wind-up frame to contact in 0.23-0.34 s (overhead chops 0.29-0.34 s), then 0.5-0.8 s held in a deep low two-handed crouch before it rises back to the ready stance (**Measured**). In duels the glint shows 0.42 s before contact. A heavy deals about 2x a light (7-10% of health against 3-5%).

**4. Parry as the pivot of the whole duel.** A parry flashes for 5 frames and leaves the attacker's bar red for about 0.38 s. The parrier's first counter lands 0.56-0.80 s after the flash (human reaction included). The parrier *gains* 8-10% guard, and a parry can empty the attacker's guard outright (108.87 s parry, break at 109.0 s) (**Measured**). The parry window itself isn't visible. A community guide gives 8, 10 or 14 active frames, or 0.13-0.23 s (**Unverified**).

**5. Posture breaks that pay off instantly.** Guard drops by 17% from a light hit taken, 19% from a heavy taken, and 26% from a heavy blocked (**Measured**, from the local bar). On a break the victim is stunned under the red X for about 0.73 s, the breaker's heavy lands 0.88-1.18 s later, and a heavy after a break dealt 13% and knocked down for about 1.2 s (one case). Guard doesn't regenerate for at least 7 s of fighting without a break, then refills fast (0.65 to 0.96 in 0.4 s). After a break it is full again 1.2 s later (**Measured**; the rule behind the regen is **Est.**).

**6. Sound on the contact frame.** Every gold or red flash has a sharp audio onset 0.07-0.12 s later in the recording. The lag is the same on every event, which matches the recorder's audio offset, so the sound is on the contact frame in the game (**Est.**). The sound reads as bright metal: spectral centroid 5-6 kHz, parry ring about 0.5 s.

## Progression and economy

Not covered: the recordings show combat only.

## Monetization and 2026 rules

Not covered. The recordings show an ultimate meter; any purchasable power in a 1v1 skill game has to pass [[Policy and Age Rules 2026]] and keep the duel fair.

## Retention

Not covered by the footage. **Rule of thumb:** short rounds and a highlight-dense parry game give natural clips, like [[Teardown - Blade Ball]].

## Tech notes

- **Spacing (Est.).** Contact happens at about 1-1.5 body heights between bodies (about 5-7 studs). In neutral, players circle at 2-4 body heights and back off to 6+. A gap-closing dash covers about 5-6 body heights in about 0.7 s. A dodge dash is about 0.25 s long and about 2 body heights (about 10 studs) far, with about 0.35 s of translucency.
- **Damage scale (Measured).** Light 3-5% of health, heavy 7-10%, heavy after a break 13%.
- **Latency.** The recorded player was on 100-117 ms ping and the opponent on 50-67 ms, and parries still landed. Validate parries with timestamps, not server arrival time: [[Client Prediction and Reconciliation for Combat]] and [[Server-Authoritative Freeze and Stun Timestamps]].
- **Hit feel on the attacker's client** at the contact frame, outcomes from the server: [[Client Hit Prediction for Melee Feedback]].

## How this was measured (reuse it on any reference clip)

1. **Pick the right audio track.** The recorder had three tracks (all audio, PC audio, microphone). Use game-only audio. Check the mic track is silent (here the mean was -59 dB).
2. **Read the HUD per frame.** A small script samples both health bars, the ultimate meters, the local guard bar, and the share of gold, white and red pixels in the action area for every frame. It writes a CSV and plots it. Breaks, hits and parries show up as steps and spikes you can timestamp.
3. **Find sound onsets** with a simple onset detector on the game-audio track, then match each onset to its visual event.
4. **Confirm by eye** on 10 fps and 5 fps contact sheets, then on 30 or 59 fps strips with each tile labelled by its frame time.
5. **Label everything.** Seen, Est. or Unknown, with the ±1-2 frame precision stated once. List what the footage can't show (here: the parry window, input buffering, the exact range in studs, the guard regen rule, light startup from the press).
6. **Compare your build the same way.** Take your times from server timestamps, check the video against them (here within 2 ms), and put side-by-sides at real speed and at quarter speed, aligned on the first shared cue.

## Team and cadence

Not covered.

## Build it yourself

1. **Light chain and separate heavy:** [[Melee Combat System End to End]] and [[Combat Combo Systems]]. Start the light gap at about 0.42 s and the heavy contact at about 0.3-0.35 s after the press, then tune.
2. **Guard as a resource:** a posture bar as here, or fold it into [[Stamina]]. Decide what blocking, being hit and parrying each do to it, using the percentages above as a starting point.
3. **Parry and break windows** validated on the server: [[Server-Authoritative Freeze and Stun Timestamps]].
4. **Feedback vocabulary:** one look per outcome, with no burst on a plain light hit. See [[VFX Design Playbook]] and the `lihfolk:lihfolk-vfx` skill. Put hit-stop and shake on parries and breaks: [[Hit-Stop and Screen Shake Tuning by Numbers]].
5. **Reactions:** stagger on a parry, stun on a break, knockdown only on the punish: [[Hit Reactions Stagger and Hit-Stop]].
6. **Sound on the contact frame:** [[Feedback Timing - Sound Visual and Haptic Sync]].

## Numbers to remember

| Number | Label | Source and date |
|---|---|---|
| Light chain gap 0.33-0.52 s, median 0.42 s | Measured | Duel recording, 2026-11-03 |
| Heavy first wind-up frame to contact 0.29-0.34 s (overhead) | Measured | Heavy clip, 2026-11-03 |
| Heavy recovery 0.5-0.8 s in a held low crouch | Measured | Heavy clip, 2026-11-03 |
| Parry flash 5 frames (0.08 s), sparks about 0.5 s, attacker marked about 0.38 s | Measured | Duel recording, 2026-11-03 |
| Break stun about 0.73 s; punish heavy 0.88-1.18 s after the break | Measured | Duel recording, 2026-11-03 |
| Damage: light 3-5%, heavy 7-10%, punish heavy 13% | Measured | HUD read, 2026-11-03 |
| Parry window 0.13-0.23 s | Unverified | Community guide |

## Common mistakes when copying this style

- **Text pop-ups for hit, block and parry** -> drop them; give each outcome its own look and colour.
- **A burst on every light hit** -> keep bursts for parries, blocks and breaks, and give a plain hit only the slash line.
- **A heavy that shares the light's button with a hold** -> put it on its own button, so it lands about 0.3 s after the press.
- **Sound triggered when the server confirms** -> play it on the contact frame on the attacker's client.
- **Parry checked on server arrival time** -> high-ping players can't parry; validate with timestamps inside a capped rewind.
- **Copying numbers from one recording as exact** -> keep the ±1-2 frame precision and re-measure your own build the same way.

## Checklist

- [ ] Light chain gap near 0.42 s; heavy contact about 0.3-0.35 s after the press.
- [ ] One distinct look per outcome; no text pop-ups.
- [ ] Bursts only on parry, block and break.
- [ ] Parry, block and break windows validated on the server with timestamps.
- [ ] Hit sound on the contact frame.
- [ ] Your build measured against the reference with server timestamps and side-by-sides.

## See also

[[High-Craft Teardowns Start Here]] · [[Teardown - Deepwoken]] · [[Teardown - Blade Ball]] · [[Teardown - Jujutsu Shenanigans]] · [[Genre Playbook - Battlegrounds and Arena Fighters]] · [[Locomotion and Melee Reference Numbers]] · [[Competitor Analysis Method]]

<!-- related:start -->
## Related
[[Teardown - BedWars]] · [[What Craft Looks Like on Roblox - Lessons from Top Studios]] · [[Teardown - Funky Friday]] · [[Teardown - Grace]] · [[Teardown - Phantom Forces]] · [[Teardown - Anime Vanguards]] · [[Teardown - Arsenal]] · [[Teardown - Apocalypse Rising 2]]
<!-- related:end -->
