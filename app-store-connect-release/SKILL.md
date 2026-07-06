---
name: app-store-connect-release
description: Use when preparing, updating, reviewing, or submitting an iOS app in App Store Connect, including metadata, screenshots, app icons, subscription copy, App Review notes, build selection, export compliance, and final submission. Especially useful for XM LLC apps and AI apps that need task-specific operator/reviewer agent roles and review notes explaining owner-hosted/local AI processing rather than third-party AI services.
---

# App Store Connect Release

Use this skill for App Store Connect release work: resubmissions, screenshot replacement, subscription metadata checks, App Review notes, build/version selection, and final submission.

## Safety Rules

- Always identify the exact target app, version, build, bundle ID, and App Store app ID before editing anything.
- If local release config exists, read it before asking the user for repeated metadata. Treat it as private local data: never commit it, paste secrets from it, or echo sensitive fields.
- Prefer separate task agents when the environment supports it: `Release Operator` agents for edits/uploads and task-specific read-only reviewers before final submission.
- Use the already-authenticated browser session when possible. If using Safari/Chrome UI automation, call the app-state inspection tool before interacting each turn.
- Never change `Pricing and Availability`, remove an app from sale, add it back to sale, delete an app, create a new app, change subscription price, or add/remove introductory offers unless the user explicitly asks for that exact change and confirms at action time.
- Final `Submit for Review` always requires action-time confirmation, even if the user said “no need to confirm” earlier.
- If the browser lands on the wrong app, a New App dialog, or another product page, stop and navigate back to the target app before continuing.
- Do not paste or repeat account passwords, OTPs, API keys, or other secrets in responses.

## Local Release Config

- Preferred directory: `~/.codex/app-store-connect-release/`.
- `profile.json` is the only manually required local config. Read it first when present. Use it for reusable publisher defaults: company/developer name, default support/marketing URLs, standard EULA URL, common legal/review notes, default screenshot conventions, and standard AI/private-model disclosure snippets.
- `apps.json` is optional generated cache, not required setup. If absent, discover app-specific facts from the user's request, local Xcode project, and live App Store Connect state.
- When exact app facts are confirmed, you may create or update `apps.json` with non-secret app-specific facts: app display name, slug, App Store app ID, bundle ID, local project path, Xcode project/scheme, API/backend domain, legal URLs, subscription product IDs, default review notes, selected reusable snippet names, and screenshot paths.
- App-specific values override profile defaults. If profile values contain `{app_slug}`, replace it with the matched app's `slug`.
- Combine profile-level `default_review_notes` with app-level `default_review_notes` unless the app explicitly sets `replace_profile_review_notes: true`.
- Use profile-level `review_note_snippets` as reusable named blocks. Only add snippets listed by the app's `review_note_snippets` array or explicitly requested by the user; do not automatically paste AI/private-model disclosures into unrelated apps.
- Do not store Apple account passwords, OTPs, API private keys, session cookies, or payment credentials in local release config.
- If multiple entries match, use exact `app_store_id`, then exact `bundle_id`, then exact app name. If still ambiguous, ask the user which app to use.
- Values discovered live in App Store Connect or Xcode override stale config values. Report mismatches before mutating anything.

## Task-Based Agent Workflow

- Split the release into task lanes. Each lane has an Operator, a read-only Reviewer, and a written pass/block result.
- **Target & Build**: Operator selects the app/version/build. `Build Reviewer` verifies app name, App Store app ID, bundle ID, marketing version, build number, selected build, and export compliance state.
- **Metadata Copy**: Operator edits Promotional Text, Description, What's New, keywords, support/marketing URLs, and review notes. `Metadata Reviewer` checks grammar, product claims, stale trial copy, legal links, and consistency with the actual app behavior.
- **Assets**: Operator uploads screenshots and app icon assets. `Asset Reviewer` checks screenshot counts/dimensions, removes stale screenshots, and verifies App Icon source images are full-bleed square with no pre-rounded corners, internal frame, or border.
- **Subscription & Legal**: Operator checks product IDs, price copy, renewal copy, Privacy Policy, and Terms of Use. `Subscription Legal Reviewer` checks that trial/introductory-offer claims match App Store Connect and that required links are visible.
- **AI & Privacy**: Operator writes AI-processing and privacy notes. `AI Privacy Reviewer` verifies owner-hosted/private AI claims, backend domains, data-retention claims, and confirms no third-party AI-service claim is made unless true.
- **Store State & Submission**: Operator prepares `Add for Review`. `Store State Reviewer` verifies `Pricing and Availability` and sale/removal status were not changed unless explicitly requested.
- **Release Captain**: combines all reviewer results. Operator may proceed to final `Submit for Review` only when every reviewer reports `PASS` or the user explicitly accepts a named risk.
- If multi-agent tools are unavailable, simulate the split with separate passes in the same session. Reviewer passes must be read-only: no clicks, uploads, edits, availability changes, or submission.
- Final `Submit for Review` still needs immediate user confirmation.

## Standard Workflow

1. **Ground the target**
   - Read local release profile if present, then confirm live App Store Connect/Xcode facts. Use optional app cache only when it already exists or after exact app facts have been confirmed.
   - Confirm target app name, App Store app ID, bundle ID, local project path, marketing version, build number, and selected build.
   - Inspect current App Store Connect state before acting: `Prepare for Submission`, `Ready for Review`, `Waiting for Review`, rejected, or removed from sale.
   - Treat “Removed from App Store” as an existing availability state unless the user specifically asks to change availability.

2. **Check local release facts**
   - Verify version/build from Xcode project settings.
   - Build if needed with the app's known `xcodebuild` command.
   - Search for stale trial/free-trial copy before submission:
     ```sh
     rg -n "3 Days|3 days|Try 3|free trial|trial|introductoryOffer|免费试用|试用" ios -S || true
     ```
   - If Safari is on the App Store Connect page, scan visible page text:
     ```sh
     osascript -e 'tell application "Safari" to do JavaScript "document.body.innerText" in document 1' \
       | rg -i "3-day|3 day|try 3|free trial|trial|试用|introductory" || true
     ```

3. **Metadata, icons, and screenshots**
   - Subscription copy should be concise and professional: state the free limit, Pro limit, price, renewal nature if relevant, and link to Privacy Policy and Terms of Use.
   - If free trial has been canceled, remove all trial claims from app UI, StoreKit config, screenshots, Promotional Text, What's New, Description, Review Notes, and subscription offers.
   - App Icon assets must be full-bleed square images with no pre-rounded corners, no internal rounded-square frame, and no visible border. Xcode/App Store/iOS apply the final rounded-corner mask automatically.
   - If generated icon artwork already includes rounded corners or a border, regenerate or edit the source PNGs so the artwork fills the full square canvas.
   - For screenshots, verify dimensions before upload. Common App Store sizes:
     - iPhone 6.5": `1242 x 2688`
     - iPad 12.9": `2048 x 2732`
   - After replacing screenshots, verify the old screenshots are gone and each required device family has at least one valid screenshot.

4. **AI App Review Notes**
   - For AI apps using owner-hosted/local/private models, include this note only when true:
     ```text
     AI processing is performed by XM LLC's own private/local model service. User audio is not sent to third-party AI model providers or third-party AI APIs. The app uses Apple's in-app purchase system for subscription payments.
     ```
   - For AI Song Cover-style apps, use or adapt:
     ```text
     No account sign-in is required. Reviewers can import a song file from Files and record or import a voice reference, then generate a free 30-second preview. The Pro subscription com.xmaillc.aisongcover.monthly is $4.99/month and enables renders up to 5 minutes. Backend API domain: https://cover.xmaillc.com.

     AI processing is performed by XM LLC's own private/local model service. User audio is not sent to third-party AI model providers or third-party AI APIs.

     The app purchase flow includes functional links to Privacy Policy and Terms of Use. Privacy Policy: https://www.xmaillc.com/ai-song-cover-privacy.html. Terms of Use: https://www.apple.com/legal/internet-services/itunes/dev/stdeula/.
     ```
   - Do not claim “local/private AI” if the app actually calls third-party model APIs.

5. **Submit and verify**
   - Click `Add for Review` only after metadata, app icon, screenshots, selected build, legal links, and review notes are correct.
   - Run all task-specific reviewer passes after `Add for Review` but before final `Submit for Review`.
   - Before clicking `Submit for Review`, ask for immediate confirmation: “最后一步会正式提交给 Apple 审核。请确认现在提交吗？”
   - After submission, verify status changes to `Waiting for Review` and capture the key facts: app, version, build, submission result, and any review submission link or ID shown.

## Common Copy Defaults

- Promotional Text:
  ```text
  Create AI song covers with your song and voice reference. Free previews are capped at 30 seconds. Pro renders up to 5 minutes.
  ```
- What's New:
  ```text
  Updated subscription copy. Refreshed screenshots and legal links. Clarified Free and Pro render limits.
  ```
- Subscription description:
  ```text
  AI Song Cover Pro unlocks renders up to 5 minutes for $4.99/month. Free users can create 30-second previews.
  ```

## Final Response

Summarize only the result and the important state:

- Target app, version, and build.
- What was changed.
- Final App Store Connect status.
- Anything intentionally not changed, especially `Pricing and Availability` or sale availability.
