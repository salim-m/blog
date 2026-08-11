+++
title = 'Feature flags with OpenFeature and Node.js'
date = 2026-08-11T19:29:43.022Z
draft = false
+++

### What are feature flags?

Feature flags (a.k.a. feature toggles) are a technique that lets you turn features on or off at runtime without deploying new code. They separate code deployment from feature release.

#### Use cases

- **A/B Testing:** Targeting different user groups with different variants of a feature or UI to gather data and measure which version performs better. (Experiment Toggles)
- **Enabling trunk-based development:** Allows teams to ship in-progress features into production while keeping them off. (Release Toggles)
- **Kill Switches:** Disable a broken piece of code in production wrapped by a feature flag without deploying new code. (Ops Toggles)
- **Access Control:** Dynamically grant users access to a certain feature based on their tier or subscription plan. (Permission Toggles)

### What is OpenFeature?

OpenFeature is an open specification that provides a vendor-agnostic, community-driven API for feature flagging. In other words, it allows you to plug into different feature flagging providers that implement this specification. You are not locked into a single vendor-specific API.

Here's how to get started with OpenFeature.

> **Note:** This example uses [ConfigCat](https://configcat.com/) as the provider, but you can swap in any OpenFeature-compliant provider without changing your application code. To follow along with ConfigCat specifically, sign up for a free account, create a boolean flag named `isSignUpEnabled` set to `true`, and export your SDK key as `CONFIGCAT_SDK_KEY`.

#### Installation

```bash
npm install --save @openfeature/server-sdk @openfeature/config-cat-provider @configcat/sdk
```

#### Usage

```javascript
import { OpenFeature } from "@openfeature/server-sdk";
import { ConfigCatProvider } from "@openfeature/config-cat-provider";
import { PollingMode } from "@configcat/sdk";

const provider = ConfigCatProvider.create(
  process.env.CONFIGCAT_SDK_KEY,
  PollingMode.AutoPoll,
);

await OpenFeature.setProviderAndWait(provider);

const client = OpenFeature.getClient();
const isSignUpEnabled = await client.getBooleanValue("isSignUpEnabled", false);
console.log("isSignUpEnabled", isSignUpEnabled); // logs: true, since we set the flag's value to true in the dashboard
```

A quick walkthrough of what's happening:

- **`ConfigCatProvider.create`** connects OpenFeature to ConfigCat using your SDK key, and `PollingMode.AutoPoll` tells the provider to automatically refresh flag values in the background at a set interval, rather than fetching on every request.
- **`OpenFeature.setProviderAndWait`** registers the provider globally and waits until it's fully initialized before continuing, so your first flag evaluation doesn't run against stale or missing data.
- **`getBooleanValue("isSignUpEnabled", false)`** evaluates the `isSignUpEnabled` flag and returns its current boolean value. The second argument, `false`, is the default value returned if the flag can't be evaluated (for example, if the provider fails to connect).

From here, you can toggle `isSignUpEnabled` from your ConfigCat dashboard at any time without redeploying your app. To go further, check out the [OpenFeature docs](https://openfeature.dev/docs/) for other providers, or explore boolean, string, number, and object flag types beyond the basic example above.
