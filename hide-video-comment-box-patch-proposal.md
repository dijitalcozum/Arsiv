# Hushfacebook patch proposal: Hide video comment composer

Status: **Draft / not ready to apply**  
Target repository: https://github.com/SysAdminDoc/HushFacebook  
Issue: https://github.com/SysAdminDoc/HushFacebook/issues/213

## Intended behavior

Add an opt-in setting named **Hide the video comment box**. When enabled, hide only the `Yorum ekle` / “Add a comment” composer at the bottom of Facebook's full-screen video viewer (`fb://watch`). Keep the video, timeline, playback controls, and separate Like/Comment/Share actions working.

The setting should default to off and must not hide comment boxes on ordinary feed posts.

## Why this is not yet a safe, ready-to-apply source patch

The current repository's `CommentSheetOptionsPatch.kt` already locates the full-screen viewer's `render` method (`LX/AET;->render` in the documented Facebook build) and can remove the viewer's GIF and sticker buttons. It does **not** currently identify the composer container itself. The existing hook is applied after the GIF button is built and only clears the GIF/sticker registers.

To safely remove the whole composer, the patch needs to identify the exact register or child-list entry for the composer container in each supported Facebook APK version. Guessing that register or clearing the wrong child could break the viewer layout or remove other controls. That information is not available from the repository source alone; it comes from inspecting the relevant Facebook APK bytecode / fixture.

## Implementation checklist

1. In `extensions/facebook/src/main/java/app/morphe/extension/facebook/settings/Settings.java`, add an off-by-default `BooleanSetting`, for example:
   `HIDE_VIDEO_COMMENT_COMPOSER = new BooleanSetting("hushfacebook_hide_video_comment_composer", FALSE);`
2. In `extensions/facebook/src/main/java/app/morphe/extension/facebook/comments/CommentSheetOptions.java`, add a narrowly scoped helper that returns whether the composer should be hidden, checking `Utils.settingsReady()`, the new setting, and `HookStatus` diagnostics using the existing class conventions.
3. In `patches/src/main/kotlin/app/morphe/patches/facebook/comments/options/CommentSheetOptionsPatch.kt`, extend the `viewerButtons` discovery result / patch logic to identify the composer container in the viewer's render method and omit only that child while the new setting is enabled. Preserve the existing GIF/sticker and mention-button behavior.
4. Add the setting to the existing settings screen, `SwitchLabels.java`, `PatchFamily.java`, backup/restore registration, and the appropriate `extensions/shared/library/src/main/l10n/*.tsv` translations.
5. Add bytecode fixture tests for each supported viewer layout and extension tests verifying the default-off behavior and enabled behavior.
6. Build the patch bundle and test against each supported Facebook version before submitting a pull request.

## Acceptance criteria

- Default off: no behavior change.
- Enabled: the bottom “Add a comment” composer is absent in the full-screen video viewer.
- The video, seek bar, playback controls, and Like/Comment/Share actions remain usable.
- Comment composers elsewhere in Facebook are unchanged.
- Unsupported or changed viewer layouts cause the patch to refuse safely instead of patching a guessed register.

## Current status

This document is an implementation proposal, not a compiled or tested code patch. GitHub branch creation was denied to the connected integration with HTTP 403, so the changes have not been pushed to the upstream repository. The issue describes the requested behavior: https://github.com/SysAdminDoc/HushFacebook/issues/213
