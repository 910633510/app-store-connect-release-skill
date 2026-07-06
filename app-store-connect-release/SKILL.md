---
name: app-store-connect-release
description: Use when preparing, updating, reviewing, or submitting an iOS app in App Store Connect, including metadata, screenshots, app icons, subscription copy, App Review notes, build selection, export compliance, and final submission. Especially useful for XM LLC apps and AI apps that need separate operator/reviewer agent roles and review notes explaining owner-hosted/local AI processing rather than third-party AI services.
---

# App Store Connect Release

Use this skill for App Store Connect release work: resubmissions, screenshot replacement, subscription metadata checks, App Review notes, build/version selection, and final submission.

## Safety Rules

- Always identify the exact target app, version, build, bundle ID, and App Store app ID before editing anything.
- Prefer two separate agents when the environment supports it: one `Release Operator` for edits/uploads, and one `Release Reviewer` for read-only QA before final submission.
- Use the already-authenticated browser session when possible. If using Safari/Chrome UI automation, call the app-state inspection tool before interacting each turn.
- Never change `Pricing and Availability`, remove an app from sale, add it back to sale, delete an app, create a new app, change subscription price, or add/remove introductory offers unless the user explicitly asks for that exact change and confirms at action time.
- Final `Submit for Review` always requires action-time confirmation, even if the user said “no need to confirm” earlier.
- If the browser lands on the wrong app, a New App dialog, or another product page, stop and navigate back to the target app before continuing.
- Do not paste or repeat account passwords, OTPs, API keys, or other secrets in responses.

## Two-Agent Workflow

- **Release Operator**: performs the actual App Store Connect work: metadata edits, screenshot uploads, build selection, export compliance, `Add for Review`, and final submission after user confirmation.
- **Release Reviewer**: performs a read-only review after Operator finishes edits and before final `Submit for Review`. The Reviewer must not click buttons, upload files, edit metadata, change availability, or submit.
- If multi-agent tools are unavailable, simulate the split with two explicit passes: first an Operator pass, then a separate Reviewer pass with fresh checks and no mutating actions.
- Operator must hand Reviewer a short release packet: app name, App Store app ID, version/build, bundle ID, selected build, screenshots replaced, subscription/trial status, App Review notes, legal URLs, and anything intentionally not changed.
- Reviewer must check target app identity, sale availability state, stale trial copy, app icon full-bleed/no-border rule, screenshot counts/dimensions, subscription copy, legal links, AI-processing claims, selected build, and review notes.
- Operator can proceed to final submission only after Reviewer reports no blocking issues. Final `Submit for Review` still needs immediate user confirmation.

## Standard Workflow

1. **Ground the target**
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
   - Run the Release Reviewer pass after `Add for Review` but before final `Submit for Review`.
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
