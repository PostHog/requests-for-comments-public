# Request for comments: Browser SDK v2 public API

- **Status:** Proposed
- **Decision maker:** @Team Client Libraries
- **Owner:** @dustinbyrne
- **Last updated:** 2026-06-25

## Summary

Browser SDK v2 should be a clean public API boundary for the JavaScript browser SDK. The v2 API removes long-deprecated
aliases, standardizes public JavaScript SDK naming on camelCase, makes feature flags return one consistent result
object, and stops exposing internal implementation objects as public API.

Event wire protocol and persisted identity/session semantics stay stable. This RFC makes the public API smaller, more
consistent with newer SDKs, and easier to document and maintain.

## Goals

- Keep the public SDK API focused on top-level supported methods.
- Remove deprecated v1 aliases instead of carrying them into the next major version.
- Standardize public JavaScript APIs and top-level config keys on camelCase instead of continuing the current mixed
  snake_case / camelCase surface.
- Keep v2 feature flag reads on the root `posthog` object.
- Keep internal objects (`posthog.featureFlags`, underscored `PostHog` internals, extension globals) out of the
  documented public API.
- Keep the `defaults` mechanism for future default-behavior versioning.

## Scope

This RFC covers the browser SDK v2 public API and top-level config migration. Event names, event `$properties`, request
body fields, query/body wire protocol fields, and persisted identity/session semantics stay stable. The observable
behavior/config changes are listed below.

Nested option-object keys stay unchanged in this RFC. Top-level config keys move to camelCase; nested keys remain as
they are for now.

Final npm package naming is tracked separately. This RFC applies to the browser v2 API whether it ships as `posthog-js`,
`@posthog/browser`, or both during a transition.

Observable behavior/config changes covered by this RFC:

- Drop `$bot_pageview` routing.
- Keep `defaults` as the versioning mechanism for future default changes.

## Decisions

### Feature flags

v2 uses one root feature flag read API:

```ts
const result = posthog.getFeatureFlag("new-onboarding");
if (result?.enabled) {
  console.log(result.variant, result.payload);
}
```

- `posthog.getFeatureFlag('name')` returns `FeatureFlagResult | undefined`.
- `posthog.getFeatureFlagPayload()` is removed.
- `posthog.getFeatureFlagResult()` is internal; its semantics are folded into `getFeatureFlag()`.
- `posthog.featureFlags` is internal and excluded from public docs.
- `posthog.featureFlags.override()` is removed from the public API. The existing root
  `posthog.updateFlags(flags, payloads?, { merge })` API is the public way to inject or override flag values.

### Remote feature flags config

- Rename browser `advanced_disable_feature_flags` to `disableRemoteFeatureFlags`.
- Match the recent core / React Native / web-lite behavior: with `disableRemoteFeatureFlags: true`, reads use
  bootstrapped or runtime-supplied flag values instead of remote flag evaluation.
- Keep remote config available when `disableRemoteFeatureFlags` is enabled.
- Drop `advanced_disable_flags` entirely. Remove the broader “kill remote config and flags APIs” switch.
- Drop `advanced_disable_decide` with `advanced_disable_flags`.

### Defaults

- Keep `defaults` as the mechanism for future default-behavior versioning.
- Users can omit `defaults` for the initial v2 baseline.
- Future dated default sets continue to use `defaults: '<date>'`.
- `$config_defaults` continues to reflect the configured `defaults` value.

### Async init and extension loading

- v2 init is async.
- Extension initialization is deferred by default.
- Remove `__preview_deferred_init_extensions` / `__previewDeferredInitExtensions` as a result.

### Bot pageviews

`$bot_pageview` routing was a preview experiment: when bot detection matched and `__preview_capture_bot_pageviews` was
enabled, the SDK sent bot `$pageview` events as a separate `$bot_pageview` event instead of dropping them. Non-pageview
bot events kept their original names.

v2 removes this split event taxonomy. The SDK has one bot policy:

- Drop bot events by default.
- Use `optOutUseragentFilter` to opt out of user-agent filtering and capture bot traffic with the original event names.
- Add `$browser_type: 'bot' | 'browser'` when user-agent filtering is opted out.

This keeps pageview analytics and event naming consistent: `$pageview` remains the pageview event, and bot-vs-browser is
represented as a property when bot traffic is intentionally captured.

Implementation changes:

- Remove `__preview_capture_bot_pageviews`.
- Remove `$bot_pageview` routing.

## Why camelCase

The browser SDK currently has a mixed public naming surface: snake_case methods and config keys sit next to newer
camelCase APIs. v2 is the right boundary to standardize on idiomatic JavaScript naming across the public API.

This standardization makes the SDK easier to teach, document, type, and compare with newer SDK APIs. It also removes
pressure to carry both spellings forward as permanent aliases.

This RFC intentionally limits the rename to root methods and top-level config keys. Nested option-object keys remain
unchanged for now so the migration is large but bounded.

## Public root API changes

### Removed or replaced methods/properties

| v1 API                                                                     | v2 API                                                                                                     | Notes                                                            |
| -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `posthog.people.set(props)` / `posthog.people.set('key', value)`           | `posthog.setPersonProperties(props)` / `posthog.setPersonProperties({ key: value })`                       | Callback argument is removed.                                    |
| `posthog.people.set_once(props)` / `posthog.people.set_once('key', value)` | `posthog.setPersonProperties(undefined, props)` / `posthog.setPersonProperties(undefined, { key: value })` | Preserves `$set_once` behavior through the second argument.      |
| `posthog.decideEndpointWasHit`                                             | `posthog.flagsEndpointWasHit`                                                                              | `/decide` naming is retired.                                     |
| `posthog._calculate_event_properties(...)`                                 | `posthog.calculateEventProperties(...)`                                                                    | Deprecated since 1.241.0.                                        |
| `posthog.getFeatureFlag(key)` returning a raw flag value                   | `posthog.getFeatureFlag(key)` returning `FeatureFlagResult                                                 | undefined`                                                       | Root feature flag API returns value, variant, payload, and metadata together. |
| `posthog.getFeatureFlagPayload(key)`                                       | `posthog.getFeatureFlag(key)?.payload`                                                                     | Payload is part of `FeatureFlagResult`.                          |
| `posthog.getFeatureFlagResult(key)`                                        | `posthog.getFeatureFlag(key)`                                                                              | Result semantics move to the main API.                           |
| `posthog.renderSurvey(surveyId, selector)`                                 | `posthog.displaySurvey(surveyId, { displayType: DisplaySurveyType.Inline, selector })`                     | `displaySurvey` also supports popovers and richer options.       |
| `posthog.canRenderSurvey(surveyId)`                                        | `const result = await posthog.canRenderSurveyAsync(surveyId); result.visible`                              | Async API returns a `SurveyRenderReason`; read `result.visible`. |
| `posthog.webPerformance`                                                   | No replacement                                                                                             | Deprecated compatibility shim removed.                           |
| `posthog.featureFlags`                                                     | Internal                                                                                                   | Exclude from public docs.                                        |

### Root method renames

| v1                              | v2                           |
| ------------------------------- | ---------------------------- |
| `get_distinct_id()`             | `getDistinctId()`            |
| `get_session_id()`              | `getSessionId()`             |
| `get_session_replay_url()`      | `getSessionReplayUrl()`      |
| `get_property()`                | `getProperty()`              |
| `set_config()`                  | `setConfig()`                |
| `register_once()`               | `registerOnce()`             |
| `register_for_session()`        | `registerForSession()`       |
| `unregister_for_session()`      | `unregisterForSession()`     |
| `opt_in_capturing()`            | `optInCapturing()`           |
| `opt_out_capturing()`           | `optOutCapturing()`          |
| `has_opted_in_capturing()`      | `hasOptedInCapturing()`      |
| `has_opted_out_capturing()`     | `hasOptedOutCapturing()`     |
| `get_explicit_consent_status()` | `getExplicitConsentStatus()` |
| `clear_opt_in_out_capturing()`  | `clearOptInOutCapturing()`   |
| `is_capturing()`                | `isCapturing()`              |

Related entrypoint renames:

| v1                    | v2                  |
| --------------------- | ------------------- |
| `init_from_snippet()` | `initFromSnippet()` |
| `init_as_module()`    | `initAsModule()`    |

## Child/internal API cleanup

These APIs are reachable in v1 and become internal in v2.

| v1 API                                                   | v2 API                                                                 | Notes                                                          |
| -------------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------- |
| `posthog.featureFlags.getFeatureFlagPayload(key)`        | No public child replacement                                            | Use `posthog.getFeatureFlag(key)?.payload`.                    |
| `posthog.featureFlags.override(flags, suppressWarning?)` | `posthog.updateFlags(flags, payloads?, { merge })`                     | Use the existing root API to inject or override flag values.   |
| `posthog.toolbar._loadEditor(params)`                    | `posthog.loadToolbar(params)` or `posthog.toolbar.loadToolbar(params)` | Toolbar is no longer called “editor” in the API.               |
| `posthog.toolbar.maybeLoadEditor(...)`                   | `posthog.toolbar.maybeLoadToolbar(...)`                                | Internal-ish but reachable.                                    |
| `posthog.sessionRecording.onRRwebEmit(rawEvent)`         | No public replacement                                                  | Direct callers are relying on internals.                       |
| `posthog.experiments._is_bot()`                          | `posthog.experiments.isBot()`                                          | Internal helper renamed during underscore cleanup.             |
| `URLTriggerMatching.onRemoteConfig(response)`            | `URLTriggerMatching.onConfig(response)`                                | Deep replay trigger-matching export; deprecated alias removed. |
| `LinkedFlagMatching.onRemoteConfig(response, onStarted)` | `LinkedFlagMatching.onConfig(response, onStarted)`                     | Deep replay trigger-matching export; deprecated alias removed. |
| `EventTriggerMatching.onRemoteConfig(response)`          | `EventTriggerMatching.onConfig(response)`                              | Deep replay trigger-matching export; deprecated alias removed. |

## `PostHogPersistence` method cleanup

`PostHogPersistence` is internal but technically reachable from `posthog.persistence` / `posthog.sessionPersistence`.

| v1                          | v2                       |
| --------------------------- | ------------------------ |
| `register_once()`           | `registerOnce()`         |
| `update_campaign_params()`  | `updateCampaignParams()` |
| `update_search_keyword()`   | `updateSearchKeyword()`  |
| `update_referrer_info()`    | `updateReferrerInfo()`   |
| `set_initial_person_info()` | `setInitialPersonInfo()` |
| `get_initial_props()`       | `getInitialProps()`      |
| `update_config()`           | `updateConfig()`         |
| `set_disabled()`            | `setDisabled()`          |
| `set_cross_subdomain()`     | `setCrossSubdomain()`    |
| `set_secure()`              | `setSecure()`            |
| `set_event_timer()`         | `setEventTimer()`        |
| `remove_event_timer()`      | `removeEventTimer()`     |
| `get_property()`            | `getProperty()`          |
| `set_property()`            | `setProperty()`          |

## Top-level config changes

Top-level config keys use camelCase as part of the standardization effort. Nested option-object keys stay unchanged in
this pass.

### Active config key renames

| v1                                             | v2                                       |
| ---------------------------------------------- | ---------------------------------------- |
| `api_host`                                     | `apiHost`                                |
| `flags_api_host`                               | `flagsApiHost`                           |
| `ui_host`                                      | `uiHost`                                 |
| `api_transport`                                | `apiTransport`                           |
| `cross_subdomain_cookie`                       | `crossSubdomainCookie`                   |
| `persistence_name`                             | `persistenceName`                        |
| `cookie_persisted_properties`                  | `cookiePersistedProperties`              |
| `save_referrer`                                | `saveReferrer`                           |
| `save_campaign_params`                         | `saveCampaignParams`                     |
| `custom_campaign_params`                       | `customCampaignParams`                   |
| `custom_blocked_useragents`                    | `customBlockedUseragents`                |
| `capture_pageview`                             | `capturePageview`                        |
| `capture_pageleave`                            | `capturePageleave`                       |
| `cookie_expiration`                            | `cookieExpiration`                       |
| `disable_session_recording`                    | `disableSessionRecording`                |
| `disable_persistence`                          | `disablePersistence`                     |
| `persistence_save_debounce_ms`                 | `persistenceSaveDebounceMs`              |
| `split_storage`                                | `splitStorage`                           |
| `detect_google_search_app`                     | `detectGoogleSearchApp`                  |
| `disable_surveys`                              | `disableSurveys`                         |
| `disable_surveys_automatic_display`            | `disableSurveysAutomaticDisplay`         |
| `disable_product_tours`                        | `disableProductTours`                    |
| `disable_conversations`                        | `disableConversations`                   |
| `identity_distinct_id`                         | `identityDistinctId`                     |
| `identity_hash`                                | `identityHash`                           |
| `disable_web_experiments`                      | `disableWebExperiments`                  |
| `disable_external_dependency_loading`          | `disableExternalDependencyLoading`       |
| `strict_script_versioning`                     | `strictScriptVersioning`                 |
| `asset_host`                                   | `assetHost`                              |
| `prepare_external_dependency_script`           | `prepareExternalDependencyScript`        |
| `prepare_external_dependency_stylesheet`       | `prepareExternalDependencyStylesheet`    |
| `external_scripts_inject_target`               | `externalScriptsInjectTarget`            |
| `enable_recording_console_log`                 | `enableRecordingConsoleLog`              |
| `secure_cookie`                                | `secureCookie`                           |
| `opt_out_capturing_by_default`                 | `optOutCapturingByDefault`               |
| `opt_out_capturing_persistence_type`           | `optOutCapturingPersistenceType`         |
| `opt_out_persistence_by_default`               | `optOutPersistenceByDefault`             |
| `opt_out_useragent_filter`                     | `optOutUseragentFilter`                  |
| `consent_persistence_name`                     | `consentPersistenceName`                 |
| `opt_in_site_apps`                             | `optInSiteApps`                          |
| `respect_dnt`                                  | `respectDnt`                             |
| `property_denylist`                            | `propertyDenylist`                       |
| `request_headers`                              | `requestHeaders`                         |
| `on_request_error`                             | `onRequestError`                         |
| `request_batching`                             | `requestBatching`                        |
| `properties_string_max_length`                 | `propertiesStringMaxLength`              |
| `session_recording`                            | `sessionRecording`                       |
| `error_tracking`                               | `errorTracking`                          |
| `session_idle_timeout_seconds`                 | `sessionIdleTimeoutSeconds`              |
| `mask_all_element_attributes`                  | `maskAllElementAttributes`               |
| `mask_all_text`                                | `maskAllText`                            |
| `mask_personal_data_properties`                | `maskPersonalDataProperties`             |
| `custom_personal_data_properties`              | `customPersonalDataProperties`           |
| `advanced_disable_feature_flags`               | `disableRemoteFeatureFlags`              |
| `advanced_disable_feature_flags_on_first_load` | `advancedDisableFeatureFlagsOnFirstLoad` |
| `evaluation_contexts`                          | `evaluationContexts`                     |
| `flag_keys`                                    | `flagKeys`                               |
| `advanced_disable_toolbar_metrics`             | `advancedDisableToolbarMetrics`          |
| `advanced_only_evaluate_survey_feature_flags`  | `advancedOnlyEvaluateSurveyFeatureFlags` |
| `advanced_enable_surveys`                      | `advancedEnableSurveys`                  |
| `feature_flag_request_timeout_ms`              | `featureFlagRequestTimeoutMs`            |
| `feature_flag_cache_ttl_ms`                    | `featureFlagCacheTtlMs`                  |
| `advanced_feature_flags_dedup_per_session`     | `advancedFeatureFlagsDedupPerSession`    |
| `surveys_request_timeout_ms`                   | `surveysRequestTimeoutMs`                |
| `remote_config_refresh_interval_ms`            | `remoteConfigRefreshIntervalMs`          |
| `get_device_id`                                | `getDeviceId`                            |
| `before_send`                                  | `beforeSend`                             |
| `capture_performance`                          | `capturePerformance`                     |
| `disable_compression`                          | `disableCompression`                     |
| `capture_heatmaps`                             | `captureHeatmaps`                        |
| `capture_dead_clicks`                          | `captureDeadClicks`                      |
| `capture_exceptions`                           | `captureExceptions`                      |
| `disable_scroll_properties`                    | `disableScrollProperties`                |
| `scroll_root_selector`                         | `scrollRootSelector`                     |
| `person_profiles`                              | `personProfiles`                         |
| `rate_limiting`                                | `rateLimiting`                           |
| `fetch_options`                                | `fetchOptions`                           |
| `request_queue_config`                         | `requestQueueConfig`                     |
| `cookieless_mode`                              | `cookielessMode`                         |
| `internal_or_test_user_hostname`               | `internalOrTestUserHostname`             |
| `override_display_language`                    | `overrideDisplayLanguage`                |
| `tracing_headers`                              | `tracingHeaders`                         |
| `disable_beacon`                               | `disableBeacon`                          |

Top-level keys that intentionally remain unchanged: `token`, `name`, `autocapture`, `rageclick`, `persistence`,
`loaded`, `debug`, `upgrade`, `surveys`, `logs`, `bootstrap`, `segment`, `integrations`, `defaults`.

Internal `__extensionClasses` remains internal.

Wire fields that look like config remain snake_case in payloads: flags request body `evaluation_contexts` / `flag_keys`,
conversations payload/query `identity_distinct_id` / `identity_hash`.

### Removed config aliases and dead keys

| Removed v1 key                                  | v2 replacement / behavior                   | Notes                                                                                                                 |
| ----------------------------------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `sanitize_properties`                           | `beforeSend`                                | Old hook also ran over `$set_once`; migrations must verify equivalent behavior for that use case.                     |
| `ip`                                            | None                                        | v1 warned this config key had no effect. Use project-level “Discard IP data” or transformations for IP handling.      |
| `on_xhr_error`                                  | `onRequestError`                            | Signature changes from XHR-focused callback to request response callback.                                             |
| `xhr_headers`                                   | `requestHeaders`                            | Alias removed.                                                                                                        |
| `process_person`                                | `personProfiles`                            | Alias removed.                                                                                                        |
| `advanced_disable_decide`                       | None                                        | Deprecated `/decide` kill switch removed.                                                                             |
| `advanced_disable_flags`                        | None                                        | Broad remote-config/flags kill switch removed.                                                                        |
| `cookie_name`                                   | `persistenceName`                           | Alias removed.                                                                                                        |
| `disable_cookie`                                | `disablePersistence`                        | Alias removed.                                                                                                        |
| `store_google`                                  | `saveCampaignParams`                        | Alias removed.                                                                                                        |
| `verbose`                                       | `debug`                                     | Alias removed.                                                                                                        |
| `property_blacklist`                            | `propertyDenylist`                          | v1 merged blacklist into denylist; v2 reads `propertyDenylist`.                                                       |
| `__preview_disable_beacon`                      | `disableBeacon`                             | Preview alias removed.                                                                                                |
| `__preview_external_dependency_versioned_paths` | `strictScriptVersioning` and/or `assetHost` | v1 accepted boolean/string and mapped to supported options.                                                           |
| `__preview_deferred_init_extensions`            | None                                        | Async v2 init defers extensions by default.                                                                           |
| `evaluation_environments`                       | `evaluationContexts`                        | Config-side feature flag evaluation context.                                                                          |
| `enable_heatmaps`                               | `captureHeatmaps`                           | Superseded.                                                                                                           |
| `opt_out_capturing_cookie_prefix`               | `consentPersistenceName`                    | `consentPersistenceName` is the full storage key; the token is no longer appended as a prefix.                        |
| `_onCapture`                                    | Split by use case                           | Use `posthog.on('eventCaptured', ...)` for observe-only behavior, or `beforeSend` for mutation/filtering before send. |
| `__add_tracing_headers`                         | `tracingHeaders`                            | Preview/internal alias removed.                                                                                       |
| `addTracingHeaders`                             | `tracingHeaders`                            | Alias removed.                                                                                                        |
| `api_method`                                    | None                                        | No reader.                                                                                                            |
| `inapp_protocol`                                | None                                        | No reader.                                                                                                            |
| `inapp_link_new_window`                         | None                                        | No reader.                                                                                                            |
| `__preview_flags_v2`                            | None                                        | Concluded experiment.                                                                                                 |
| `__preview_eager_load_replay`                   | None                                        | Concluded experiment.                                                                                                 |
| `__preview_lazy_load_replay`                    | None                                        | Concluded experiment.                                                                                                 |
| `__preview_disable_xhr_credentials`             | None                                        | Concluded experiment.                                                                                                 |
| `__preview_cookie_wins_on_conflict`             | None                                        | Cookie-wins merge behavior is the only behavior.                                                                      |
| `__preview_capture_bot_pageviews`               | None                                        | `$bot_pageview` routing removed.                                                                                      |

## Defaults behavior

v2 keeps the `defaults` config key so future default changes can be introduced behind dated default sets. Initial v2
uses the package baseline defaults.

The v2 package baseline includes the latest default behavior that had already been staged in v1:

| Config key                    | v2 baseline default                                                                             | v1 undated default |
| ----------------------------- | ----------------------------------------------------------------------------------------------- | ------------------ | ----------- |
| `capturePageview`             | `'history_change'`                                                                              | `true`             |
| `rageclick`                   | `{ content_ignorelist: DEFAULT_CONTENT_IGNORELIST_WITH_STEPPERS, ignore_text_selection: true }` | `true`             |
| `sessionRecording`            | `{ strictMinimumDuration: true }`                                                               | `{}`               |
| `externalScriptsInjectTarget` | `'head'`                                                                                        | `'body'`           |
| `internalOrTestUserHostname`  | `/^(localhost                                                                                   | 127\.0\.0\.1)$/`   | `undefined` |
| `persistenceSaveDebounceMs`   | `250`                                                                                           | `0`                |
| `splitStorage`                | `true`                                                                                          | `false`            |
| `detectGoogleSearchApp`       | `true`                                                                                          | `false`            |

Users should use individual config overrides to preserve older behavior. Future default changes should use
`defaults: '<date>'` again.

## Window globals and extension contract removals

For lazy-loaded extension bundles, `window.__PosthogExtensions__` is the supported contract. The toolbar loader still
uses direct `window.ph_load_toolbar`; legacy `window.ph_load_editor` is removed.

| Removed global                         | v2 contract                                                        |
| -------------------------------------- | ------------------------------------------------------------------ |
| `window.rrweb`                         | `window.__PosthogExtensions__.rrweb`                               |
| `window.rrwebConsoleRecord`            | `window.__PosthogExtensions__.rrwebPlugins.getRecordConsolePlugin` |
| `window.getRecordNetworkPlugin`        | `window.__PosthogExtensions__.rrwebPlugins.getRecordNetworkPlugin` |
| `window.posthogErrorWrappingFunctions` | `window.__PosthogExtensions__.errorWrappingFunctions`              |
| `window.postHogWebVitalsCallbacks`     | `window.__PosthogExtensions__.postHogWebVitalsCallbacks`           |
| `window.postHogTracingHeadersPatchFns` | `window.__PosthogExtensions__.tracingHeadersPatchFns`              |
| `window.extendPostHogWithSurveys`      | `window.__PosthogExtensions__.generateSurveys`                     |
| `window.ph_load_editor`                | `window.ph_load_toolbar`                                           |

Also remove `window.__PosthogExtensions__.loadWebVitalsCallbacks()`.

## Survey and remote-config type cleanup

| Type/member                             | v2 status | Replacement / notes                                          |
| --------------------------------------- | --------- | ------------------------------------------------------------ |
| `SurveyAppearance.descriptionTextColor` | Removed   | Deprecated and unused.                                       |
| `SurveyAppearance.inputBackgroundColor` | Removed   | Use `inputBackground`.                                       |
| `ActionStepType.tag_name`               | Removed   | Use `selector`.                                              |
| `SurveyConfig.autoSubmitIfComplete`     | Removed   | No longer used; prefilled skip-submit behavior is automatic. |
| `SurveyConfig.autoSubmitDelay`          | Removed   | No longer used.                                              |
| `RemoteConfig.editorParams`             | Removed   | Use `toolbarParams`.                                         |
| `RemoteConfig.toolbarVersion`           | Removed   | Obsolete toolbar metadata.                                   |

## Public-ish underscored `PostHog` internals

These were reachable on the class and consumed by some internal modules, external bundles, wrapper SDKs, or snippets. v2
gives the cross-module ones stable non-underscored names and marks them `@internal`. They are internal APIs, but they
are breaking changes for deep integrations. They may be exposed by some other internal means.

| v1                          | v2                                                          |
| --------------------------- | ----------------------------------------------------------- |
| `__loaded`                  | `isLoaded` (`@internal`)                                    |
| `_init()`                   | `internalInit()` (`@internal`)                              |
| `_onRemoteConfig()`         | `onRemoteConfig()` (`@internal`)                            |
| `_dom_loaded()`             | `domLoaded()` (`@internal`)                                 |
| `_send_request()`           | `sendRequest()` (`@internal`)                               |
| `_send_retriable_request()` | `sendRetriableRequest()` (`@internal`)                      |
| `_execute_array()`          | `executeArray()` (`@internal`)                              |
| `_addCaptureHook()`         | `addCaptureHook()` (`@internal`)                            |
| `_internalEventEmitter`     | `internalEventEmitter` (`@internal`)                        |
| `_overrideSDKInfo()`        | `overrideSDKInfo()` (`@internal`; wrapper SDKs must update) |
| `_is_bot()`                 | `isBot()` (`@internal`)                                     |
| `_shouldDisableFlags()`     | Removed with `advanced_disable_flags`                       |
| `_isIdentified()`           | `isIdentified()` (promoted to public)                       |

The remaining class-only internals are explicitly private and can be mangled in production builds. Non-exhaustive
examples include `_requestQueue`, `_retryQueue`, `_loaded`, `_startQueueIfOptedIn`, `_handleUnload`,
`_calculateSetOnceProperties`, `_registerSingle`, `_hasPersonProcessing`, `_shouldCapturePageleave`,
`_requirePersonProcessing`, `_captureInitialPageview`, `_initRequestQueue` (from `__request_queue`), `_triggeredNotifs`
(from `_triggered_notifs`), and removed `_originalUserConfig`.

## Snippet and queued-call migration

The snippet queue dispatches by method name. Queued calls must use v2 method names.

Examples:

```js
posthog.setConfig({ disableSessionRecording: true });
posthog.registerOnce({ plan: "pro" });
posthog.getDistinctId();
```

Old queued calls such as `posthog.set_config(...)`, `posthog.register_once(...)`, or `posthog.get_distinct_id()` fail in
v2 because the queue processor dispatches by exact method name.

## Runtime behavior for removed aliases

- Removed methods and properties are absent at runtime.
- Removed config keys in plain JavaScript can still exist on user objects; the SDK ignores them.
- Removed config keys are ignored directly rather than translated silently.
- Any one-release runtime alias must be an explicit compatibility shim with warnings and a removal date, rather than
  accidental public API.

## Feedback requested

@Team Client Libraries is the decision maker for this RFC. Feedback should focus on:

- Known customer or wrapper SDK usage of removed aliases.
- Any runtime aliases that are worth keeping for one compatibility release.
- Whether nested config keys should be renamed in v2 or deferred.
- Any debug/test flag override use cases beyond `posthog.updateFlags(flags, payloads?, { merge })`.
- Any migration risks around changed default behavior, bot pageviews, `sanitize_properties` / `$set_once`, or split
  storage.
