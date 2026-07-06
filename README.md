# App Store Connect Release Skill

一键发布你的应用，避开所有审核坑。

A Codex skill for preparing and submitting iOS apps in App Store Connect with fewer release-day mistakes.

It is designed for workflows that involve:

- App Store metadata updates
- app icon checks
- screenshot replacement
- subscription copy review
- App Review notes
- task-based operator/reviewer workflow
- final `Submit for Review` safety checks
- AI app review notes for owner-hosted/private model services

The skill is intentionally cautious around App Store state changes. It splits release work into task lanes with separate reviewers, and tells the agent not to touch `Pricing and Availability`, remove an app from sale, add it back to sale, change subscription price, or submit for review without action-time confirmation.

## Install

Copy the skill folder into your Codex skills directory:

```sh
mkdir -p ~/.codex/skills
cp -R app-store-connect-release ~/.codex/skills/
```

Restart Codex or start a new session so the skill metadata is loaded.

## Optional Local Release Config

You can keep reusable publisher details in a private local profile. This is the only file you need to create manually:

```sh
mkdir -p ~/.codex/app-store-connect-release
cp app-store-connect-profile.example.json ~/.codex/app-store-connect-release/profile.json
```

Edit `~/.codex/app-store-connect-release/profile.json` with reusable information such as company name, default support/marketing URLs, standard EULA URL, common App Review notes, and reusable App Review note snippets such as owner-hosted AI or subscription payment disclosures.

`~/.codex/app-store-connect-release/apps.json` is optional. Do not maintain it by hand unless you want to. During a release, the skill can discover app-specific facts from Xcode and App Store Connect, then create or update `apps.json` as a local cache for future releases.

Do not put Apple ID passwords, OTPs, API keys, App Store Connect API private keys, session cookies, or payment credentials in any local release config file.

## What It Covers

- Identifying the exact app, version, build, bundle ID, and App Store app ID before editing.
- Checking for stale free-trial or introductory-offer copy before submission.
- Checking that App Icon assets are full-bleed square images with no pre-rounded corners, internal frame, or border. Xcode and iOS apply the rounded mask.
- Replacing App Store screenshots while validating common required dimensions.
- Filling App Review notes with privacy, legal, subscription, and AI-processing context.
- Splitting work into task lanes with separate read-only reviewers for build, assets, subscription/legal, AI/privacy, store state, and final submission.
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
