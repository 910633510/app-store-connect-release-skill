# App Store Connect Release Skill

A Codex skill for preparing and submitting iOS apps in App Store Connect with fewer release-day mistakes.

It is designed for workflows that involve:

- App Store metadata updates
- screenshot replacement
- subscription copy review
- App Review notes
- final `Submit for Review` safety checks
- AI app review notes for owner-hosted/private model services

The skill is intentionally cautious around App Store state changes. It tells the agent not to touch `Pricing and Availability`, remove an app from sale, add it back to sale, change subscription price, or submit for review without action-time confirmation.

## Install

Copy the skill folder into your Codex skills directory:

```sh
mkdir -p ~/.codex/skills
cp -R app-store-connect-release ~/.codex/skills/
```

Restart Codex or start a new session so the skill metadata is loaded.

## What It Covers

- Identifying the exact app, version, build, bundle ID, and App Store app ID before editing.
- Checking for stale free-trial or introductory-offer copy before submission.
- Replacing App Store screenshots while validating common required dimensions.
- Filling App Review notes with privacy, legal, subscription, and AI-processing context.
- Requiring a final confirmation before `Submit for Review`.
- Recording final status such as `Waiting for Review`.

## AI App Review Notes

For apps that use an owner-hosted/private AI model service, the skill includes a review-note template:

```text
AI processing is performed by XM LLC's own private/local model service. User audio is not sent to third-party AI model providers or third-party AI APIs. The app uses Apple's in-app purchase system for subscription payments.
```

Only use that note when it is true for the app being submitted.

## Status

This is a small operational skill extracted from a real App Store Connect submission workflow.
