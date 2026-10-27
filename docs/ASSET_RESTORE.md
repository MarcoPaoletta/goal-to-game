# Asset restoration

**WORK IN PROGRESS. DO NOT DELETE THIS PROJECT.**

This game or its production copy is protected because its video is unpublished, it is reused by pending production, or it is an unfinished direction. This documentation does not change that protection. No code, scene, asset, cache or file location was changed to prepare it.

Recorded on 2026-10-27.

Repository: https://github.com/MarcoPaoletta/goal-to-game.git

Local branch: `main`. Metadata baseline commit: `e5878de3d913747a05ae73118d40539d2c9e7464`.

## Restore workflow

1. When restoration is actually needed, clone the matching code revision. Restore Git-managed files first and use `git lfs pull` if the repository uses LFS.
2. Consult the source manifests below for the exact models, packs, relative destinations, versions and import settings. Keep downloaded models and their textures together.
3. For generated assets, use the original provider account and original model/export name or persistent ID. Marco confirmed on 2026-10-27 that his generated originals remain in those accounts. Missing direct URLs or IDs are not a reason to generate a replacement. No current API download test is claimed here.
4. Retrieve the same rigged/animated and texture variant, preserve filenames, and reapply the recorded scale, orientation, animation mapping and postprocess scripts.
5. SDKs, editor integration and the private development skill are installed separately. Do not commit account credentials or the private skill copy. Godot/Unity imports and other generated caches can rebuild when the project is reopened.

## Current authoritative model and source records

* `README.md`

## Stable source/reference URLs already recorded

No signed download URL or credential is stored here.

* https://astral.sh/uv/install.ps1
* https://astral.sh/uv/install.sh
* https://claude.ai/install.ps1
* https://claude.ai/install.sh
* https://claude.com/claude-code
* https://code.claude.com/docs/en/plugins
* https://code.claude.com/docs/en/skills
* https://github.com/thrixel/goal-to-game
* https://lettacorporation.com
* https://modelcontextprotocol.io
* https://raw.githubusercontent.com/ThatCharlieK/READMEAssets/main/Thrixel-1prompt-to-game-readme.gif
* https://thrixel.com/
* https://thrixel.com/create
* https://thrixel.com/create/#settings/billing
* https://thrixel.com/create/#upgrade
* https://thrixel.com/docs/goal-to-game
* https://thrixel.com/docs/goal-to-game#codex
* https://thrixel.com/docs/goal-to-game#gemini-cli

## Cleanup status

**Protected and not ready for deletion.** Keep this project local. Update this file as new models are imported, but do not archive the game merely because this file exists.
