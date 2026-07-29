# ChristopherBell.dev Tools Menu Access Navigation

- Status: active

## Objective

Move Music, Command Center, and Back Office into the Tools dropdown, remove their prior top-level/profile-menu copies, show each only to accounts with effective access, and keep every visible Tools entry alphabetized.

## Owner and Context

- Coordinator: Codex in the Builder hub
- User authorization: proceed autonomously; commit and push completed work
- Date opened: 2026-07-29

## Spoke Repository

- Repository: `azurras/christopherbell.dev`
- Base: refreshed `origin/main` at `e393687d10c40b856f35d669c25bf3ea65c5c083`
- Preserve the dirty authoritative checkout; implement in a new isolated worktree.

## Access Contract

- Music: visible only with effective `MUSIC_READ` or `MUSIC_WRITE`; `ADMIN` has effective access.
- Back Office and Command Center: visible only for exact `ADMIN` role.
- Shared Folder: retain existing effective read gating.
- Existing public tools remain public.
- Sort the final visible list alphabetically by label.

## Validation

- Test-first JavaScript unit coverage for visibility, source-menu removal, and ordering.
- Full website check.
- Local runtime/browser navigation checks for signed-out, authorized listener, and administrator states when fixtures allow.
- One PR, green CI, merge, automatic deployment, and public smoke verification.

## Next Steps

1. Save the approved spec and implementation plan.
2. Implement from fresh main in an isolated worktree.
3. Test, publish, merge, deploy, verify, and close the record.
