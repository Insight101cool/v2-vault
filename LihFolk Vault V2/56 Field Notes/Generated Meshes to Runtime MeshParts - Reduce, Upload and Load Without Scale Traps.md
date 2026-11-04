---
tags: [field-notes, meshes, open-cloud, upload, rojo, blender, ai-models, import, meshpart, surfaceappearance, type/guide, area/art, level/intermediate, area/ai]
---
# Generated Meshes to Runtime MeshParts - Reduce, Upload and Load Without Scale Traps

<!-- tldr:start -->
> [!tldr] TL;DR
> - Blender runs for an hour and dies on a generated model: Decimate on millions of triangles -> `pyfqmr`, after welding the seams
> - Reducer stops far above the target: Seam-split vertices leave open edges -> Merge by distance first
> - Setting the rotation does nothing: glTF import uses quaternion rotation -> `rotation_mode = "XYZ"` first
> - Generated meshes welded, then reduced with a fast reducer, to the prop's budget.
> - Each prop's facing fixed from a preview before export.
> - Textures uploaded as `Image`; maps applied through synced templates.
>
> Cost: TL;DR ~180 tok · whole note ~2.1k tok. Read more: ## 1. Reduce in Blender without waiting an hour · ## 3. Loading at runtime: two routes work, one doesn't · ## Common mistakes · ## Checklist
<!-- tldr:end -->

A Rojo project that never saves its place has to rebuild every model on each fresh Play from files and asset ids on disk. That rules out "import it in Studio and save". This note covers the route that worked for million-triangle generated models (for example, from an image-to-3D tool): reduce them fast in Blender, upload through Open Cloud, and load them at runtime at the right size. It also covers the three traps that cost the most time. The general import rules (units, pivots, Apply Scalings) are in [[AI Pitfalls - Assets and Import]] and [[Exporting and Validating Models for Roblox]]. Upload automation is in [[Asset Upload Automation with Open Cloud]].

Source labels: **Roblox docs (as of October 2026)**, **Measured** (field-tested in a Roblox game in development, 2026-10-22 to 2026-10-27), **Rule of thumb**.

---

## 1. Reduce in Blender without waiting an hour

**Measured.** Collapsing a 1,883,271-triangle generated throne to 10,000 triangles with Blender's Decimate modifier ran on one core for over 45 minutes, and the process ended with no output.

- **Use a fast quadric reducer.** `pyfqmr` installed for Blender's Python (`--user`) did the same reduction in under a minute, with the same look after the texture bake. Blender's Python skips the user site folder, so add it to `sys.path` in the script.
- **Weld first.** A glTF import splits vertices along every UV seam, and a reducer won't collapse open edges. One torch stopped at 13,218 triangles against a 3,000 target until its vertices were merged by distance.
- **Rotation does nothing?** glTF imports set `rotation_mode` to `QUATERNION`, so writing `rotation_euler` has no effect. Set `obj.rotation_mode = "XYZ"` first.
- **Facing isn't consistent between models.** One generated knight came in facing backwards and a torch sideways. Give each prop its own turn, read off a preview render, before export. The facing convention itself is in [[AI 3D Model Generation Outside Roblox]].

## 2. Uploading through Open Cloud

**Measured.** `POST https://apis.roblox.com/assets/v1/assets` (multipart: a `request` JSON with `assetType`, `displayName`, `description` and `creationContext.creator`; a `fileContent` part), then poll the returned operation until `done` (**Roblox docs** for the endpoint).

| assetType | File | What came back |
|---|---|---|
| `Image` | png | The image id directly (no decal wrapper). Usable for textures, UI and sky faces. Reviewed in about a minute. |
| `Animation` | `.rbxm` | Plays, and its markers fire on time. |
| `Model` | `.fbx` / `.glb` | A package Model of MeshParts with new mesh ids. **FBX textures did not come through.** Upload every map as an `Image` and apply them yourself (section 4). |

## 3. Loading at runtime: two routes work, one doesn't

| Route | Result |
|---|---|
| `InsertService:LoadAsset(modelId)` on the server | **Works** when the place owner owns the asset. Best for a multi-part model whose parts sit relative to each other. Load once, keep it in ServerStorage, clone per use. |
| `AssetService:CreateMeshPartAsync(Content.fromAssetId(meshId), options)` | **Works**, at the correct size. Make each mesh once per server and clone it. Get the mesh ids by loading the model once in Edit and reading each MeshPart's `MeshId`, then store them in config. |
| A Rojo `.model.json` MeshPart with only `MeshId` set | **Don't.** The mesh renders, but `MeshSize` stays 0, so the part ignores its `Size` and drew about 3x too big. |

```lua
-- Server: build each mesh once, clone per use. Ids come from your config.
local AssetService = game:GetService("AssetService")

local templates: { [string]: MeshPart } = {}

local function meshPart(name: string, meshId: number): MeshPart
	local template = templates[name]
	if not template then
		template = AssetService:CreateMeshPartAsync(Content.fromAssetId(meshId), {
			CollisionFidelity = Enum.CollisionFidelity.Box,
			RenderFidelity = Enum.RenderFidelity.Automatic,
		})
		templates[name] = template
	end
	return template:Clone()
end
```

**The scale trap (Measured).** One multi-part kit (authored at 1 unit = 1 stud) came back from its `Model` upload with every MeshPart about 21.5x too big. The tallest part was exactly 2048 studs, so Roblox had scaled the whole model uniformly. Axes and each part's centre were kept. The same import read another FBX at 100x. A later single-prop upload loaded at the right size. So never trust imported sizes: export each part's real size and centre from the source (a small generated Luau table works), and set `Size` and `CFrame` from it after loading. A check like the `MeshSize` ratio in [[Exporting and Validating Models for Roblox]] catches it.

**Not a route (Measured, 2026-10-22).** `AssetService:CreateAssetAsync` with an `EditableMesh` and `Enum.AssetType.Mesh` returned "CreateAssetAsync and CreateAssetVersionAsync are not available yet". Building meshes at runtime from an `EditableMesh` does work in Studio. In published experiences, though, the Mesh/Image APIs have to be switched on in Experience Settings by an eligible creator, and the content isn't replicated. A MeshPart made from an `EditableMesh` also keeps the vertices' own origin, so centre the vertices on their bounding box first.

## 4. Materials that survive without a save

- **SurfaceAppearance.** Game scripts can't set its map properties (Plugin security, **Roblox docs**), but a Rojo-synced template can. Write a `SurfaceAppearance` `.model.json` with the uploaded image ids, then clone it onto the runtime MeshPart.
- **MaterialVariant.** Same idea: `.model.json` files in a folder mapped into `MaterialService` by the project file. At runtime, set `part.Material` to the base material and `part.MaterialVariant` to the variant's name. Keep variant parts near-white, because the part's `Color` multiplies the map ([[Materials MaterialVariants and SurfaceAppearance]]).
- **Lighting technology.** `Lighting.Technology` is deprecated and not scriptable, so not even Rojo can set it. Set the lighting style in Studio and say so in the hand-off.

## Common mistakes

| Symptom | Cause | Fix |
|---|---|---|
| Blender runs for an hour and dies on a generated model | Decimate on millions of triangles | `pyfqmr`, after welding the seams |
| Reducer stops far above the target | Seam-split vertices leave open edges | Merge by distance first |
| Setting the rotation does nothing | glTF import uses [[Quaternions\|quaternion]] rotation | `rotation_mode = "XYZ"` first |
| Uploaded model loads 20x or 100x too big | Uniform rescale on upload (largest part to 2048 in one case) | Set `Size` and `CFrame` from source data |
| Model uploads with no textures | FBX textures don't come through | Upload maps as `Image`; synced SurfaceAppearance template |
| Rojo MeshPart draws at the wrong size | `MeshSize` stays 0 with only `MeshId` set | `CreateMeshPartAsync` or `LoadAsset` at runtime |
| Script can't upload a mesh | `CreateAssetAsync` for meshes not available | Upload the file through Open Cloud |

## Checklist

- [ ] Generated meshes welded, then reduced with a fast reducer, to the prop's budget.
- [ ] Each prop's facing fixed from a preview before export.
- [ ] Textures uploaded as `Image`; maps applied through synced templates.
- [ ] Meshes built once per server with `CreateMeshPartAsync` or `LoadAsset`, then cloned.
- [ ] Sizes and centres set from source data, never trusted from the import.
- [ ] No Rojo MeshPart that relies on `MeshId` alone.
- [ ] Every id lives in a committed config or id map.

## See also

[[Asset Upload Automation with Open Cloud]] · [[AI 3D Model Generation Outside Roblox]] · [[AI Pitfalls - Assets and Import]] · [[Importing Models and Rigs]] · [[Field Notes Start Here]]

<!-- related:start -->
## Related
[[Moving Data Out of a Play Session]] · [[Running Many AI Agents on One Roblox Place]] · [[Deferred Events - Decide From the Arguments, Not the Property]] · [[Centre-Screen Announcements That Stay Centred]] · [[Studio Automation Without Freezing Studio]] · [[Sound Design Pitfalls - Reused Ids Onsets and Layered Loops]] · [[Comparable Studio Look Shots - Quality Level, Settle Time and Camera Ownership]] · [[Animation Data Hygiene - Euler Wrap Joint Holes and Easing]]
<!-- related:end -->
