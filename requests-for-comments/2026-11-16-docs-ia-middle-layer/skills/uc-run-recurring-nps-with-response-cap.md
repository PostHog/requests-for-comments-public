---
name: surveys-uc-run-recurring-nps-with-response-cap
code: UC-8
goal: "Run a recurring NPS survey across web and mobile with automatic score breakdowns and an automatic response cap."
surfaces: [UI, MCP, API, WEB, AND, FLT, IOS, RN]
prerequisites: [SV-0.1, SV-0.3, SV-0.2, SV-3.1, SV-5.1]
procedures: [SV-UI-PR-01, SV-MCP-PR-05, SV-API-PR-01, SV-API-PR-07, SV-MCP-PR-06, SV-UI-PR-02, SV-API-PR-02, SV-WEB-PR-04, SV-AND-PR-02, SV-FLT-PR-02, SV-IOS-PR-03, SV-RN-PR-02, SV-UI-PR-03, SV-API-PR-10, SV-API-PR-11, SV-API-PR-14, SV-UI-PR-13, SV-UI-PR-18, SV-MCP-PR-04, SV-API-PR-12, SV-API-PR-03, SV-MCP-PR-07]
facts_resolved: 849
---

# UC-8: Run a recurring NPS survey across web and mobile with automatic score breakdowns and an automatic response cap.

## Prerequisites

Complete these before starting. Each code links to its full steps and facts under [Resolved prerequisites](#resolved-prerequisites):

- [`SV-0.1`](#prereq-sv-0-1) Enable surveys in project settings and set defaults (`SV-UI-P-01`, `SV-MCP-P-01`)
- [`SV-0.3`](#prereq-sv-0-3) Authenticate with the Surveys API and MCP server (`SV-API-P-01`, `SV-MCP-P-02`)
- [`SV-0.2`](#prereq-sv-0-2) Install and initialize the SDK for surveys (`SV-WEB-P-01`, `SV-RN-P-01`, `SV-IOS-P-01`, `SV-AND-P-01`, `SV-FLT-P-01`)
- [`SV-3.1`](#prereq-sv-3-1) Target surveys by URL and estimate reachable audience
- [`SV-5.1`](#prereq-sv-5-1) Fetch and cache active surveys on the client

## Procedures

Follow these procedures in order for your surface:

1. **[`SV-UI-PR-01`](#sv-ui-pr-01)** (`UI`): Create and launch or save a draft survey through the guided Survey Wizard with template selection and branding entitlement gating.
2. **[`SV-MCP-PR-05`](#sv-mcp-pr-05)** (`MCP`): Detect Surveys setup status and create a first survey via the Survey Wizard or the PostHog AI create_survey tool.
3. **[`SV-API-PR-01`](#sv-api-pr-01)** (`API`): Validate and create a new survey with questions, appearance, conditions, translations, and targeting flags via the authenticated Surveys REST API.
4. **[`SV-API-PR-07`](#sv-api-pr-07)** (`API`): Persist a Survey record with normalized recurring iterations, response sampling limits, and question UUIDs, then trigger feature flag cache synchronization on commit.
5. **[`SV-MCP-PR-06`](#sv-mcp-pr-06)** (`MCP`): Create, list, get, update, launch, stop, and soft-archive surveys via PostHog MCP tools (survey-create, survey-update, survey-delete, survey-get, surveys-get-all).
6. **[`SV-UI-PR-02`](#sv-ui-pr-02)** (`UI`): Edit an existing survey in the full or hosted editor, modify questions, preview branching flow, manage translations, and save changes.
7. **[`SV-API-PR-02`](#sv-api-pr-02)** (`API`): Update an existing survey's configuration, schedule, actions, lifecycle dates, and targeting flags.
8. **[`SV-WEB-PR-04`](#sv-web-pr-04)** (`WEB`): Validate and record end-user survey question answers, persist in-progress state, and capture the 'survey sent' event.
9. **[`SV-AND-PR-02`](#sv-and-pr-02)** (`AND`): Validate user input across survey question types, evaluate response-based branching, persist progress, and capture the survey sent event.
10. **[`SV-FLT-PR-02`](#sv-flt-pr-02)** (`FLT`): Render interactive open-text, choice, rating, or link survey questions, validate user input, and submit responses to advance or complete the survey.
11. **[`SV-IOS-PR-03`](#sv-ios-pr-03)** (`IOS`): Capture a user's question response, advance or complete the survey using branching rules, update persisted progress, and emit 'survey sent'.
12. **[`SV-RN-PR-02`](#sv-rn-pr-02)** (`RN`): Capture user answers, evaluate question branching, dismiss surveys, and emit survey shown/sent/dismissed events in React Native.
13. **[`SV-UI-PR-03`](#sv-ui-pr-03)** (`UI`): Configure survey display triggers, completion limits, widget styling, and linked feature flag targeting in SurveyEdit.
14. **[`SV-API-PR-10`](#sv-api-pr-10)** (`API`): Count unique survey responses per active response-limited survey via HogQL to determine which surveys have reached responses_limit.
15. **[`SV-API-PR-11`](#sv-api-pr-11)** (`API`): Stop a survey that has reached its responses_limit and record the system action in the activity log.
16. **[`SV-API-PR-14`](#sv-api-pr-14)** (`API`): Advance active recurring surveys to their next iteration or close them after their final iteration while syncing their internal targeting feature flag.
17. **[`SV-UI-PR-13`](#sv-ui-pr-13)** (`UI`): Filter survey results by date, properties, answer values, or archived state, inspect metrics and individual response rows, and build the underlying HogQL response query.
18. **[`SV-UI-PR-18`](#sv-ui-pr-18)** (`UI`): Preview simulated pre-launch survey results and interact with multiple-choice and rating breakdown charts.
19. **[`SV-MCP-PR-04`](#sv-mcp-pr-04)** (`MCP`): Inspect survey response results, completion outcomes, and customize response table and export columns.
20. **[`SV-API-PR-12`](#sv-api-pr-12)** (`API`): Validate date bounds and compute shown, dismissed, sent, and dismissed-and-sent statistics for a single survey or all non-archived team surveys.
21. **[`SV-API-PR-03`](#sv-api-pr-03)** (`API`): List, filter, and read surveys, stats, responses, site snapshots, question labels, and activity logs under authenticated DRF access control.
22. **[`SV-MCP-PR-07`](#sv-mcp-pr-07)** (`MCP`): Query global or per-survey statistics and list individual responses via PostHog MCP tools (surveys-global-stats, survey-stats, surveys-responses-list requiring survey:read + query:read).

---

### <a id="sv-ui-pr-01"></a>`SV-UI-PR-01`: Create and launch or save a draft survey through the guided Survey Wizard with template selection and branding entitlement gating.

- **Surface:** PostHog web app (`UI`)
- **Capability:** `SV-1.1` Create a survey with templates, the wizard, or PostHog AI
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3)
- **Entry:** `NewSurveyButton on the Surveys scene or SurveyWizardComponent route`

1. Initiate survey creation from NewSurveyButton (selecting Button or LemonButton via TODAY_RAIL_NAV and recording SURVEY_ADD_NEW product intent) and mount SurveyWizardComponent bound to surveyLogic.
   - **Facts:** [`SV-UI-F-0878`](#sv-ui-f-0878), [`SV-UI-F-0879`](#sv-ui-f-0879), [`SV-UI-F-0880`](#sv-ui-f-0880), [`SV-UI-F-2591`](#sv-ui-f-2591), [`SV-UI-F-2594`](#sv-ui-f-2594)

2. Select a template in TemplateStep or generate a survey via the Max AI create_survey tool (prefixing the prompt with '!' and invoking handleMaxSurveyCreated); hosted templateMode redirects to the full editor, and users can manually switch to the full editor (persisting preferredEditor as 'full').
   - **Facts:** [`SV-UI-F-2606`](#sv-ui-f-2606), [`SV-UI-F-2762`](#sv-ui-f-2762), [`SV-UI-F-2884`](#sv-ui-f-2884)

3. Configure branding in AppearanceStep, where checking 'Hide PostHog branding' is gated behind AvailableFeature.WHITE_LABELLING via upgradeModalLogic.guardAvailableFeature while unchecking is permitted directly.
   - **Facts:** [`SV-UI-F-2642`](#sv-ui-f-2642), [`SV-UI-F-2662`](#sv-ui-f-2662)

4. Persist the new survey via surveyWizardLogic.launchSurvey (setting start_date to the current ISO timestamp) or surveyWizardLogic.saveDraft (omitting start_date), sending POST /api/projects/*/surveys through the authenticated Surveys API.
   - **Facts:** [`SV-UI-F-2886`](#sv-ui-f-2886), [`SV-UI-F-2889`](#sv-ui-f-2889)

---

### <a id="sv-mcp-pr-05"></a>`SV-MCP-PR-05`: Detect Surveys setup status and create a first survey via the Survey Wizard or the PostHog AI create_survey tool.

- **Surface:** PostHog MCP (`MCP`)
- **Capability:** `SV-1.1` Create a survey with templates, the wizard, or PostHog AI
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3)
- **Entry:** `surveysEmptyState and NewSurveyActions`

1. Evaluate non-skippable surveysEmptyState via surveysSetupLogic, which reads currentProjectId from projectLogic and queries at most 1 non-archived and 1 archived survey via surveysList to determine whether any live or archived survey exists.
   - **Facts:** [`SV-UI-F-3126`](#sv-ui-f-3126), [`SV-UI-F-3130`](#sv-ui-f-3130), [`SV-UI-F-3131`](#sv-ui-f-3131), [`SV-UI-F-3133`](#sv-ui-f-3133), [`SV-UI-F-3134`](#sv-ui-f-3134), [`SV-UI-F-3135`](#sv-ui-f-3135)

2. Gate creation actions in NewSurveyActions behind AccessControlAction requiring AccessControlLevel.Editor on AccessControlResourceType.Survey; clicking 'Create your first survey' dispatches teamLogic.addProductIntent (ProductKey.SURVEYS, ProductIntentContext.SURVEY_ADD_NEW) and navigates to urls.surveyWizard().
   - **Facts:** [`SV-UI-F-3116`](#sv-ui-f-3116), [`SV-UI-F-3117`](#sv-ui-f-3117), [`SV-UI-F-3118`](#sv-ui-f-3118)

3. When isMaxAvailable is true, register the 'create_survey' Max AI tool (disabling 'Create with AI' when openMax is unavailable); clicking 'Create with AI' opens Max and invokes surveysLogic.handleMaxSurveyCreated with SURVEY_CREATED_SOURCE.SURVEY_EMPTY_STATE when the tool callback fires.
   - **Facts:** [`SV-UI-F-3113`](#sv-ui-f-3113), [`SV-UI-F-3114`](#sv-ui-f-3114), [`SV-UI-F-3115`](#sv-ui-f-3115), [`SV-UI-F-3120`](#sv-ui-f-3120), [`SV-UI-F-3121`](#sv-ui-f-3121)

4. Emit survey lifecycle and theme analytics events via reportSurveyCreated (defaulting is_duplicate to false and creation_source to 'full_editor'), reportSurveyEdited (defaulting recurring iteration count and interval to 0), and site/theme reporting helpers.
   - **Facts:** [`SV-UI-F-3138`](#sv-ui-f-3138), [`SV-UI-F-3139`](#sv-ui-f-3139), [`SV-UI-F-3142`](#sv-ui-f-3142), [`SV-UI-F-3143`](#sv-ui-f-3143), [`SV-UI-F-3145`](#sv-ui-f-3145), [`SV-UI-F-3146`](#sv-ui-f-3146), [`SV-UI-F-3147`](#sv-ui-f-3147), [`SV-UI-F-3148`](#sv-ui-f-3148), [`SV-UI-F-3149`](#sv-ui-f-3149), [`SV-UI-F-3150`](#sv-ui-f-3150)

---

### <a id="sv-api-pr-01"></a>`SV-API-PR-01`: Validate and create a new survey with questions, appearance, conditions, translations, and targeting flags via the authenticated Surveys REST API.

- **Surface:** REST API (`API`)
- **Capability:** `SV-1.1` Create a survey with templates, the wizard, or PostHog AI
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3)
- **Entry:** `POST /api/projects/{project_id}/surveys/ (SurveyViewSet -> SurveySerializerCreateUpdateOnly)`

1. Validate top-level survey fields: enforce name min_length of 1 and project-scoped uniqueness (providing a free-name hint on collision), base_language max_length of 20 with valid non-reserved BCP47 format, schedule in ('once', 'recurring', 'always'), iteration_count within 0..MAX_ITERATION_COUNT, and responses_limit.
   - **Facts:** [`SV-API-F-0120`](#sv-api-f-0120), [`SV-API-F-0124`](#sv-api-f-0124), [`SV-API-F-0125`](#sv-api-f-0125), [`SV-API-F-0129`](#sv-api-f-0129), [`SV-API-F-0139`](#sv-api-f-0139), [`SV-API-F-0168`](#sv-api-f-0168), [`SV-API-F-0169`](#sv-api-f-0169), [`SV-API-F-0170`](#sv-api-f-0170), [`SV-API-F-0235`](#sv-api-f-0235), [`SV-API-F-0289`](#sv-api-f-0289)

2. Validate questions, appearance, conditions, form_content, and translations: enforce question text, non-empty string choices, allowed link URL schemes (including valid mailto and https netloc), validation rules (min_length/max_length/email with min_length <= max_length), matching translation choice counts and non-colliding BCP47 keys, no shuffleQuestions combined with branching, AvailableFeature.WHITE_LABELLING entitlement when whiteLabel is enabled, valid seenSurveyWaitPeriodInDays, viewer access to newly added actions, and EXTERNAL_SURVEY field/condition exclusions.
   - **Facts:** [`SV-API-F-0128`](#sv-api-f-0128), [`SV-API-F-0132`](#sv-api-f-0132), [`SV-API-F-0133`](#sv-api-f-0133), [`SV-API-F-0134`](#sv-api-f-0134), [`SV-API-F-0136`](#sv-api-f-0136), [`SV-API-F-0137`](#sv-api-f-0137), [`SV-API-F-0141`](#sv-api-f-0141), [`SV-API-F-0143`](#sv-api-f-0143), [`SV-API-F-0144`](#sv-api-f-0144), [`SV-API-F-0145`](#sv-api-f-0145), [`SV-API-F-0146`](#sv-api-f-0146), [`SV-API-F-0147`](#sv-api-f-0147), [`SV-API-F-0149`](#sv-api-f-0149), [`SV-API-F-0152`](#sv-api-f-0152), [`SV-API-F-0154`](#sv-api-f-0154), [`SV-API-F-0155`](#sv-api-f-0155), [`SV-API-F-0159`](#sv-api-f-0159), [`SV-API-F-0163`](#sv-api-f-0163), [`SV-API-F-0164`](#sv-api-f-0164), [`SV-API-F-0165`](#sv-api-f-0165), [`SV-API-F-0166`](#sv-api-f-0166), [`SV-API-F-0172`](#sv-api-f-0172), [`SV-API-F-0192`](#sv-api-f-0192), [`SV-API-F-0195`](#sv-api-f-0195), [`SV-API-F-0207`](#sv-api-f-0207), [`SV-API-F-0212`](#sv-api-f-0212), [`SV-API-F-0214`](#sv-api-f-0214), [`SV-API-F-0242`](#sv-api-f-0242), [`SV-API-F-0243`](#sv-api-f-0243), [`SV-API-F-0244`](#sv-api-f-0244), [`SV-API-F-0245`](#sv-api-f-0245)

3. Validate linked resources and targeting/sampling settings: verify team-scoped linked_flag_id, targeting_flag_id (asserting FLAG_OWNER_SURVEY availability), linked_insight_id, and conditions.linkedFlagVariant against linked flag variants; reject targeting_flag_filters groups with 100%/None rollout and no properties; enforce response_sampling_start_date >= today 00:00 UTC and response_sampling_interval > 0.
   - **Facts:** [`SV-API-F-0157`](#sv-api-f-0157), [`SV-API-F-0223`](#sv-api-f-0223), [`SV-API-F-0224`](#sv-api-f-0224), [`SV-API-F-0225`](#sv-api-f-0225), [`SV-API-F-0226`](#sv-api-f-0226), [`SV-API-F-0227`](#sv-api-f-0227), [`SV-API-F-0228`](#sv-api-f-0228), [`SV-API-F-0229`](#sv-api-f-0229), [`SV-API-F-0230`](#sv-api-f-0230), [`SV-API-F-0231`](#sv-api-f-0231), [`SV-API-F-0232`](#sv-api-f-0232), [`SV-API-F-0233`](#sv-api-f-0233), [`SV-API-F-0237`](#sv-api-f-0237), [`SV-API-F-0238`](#sv-api-f-0238), [`SV-API-F-0239`](#sv-api-f-0239), [`SV-API-F-0240`](#sv-api-f-0240)

4. Enforce feature flag write scope when targeting_flag_id or targeting_flag_filters is supplied, load the Team, persist the new Survey with created_by inside a database transaction, associate matching Action records, create any internal_response_sampling_flag, and log a 'created' Survey activity entry.
   - **Facts:** [`SV-API-F-0126`](#sv-api-f-0126), [`SV-API-F-0249`](#sv-api-f-0249), [`SV-API-F-0251`](#sv-api-f-0251), [`SV-API-F-0252`](#sv-api-f-0252), [`SV-API-F-0255`](#sv-api-f-0255), [`SV-API-F-0257`](#sv-api-f-0257), [`SV-API-F-0275`](#sv-api-f-0275), [`SV-API-F-0279`](#sv-api-f-0279)

5. Atomically create or update the survey's FLAG_OWNER_SURVEY targeting and internal targeting FeatureFlags (translating ApprovalRequired and behavioural-cohort errors into ValidationError) through the Feature Flags facade.
   - **Facts:** [`SV-API-F-0281`](#sv-api-f-0281), [`SV-API-F-0284`](#sv-api-f-0284), [`SV-API-F-0285`](#sv-api-f-0285), [`SV-API-F-0474`](#sv-api-f-0474)

---

### <a id="sv-api-pr-07"></a>`SV-API-PR-07`: Persist a Survey record with normalized recurring iterations, response sampling limits, and question UUIDs, then trigger feature flag cache synchronization on commit.

- **Surface:** REST API (`API`)
- **Capability:** `SV-1.1` Create a survey with templates, the wizard, or PostHog AI
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3)
- **Entry:** `Survey.save()`

1. Validate and persist the Survey model (inheriting FileSystemSyncMixin, RootTeamMixin, and UUIDTModel) to posthog_survey, enforcing per-team unique name (max 400 chars), type (max 40 chars: popover, widget, external_survey, api), schedule (max 40 chars: once, recurring, always, default once), base_language (max 20 chars, default "en"), response_sampling_interval_type (max 6 chars: day, week, month, default week), optional positive responses_limit, and boolean defaults (archived=False, enable_partial_responses=False, enable_iframe_embedding=False).
   - **Facts:** [`SV-API-F-0559`](#sv-api-f-0559), [`SV-API-F-0560`](#sv-api-f-0560), [`SV-API-F-0561`](#sv-api-f-0561), [`SV-API-F-0562`](#sv-api-f-0562), [`SV-API-F-0563`](#sv-api-f-0563), [`SV-API-F-0564`](#sv-api-f-0564), [`SV-API-F-0565`](#sv-api-f-0565), [`SV-API-F-0566`](#sv-api-f-0566), [`SV-API-F-0567`](#sv-api-f-0567), [`SV-API-F-0568`](#sv-api-f-0568), [`SV-API-F-0569`](#sv-api-f-0569), [`SV-API-F-0570`](#sv-api-f-0570), [`SV-API-F-0572`](#sv-api-f-0572), [`SV-API-F-0573`](#sv-api-f-0573), [`SV-API-F-0574`](#sv-api-f-0574), [`SV-API-F-0575`](#sv-api-f-0575), [`SV-API-F-0576`](#sv-api-f-0576)

2. Trigger the pre_save_survey Django pre_save signal to run update_survey_iterations, update_response_sampling_limits, and ensure_question_ids before writing the row.
   - **Facts:** [`SV-API-F-0591`](#sv-api-f-0591)

3. Compute recurring iteration schedules in update_survey_iterations: reset iteration_start_dates to [] and clear current iteration fields when repeats_on_schedule is False (iteration_frequency_days unset or zero); set iteration_start_dates to None when start_date is None; for repeat_until_stopped recurring surveys, ignore iteration_count, compute the 1-based iteration index via open_ended_iteration (clamped >= 1), and generate a 2-date rrule window; for fixed-count recurring surveys, cap generated dates at MAX_ITERATION_COUNT (500) with frequency up to MAX_ITERATION_FREQUENCY_DAYS (365) and initialize current_iteration to 1 and current_iteration_start_date to start_date when unset or 0.
   - **Facts:** [`SV-API-F-0557`](#sv-api-f-0557), [`SV-API-F-0558`](#sv-api-f-0558), [`SV-API-F-0571`](#sv-api-f-0571), [`SV-API-F-0579`](#sv-api-f-0579), [`SV-API-F-0580`](#sv-api-f-0580), [`SV-API-F-0594`](#sv-api-f-0594), [`SV-API-F-0595`](#sv-api-f-0595), [`SV-API-F-0596`](#sv-api-f-0596), [`SV-API-F-0597`](#sv-api-f-0597), [`SV-API-F-0598`](#sv-api-f-0598), [`SV-API-F-0599`](#sv-api-f-0599)

4. Recompute response_sampling_daily_limits in update_response_sampling_limits: clear it to None if response_sampling_interval_type or response_sampling_start_date is None or response_sampling_limit is 0; otherwise map interval duration (day=1, week=7, month=30) and store a cumulative daily response_sampling_limit and rollout_percentage JSON schedule (rounded to 2 decimals, final day forced to 100.0).
   - **Facts:** [`SV-API-F-0588`](#sv-api-f-0588), [`SV-API-F-0589`](#sv-api-f-0589), [`SV-API-F-0590`](#sv-api-f-0590)

5. Run ensure_question_ids to return early when questions is empty or falsy, or assign a UUID4 string id to any question missing one.
   - **Facts:** [`SV-API-F-0592`](#sv-api-f-0592), [`SV-API-F-0593`](#sv-api-f-0593)

6. On post_save (or post_delete), survey_changed registers a transaction on_commit callback that enqueues update_team_flags_cache for instance.team_id.
   - **Facts:** [`SV-API-F-0600`](#sv-api-f-0600), [`SV-API-F-0601`](#sv-api-f-0601), [`SV-API-F-0603`](#sv-api-f-0603)

---

### <a id="sv-mcp-pr-06"></a>`SV-MCP-PR-06`: Create, list, get, update, launch, stop, and soft-archive surveys via PostHog MCP tools (survey-create, survey-update, survey-delete, survey-get, surveys-get-all).

- **Surface:** PostHog MCP (`MCP`)
- **Capability:** `SV-1.1` Create a survey with templates, the wizard, or PostHog AI
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3)
- **Entry:** `products/surveys/mcp/tools.yaml#survey-create`

1. Call survey-create, survey-update, survey-delete, survey-get, or surveys-get-all with survey:read/survey:write scopes.
   - **Facts:** [`SV-MCP-F-0024`](#sv-mcp-f-0024), [`SV-MCP-F-0025`](#sv-mcp-f-0025), [`SV-MCP-F-0026`](#sv-mcp-f-0026), [`SV-MCP-F-0027`](#sv-mcp-f-0027), [`SV-MCP-F-0028`](#sv-mcp-f-0028), [`SV-MCP-F-0029`](#sv-mcp-f-0029), [`SV-MCP-F-0030`](#sv-mcp-f-0030), [`SV-MCP-F-0031`](#sv-mcp-f-0031), [`SV-MCP-F-0032`](#sv-mcp-f-0032), [`SV-MCP-F-0033`](#sv-mcp-f-0033), [`SV-MCP-F-0034`](#sv-mcp-f-0034), [`SV-MCP-F-0035`](#sv-mcp-f-0035), [`SV-MCP-F-0036`](#sv-mcp-f-0036), [`SV-MCP-F-0037`](#sv-mcp-f-0037), [`SV-MCP-F-0038`](#sv-mcp-f-0038), [`SV-MCP-F-0039`](#sv-mcp-f-0039), [`SV-MCP-F-0040`](#sv-mcp-f-0040), [`SV-MCP-F-0041`](#sv-mcp-f-0041), [`SV-MCP-F-0042`](#sv-mcp-f-0042), [`SV-MCP-F-0043`](#sv-mcp-f-0043)

---

### <a id="sv-ui-pr-02"></a>`SV-UI-PR-02`: Edit an existing survey in the full or hosted editor, modify questions, preview branching flow, manage translations, and save changes.

- **Surface:** PostHog web app (`UI`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `SurveyViewRedesign Edit button or SurveyComponent route with ?edit=true`

1. Enter edit mode on the survey route: SurveyViewRedesign routes to the guided wizard when canUseSurveyWizard(survey) and preferredEditor === 'guided', or invokes editingSurvey(true) otherwise; surveyLogic syncs ?edit=true in URL params and SurveyComponent registers the 'edit_survey' MaxTool for existing surveys.
   - **Facts:** [`SV-UI-F-0158`](#sv-ui-f-0158), [`SV-UI-F-0159`](#sv-ui-f-0159), [`SV-UI-F-0161`](#sv-ui-f-0161), [`SV-UI-F-0837`](#sv-ui-f-0837), [`SV-UI-F-2146`](#sv-ui-f-2146), [`SV-UI-F-2148`](#sv-ui-f-2148)

2. Render SurveyForm bound to surveyLogic, routing to HostedSurveyEdit when SURVEYS_HOSTED_EDITOR is enabled and survey.type is SurveyType.ExternalSurvey, or SurveyEdit otherwise (which permits converting non-hosted surveys to ExternalSurvey under SURVEYS_HOSTED_EDITOR).
   - **Facts:** [`SV-UI-F-0170`](#sv-ui-f-0170), [`SV-UI-F-0171`](#sv-ui-f-0171), [`SV-UI-F-0247`](#sv-ui-f-0247)

3. Edit intro screens, questions, and branching rules via SurveyEditIntroScreenHeader, SurveyEditQuestionGroup, and QuestionBranchingInput; when hasBranchingLogic is true, open SurveyBranchingFlowModal via the 'Preview branching flow' button.
   - **Facts:** [`SV-UI-F-0117`](#sv-ui-f-0117), [`SV-UI-F-0283`](#sv-ui-f-0283), [`SV-UI-F-0348`](#sv-ui-f-0348), [`SV-UI-F-0375`](#sv-ui-f-0375), [`SV-UI-F-0376`](#sv-ui-f-0376), [`SV-UI-F-0388`](#sv-ui-f-0388), [`SV-UI-F-0389`](#sv-ui-f-0389), [`SV-UI-F-0903`](#sv-ui-f-0903), [`SV-UI-F-1204`](#sv-ui-f-1204)

4. When FEATURE_FLAGS.SURVEYS_TRANSLATIONS is enabled, edit localized fields via SurveyTranslations and SurveyTranslationFields or invoke generateTranslationDrafts.
   - **Facts:** [`SV-UI-F-0096`](#sv-ui-f-0096), [`SV-UI-F-0246`](#sv-ui-f-0246), [`SV-UI-F-0677`](#sv-ui-f-0677), [`SV-UI-F-0678`](#sv-ui-f-0678), [`SV-UI-F-0713`](#sv-ui-f-0713), [`SV-UI-F-0714`](#sv-ui-f-0714)

5. Submit translation generation via surveysGenerateTranslationsCreate or persist survey edits via surveyLogic.updateSurvey (PATCH /api/projects/{}/surveys/{}/) through the authenticated Surveys API.
   - **Facts:** [`SV-UI-F-1903`](#sv-ui-f-1903), [`SV-UI-F-1934`](#sv-ui-f-1934)

---

### <a id="sv-api-pr-02"></a>`SV-API-PR-02`: Update an existing survey's configuration, schedule, actions, lifecycle dates, and targeting flags.

- **Surface:** REST API (`API`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `PATCH /api/projects/{project_id}/surveys/{id}/ (SurveyViewSet -> SurveySerializerCreateUpdateOnly.update)`

1. Fetch the pre-update Survey record and validate update-specific rules: project-scoped name uniqueness excluding self.instance.id, base_language collision checks against preserved translations, viewer access only for newly added action IDs, repeat_until_stopped and iteration_frequency_days reconciliation, and iteration_count >= instance.current_iteration.
   - **Facts:** [`SV-API-F-0153`](#sv-api-f-0153), [`SV-API-F-0154`](#sv-api-f-0154), [`SV-API-F-0155`](#sv-api-f-0155), [`SV-API-F-0221`](#sv-api-f-0221), [`SV-API-F-0222`](#sv-api-f-0222), [`SV-API-F-0236`](#sv-api-f-0236), [`SV-API-F-0246`](#sv-api-f-0246), [`SV-API-F-0258`](#sv-api-f-0258), [`SV-API-F-0260`](#sv-api-f-0260)

2. Enforce feature flag write scope whenever targeting_flag_id, remove_targeting_flag, or targeting_flag_filters is modified on the update request.
   - **Facts:** [`SV-API-F-0259`](#sv-api-f-0259), [`SV-API-F-0261`](#sv-api-f-0261), [`SV-API-F-0263`](#sv-api-f-0263)

3. Persist updated Survey fields inside a gated_atomic transaction, load the Team, update or clear associated Action records based on conditions.actions.values, log an 'updated' Survey activity entry with computed diffs, and emit 'survey launched', 'survey stopped', or 'survey resumed' events on start_date/end_date transitions.
   - **Facts:** [`SV-API-F-0126`](#sv-api-f-0126), [`SV-API-F-0265`](#sv-api-f-0265), [`SV-API-F-0266`](#sv-api-f-0266), [`SV-API-F-0269`](#sv-api-f-0269), [`SV-API-F-0270`](#sv-api-f-0270), [`SV-API-F-0271`](#sv-api-f-0271), [`SV-API-F-0272`](#sv-api-f-0272), [`SV-API-F-0276`](#sv-api-f-0276), [`SV-API-F-0277`](#sv-api-f-0277), [`SV-API-F-0278`](#sv-api-f-0278), [`SV-API-F-0279`](#sv-api-f-0279)

4. Update existing targeting and internal targeting FeatureFlag filters and synchronize targeting_flag.active via _should_survey_flags_be_active, triggering the FeatureFlag post_save remote-config pipeline.
   - **Facts:** [`SV-API-F-0267`](#sv-api-f-0267), [`SV-API-F-0281`](#sv-api-f-0281), [`SV-API-F-0283`](#sv-api-f-0283)

---

### <a id="sv-web-pr-04"></a>`SV-WEB-PR-04`: Validate and record end-user survey question answers, persist in-progress state, and capture the 'survey sent' event.

- **Surface:** JavaScript Web SDK (`WEB`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `Questions (onNextButtonClick)`

1. Initialize question state from previewPageIndex or saved InProgressSurveyState in localStorage (lastQuestionIndex, visitedIndices, questionSnapshots, and surveySubmissionId or a newly generated uuidv7()), falling back to defaults if localStorage fails.
   - **Facts:** [`SV-WEB-F-0160`](#sv-web-f-0160), [`SV-WEB-F-0180`](#sv-web-f-0180), [`SV-WEB-F-0181`](#sv-web-f-0181), [`SV-WEB-F-0182`](#sv-web-f-0182), [`SV-WEB-F-0466`](#sv-web-f-0466), [`SV-WEB-F-0468`](#sv-web-f-0468)

2. Guard submission in Questions: log an error and abort if posthog or questionId is missing; enforce required-field, MinLength, MaxLength, and rating scale (2, 3, 5, 7, 10) integer/range validation before recording the response.
   - **Facts:** [`SV-WEB-F-0185`](#sv-web-f-0185), [`SV-WEB-F-0187`](#sv-web-f-0187), [`SV-WEB-F-0353`](#sv-web-f-0353), [`SV-WEB-F-0355`](#sv-web-f-0355), [`SV-WEB-F-0357`](#sv-web-f-0357), [`SV-WEB-F-0359`](#sv-web-f-0359), [`SV-WEB-F-0361`](#sv-web-f-0361), [`SV-WEB-F-0363`](#sv-web-f-0363), [`SV-WEB-F-0373`](#sv-web-f-0373), [`SV-WEB-F-0583`](#sv-web-f-0583), [`SV-WEB-F-0585`](#sv-web-f-0585), [`SV-WEB-F-0586`](#sv-web-f-0586)

3. Persist partial progress to localStorage via setInProgressSurveyState (falling back to inMemoryInProgressSurveyState on storage error), ignore partial 'survey sent' events ($survey_completed === false) for activation consumption, and if autoDisappear is enabled on completion, schedule hiding the popup after 5000ms.
   - **Facts:** [`SV-WEB-F-0047`](#sv-web-f-0047), [`SV-WEB-F-0146`](#sv-web-f-0146), [`SV-WEB-F-0462`](#sv-web-f-0462), [`SV-WEB-F-0464`](#sv-web-f-0464)

4. Guard sendSurveyEvent (returning early if posthog is missing or client.canCapture is falsy), then capture 'survey sent' with survey metadata, response properties, replay URL, and $survey_responded/{id} person property set to true, and dispatch a 'PHSurveySent' CustomEvent on window when completed.
   - **Facts:** [`SV-WEB-F-0050`](#sv-web-f-0050), [`SV-WEB-F-0435`](#sv-web-f-0435), [`SV-WEB-F-0436`](#sv-web-f-0436), [`SV-WEB-F-0438`](#sv-web-f-0438), [`SV-WEB-F-0439`](#sv-web-f-0439)

---

### <a id="sv-and-pr-02"></a>`SV-AND-PR-02`: Validate user input across survey question types, evaluate response-based branching, persist progress, and capture the survey sent event.

- **Surface:** Android SDK (`AND`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `User interacts with SurveySheet question controls and taps the submit button`

1. Render the active question in SurveySheet using resolved SurveyAppearance defaults and translations (falling back to base survey text and UnsupportedQuestionPlaceholder for unknown question types) across OpenText, NumberRating, EmojiRating (2-, 3-, or default 5-emoji scales), SingleChoice, MultipleChoice (optionally shuffling regular choices while keeping an open choice last), and Link questions.
   - **Facts:** [`SV-AND-F-0041`](#sv-and-f-0041), [`SV-AND-F-0042`](#sv-and-f-0042), [`SV-AND-F-0049`](#sv-and-f-0049), [`SV-AND-F-0073`](#sv-and-f-0073), [`SV-AND-F-0074`](#sv-and-f-0074), [`SV-AND-F-0089`](#sv-and-f-0089), [`SV-AND-F-0091`](#sv-and-f-0091), [`SV-AND-F-0099`](#sv-and-f-0099), [`SV-AND-F-0100`](#sv-and-f-0100), [`SV-AND-F-0101`](#sv-and-f-0101), [`SV-AND-F-0115`](#sv-and-f-0115), [`SV-AND-F-0122`](#sv-and-f-0122), [`SV-AND-F-0137`](#sv-and-f-0137), [`SV-AND-F-0297`](#sv-and-f-0297), [`SV-AND-F-0311`](#sv-and-f-0311)

2. Enforce per-question submission guards unless question.isOptional is true: require a selected rating for Rating, non-blank text for OpenText, and a selected choice plus non-blank openInput when an open choice is selected for SingleChoice/MultipleChoice (submitting null instead of an empty list when no multiple choices are selected); Link buttons stay enabled and open URLs via ACTION_VIEW while silently catching intent errors.
   - **Facts:** [`SV-AND-F-0138`](#sv-and-f-0138), [`SV-AND-F-0142`](#sv-and-f-0142), [`SV-AND-F-0145`](#sv-and-f-0145), [`SV-AND-F-0148`](#sv-and-f-0148), [`SV-AND-F-0154`](#sv-and-f-0154), [`SV-AND-F-0157`](#sv-and-f-0157), [`SV-AND-F-0112`](#sv-and-f-0112), [`SV-AND-F-0113`](#sv-and-f-0113), [`SV-AND-F-0114`](#sv-and-f-0114)

3. Forward the submitted response through PostHogSurveyHost.submitResponse (returning null if the presentation is no longer current) into PostHogSurveysIntegration.surveyResponseCallback, which logs a warning and returns null if the attempt is inactive or canRecordResponse fails due to mismatched progress or resetGeneration.
   - **Facts:** [`SV-AND-F-0018`](#sv-and-f-0018), [`SV-AND-F-0025`](#sv-and-f-0025), [`SV-AND-F-0208`](#sv-and-f-0208), [`SV-AND-F-0209`](#sv-and-f-0209), [`SV-AND-F-0219`](#sv-and-f-0219)

4. Evaluate response-based branching for SingleChoice and Rating questions (mapping 3/5/7/10-point scales to buckets, logging errors and returning null on unsupported types or unresolved targets, and clamping destination indices to 0..maxOf(totalQuestions - 1, 0)), and persist updated progress to PostHogPreferences.SURVEY_PROGRESS when generation matches resetGeneration.
   - **Facts:** [`SV-AND-F-0223`](#sv-and-f-0223), [`SV-AND-F-0227`](#sv-and-f-0227), [`SV-AND-F-0230`](#sv-and-f-0230), [`SV-AND-F-0231`](#sv-and-f-0231), [`SV-AND-F-0235`](#sv-and-f-0235), [`SV-AND-F-0240`](#sv-and-f-0240), [`SV-AND-F-0292`](#sv-and-f-0292)

5. When shouldSendResponse is true, build the 'survey sent' event with response properties (resolving recorded, translated, then base question text), $survey_submission_id, $survey_completed, and $set marking the responded person property true, then emit it via postHog.capture.
   - **Facts:** [`SV-AND-F-0207`](#sv-and-f-0207), [`SV-AND-F-0210`](#sv-and-f-0210), [`SV-AND-F-0245`](#sv-and-f-0245), [`SV-AND-F-0247`](#sv-and-f-0247), [`SV-AND-F-0253`](#sv-and-f-0253), [`SV-AND-F-0169`](#sv-and-f-0169)

---

### <a id="sv-flt-pr-02"></a>`SV-FLT-PR-02`: Render interactive open-text, choice, rating, or link survey questions, validate user input, and submit responses to advance or complete the survey.

- **Surface:** Flutter SDK (`FLT`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `_SurveyBottomSheetState._buildQuestion`

1. Switch on currentQuestion.type to render OpenTextQuestion, LinkQuestion, RatingQuestion, or ChoiceQuestionWidget, each displaying a QuestionHeader with bold 18pt question text and an optional 16pt description when descriptionContentType is plain text.
   - **Facts:** [`SV-FLT-F-0142`](#sv-flt-f-0142), [`SV-FLT-F-0156`](#sv-flt-f-0156), [`SV-FLT-F-0161`](#sv-flt-f-0161), [`SV-FLT-F-0162`](#sv-flt-f-0162), [`SV-FLT-F-0186`](#sv-flt-f-0186), [`SV-FLT-F-0204`](#sv-flt-f-0204)

2. For open-text questions, render a scrollable multi-line TextField (min height 80, max height 70% of available height minus keyboard, bottom padding, and 150px), enable submission when optional or non-empty, and submit the trimmed text.
   - **Facts:** [`SV-FLT-F-0153`](#sv-flt-f-0153), [`SV-FLT-F-0154`](#sv-flt-f-0154), [`SV-FLT-F-0155`](#sv-flt-f-0155), [`SV-FLT-F-0157`](#sv-flt-f-0157), [`SV-FLT-F-0158`](#sv-flt-f-0158), [`SV-FLT-F-0159`](#sv-flt-f-0159), [`SV-FLT-F-0160`](#sv-flt-f-0160), [`SV-FLT-F-0205`](#sv-flt-f-0205)

3. For single- and multiple-choice questions, constrain list height to screenHeight - 350 - keyboardHeight - bottomPadding, optionally shuffle non-open choices while pinning open-choice last (reversing if the shuffle matches original order), auto-submit on tap only when skipSubmitButton && !isMultipleChoice && !hasOpenChoice, require a selection (and non-empty trimmed open-choice input when selected) unless optional, and submit selected strings (substituting open-choice text) or null.
   - **Facts:** [`SV-FLT-F-0116`](#sv-flt-f-0116), [`SV-FLT-F-0117`](#sv-flt-f-0117), [`SV-FLT-F-0119`](#sv-flt-f-0119), [`SV-FLT-F-0120`](#sv-flt-f-0120), [`SV-FLT-F-0121`](#sv-flt-f-0121), [`SV-FLT-F-0122`](#sv-flt-f-0122), [`SV-FLT-F-0123`](#sv-flt-f-0123), [`SV-FLT-F-0124`](#sv-flt-f-0124), [`SV-FLT-F-0125`](#sv-flt-f-0125), [`SV-FLT-F-0126`](#sv-flt-f-0126), [`SV-FLT-F-0127`](#sv-flt-f-0127), [`SV-FLT-F-0128`](#sv-flt-f-0128), [`SV-FLT-F-0130`](#sv-flt-f-0130), [`SV-FLT-F-0131`](#sv-flt-f-0131), [`SV-FLT-F-0132`](#sv-flt-f-0132), [`SV-FLT-F-0133`](#sv-flt-f-0133), [`SV-FLT-F-0208`](#sv-flt-f-0208), [`SV-FLT-F-0226`](#sv-flt-f-0226), [`SV-FLT-F-0228`](#sv-flt-f-0228), [`SV-FLT-F-0229`](#sv-flt-f-0229), [`SV-FLT-F-0230`](#sv-flt-f-0230)

4. For rating questions, render emoji RatingIcon buttons for 2-, 3-, or 5-point emoji scales (omitting bound labels on 2-point thumb scales) or NumberRatingButton otherwise; auto-submit on tap when skipSubmitButton is true, or toggle selection and gate the submit button on optional || _rating != null.
   - **Facts:** [`SV-FLT-F-0176`](#sv-flt-f-0176), [`SV-FLT-F-0177`](#sv-flt-f-0177), [`SV-FLT-F-0178`](#sv-flt-f-0178), [`SV-FLT-F-0179`](#sv-flt-f-0179), [`SV-FLT-F-0180`](#sv-flt-f-0180), [`SV-FLT-F-0181`](#sv-flt-f-0181), [`SV-FLT-F-0182`](#sv-flt-f-0182), [`SV-FLT-F-0183`](#sv-flt-f-0183), [`SV-FLT-F-0184`](#sv-flt-f-0184), [`SV-FLT-F-0185`](#sv-flt-f-0185), [`SV-FLT-F-0187`](#sv-flt-f-0187), [`SV-FLT-F-0188`](#sv-flt-f-0188), [`SV-FLT-F-0191`](#sv-flt-f-0191), [`SV-FLT-F-0192`](#sv-flt-f-0192), [`SV-FLT-F-0207`](#sv-flt-f-0207)

5. On submission (or pressing a LinkQuestion button, which submits true plus linkQuestion.link), guard against concurrent/completed submissions, block input with AbsorbPointer, await onResponse, open any non-empty link via PosthogFlutterPlatformInterface.instance.openUrl, check mounted state, close immediately if completed with displayThankYouMessage false or update _currentIndex/_isCompleted (showing ConfirmationMessage when completed), and reset _isSubmitting in finally.
   - **Facts:** [`SV-FLT-F-0135`](#sv-flt-f-0135), [`SV-FLT-F-0136`](#sv-flt-f-0136), [`SV-FLT-F-0137`](#sv-flt-f-0137), [`SV-FLT-F-0197`](#sv-flt-f-0197), [`SV-FLT-F-0198`](#sv-flt-f-0198), [`SV-FLT-F-0199`](#sv-flt-f-0199), [`SV-FLT-F-0200`](#sv-flt-f-0200), [`SV-FLT-F-0201`](#sv-flt-f-0201), [`SV-FLT-F-0202`](#sv-flt-f-0202), [`SV-FLT-F-0203`](#sv-flt-f-0203), [`SV-FLT-F-0206`](#sv-flt-f-0206), [`SV-FLT-F-0214`](#sv-flt-f-0214), [`SV-FLT-F-0216`](#sv-flt-f-0216)

---

### <a id="sv-ios-pr-03"></a>`SV-IOS-PR-03`: Capture a user's question response, advance or complete the survey using branching rules, update persisted progress, and emit 'survey sent'.

- **Surface:** iOS SDK (`IOS`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `Question view submit or action tap invoking SurveyDisplayController.onNextQuestion(_:)`

1. Collect input from the active question view (open text, link opening via UIApplication.shared.open when canOpenURL succeeds, single/multiple choice with optional open-choice input disabled while unselected, or number/emoji rating) and emit onSurveyResponse from SurveyDisplayController.onNextQuestion.
   - **Facts:** [`SV-IOS-F-0213`](#sv-ios-f-0213), [`SV-IOS-F-0215`](#sv-ios-f-0215), [`SV-IOS-F-0216`](#sv-ios-f-0216), [`SV-IOS-F-0218`](#sv-ios-f-0218), [`SV-IOS-F-0343`](#sv-ios-f-0343), [`SV-IOS-F-0346`](#sv-ios-f-0346), [`SV-IOS-F-0347`](#sv-ios-f-0347), [`SV-IOS-F-0332`](#sv-ios-f-0332), [`SV-IOS-F-0351`](#sv-ios-f-0351), [`SV-IOS-F-0230`](#sv-ios-f-0230), [`SV-IOS-F-0231`](#sv-ios-f-0231)

2. handleSurveyResponse validates the active attempt, survey ID, non-completed state, and matching in-bounds activeSurveyQuestionIndex; if activeProgressWasPersisted is true, it verifies the stored submissionId in progressStore matches before continuing, otherwise clearing the active survey and returning nil.
   - **Facts:** [`SV-IOS-F-0136`](#sv-ios-f-0136), [`SV-IOS-F-0137`](#sv-ios-f-0137), [`SV-IOS-F-0138`](#sv-ios-f-0138), [`SV-IOS-F-0146`](#sv-ios-f-0146), [`SV-IOS-F-0147`](#sv-ios-f-0147), [`SV-IOS-F-0152`](#sv-ios-f-0152)

3. Resolve the next question via branching (next_question, end, specific_question, or response_based), clamping integer targets to totalQuestions - 1 and logging/returning nil on nil responses, invalid rating scale ranges (3: 1-3, 5: 1-5, 7: 1-7, 10: 0-10), or unsupported question/response pairs.
   - **Facts:** [`SV-IOS-F-0037`](#sv-ios-f-0037), [`SV-IOS-F-0045`](#sv-ios-f-0045), [`SV-IOS-F-0187`](#sv-ios-f-0187), [`SV-IOS-F-0374`](#sv-ios-f-0374), [`SV-IOS-F-0378`](#sv-ios-f-0378), [`SV-IOS-F-0379`](#sv-ios-f-0379), [`SV-IOS-F-0380`](#sv-ios-f-0380), [`SV-IOS-F-0381`](#sv-ios-f-0381), [`SV-IOS-F-0382`](#sv-ios-f-0382), [`SV-IOS-F-0388`](#sv-ios-f-0388), [`SV-IOS-F-0393`](#sv-ios-f-0393)

4. Persist updated SurveyProgress (questionIndex, responses, questionText, language) to .surveyProgress when the storage generation and resetEpoch remain current, or remove saved progress when activeSurveyCompleted is true.
   - **Facts:** [`SV-IOS-F-0177`](#sv-ios-f-0177), [`SV-IOS-F-0178`](#sv-ios-f-0178), [`SV-IOS-F-0179`](#sv-ios-f-0179), [`SV-IOS-F-0180`](#sv-ios-f-0180), [`SV-IOS-F-0238`](#sv-ios-f-0238), [`SV-IOS-F-0254`](#sv-ios-f-0254), [`SV-IOS-F-0255`](#sv-ios-f-0255), [`SV-IOS-F-0257`](#sv-ios-f-0257), [`SV-IOS-F-0258`](#sv-ios-f-0258)

5. When enablePartialResponses is true or the survey is completed, build response properties with question-text fallbacks, attach $set person property $survey_responded/{id} with $survey_completed and $survey_submission_id, mark the survey as seen in .surveySeen, and emit 'survey sent' via postHog.capture.
   - **Facts:** [`SV-IOS-F-0151`](#sv-ios-f-0151), [`SV-IOS-F-0160`](#sv-ios-f-0160), [`SV-IOS-F-0161`](#sv-ios-f-0161), [`SV-IOS-F-0164`](#sv-ios-f-0164), [`SV-IOS-F-0127`](#sv-ios-f-0127), [`SV-IOS-F-0128`](#sv-ios-f-0128), [`SV-IOS-F-0169`](#sv-ios-f-0169)

---

### <a id="sv-rn-pr-02"></a>`SV-RN-PR-02`: Capture user answers, evaluate question branching, dismiss surveys, and emit survey shown/sent/dismissed events in React Native.

- **Surface:** React Native SDK (`RN`)
- **Capability:** `SV-2.1` Configure question types, rating scales, auto-submit, and shuffling
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `packages/react-native/src/surveys/components/SurveyModal.tsx#SurveyModal`

1. Validate React Native question inputs, advance branches, and capture survey events.
   - **Facts:** [`SV-RN-F-0008`](#sv-rn-f-0008), [`SV-RN-F-0011`](#sv-rn-f-0011), [`SV-RN-F-0019`](#sv-rn-f-0019), [`SV-RN-F-0020`](#sv-rn-f-0020), [`SV-RN-F-0023`](#sv-rn-f-0023), [`SV-RN-F-0024`](#sv-rn-f-0024), [`SV-RN-F-0026`](#sv-rn-f-0026), [`SV-RN-F-0027`](#sv-rn-f-0027), [`SV-RN-F-0028`](#sv-rn-f-0028), [`SV-RN-F-0029`](#sv-rn-f-0029), [`SV-RN-F-0030`](#sv-rn-f-0030), [`SV-RN-F-0039`](#sv-rn-f-0039), [`SV-RN-F-0040`](#sv-rn-f-0040), [`SV-RN-F-0041`](#sv-rn-f-0041), [`SV-RN-F-0043`](#sv-rn-f-0043), [`SV-RN-F-0044`](#sv-rn-f-0044), [`SV-RN-F-0047`](#sv-rn-f-0047), [`SV-RN-F-0052`](#sv-rn-f-0052), [`SV-RN-F-0053`](#sv-rn-f-0053), [`SV-RN-F-0054`](#sv-rn-f-0054)

---

### <a id="sv-ui-pr-03"></a>`SV-UI-PR-03`: Configure survey display triggers, completion limits, widget styling, and linked feature flag targeting in SurveyEdit.

- **Surface:** PostHog web app (`UI`)
- **Capability:** `SV-3.4` Stop surveys automatically at a response limit
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `SurveyEdit targeting, activation triggers, and customization sections`

1. Configure activation and cancellation triggers: bind event triggers via SurveyEventSelector/SurveyEventTrigger (property filtering scoped to posthog-js 1.268+ or posthog-react-native 4.15+), cancel-on-event triggers via SurveyCancelEventTrigger (web surveys on posthog-js 1.299+), and action triggers via SurveyActionTrigger when FEATURE_FLAGS.SURVEYS_ACTIONS is enabled.
   - **Facts:** [`SV-UI-F-0189`](#sv-ui-f-0189), [`SV-UI-F-0194`](#sv-ui-f-0194), [`SV-UI-F-0301`](#sv-ui-f-0301), [`SV-UI-F-0469`](#sv-ui-f-0469), [`SV-UI-F-0470`](#sv-ui-f-0470), [`SV-UI-F-0478`](#sv-ui-f-0478), [`SV-UI-F-0479`](#sv-ui-f-0479), [`SV-UI-F-0481`](#sv-ui-f-0481)

2. Configure response limits and repeat schedules in SurveyCompletionConditions and SurveyIterationOptions, and customize widget button position in SurveyWidgetCustomization (paywalled behind AvailableFeature.SURVEYS_STYLING, disabled when surveysStylingAvailable is false, and requiring posthog-js 1.294.0+).
   - **Facts:** [`SV-UI-F-0216`](#sv-ui-f-0216), [`SV-UI-F-0217`](#sv-ui-f-0217), [`SV-UI-F-0519`](#sv-ui-f-0519), [`SV-UI-F-0520`](#sv-ui-f-0520), [`SV-UI-F-0861`](#sv-ui-f-0861), [`SV-UI-F-0872`](#sv-ui-f-0872), [`SV-UI-F-0876`](#sv-ui-f-0876), [`SV-UI-F-0877`](#sv-ui-f-0877)

3. Select or hydrate a linked feature flag in SurveyEdit: when survey.linked_flag_id is set without a loaded linked_flag object, fetch it via api.featureFlags.get(survey.linked_flag_id) through the authenticated project API (clearing linked_flag_id to null and removing linkedFlagVariant from conditions if the fetch fails).
   - **Facts:** [`SV-UI-F-0292`](#sv-ui-f-0292), [`SV-UI-F-0353`](#sv-ui-f-0353)

---

### <a id="sv-api-pr-10"></a>`SV-API-PR-10`: Count unique survey responses per active response-limited survey via HogQL to determine which surveys have reached responses_limit.

- **Surface:** REST API (`API`)
- **Capability:** `SV-3.4` Stop surveys automatically at a response limit
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `stop_surveys_reached_target`

1. Query active Survey records with a non-null responses_limit and no end_date, then bulk-load all referenced Team records.
   - **Facts:** [`SV-API-F-0020`](#sv-api-f-0020), [`SV-API-F-0021`](#sv-api-f-0021)

2. Deduplicate response counts by non-empty properties.$survey_submission_id (falling back to toString(uuid)) and cap returned rows to the number of survey IDs in surveys_ids.
   - **Facts:** [`SV-API-F-0013`](#sv-api-f-0013), [`SV-API-F-0014`](#sv-api-f-0014)

3. Execute the offline HogQL query (query_type='stop_surveys_reached_target') against events filtered to 'survey sent' events at or after earliest_survey_creation_date for surveys_ids.
   - **Facts:** [`SV-API-F-0011`](#sv-api-f-0011), [`SV-API-F-0012`](#sv-api-f-0012)

---

### <a id="sv-api-pr-11"></a>`SV-API-PR-11`: Stop a survey that has reached its responses_limit and record the system action in the activity log.

- **Surface:** REST API (`API`)
- **Capability:** `SV-3.4` Stop surveys automatically at a response limit
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `_stop_survey_if_reached_limit`

1. Refresh the Survey instance from the database before evaluating its current responses_limit and end_date.
   - **Facts:** [`SV-API-F-0015`](#sv-api-f-0015)

2. Return early without modifying the survey if responses_limit is None or end_date is already set.
   - **Facts:** [`SV-API-F-0016`](#sv-api-f-0016)

3. Stop the survey by setting end_date to timezone.now(), clearing responses_limit to None, and saving those fields to the Survey model.
   - **Facts:** [`SV-API-F-0018`](#sv-api-f-0018)

4. Write an 'updated' activity log entry via log_activity for scope='Survey' and item_id=survey.id under survey.team_id and survey.team.organization_id.
   - **Facts:** [`SV-API-F-0026`](#sv-api-f-0026)

---

### <a id="sv-api-pr-14"></a>`SV-API-PR-14`: Advance active recurring surveys to their next iteration or close them after their final iteration while syncing their internal targeting feature flag.

- **Surface:** REST API (`API`)
- **Capability:** `SV-3.5` Schedule recurring survey iterations and repeat display rules
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Entry:** `update_survey_iteration`

1. Query active recurring Survey records (schedule=Survey.Schedule.RECURRING, start_date set, end_date null, and iteration_count set or repeat_until_stopped=True), catching per-survey exceptions via capture_exception with survey_id and team_id so other surveys continue.
   - **Facts:** [`SV-API-F-0053`](#sv-api-f-0053), [`SV-API-F-0054`](#sv-api-f-0054)

2. Refresh each Survey from the database, compute its final iteration end (catching OverflowError and returning None), and check ActivityLog for non-system entries where end_date was changed to None.
   - **Facts:** [`SV-API-F-0036`](#sv-api-f-0036), [`SV-API-F-0047`](#sv-api-f-0047), [`SV-API-F-0048`](#sv-api-f-0048)

3. Persist end_date when closing the survey after its final iteration, or save updated current_iteration, current_iteration_start_date, internal_targeting_flag_id, and iteration_start_dates when advancing.
   - **Facts:** [`SV-API-F-0040`](#sv-api-f-0040), [`SV-API-F-0043`](#sv-api-f-0043)

4. Build internal targeting release groups requiring $survey_dismissed/{survey_key} and $survey_responded/{survey_key} to be unset (plus optional wait_period_days last-seen rules), then update the existing internal_targeting_flag or create an active FeatureFlag keyed by str(survey.id) with creation_context='surveys' and created_by backfilled.
   - **Facts:** [`SV-API-F-0541`](#sv-api-f-0541), [`SV-API-F-0542`](#sv-api-f-0542), [`SV-API-F-0544`](#sv-api-f-0544), [`SV-API-F-0050`](#sv-api-f-0050), [`SV-API-F-0051`](#sv-api-f-0051), [`SV-API-F-0052`](#sv-api-f-0052)

---

### <a id="sv-ui-pr-13"></a>`SV-UI-PR-13`: Filter survey results by date, properties, answer values, or archived state, inspect metrics and individual response rows, and build the underlying HogQL response query.

- **Surface:** PostHog web app (`UI`)
- **Capability:** `SV-7.1` View survey metrics, charts, and filtered response rows
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-5.1`](#prereq-sv-5-1)
- **Entry:** `Survey results view (SurveyResultsFiltersBar, SurveyStatsSummary, SurveyResponsesContent)`

1. Adjust the date range via DateFilter (setDateRange), expand the Filters panel to add event, person, feature flag, cohort, HogQL, or group filters via PropertyFilters (setPropertyFilters), or toggle the 'Show archived' switch (setShowArchivedResponses).
   - **Facts:** [`SV-UI-F-0733`](#sv-ui-f-0733), [`SV-UI-F-0737`](#sv-ui-f-0737), [`SV-UI-F-0738`](#sv-ui-f-0738), [`SV-UI-F-0745`](#sv-ui-f-0745), [`SV-UI-F-0746`](#sv-ui-f-0746), [`SV-UI-F-0755`](#sv-ui-f-0755), [`SV-UI-F-0756`](#sv-ui-f-0756)

2. Filter by specific question answers by selecting an operator from OPERATOR_OPTIONS[question.type] (defaulting to PropertyOperator.Exact for new filters) and entering a response value scoped to the 'survey sent' event, or copy the response property key ($survey_response_<questionId>).
   - **Facts:** [`SV-UI-F-0534`](#sv-ui-f-0534), [`SV-UI-F-0535`](#sv-ui-f-0535), [`SV-UI-F-0536`](#sv-ui-f-0536), [`SV-UI-F-0732`](#sv-ui-f-0732), [`SV-UI-F-0734`](#sv-ui-f-0734), [`SV-UI-F-0735`](#sv-ui-f-0735), [`SV-UI-F-0749`](#sv-ui-f-0749), [`SV-UI-F-0750`](#sv-ui-f-0750), [`SV-UI-F-0752`](#sv-ui-f-0752), [`SV-UI-F-0754`](#sv-ui-f-0754)

3. Inspect shown, unanswered, dismissed, and sent metrics in SurveyStatsSummary and toggle 'Count each person once' (setFilterSurveyStatsByDistinctId) to switch between unique_persons and total_count; when no responses match, view SurveyNoResponsesBanner with filter-specific guidance and use 'Clear current filters' (clearFilters).
   - **Facts:** [`SV-UI-F-0498`](#sv-ui-f-0498), [`SV-UI-F-0502`](#sv-ui-f-0502), [`SV-UI-F-0504`](#sv-ui-f-0504), [`SV-UI-F-0506`](#sv-ui-f-0506), [`SV-UI-F-0507`](#sv-ui-f-0507), [`SV-UI-F-0508`](#sv-ui-f-0508), [`SV-UI-F-0509`](#sv-ui-f-0509), [`SV-UI-F-0629`](#sv-ui-f-0629), [`SV-UI-F-0630`](#sv-ui-f-0630), [`SV-UI-F-0634`](#sv-ui-f-0634), [`SV-UI-F-0635`](#sv-ui-f-0635), [`SV-UI-F-0639`](#sv-ui-f-0639), [`SV-UI-F-0640`](#sv-ui-f-0640), [`SV-UI-F-0641`](#sv-ui-f-0641), [`SV-UI-F-0642`](#sv-ui-f-0642), [`SV-UI-F-0651`](#sv-ui-f-0651), [`SV-UI-F-0652`](#sv-ui-f-0652), [`SV-UI-F-0656`](#sv-ui-f-0656), [`SV-UI-F-0740`](#sv-ui-f-0740)

4. Inspect individual merged completed, dismissed, and abandoned responses in SurveyResponsesContent: expand non-interactive rows or long (>100 char) response cards (virtualized above 24 static responses), open session recordings or $ai_trace_id LLM traces, and archive or unarchive responses via ArchiveSurveyButton.
   - **Facts:** [`SV-UI-F-0794`](#sv-ui-f-0794), [`SV-UI-F-0848`](#sv-ui-f-0848), [`SV-UI-F-0849`](#sv-ui-f-0849), [`SV-UI-F-0858`](#sv-ui-f-0858), [`SV-UI-F-0859`](#sv-ui-f-0859), [`SV-UI-F-0860`](#sv-ui-f-0860), [`SV-UI-F-0926`](#sv-ui-f-0926), [`SV-UI-F-0927`](#sv-ui-f-0927), [`SV-UI-F-0928`](#sv-ui-f-0928), [`SV-UI-F-0929`](#sv-ui-f-0929), [`SV-UI-F-1396`](#sv-ui-f-1396), [`SV-UI-F-1398`](#sv-ui-f-1398), [`SV-UI-F-1399`](#sv-ui-f-1399), [`SV-UI-F-1402`](#sv-ui-f-1402), [`SV-UI-F-1483`](#sv-ui-f-1483), [`SV-UI-F-1485`](#sv-ui-f-1485), [`SV-UI-F-1493`](#sv-ui-f-1493), [`SV-UI-F-2434`](#sv-ui-f-2434), [`SV-UI-F-2435`](#sv-ui-f-2435), [`SV-UI-F-2446`](#sv-ui-f-2446), [`SV-UI-F-2452`](#sv-ui-f-2452)

5. Build the HogQL SELECT query over merged survey submissions ordered by submitted_at DESC via buildSurveyResponsesQuery (or view a 100-row standalone SQL query in SurveySQLHelper) and hand execution off to the HogQL query runner.
   - **Facts:** [`SV-UI-F-0546`](#sv-ui-f-0546), [`SV-UI-F-0559`](#sv-ui-f-0559), [`SV-UI-F-2455`](#sv-ui-f-2455), [`SV-UI-F-2462`](#sv-ui-f-2462), [`SV-UI-F-2464`](#sv-ui-f-2464)

---

### <a id="sv-ui-pr-18"></a>`SV-UI-PR-18`: Preview simulated pre-launch survey results and interact with multiple-choice and rating breakdown charts.

- **Surface:** PostHog web app (`UI`)
- **Capability:** `SV-7.1` View survey metrics, charts, and filtered response rows
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-5.1`](#prereq-sv-5-1)
- **Entry:** `SurveyResultDemo, MultipleChoiceBarChart, and RatingBarChart`

1. Generate simulated demo results (defaulting to 76 responses via generateDemoSurveyResults, rating scale defaulting to 5, multiple-choice selecting 1+ options) and render DemoStatsSummary, SurveyResponsesByQuestionV2Demo, and DemoDataTable (silently falling back to {} on invalid person-property JSON and using 'https://app.example.com/dashboard').
   - **Facts:** [`SV-UI-F-0567`](#sv-ui-f-0567), [`SV-UI-F-0570`](#sv-ui-f-0570), [`SV-UI-F-0571`](#sv-ui-f-0571), [`SV-UI-F-0573`](#sv-ui-f-0573), [`SV-UI-F-0574`](#sv-ui-f-0574), [`SV-UI-F-2501`](#sv-ui-f-2501), [`SV-UI-F-2510`](#sv-ui-f-2510), [`SV-UI-F-2512`](#sv-ui-f-2512), [`SV-UI-F-2514`](#sv-ui-f-2514)

2. Render the disabled 'Explore results' button (data-attr='survey-results-explore-demo') with disabledReason 'Demo mode - launch survey to explore real results' until the survey is launched.
   - **Facts:** [`SV-UI-F-0578`](#sv-ui-f-0578)

3. In breakdown charts, hover bars to display ChoiceTooltip or RatingTooltip (with multiple-choice category labels capped at 280px, minimum 32px bar bands, 0-100% domain, and rating values defaulting to 0) and click a bar to invoke onBarClick(dataIndex).
   - **Facts:** [`SV-UI-F-1228`](#sv-ui-f-1228), [`SV-UI-F-1232`](#sv-ui-f-1232), [`SV-UI-F-1234`](#sv-ui-f-1234), [`SV-UI-F-1235`](#sv-ui-f-1235), [`SV-UI-F-1238`](#sv-ui-f-1238), [`SV-UI-F-1239`](#sv-ui-f-1239), [`SV-UI-F-1301`](#sv-ui-f-1301), [`SV-UI-F-1304`](#sv-ui-f-1304), [`SV-UI-F-1305`](#sv-ui-f-1305)

---

### <a id="sv-mcp-pr-04"></a>`SV-MCP-PR-04`: Inspect survey response results, completion outcomes, and customize response table and export columns.

- **Surface:** PostHog MCP (`MCP`)
- **Capability:** `SV-7.1` View survey metrics, charts, and filtered response rows
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-5.1`](#prereq-sv-5-1)
- **Entry:** `SurveyResult, SurveyResponseBreakdown, and SurveyResponseColumnsMenu`

1. Subscribe to surveyLogic in SurveyResult to check processedSurveyStats for 'survey sent' responses, rendering LemonSkeleton during initial load, SurveyNoResponsesBanner (with clearFilters when hasActiveFilters is true) when no responses exist, or stats summary, optional SurveyHeadline, per-question visualizations, 'Explore results' link to surveyAsInsightURL, and the events table (unless disableEventsTable is true).
   - **Facts:** [`SV-UI-F-2906`](#sv-ui-f-2906), [`SV-UI-F-2907`](#sv-ui-f-2907), [`SV-UI-F-2908`](#sv-ui-f-2908), [`SV-UI-F-2909`](#sv-ui-f-2909), [`SV-UI-F-2910`](#sv-ui-f-2910), [`SV-UI-F-2911`](#sv-ui-f-2911), [`SV-UI-F-2912`](#sv-ui-f-2912), [`SV-UI-F-2916`](#sv-ui-f-2916)

2. Render per-question SurveyQuestionVisualization cards in SurveyResponsesByQuestionV2 (skipping questions without an id or of type SurveyQuestionType.Link) alongside SurveyResponseBreakdown showing 'Response completion' counts and 1-decimal percentages mapped via OUTCOME_LABELS ('Completed', 'Partial · dismissed', 'Partial · abandoned') with green vs muted status dots.
   - **Facts:** [`SV-UI-F-2903`](#sv-ui-f-2903), [`SV-UI-F-2904`](#sv-ui-f-2904), [`SV-UI-F-2905`](#sv-ui-f-2905), [`SV-UI-F-2917`](#sv-ui-f-2917), [`SV-UI-F-2918`](#sv-ui-f-2918), [`SV-UI-F-2919`](#sv-ui-f-2919), [`SV-UI-F-2920`](#sv-ui-f-2920), [`SV-UI-F-2921`](#sv-ui-f-2921), [`SV-UI-F-2922`](#sv-ui-f-2922), [`SV-UI-F-2923`](#sv-ui-f-2923), [`SV-UI-F-2924`](#sv-ui-f-2924), [`SV-UI-F-2925`](#sv-ui-f-2925)

3. Display response events via Query with dataTableQuery, expandable response rows, responsesExportQuery (excluding 'response' and 'actions'), and transformSurveyResponseRows, dimming archived rows ('opacity-50') when result[0]?.uuid is in archivedResponseUuids.
   - **Facts:** [`SV-UI-F-2913`](#sv-ui-f-2913), [`SV-UI-F-2914`](#sv-ui-f-2914), [`SV-UI-F-2915`](#sv-ui-f-2915)

4. Open the 'Columns' dropdown in SurveyResponseColumnsMenu to toggle the 'Person ID' switch, remove property columns, or click 'Add column' to pick EventProperties or PersonProperties scoped to 'survey sent' via TaxonomicFilter (excluding already-added keys).
   - **Facts:** [`SV-UI-F-2926`](#sv-ui-f-2926), [`SV-UI-F-2927`](#sv-ui-f-2927), [`SV-UI-F-2928`](#sv-ui-f-2928), [`SV-UI-F-2929`](#sv-ui-f-2929), [`SV-UI-F-2930`](#sv-ui-f-2930), [`SV-UI-F-2931`](#sv-ui-f-2931), [`SV-UI-F-2932`](#sv-ui-f-2932), [`SV-UI-F-2933`](#sv-ui-f-2933), [`SV-UI-F-2934`](#sv-ui-f-2934), [`SV-UI-F-2935`](#sv-ui-f-2935), [`SV-UI-F-2936`](#sv-ui-f-2936)

---

### <a id="sv-api-pr-12"></a>`SV-API-PR-12`: Validate date bounds and compute shown, dismissed, sent, and dismissed-and-sent statistics for a single survey or all non-archived team surveys.

- **Surface:** REST API (`API`)
- **Capability:** `SV-7.1` View survey metrics, charts, and filtered response rows
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-5.1`](#prereq-sv-5-1)
- **Entry:** `get_survey_stats`

1. Parse and validate date_from and date_to via validate_and_parse_dates and StatsWindow, raising ValueError on invalid or overflowing date formats or when date_from > date_to.
   - **Facts:** [`SV-API-F-0722`](#sv-api-f-0722), [`SV-API-F-0728`](#sv-api-f-0728)

2. Load the Team record by team_id, then load start_date, created_at, and end_date for the requested survey_id—or load all non-archived survey IDs for the team when survey_id is omitted.
   - **Facts:** [`SV-API-F-0734`](#sv-api-f-0734), [`SV-API-F-0737`](#sv-api-f-0737), [`SV-API-F-0745`](#sv-api-f-0745)

3. Build the latest-response-event UUID subquery per (properties.$survey_id, SUBMISSION_GROUPING_KEY) filtered by RESPONSE_EVENT_FILTER ('survey sent', 'survey dismissed', 'survey abandoned') and execute the survey_stats_base and survey_stats_dismissed_and_sent HogQL queries.
   - **Facts:** [`SV-API-F-0671`](#sv-api-f-0671), [`SV-API-F-0733`](#sv-api-f-0733), [`SV-API-F-0751`](#sv-api-f-0751), [`SV-API-F-0752`](#sv-api-f-0752)

---

### <a id="sv-api-pr-03"></a>`SV-API-PR-03`: List, filter, and read surveys, stats, responses, site snapshots, question labels, and activity logs under authenticated DRF access control.

- **Surface:** REST API (`API`)
- **Capability:** `SV-7.1` View survey metrics, charts, and filtered response rows
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-5.1`](#prereq-sv-5-1)
- **Entry:** `GET /api/projects/{project_id}/surveys/ and read detail/list actions on SurveyViewSet`

1. Query Survey records with select_related for linked_flag, linked_insight, targeting_flag, and internal_targeting_flag; apply SurveyFilterSet filters (ids, created_by, linked_flag_id, status, archived, type), reject search terms exceeding MAX_SEARCH_LENGTH, and serialize via SurveySerializer.
   - **Facts:** [`SV-API-F-0111`](#sv-api-f-0111), [`SV-API-F-0114`](#sv-api-f-0114), [`SV-API-F-0291`](#sv-api-f-0291), [`SV-API-F-0294`](#sv-api-f-0294), [`SV-API-F-0299`](#sv-api-f-0299)

2. Handle read-only survey sub-endpoints with input validation and error mapping: /responses_count/, /stats/ (raising NotFound when missing and converting ValueError to ValidationError), /responses/ (limit 1..500, offset >= 0, requiring question_id with score_lte/score_gte), /site_snapshot/ (device and URL target defaulting match_type to ICONTAINS), /question_labels/ (chunked in 200-row batches), and paginated /activity/ logs.
   - **Facts:** [`SV-API-F-0062`](#sv-api-f-0062), [`SV-API-F-0063`](#sv-api-f-0063), [`SV-API-F-0064`](#sv-api-f-0064), [`SV-API-F-0066`](#sv-api-f-0066), [`SV-API-F-0107`](#sv-api-f-0107), [`SV-API-F-0108`](#sv-api-f-0108), [`SV-API-F-0110`](#sv-api-f-0110), [`SV-API-F-0297`](#sv-api-f-0297), [`SV-API-F-0318`](#sv-api-f-0318), [`SV-API-F-0321`](#sv-api-f-0321), [`SV-API-F-0323`](#sv-api-f-0323), [`SV-API-F-0324`](#sv-api-f-0324), [`SV-API-F-0339`](#sv-api-f-0339), [`SV-API-F-0341`](#sv-api-f-0341), [`SV-API-F-0346`](#sv-api-f-0346), [`SV-API-F-0347`](#sv-api-f-0347), [`SV-API-F-0353`](#sv-api-f-0353), [`SV-API-F-0359`](#sv-api-f-0359), [`SV-API-F-0360`](#sv-api-f-0360), [`SV-API-F-0363`](#sv-api-f-0363), [`SV-API-F-0364`](#sv-api-f-0364), [`SV-API-F-0366`](#sv-api-f-0366), [`SV-API-F-0367`](#sv-api-f-0367), [`SV-API-F-0368`](#sv-api-f-0368)

3. Enforce DRF authentication, team/org scoping, AccessControlViewSetMixin (scope_object = 'survey'), user_access_control queryset filtering, and per-action required_scopes (survey:read, query:read, session_recording:read, export:read, activity_log:read) via the platform auth layer.
   - **Facts:** [`SV-API-F-0292`](#sv-api-f-0292), [`SV-API-F-0293`](#sv-api-f-0293), [`SV-API-F-0317`](#sv-api-f-0317), [`SV-API-F-0322`](#sv-api-f-0322), [`SV-API-F-0340`](#sv-api-f-0340), [`SV-API-F-0343`](#sv-api-f-0343), [`SV-API-F-0345`](#sv-api-f-0345), [`SV-API-F-0351`](#sv-api-f-0351), [`SV-API-F-0352`](#sv-api-f-0352), [`SV-API-F-0358`](#sv-api-f-0358), [`SV-API-F-0362`](#sv-api-f-0362), [`SV-API-F-0365`](#sv-api-f-0365)

---

### <a id="sv-mcp-pr-07"></a>`SV-MCP-PR-07`: Query global or per-survey statistics and list individual responses via PostHog MCP tools (surveys-global-stats, survey-stats, surveys-responses-list requiring survey:read + query:read).

- **Surface:** PostHog MCP (`MCP`)
- **Capability:** `SV-7.1` View survey metrics, charts, and filtered response rows
- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.3`](#prereq-sv-0-3), [`SV-5.1`](#prereq-sv-5-1)
- **Entry:** `products/surveys/mcp/tools.yaml#surveys-responses-list`

1. Call surveys-global-stats, survey-stats, or surveys-responses-list with survey:read and query:read scopes.
   - **Facts:** [`SV-MCP-F-0005`](#sv-mcp-f-0005), [`SV-MCP-F-0028`](#sv-mcp-f-0028), [`SV-MCP-F-0034`](#sv-mcp-f-0034), [`SV-MCP-F-0035`](#sv-mcp-f-0035), [`SV-MCP-F-0036`](#sv-mcp-f-0036), [`SV-MCP-F-0056`](#sv-mcp-f-0056), [`SV-MCP-F-0079`](#sv-mcp-f-0079), [`SV-MCP-F-0097`](#sv-mcp-f-0097), [`SV-MCP-F-0100`](#sv-mcp-f-0100), [`SV-MCP-F-0101`](#sv-mcp-f-0101), [`SV-MCP-F-0102`](#sv-mcp-f-0102), [`SV-MCP-F-0103`](#sv-mcp-f-0103), [`SV-MCP-F-0104`](#sv-mcp-f-0104), [`SV-MCP-F-0105`](#sv-mcp-f-0105), [`SV-MCP-F-0106`](#sv-mcp-f-0106), [`SV-MCP-F-0107`](#sv-mcp-f-0107), [`SV-MCP-F-0112`](#sv-mcp-f-0112), [`SV-MCP-F-0113`](#sv-mcp-f-0113), [`SV-MCP-F-0114`](#sv-mcp-f-0114), [`SV-MCP-F-0117`](#sv-mcp-f-0117)

---

## <a id="resolved-prerequisites"></a>Resolved prerequisites

### <a id="prereq-sv-0-1"></a>`SV-0.1`: Enable surveys in project settings and set defaults

- **Codes:** `SV-UI-P-01`, `SV-MCP-P-01`
- **Summary:** Turn on the project-wide Surveys opt-in toggle, verify setup readiness in the empty state, and save default survey appearance settings for your team.

#### <a id="sv-ui-pr-05"></a>`SV-UI-PR-05` (`UI`): Enable project-wide surveys opt-in and persist default survey appearance settings for the current team.

- **Entry:** `QuickSurveyForm opt-in warning notice, SurveysDisabledBanner, or openSurveysSettingsDialog`

1. Detect disabled surveys via !currentTeam?.surveys_opt_in on mount in QuickSurveyForm (rendering SurveyEnableToggle) or SurveysDisabledBanner, or open the 'Surveys settings' LemonDialog via openSurveysSettingsDialog.
   - **Facts:** [`SV-UI-F-0150`](#sv-ui-f-0150), [`SV-UI-F-0152`](#sv-ui-f-0152), [`SV-UI-F-0623`](#sv-ui-f-0623), [`SV-UI-F-0624`](#sv-ui-f-0624)

2. Update currentTeam.surveys_opt_in via SurveyEnableToggle or persist sanitized default appearance to currentTeam.survey_config.appearance via SurveyDefaultAppearance (gated by globalSurveyAppearanceConfigAvailable) using teamLogic.updateCurrentTeam, which triggers the Team post_save remote config pipeline.
   - **Facts:** [`SV-UI-F-0601`](#sv-ui-f-0601), [`SV-UI-F-0602`](#sv-ui-f-0602), [`SV-UI-F-0604`](#sv-ui-f-0604), [`SV-UI-F-0610`](#sv-ui-f-0610), [`SV-UI-F-0611`](#sv-ui-f-0611), [`SV-UI-F-0612`](#sv-ui-f-0612), [`SV-UI-F-0617`](#sv-ui-f-0617)

#### <a id="sv-mcp-pr-05"></a>`SV-MCP-PR-05` (`MCP`): Detect Surveys setup status and create a first survey via the Survey Wizard or the PostHog AI create_survey tool.

- **Entry:** `surveysEmptyState and NewSurveyActions`

1. Evaluate non-skippable surveysEmptyState via surveysSetupLogic, which reads currentProjectId from projectLogic and queries at most 1 non-archived and 1 archived survey via surveysList to determine whether any live or archived survey exists.
   - **Facts:** [`SV-UI-F-3126`](#sv-ui-f-3126), [`SV-UI-F-3130`](#sv-ui-f-3130), [`SV-UI-F-3131`](#sv-ui-f-3131), [`SV-UI-F-3133`](#sv-ui-f-3133), [`SV-UI-F-3134`](#sv-ui-f-3134), [`SV-UI-F-3135`](#sv-ui-f-3135)

2. Gate creation actions in NewSurveyActions behind AccessControlAction requiring AccessControlLevel.Editor on AccessControlResourceType.Survey; clicking 'Create your first survey' dispatches teamLogic.addProductIntent (ProductKey.SURVEYS, ProductIntentContext.SURVEY_ADD_NEW) and navigates to urls.surveyWizard().
   - **Facts:** [`SV-UI-F-3116`](#sv-ui-f-3116), [`SV-UI-F-3117`](#sv-ui-f-3117), [`SV-UI-F-3118`](#sv-ui-f-3118)

3. When isMaxAvailable is true, register the 'create_survey' Max AI tool (disabling 'Create with AI' when openMax is unavailable); clicking 'Create with AI' opens Max and invokes surveysLogic.handleMaxSurveyCreated with SURVEY_CREATED_SOURCE.SURVEY_EMPTY_STATE when the tool callback fires.
   - **Facts:** [`SV-UI-F-3113`](#sv-ui-f-3113), [`SV-UI-F-3114`](#sv-ui-f-3114), [`SV-UI-F-3115`](#sv-ui-f-3115), [`SV-UI-F-3120`](#sv-ui-f-3120), [`SV-UI-F-3121`](#sv-ui-f-3121)

4. Emit survey lifecycle and theme analytics events via reportSurveyCreated (defaulting is_duplicate to false and creation_source to 'full_editor'), reportSurveyEdited (defaulting recurring iteration count and interval to 0), and site/theme reporting helpers.
   - **Facts:** [`SV-UI-F-3138`](#sv-ui-f-3138), [`SV-UI-F-3139`](#sv-ui-f-3139), [`SV-UI-F-3142`](#sv-ui-f-3142), [`SV-UI-F-3143`](#sv-ui-f-3143), [`SV-UI-F-3145`](#sv-ui-f-3145), [`SV-UI-F-3146`](#sv-ui-f-3146), [`SV-UI-F-3147`](#sv-ui-f-3147), [`SV-UI-F-3148`](#sv-ui-f-3148), [`SV-UI-F-3149`](#sv-ui-f-3149), [`SV-UI-F-3150`](#sv-ui-f-3150)

---

### <a id="prereq-sv-0-3"></a>`SV-0.3`: Authenticate with the Surveys API and MCP server

- **Codes:** `SV-API-P-01`, `SV-MCP-P-02`
- **Requires:** [`SV-0.1`](#prereq-sv-0-1)
- **Summary:** Configure personal API keys, project API tokens, and MCP scopes (including survey:read and query:read) for authenticated REST API, public delivery, and MCP access.

#### <a id="sv-api-pr-03"></a>`SV-API-PR-03` (`API`): List, filter, and read surveys, stats, responses, site snapshots, question labels, and activity logs under authenticated DRF access control.

- **Entry:** `GET /api/projects/{project_id}/surveys/ and read detail/list actions on SurveyViewSet`

1. Query Survey records with select_related for linked_flag, linked_insight, targeting_flag, and internal_targeting_flag; apply SurveyFilterSet filters (ids, created_by, linked_flag_id, status, archived, type), reject search terms exceeding MAX_SEARCH_LENGTH, and serialize via SurveySerializer.
   - **Facts:** [`SV-API-F-0111`](#sv-api-f-0111), [`SV-API-F-0114`](#sv-api-f-0114), [`SV-API-F-0291`](#sv-api-f-0291), [`SV-API-F-0294`](#sv-api-f-0294), [`SV-API-F-0299`](#sv-api-f-0299)

2. Handle read-only survey sub-endpoints with input validation and error mapping: /responses_count/, /stats/ (raising NotFound when missing and converting ValueError to ValidationError), /responses/ (limit 1..500, offset >= 0, requiring question_id with score_lte/score_gte), /site_snapshot/ (device and URL target defaulting match_type to ICONTAINS), /question_labels/ (chunked in 200-row batches), and paginated /activity/ logs.
   - **Facts:** [`SV-API-F-0062`](#sv-api-f-0062), [`SV-API-F-0063`](#sv-api-f-0063), [`SV-API-F-0064`](#sv-api-f-0064), [`SV-API-F-0066`](#sv-api-f-0066), [`SV-API-F-0107`](#sv-api-f-0107), [`SV-API-F-0108`](#sv-api-f-0108), [`SV-API-F-0110`](#sv-api-f-0110), [`SV-API-F-0297`](#sv-api-f-0297), [`SV-API-F-0318`](#sv-api-f-0318), [`SV-API-F-0321`](#sv-api-f-0321), [`SV-API-F-0323`](#sv-api-f-0323), [`SV-API-F-0324`](#sv-api-f-0324), [`SV-API-F-0339`](#sv-api-f-0339), [`SV-API-F-0341`](#sv-api-f-0341), [`SV-API-F-0346`](#sv-api-f-0346), [`SV-API-F-0347`](#sv-api-f-0347), [`SV-API-F-0353`](#sv-api-f-0353), [`SV-API-F-0359`](#sv-api-f-0359), [`SV-API-F-0360`](#sv-api-f-0360), [`SV-API-F-0363`](#sv-api-f-0363), [`SV-API-F-0364`](#sv-api-f-0364), [`SV-API-F-0366`](#sv-api-f-0366), [`SV-API-F-0367`](#sv-api-f-0367), [`SV-API-F-0368`](#sv-api-f-0368)

3. Enforce DRF authentication, team/org scoping, AccessControlViewSetMixin (scope_object = 'survey'), user_access_control queryset filtering, and per-action required_scopes (survey:read, query:read, session_recording:read, export:read, activity_log:read) via the platform auth layer.
   - **Facts:** [`SV-API-F-0292`](#sv-api-f-0292), [`SV-API-F-0293`](#sv-api-f-0293), [`SV-API-F-0317`](#sv-api-f-0317), [`SV-API-F-0322`](#sv-api-f-0322), [`SV-API-F-0340`](#sv-api-f-0340), [`SV-API-F-0343`](#sv-api-f-0343), [`SV-API-F-0345`](#sv-api-f-0345), [`SV-API-F-0351`](#sv-api-f-0351), [`SV-API-F-0352`](#sv-api-f-0352), [`SV-API-F-0358`](#sv-api-f-0358), [`SV-API-F-0362`](#sv-api-f-0362), [`SV-API-F-0365`](#sv-api-f-0365)

#### <a id="sv-api-pr-08"></a>`SV-API-PR-08` (`API`): Extract and validate the project API token on public Rust /api/surveys requests and map missing or malformed tokens to HTTP 400/401 errors.

- **Entry:** `HTTP GET / POST /api/surveys`

1. Extract a non-empty token (or fallback api_key) from query parameters in extract_token, or for POST requests without a query token, parse the URL-encoded form body for token or api_key.
   - **Facts:** [`SV-API-F-0860`](#sv-api-f-0860), [`SV-API-F-0861`](#sv-api-f-0861)

2. Return 401 Unauthorized with "Token not provided" if no non-empty token is supplied; otherwise validate via Token::parse and map TokenError::Empty to 401 Unauthorized and TokenError::TooLong or TokenError::InvalidCharacters to 400 Bad Request.
   - **Facts:** [`SV-API-F-0862`](#sv-api-f-0862), [`SV-API-F-0863`](#sv-api-f-0863), [`SV-API-F-0864`](#sv-api-f-0864), [`SV-API-F-0865`](#sv-api-f-0865)

3. Complete project token authentication at surveys_endpoint, returning any 400 or 401 token error response on failure.
   - **Facts:** [`SV-API-F-0855`](#sv-api-f-0855)

#### <a id="sv-mcp-pr-06"></a>`SV-MCP-PR-06` (`MCP`): Create, list, get, update, launch, stop, and soft-archive surveys via PostHog MCP tools (survey-create, survey-update, survey-delete, survey-get, surveys-get-all).

- **Entry:** `products/surveys/mcp/tools.yaml#survey-create`

1. Call survey-create, survey-update, survey-delete, survey-get, or surveys-get-all with survey:read/survey:write scopes.
   - **Facts:** [`SV-MCP-F-0024`](#sv-mcp-f-0024), [`SV-MCP-F-0025`](#sv-mcp-f-0025), [`SV-MCP-F-0026`](#sv-mcp-f-0026), [`SV-MCP-F-0027`](#sv-mcp-f-0027), [`SV-MCP-F-0028`](#sv-mcp-f-0028), [`SV-MCP-F-0029`](#sv-mcp-f-0029), [`SV-MCP-F-0030`](#sv-mcp-f-0030), [`SV-MCP-F-0031`](#sv-mcp-f-0031), [`SV-MCP-F-0032`](#sv-mcp-f-0032), [`SV-MCP-F-0033`](#sv-mcp-f-0033), [`SV-MCP-F-0034`](#sv-mcp-f-0034), [`SV-MCP-F-0035`](#sv-mcp-f-0035), [`SV-MCP-F-0036`](#sv-mcp-f-0036), [`SV-MCP-F-0037`](#sv-mcp-f-0037), [`SV-MCP-F-0038`](#sv-mcp-f-0038), [`SV-MCP-F-0039`](#sv-mcp-f-0039), [`SV-MCP-F-0040`](#sv-mcp-f-0040), [`SV-MCP-F-0041`](#sv-mcp-f-0041), [`SV-MCP-F-0042`](#sv-mcp-f-0042), [`SV-MCP-F-0043`](#sv-mcp-f-0043)

#### <a id="sv-mcp-pr-07"></a>`SV-MCP-PR-07` (`MCP`): Query global or per-survey statistics and list individual responses via PostHog MCP tools (surveys-global-stats, survey-stats, surveys-responses-list requiring survey:read + query:read).

- **Entry:** `products/surveys/mcp/tools.yaml#surveys-responses-list`

1. Call surveys-global-stats, survey-stats, or surveys-responses-list with survey:read and query:read scopes.
   - **Facts:** [`SV-MCP-F-0005`](#sv-mcp-f-0005), [`SV-MCP-F-0028`](#sv-mcp-f-0028), [`SV-MCP-F-0034`](#sv-mcp-f-0034), [`SV-MCP-F-0035`](#sv-mcp-f-0035), [`SV-MCP-F-0036`](#sv-mcp-f-0036), [`SV-MCP-F-0056`](#sv-mcp-f-0056), [`SV-MCP-F-0079`](#sv-mcp-f-0079), [`SV-MCP-F-0097`](#sv-mcp-f-0097), [`SV-MCP-F-0100`](#sv-mcp-f-0100), [`SV-MCP-F-0101`](#sv-mcp-f-0101), [`SV-MCP-F-0102`](#sv-mcp-f-0102), [`SV-MCP-F-0103`](#sv-mcp-f-0103), [`SV-MCP-F-0104`](#sv-mcp-f-0104), [`SV-MCP-F-0105`](#sv-mcp-f-0105), [`SV-MCP-F-0106`](#sv-mcp-f-0106), [`SV-MCP-F-0107`](#sv-mcp-f-0107), [`SV-MCP-F-0112`](#sv-mcp-f-0112), [`SV-MCP-F-0113`](#sv-mcp-f-0113), [`SV-MCP-F-0114`](#sv-mcp-f-0114), [`SV-MCP-F-0117`](#sv-mcp-f-0117)

---

### <a id="prereq-sv-0-2"></a>`SV-0.2`: Install and initialize the SDK for surveys

- **Codes:** `SV-WEB-P-01`, `SV-RN-P-01`, `SV-IOS-P-01`, `SV-AND-P-01`, `SV-FLT-P-01`
- **Requires:** [`SV-0.1`](#prereq-sv-0-1)
- **Summary:** Install and initialize the PostHog SDK on web or mobile so surveys load from remote config, mount platform UI providers, and stay synced with targeting flags.

#### <a id="sv-web-pr-01"></a>`SV-WEB-PR-01` (`WEB`): Initialize the browser surveys extension on SDK startup, evaluate remote config and enablement guards, and lazily load the external surveys renderer bundle.

- **Entry:** `createSurveys / PostHogSurveys.setup`

1. Resolve the analytics extension and initialize client.kv storage during setup (aborting early if disposed while awaiting), and subscribe to remote config updates, client reset, analytics before-teardown, and window pagehide events.
   - **Facts:** [`SV-WEB-F-0208`](#sv-web-f-0208), [`SV-WEB-F-0210`](#sv-web-f-0210), [`SV-WEB-F-0495`](#sv-web-f-0495), [`SV-WEB-F-0496`](#sv-web-f-0496), [`SV-WEB-F-0497`](#sv-web-f-0497), [`SV-WEB-F-0501`](#sv-web-f-0501), [`SV-WEB-F-0502`](#sv-web-f-0502), [`SV-WEB-F-0522`](#sv-web-f-0522), [`SV-WEB-F-0523`](#sv-web-f-0523)

2. Process remote config updates in PostHogSurveys.onRemoteConfig: ignore updates when config.disableSurveys (config.disable_surveys) is true; on a non-ok result, set _surveyInitializationFailed = true, log a warning, and notify active-matching-survey callbacks with an error.
   - **Facts:** [`SV-WEB-F-0214`](#sv-web-f-0214), [`SV-WEB-F-0215`](#sv-web-f-0215), [`SV-WEB-F-0529`](#sv-web-f-0529)

3. Guard lazy loading in PostHogSurveys.loadIfEnabled: skip if disableSurveys is true, if opted out in cookielessMode, or if _isSurveysEnabled is undefined and advancedEnableSurveys is falsy; invoke _handleSurveyLoadError if extensions or loadExternalDependency are unavailable.
   - **Facts:** [`SV-WEB-F-0223`](#sv-web-f-0223), [`SV-WEB-F-0224`](#sv-web-f-0224), [`SV-WEB-F-0225`](#sv-web-f-0225), [`SV-WEB-F-0226`](#sv-web-f-0226), [`SV-WEB-F-0228`](#sv-web-f-0228), [`SV-WEB-F-0235`](#sv-web-f-0235), [`SV-WEB-F-0530`](#sv-web-f-0530), [`SV-WEB-F-0531`](#sv-web-f-0531)

4. Lazily load the external 'surveys' extension bundle via loadExternalDependency, completing initialization with cached SURVEYS from client.kv on success or reporting load/initialization errors via _handleSurveyLoadError.
   - **Facts:** [`SV-WEB-F-0229`](#sv-web-f-0229), [`SV-WEB-F-0230`](#sv-web-f-0230), [`SV-WEB-F-0234`](#sv-web-f-0234), [`SV-WEB-F-0540`](#sv-web-f-0540)

#### <a id="sv-rn-pr-01"></a>`SV-RN-PR-01` (`RN`): Mount PostHogSurveyProvider (passing client prop if outside PostHogProvider), fetch active surveys, and display React Native SurveyModal.

- **Entry:** `packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider`

1. Wrap the app in PostHogSurveyProvider and render matched surveys in SurveyModal.
   - **Facts:** [`SV-RN-F-0001`](#sv-rn-f-0001), [`SV-RN-F-0002`](#sv-rn-f-0002), [`SV-RN-F-0003`](#sv-rn-f-0003), [`SV-RN-F-0004`](#sv-rn-f-0004), [`SV-RN-F-0005`](#sv-rn-f-0005), [`SV-RN-F-0006`](#sv-rn-f-0006), [`SV-RN-F-0007`](#sv-rn-f-0007), [`SV-RN-F-0008`](#sv-rn-f-0008), [`SV-RN-F-0009`](#sv-rn-f-0009), [`SV-RN-F-0010`](#sv-rn-f-0010), [`SV-RN-F-0011`](#sv-rn-f-0011), [`SV-RN-F-0012`](#sv-rn-f-0012), [`SV-RN-F-0013`](#sv-rn-f-0013), [`SV-RN-F-0014`](#sv-rn-f-0014), [`SV-RN-F-0015`](#sv-rn-f-0015), [`SV-RN-F-0016`](#sv-rn-f-0016), [`SV-RN-F-0017`](#sv-rn-f-0017), [`SV-RN-F-0018`](#sv-rn-f-0018), [`SV-RN-F-0019`](#sv-rn-f-0019), [`SV-RN-F-0020`](#sv-rn-f-0020)

#### <a id="sv-ios-pr-01"></a>`SV-IOS-PR-01` (`IOS`): Install the iOS survey integration, decode surveys from remote config, reconcile stored progress, and refresh targeting feature flags.

- **Entry:** `PostHogSurveyIntegration.install(_:) and remoteConfig.onRemoteConfigLoaded`

1. Install the survey integration once via integrationInstallState, bind the PostHogSDK reference, initialize SurveyProgressStore when storage is available, and call start() to subscribe to person-property flag changes, captured events, a 5-second throttled run-loop opportunity publisher, and app-did-become-active events.
   - **Facts:** [`SV-IOS-F-0093`](#sv-ios-f-0093), [`SV-IOS-F-0095`](#sv-ios-f-0095), [`SV-IOS-F-0096`](#sv-ios-f-0096), [`SV-IOS-F-0097`](#sv-ios-f-0097), [`SV-IOS-F-0098`](#sv-ios-f-0098)

2. When loading surveys via getSurveys or onRemoteConfigLoaded, return early if remoteConfig is nil or invoke the callback with an empty array when surveys are disabled; otherwise acquire surveyRefreshProcessingLock and start a new refresh generation.
   - **Facts:** [`SV-IOS-F-0106`](#sv-ios-f-0106), [`SV-IOS-F-0107`](#sv-ios-f-0107), [`SV-IOS-F-0110`](#sv-ios-f-0110), [`SV-IOS-F-0365`](#sv-ios-f-0365)

3. Decode remoteConfig["surveys"] with PostHogApi.jsonDecoder—returning empty if missing or malformed, logging and skipping invalid entries while preserving unknown survey, question, branching, and schedule types—and abort caching if beforeCacheUpdate returns false.
   - **Facts:** [`SV-IOS-F-0361`](#sv-ios-f-0361), [`SV-IOS-F-0362`](#sv-ios-f-0362), [`SV-IOS-F-0359`](#sv-ios-f-0359), [`SV-IOS-F-0025`](#sv-ios-f-0025), [`SV-IOS-F-0026`](#sv-ios-f-0026), [`SV-IOS-F-0042`](#sv-ios-f-0042), [`SV-IOS-F-0046`](#sv-ios-f-0046), [`SV-IOS-F-0036`](#sv-ios-f-0036)

4. Reconcile .surveyProgress storage to retain only entries for active surveys, aborting without writing if the survey state generation is no longer current.
   - **Facts:** [`SV-IOS-F-0259`](#sv-ios-f-0259), [`SV-IOS-F-0260`](#sv-ios-f-0260), [`SV-IOS-F-0261`](#sv-ios-f-0261), [`SV-IOS-F-0262`](#sv-ios-f-0262)

5. If surveys require feature flags, call reloadFeatureFlagsForSurvey (or react to onFeatureFlagsLoaded when not awaiting a refresh); on completion for the matching generation, clear the awaiting state, mark surveyFeatureFlagsUnavailable when flags are nil, and invoke showNextSurvey().
   - **Facts:** [`SV-IOS-F-0368`](#sv-ios-f-0368), [`SV-IOS-F-0366`](#sv-ios-f-0366), [`SV-IOS-F-0111`](#sv-ios-f-0111), [`SV-IOS-F-0356`](#sv-ios-f-0356)

#### <a id="sv-and-pr-01"></a>`SV-AND-PR-01` (`AND`): Install the Android surveys integration, match loaded or event-activated surveys, present the Jetpack Compose survey sheet, and emit the survey shown event.

- **Entry:** `PostHogSurveysIntegration.install, onSurveysLoaded, or onEvent`

1. On install, mark PostHogSurveysIntegration started, resolve the UI delegate (defaulting to the no-op PostHogSurveysDefaultDelegate unless PostHogSurveysComposeDelegate is configured, which declares supportsSurveyResume=true and registers ActivityProvider with Application.registerActivityLifecycleCallbacks), and attempt to show the next survey.
   - **Facts:** [`SV-AND-F-0171`](#sv-and-f-0171), [`SV-AND-F-0325`](#sv-and-f-0325), [`SV-AND-F-0329`](#sv-and-f-0329), [`SV-AND-F-0001`](#sv-and-f-0001), [`SV-AND-F-0002`](#sv-and-f-0002), [`SV-AND-F-0005`](#sv-and-f-0005)

2. When surveys load via onSurveysLoaded, cache them, reconcile SurveyProgressStore (reading PostHogPreferences.SURVEY_PROGRESS keyed by '{survey.id}/{currentIteration ?: 0}' and discarding invalid saved entries), and rebuild the event-to-surveys map; when SDK events arrive via onEvent, evaluate property filters (defaulting to true when empty and rejecting missing properties or unknown operators) if canRenderSurveys() is true.
   - **Facts:** [`SV-AND-F-0173`](#sv-and-f-0173), [`SV-AND-F-0281`](#sv-and-f-0281), [`SV-AND-F-0286`](#sv-and-f-0286), [`SV-AND-F-0290`](#sv-and-f-0290), [`SV-AND-F-0266`](#sv-and-f-0266), [`SV-AND-F-0267`](#sv-and-f-0267), [`SV-AND-F-0274`](#sv-and-f-0274), [`SV-AND-F-0275`](#sv-and-f-0275)

3. Guard survey eligibility and display in getActiveMatchingSurveys (empty when config.surveys is disabled or postHog is uninstalled) and showNextSurvey/showSurvey (requiring integration started, canRenderSurveys(), cachePreferences available, no activeSurvey, and no pendingPresentation), checking seen state in PostHogPreferences.SURVEY_SEEN and last-seen date in PostHogPreferences.LAST_SEEN_SURVEY_DATE before incrementing presentationGeneration and setting pendingPresentation under activeSurveyLock.
   - **Facts:** [`SV-AND-F-0175`](#sv-and-f-0175), [`SV-AND-F-0176`](#sv-and-f-0176), [`SV-AND-F-0241`](#sv-and-f-0241), [`SV-AND-F-0168`](#sv-and-f-0168), [`SV-AND-F-0184`](#sv-and-f-0184), [`SV-AND-F-0188`](#sv-and-f-0188), [`SV-AND-F-0189`](#sv-and-f-0189), [`SV-AND-F-0257`](#sv-and-f-0257), [`SV-AND-F-0259`](#sv-and-f-0259), [`SV-AND-F-0260`](#sv-and-f-0260), [`SV-AND-F-0264`](#sv-and-f-0264), [`SV-AND-F-0265`](#sv-and-f-0265)

4. On the main thread via renderIfCurrent, verify isStarted and matching resetGeneration/presentationGeneration, then invoke delegate.renderSurvey -> PostHogSurveyHost.show, which dismisses any prior active survey, schedules presentation after surveyPopupDelaySeconds (default 0.0, negative clamped to 0.0), and opens a transparent, non-cancelable ComponentDialog hosting SurveyComposition and SurveySheet (deferring via awaitingForeground if the target Activity is null or finishing).
   - **Facts:** [`SV-AND-F-0193`](#sv-and-f-0193), [`SV-AND-F-0187`](#sv-and-f-0187), [`SV-AND-F-0004`](#sv-and-f-0004), [`SV-AND-F-0009`](#sv-and-f-0009), [`SV-AND-F-0010`](#sv-and-f-0010), [`SV-AND-F-0011`](#sv-and-f-0011), [`SV-AND-F-0012`](#sv-and-f-0012), [`SV-AND-F-0015`](#sv-and-f-0015), [`SV-AND-F-0019`](#sv-and-f-0019), [`SV-AND-F-0027`](#sv-and-f-0027), [`SV-AND-F-0031`](#sv-and-f-0031), [`SV-AND-F-0125`](#sv-and-f-0125), [`SV-AND-F-0126`](#sv-and-f-0126)

5. Once displayed, PostHogSurveyHost.reportShownOnce invokes onShownCallback at most once per presentation; surveyShownCallback verifies isCurrentContext, activates the survey, removes it from eventActivatedSurveys, persists the ISO-8601 last-seen survey date to cachePreferences, and captures the 'survey shown' event via postHog.capture.
   - **Facts:** [`SV-AND-F-0026`](#sv-and-f-0026), [`SV-AND-F-0214`](#sv-and-f-0214), [`SV-AND-F-0215`](#sv-and-f-0215), [`SV-AND-F-0213`](#sv-and-f-0213), [`SV-AND-F-0170`](#sv-and-f-0170), [`SV-AND-F-0166`](#sv-and-f-0166), [`SV-AND-F-0169`](#sv-and-f-0169)

#### <a id="sv-flt-pr-01"></a>`SV-FLT-PR-01` (`FLT`): Schedule a survey with optional popup delay, wait for a mounted navigator context, and present the modal bottom sheet with its initial intro or question screen.

- **Entry:** `SurveyService.showSurvey`

1. Guard against concurrent survey displays on the singleton SurveyService by logging a debug message and returning early if _isShowingSurvey is already true.
   - **Facts:** [`SV-FLT-F-0089`](#sv-flt-f-0089), [`SV-FLT-F-0091`](#sv-flt-f-0091)

2. Compute the popup delay via surveyPopupDelay (returning Duration.zero for null, NaN, or <= 0, capping at 1 day, and converting positive seconds to rounded milliseconds); if positive, start a Timer via _waitForPopupDelay (cancelling any prior pending show) and abort if the wait is cancelled.
   - **Facts:** [`SV-FLT-F-0086`](#sv-flt-f-0086), [`SV-FLT-F-0087`](#sv-flt-f-0087), [`SV-FLT-F-0088`](#sv-flt-f-0088), [`SV-FLT-F-0092`](#sv-flt-f-0092), [`SV-FLT-F-0096`](#sv-flt-f-0096), [`SV-FLT-F-0097`](#sv-flt-f-0097)

3. If PosthogObserver.currentContext is null, wait in a loop until onContextAvailable() completes the pending show (only when no popup delay timer is active), aborting if cancelled, another survey starts showing, or the obtained BuildContext is no longer mounted.
   - **Facts:** [`SV-FLT-F-0090`](#sv-flt-f-0090), [`SV-FLT-F-0093`](#sv-flt-f-0093), [`SV-FLT-F-0094`](#sv-flt-f-0094)

4. Present a non-dismissible, scroll-controlled modal bottom sheet on the root navigator via _showSurveyWithNavigator, building a SurveyBottomSheet themed with SurveyAppearance.fromPostHog(survey.appearance).
   - **Facts:** [`SV-FLT-F-0095`](#sv-flt-f-0095), [`SV-FLT-F-0098`](#sv-flt-f-0098), [`SV-FLT-F-0103`](#sv-flt-f-0103)

5. Initialize _SurveyBottomSheetState at survey.initialQuestionIndex, invoke onShown(survey) in initState, pad for keyboard viewInsets, optionally wrap with fontFamily theming, and display IntroMessage first only when initialQuestionIndex is 0, displayIntroScreen is true, and either intro header or description is non-empty.
   - **Facts:** [`SV-FLT-F-0139`](#sv-flt-f-0139), [`SV-FLT-F-0140`](#sv-flt-f-0140), [`SV-FLT-F-0141`](#sv-flt-f-0141), [`SV-FLT-F-0193`](#sv-flt-f-0193), [`SV-FLT-F-0194`](#sv-flt-f-0194), [`SV-FLT-F-0195`](#sv-flt-f-0195), [`SV-FLT-F-0209`](#sv-flt-f-0209), [`SV-FLT-F-0210`](#sv-flt-f-0210), [`SV-FLT-F-0212`](#sv-flt-f-0212), [`SV-FLT-F-0215`](#sv-flt-f-0215)

---

### <a id="prereq-sv-3-1"></a>`SV-3.1`: Target surveys by URL and estimate reachable audience

- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-UI-PR-01`](#sv-ui-pr-01)
- **Summary:** Restrict web surveys to specific URLs or URL patterns, estimate reachable audience size from recent pageviews, and support URL prefills on web popovers. (Unsupported tabs: MCP, RN, IOS, AND, FLT)

#### <a id="sv-ui-pr-03"></a>`SV-UI-PR-03` (`UI`): Configure survey display triggers, completion limits, widget styling, and linked feature flag targeting in SurveyEdit.

- **Entry:** `SurveyEdit targeting, activation triggers, and customization sections`

1. Configure activation and cancellation triggers: bind event triggers via SurveyEventSelector/SurveyEventTrigger (property filtering scoped to posthog-js 1.268+ or posthog-react-native 4.15+), cancel-on-event triggers via SurveyCancelEventTrigger (web surveys on posthog-js 1.299+), and action triggers via SurveyActionTrigger when FEATURE_FLAGS.SURVEYS_ACTIONS is enabled.
   - **Facts:** [`SV-UI-F-0189`](#sv-ui-f-0189), [`SV-UI-F-0194`](#sv-ui-f-0194), [`SV-UI-F-0301`](#sv-ui-f-0301), [`SV-UI-F-0469`](#sv-ui-f-0469), [`SV-UI-F-0470`](#sv-ui-f-0470), [`SV-UI-F-0478`](#sv-ui-f-0478), [`SV-UI-F-0479`](#sv-ui-f-0479), [`SV-UI-F-0481`](#sv-ui-f-0481)

2. Configure response limits and repeat schedules in SurveyCompletionConditions and SurveyIterationOptions, and customize widget button position in SurveyWidgetCustomization (paywalled behind AvailableFeature.SURVEYS_STYLING, disabled when surveysStylingAvailable is false, and requiring posthog-js 1.294.0+).
   - **Facts:** [`SV-UI-F-0216`](#sv-ui-f-0216), [`SV-UI-F-0217`](#sv-ui-f-0217), [`SV-UI-F-0519`](#sv-ui-f-0519), [`SV-UI-F-0520`](#sv-ui-f-0520), [`SV-UI-F-0861`](#sv-ui-f-0861), [`SV-UI-F-0872`](#sv-ui-f-0872), [`SV-UI-F-0876`](#sv-ui-f-0876), [`SV-UI-F-0877`](#sv-ui-f-0877)

3. Select or hydrate a linked feature flag in SurveyEdit: when survey.linked_flag_id is set without a loaded linked_flag object, fetch it via api.featureFlags.get(survey.linked_flag_id) through the authenticated project API (clearing linked_flag_id to null and removing linkedFlagVariant from conditions if the fetch fails).
   - **Facts:** [`SV-UI-F-0292`](#sv-ui-f-0292), [`SV-UI-F-0353`](#sv-ui-f-0353)

#### <a id="sv-ui-pr-04"></a>`SV-UI-PR-04` (`UI`): Estimate the reachable audience size for URL-targeted survey conditions from recent pageview events.

- **Entry:** `SurveyUrlAudienceEstimate within survey targeting configuration`

1. Read the active survey URL targeting conditions from surveyLogic in SurveyUrlAudienceEstimate.
   - **Facts:** [`SV-UI-F-1125`](#sv-ui-f-1125)

2. Execute a HogQL query against the events table for unique person_id counts of $pageview events matching $current_url over the last URL_AUDIENCE_ESTIMATE_DAYS.
   - **Facts:** [`SV-UI-F-1128`](#sv-ui-f-1128)

#### <a id="sv-web-pr-03"></a>`SV-WEB-PR-03` (`WEB`): Track event and action survey activations, evaluate display eligibility and URL prefills, render a popover survey after its delay, and emit the 'survey shown' event.

- **Entry:** `generateSurveys / SurveyManager.startAutomaticDisplay`

1. Register event- and action-based surveys in SurveyEventReceiver and ActionMatcher, subscribing to capture events and session changes, registering element selectors on autocapture, and persisting activated survey IDs, timestamps, and session IDs in client.kv (SURVEYS_ACTIVATED, SURVEYS_ACTIVATED_TIMESTAMPS, SURVEYS_ACTIVATED_SESSION).
   - **Facts:** [`SV-WEB-F-0001`](#sv-web-f-0001), [`SV-WEB-F-0004`](#sv-web-f-0004), [`SV-WEB-F-0005`](#sv-web-f-0005), [`SV-WEB-F-0027`](#sv-web-f-0027), [`SV-WEB-F-0031`](#sv-web-f-0031), [`SV-WEB-F-0038`](#sv-web-f-0038), [`SV-WEB-F-0040`](#sv-web-f-0040), [`SV-WEB-F-0041`](#sv-web-f-0041), [`SV-WEB-F-0043`](#sv-web-f-0043), [`SV-WEB-F-0044`](#sv-web-f-0044), [`SV-WEB-F-0045`](#sv-web-f-0045), [`SV-WEB-F-0046`](#sv-web-f-0046)

2. Start automatic display polling every 1000ms (unless document/window is missing, automaticDisplay is false, isSurveysEnabled is false, or the manager is disposed) and listen for matching-condition triggers ($pageview, survey events, PHSurveyShown, onFeatureFlags, and languagechange).
   - **Facts:** [`SV-WEB-F-0071`](#sv-web-f-0071), [`SV-WEB-F-0072`](#sv-web-f-0072), [`SV-WEB-F-0073`](#sv-web-f-0073), [`SV-WEB-F-0125`](#sv-web-f-0125), [`SV-WEB-F-0126`](#sv-web-f-0126), [`SV-WEB-F-0127`](#sv-web-f-0127), [`SV-WEB-F-0130`](#sv-web-f-0130), [`SV-WEB-F-0541`](#sv-web-f-0541), [`SV-WEB-F-0542`](#sv-web-f-0542), [`SV-WEB-F-0543`](#sv-web-f-0543), [`SV-WEB-F-0544`](#sv-web-f-0544), [`SV-WEB-F-0545`](#sv-web-f-0545), [`SV-WEB-F-0546`](#sv-web-f-0546), [`SV-WEB-F-0547`](#sv-web-f-0547)

3. Evaluate eligibility in SurveyManager: require client.canCapture, check getSurveySeen and hasWaitPeriodPassed (reading localStorage with error fallbacks), evaluate targeting feature flags without emitting flag call events, and parse/persist any URL prefill responses to localStorage via setInProgressSurveyState (capping branching traversal at survey.questions.length + 1).
   - **Facts:** [`SV-WEB-F-0092`](#sv-web-f-0092), [`SV-WEB-F-0094`](#sv-web-f-0094), [`SV-WEB-F-0096`](#sv-web-f-0096), [`SV-WEB-F-0099`](#sv-web-f-0099), [`SV-WEB-F-0107`](#sv-web-f-0107), [`SV-WEB-F-0108`](#sv-web-f-0108), [`SV-WEB-F-0109`](#sv-web-f-0109), [`SV-WEB-F-0385`](#sv-web-f-0385), [`SV-WEB-F-0401`](#sv-web-f-0401), [`SV-WEB-F-0402`](#sv-web-f-0402), [`SV-WEB-F-0451`](#sv-web-f-0451), [`SV-WEB-F-0452`](#sv-web-f-0452), [`SV-WEB-F-0454`](#sv-web-f-0454), [`SV-WEB-F-0455`](#sv-web-f-0455)

4. Schedule popover display after surveyPopupDelaySeconds (default 0) via window.setTimeout, re-verify client.canCapture and _shouldDisplaySurvey when the timer fires, sanitize survey HTML (XHTML namespace, stripping ARIA/data-* and non-allowlisted attributes), and render SurveyPopup while writing 'lastSeenSurveyDate' to localStorage.
   - **Facts:** [`SV-WEB-F-0065`](#sv-web-f-0065), [`SV-WEB-F-0075`](#sv-web-f-0075), [`SV-WEB-F-0079`](#sv-web-f-0079), [`SV-WEB-F-0081`](#sv-web-f-0081), [`SV-WEB-F-0083`](#sv-web-f-0083), [`SV-WEB-F-0137`](#sv-web-f-0137), [`SV-WEB-F-0148`](#sv-web-f-0148), [`SV-WEB-F-0157`](#sv-web-f-0157), [`SV-WEB-F-0378`](#sv-web-f-0378), [`SV-WEB-F-0379`](#sv-web-f-0379), [`SV-WEB-F-0380`](#sv-web-f-0380)

5. Capture the 'survey shown' event via posthog.client.capture with survey metadata, optional language, and sessionRecordingUrl (unless skipShownEvent is true), updating SurveyEventReceiver activation state (persisting non-repeating activations or consuming repeating ones).
   - **Facts:** [`SV-WEB-F-0035`](#sv-web-f-0035), [`SV-WEB-F-0048`](#sv-web-f-0048), [`SV-WEB-F-0049`](#sv-web-f-0049), [`SV-WEB-F-0150`](#sv-web-f-0150)

---

### <a id="prereq-sv-5-1"></a>`SV-5.1`: Fetch and cache active surveys on the client

- **Requires:** [`SV-0.1`](#prereq-sv-0-1), [`SV-0.2`](#prereq-sv-0-2), [`SV-UI-PR-01`](#sv-ui-pr-01), [`SV-3.1`](#prereq-sv-3-1)
- **Summary:** Fetch active survey definitions from PostHog with TTL caching, failure backoff, and local progress reconciliation for automatic popovers or custom UI integrations. (Unsupported tabs: AND)

#### <a id="sv-web-pr-02"></a>`SV-WEB-PR-02` (`WEB`): Fetch survey definitions from the PostHog API with 5-minute TTL caching and failure backoff, and deliver results or errors to survey callbacks.

- **Entry:** `PostHogSurveys.getSurveys / PostHogSurveys.onActiveMatchingSurveysChanged`

1. Guard getSurveys: if disableSurveys is enabled in config, log SURVEY_DISABLED and invoke the callback with an empty array; check cache freshness via SURVEYS_LOADED_AT against the 5-minute SURVEYS_CACHE_TTL_MS and refresh backoff against the 5-minute SURVEYS_REFRESH_BACKOFF_MS.
   - **Facts:** [`SV-WEB-F-0051`](#sv-web-f-0051), [`SV-WEB-F-0052`](#sv-web-f-0052), [`SV-WEB-F-0206`](#sv-web-f-0206), [`SV-WEB-F-0238`](#sv-web-f-0238), [`SV-WEB-F-0249`](#sv-web-f-0249)

2. When subscribing via onActiveMatchingSurveysChanged, immediately notify { isLoaded: false } if surveys are disabled, initialization failed, or no cached surveys exist after a failed refresh with no in-flight request; catch and log any synchronous or async user callback errors.
   - **Facts:** [`SV-WEB-F-0244`](#sv-web-f-0244), [`SV-WEB-F-0252`](#sv-web-f-0252), [`SV-WEB-F-0254`](#sv-web-f-0254), [`SV-WEB-F-0260`](#sv-web-f-0260), [`SV-WEB-F-0491`](#sv-web-f-0491)

3. Fetch surveys via GET /api/surveys/ authenticated with the public project token, sentAt query param, and configured requestTimeoutMs; on non-200 status, missing JSON, or response-processing exceptions, record _lastSurveyRefreshFailedAt, log the error or warning, and return { isLoaded: false, error }.
   - **Facts:** [`SV-WEB-F-0241`](#sv-web-f-0241), [`SV-WEB-F-0242`](#sv-web-f-0242), [`SV-WEB-F-0247`](#sv-web-f-0247)

#### <a id="sv-ios-pr-01"></a>`SV-IOS-PR-01` (`IOS`): Install the iOS survey integration, decode surveys from remote config, reconcile stored progress, and refresh targeting feature flags.

- **Entry:** `PostHogSurveyIntegration.install(_:) and remoteConfig.onRemoteConfigLoaded`

1. Install the survey integration once via integrationInstallState, bind the PostHogSDK reference, initialize SurveyProgressStore when storage is available, and call start() to subscribe to person-property flag changes, captured events, a 5-second throttled run-loop opportunity publisher, and app-did-become-active events.
   - **Facts:** [`SV-IOS-F-0093`](#sv-ios-f-0093), [`SV-IOS-F-0095`](#sv-ios-f-0095), [`SV-IOS-F-0096`](#sv-ios-f-0096), [`SV-IOS-F-0097`](#sv-ios-f-0097), [`SV-IOS-F-0098`](#sv-ios-f-0098)

2. When loading surveys via getSurveys or onRemoteConfigLoaded, return early if remoteConfig is nil or invoke the callback with an empty array when surveys are disabled; otherwise acquire surveyRefreshProcessingLock and start a new refresh generation.
   - **Facts:** [`SV-IOS-F-0106`](#sv-ios-f-0106), [`SV-IOS-F-0107`](#sv-ios-f-0107), [`SV-IOS-F-0110`](#sv-ios-f-0110), [`SV-IOS-F-0365`](#sv-ios-f-0365)

3. Decode remoteConfig["surveys"] with PostHogApi.jsonDecoder—returning empty if missing or malformed, logging and skipping invalid entries while preserving unknown survey, question, branching, and schedule types—and abort caching if beforeCacheUpdate returns false.
   - **Facts:** [`SV-IOS-F-0361`](#sv-ios-f-0361), [`SV-IOS-F-0362`](#sv-ios-f-0362), [`SV-IOS-F-0359`](#sv-ios-f-0359), [`SV-IOS-F-0025`](#sv-ios-f-0025), [`SV-IOS-F-0026`](#sv-ios-f-0026), [`SV-IOS-F-0042`](#sv-ios-f-0042), [`SV-IOS-F-0046`](#sv-ios-f-0046), [`SV-IOS-F-0036`](#sv-ios-f-0036)

4. Reconcile .surveyProgress storage to retain only entries for active surveys, aborting without writing if the survey state generation is no longer current.
   - **Facts:** [`SV-IOS-F-0259`](#sv-ios-f-0259), [`SV-IOS-F-0260`](#sv-ios-f-0260), [`SV-IOS-F-0261`](#sv-ios-f-0261), [`SV-IOS-F-0262`](#sv-ios-f-0262)

5. If surveys require feature flags, call reloadFeatureFlagsForSurvey (or react to onFeatureFlagsLoaded when not awaiting a refresh); on completion for the matching generation, clear the awaiting state, mark surveyFeatureFlagsUnavailable when flags are nil, and invoke showNextSurvey().
   - **Facts:** [`SV-IOS-F-0368`](#sv-ios-f-0368), [`SV-IOS-F-0366`](#sv-ios-f-0366), [`SV-IOS-F-0111`](#sv-ios-f-0111), [`SV-IOS-F-0356`](#sv-ios-f-0356)

#### <a id="sv-rn-pr-01"></a>`SV-RN-PR-01` (`RN`): Mount PostHogSurveyProvider (passing client prop if outside PostHogProvider), fetch active surveys, and display React Native SurveyModal.

- **Entry:** `packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider`

1. Wrap the app in PostHogSurveyProvider and render matched surveys in SurveyModal.
   - **Facts:** [`SV-RN-F-0001`](#sv-rn-f-0001), [`SV-RN-F-0002`](#sv-rn-f-0002), [`SV-RN-F-0003`](#sv-rn-f-0003), [`SV-RN-F-0004`](#sv-rn-f-0004), [`SV-RN-F-0005`](#sv-rn-f-0005), [`SV-RN-F-0006`](#sv-rn-f-0006), [`SV-RN-F-0007`](#sv-rn-f-0007), [`SV-RN-F-0008`](#sv-rn-f-0008), [`SV-RN-F-0009`](#sv-rn-f-0009), [`SV-RN-F-0010`](#sv-rn-f-0010), [`SV-RN-F-0011`](#sv-rn-f-0011), [`SV-RN-F-0012`](#sv-rn-f-0012), [`SV-RN-F-0013`](#sv-rn-f-0013), [`SV-RN-F-0014`](#sv-rn-f-0014), [`SV-RN-F-0015`](#sv-rn-f-0015), [`SV-RN-F-0016`](#sv-rn-f-0016), [`SV-RN-F-0017`](#sv-rn-f-0017), [`SV-RN-F-0018`](#sv-rn-f-0018), [`SV-RN-F-0019`](#sv-rn-f-0019), [`SV-RN-F-0020`](#sv-rn-f-0020)

---

## <a id="resolved-facts"></a>Resolved facts (849)

### <a id="sv-and-f-0001"></a>`SV-AND-F-0001`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/PostHogSurveysComposeDelegate.kt#PostHogSurveysComposeDelegate (L87-115)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Declares supportsSurveyResume as true on PostHogSurveysComposeDelegate

```
override val supportsSurveyResume: Boolean = true
```

### <a id="sv-and-f-0002"></a>`SV-AND-F-0002`

- **Surface:** `AND` | **Type:** `wiring` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/PostHogSurveysComposeDelegate.kt#PostHogSurveysComposeDelegate (L87-115)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Registers ActivityProvider with Application.registerActivityLifecycleCallbacks on initialization using context.applicationContext

```
application.registerActivityLifecycleCallbacks(activityProvider)
```

### <a id="sv-and-f-0004"></a>`SV-AND-F-0004`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/PostHogSurveysComposeDelegate.kt#PostHogSurveysComposeDelegate.renderSurvey (L98-110)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** renderSurvey delegates survey presentation to PostHogSurveyHost.show with the survey and onSurveyShown, onSurveyResponse, and onSurveyClosed callbacks

```
override fun renderSurvey(
```

### <a id="sv-and-f-0005"></a>`SV-AND-F-0005`

- **Surface:** `AND` | **Type:** `wiring` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ActivityProvider.kt#ActivityProvider (L16-68)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Registers as an Android Application.ActivityLifecycleCallbacks implementation to track foreground activity lifecycle events

```
internal class ActivityProvider : Application.ActivityLifecycleCallbacks {
```

### <a id="sv-and-f-0009"></a>`SV-AND-F-0009`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.show (L122-158)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Computes popup delay in milliseconds from surveyPopupDelaySeconds, defaulting to 0.0 and clamping negative values to 0.0.

```
((survey.appearance?.surveyPopupDelaySeconds ?: 0.0).coerceAtLeast(0.0) * 1000).toLong()
```

### <a id="sv-and-f-0010"></a>`SV-AND-F-0010`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.show (L122-158)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Aborts showing the survey on the main thread if renderCounter has advanced past the current render ID, superseding stale show requests.

```
if (render != renderCounter.get()) return@runOnMain
```

### <a id="sv-and-f-0011"></a>`SV-AND-F-0011`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.show (L122-158)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Dismisses any active survey and notifies its closed callback before starting the new survey presentation.

```
dismissInternal(notifyClosed = true)
```

### <a id="sv-and-f-0012"></a>`SV-AND-F-0012`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.show (L122-158)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Schedules presentation on mainHandler after delayMillis when delayMillis > 0L; otherwise presents immediately.

```
if (delayMillis > 0L) {
```

### <a id="sv-and-f-0015"></a>`SV-AND-F-0015`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.present (L167-245)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Sets awaitingForeground to true and defers presentation when the target activity is null or finishing.

```
if (activity == null || activity.isFinishing) {
```

### <a id="sv-and-f-0018"></a>`SV-AND-F-0018`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.present (L167-245)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Forwards survey responses to onResponseCallback only when the presentation is still current, returning null otherwise.

```
if (isCurrentPresentation(
```

### <a id="sv-and-f-0019"></a>`SV-AND-F-0019`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.present (L167-245)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Configures the survey ComponentDialog so it cannot be canceled by back press or outside touches.

```
setCanceledOnTouchOutside(false)
```

### <a id="sv-and-f-0025"></a>`SV-AND-F-0025`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.submitResponse (L271-276)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Invokes onResponseCallback and returns the next question only if the presentation is current, otherwise returns null.

```
): PostHogNextSurveyQuestion? = if (isCurrentPresentation(presentation)) onResponseCallback?.invoke(survey, index, response) else null
```

### <a id="sv-and-f-0026"></a>`SV-AND-F-0026`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.reportShownOnce (L278-283)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Guards against duplicate or stale shown events and invokes onShownCallback at most once per current presentation.

```
if (!isCurrentPresentation(presentation) || shownReported) return
```

### <a id="sv-and-f-0027"></a>`SV-AND-F-0027`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/PostHogSurveyHost.kt#PostHogSurveyHost.configureWindow (L290-299)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Sets the dialog window background to transparent, layout to MATCH_PARENT, and clears FLAG_DIM_BEHIND when window is non-null.

```
window.clearFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND)
```

### <a id="sv-and-f-0031"></a>`SV-AND-F-0031`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/SurveyComposition.kt#SurveyComposition (L23-100)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Wraps a ComposeView inside a FrameLayout using MATCH_PARENT width and height layout parameters.

```
addView(composeView, FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT, FrameLayout.LayoutParams.MATCH_PARENT))
```

### <a id="sv-and-f-0041"></a>`SV-AND-F-0041`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/theme/SurveyAppearance.kt#resolve (L79-160)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Defaults survey backgroundColor to DefaultBackgroundColor (0xFFF2F2F7) when missing or invalid

```
parseSurveyColorOrDefault(this?.backgroundColor, DefaultBackgroundColor)
```

### <a id="sv-and-f-0042"></a>`SV-AND-F-0042`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/theme/SurveyAppearance.kt#resolve (L79-160)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Defaults submitButtonColor to Color.Black when missing or invalid

```
parseSurveyColorOrDefault(this?.submitButtonColor, Color.Black)
```

### <a id="sv-and-f-0049"></a>`SV-AND-F-0049`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/theme/SurveyAppearance.kt#resolve (L79-160)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Defaults submitButtonText to "Submit" when null or blank

```
val submitButtonText = this?.submitButtonText?.takeIf { it.isNotBlank() } ?: "Submit"
```

### <a id="sv-and-f-0073"></a>`SV-AND-F-0073`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/ChoiceOptions.kt#ChoiceOptions (L47-83)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Preserves the computed choice displayOrder across recompositions and saved state keyed by question.id, options.size, question.hasOpenChoice, and question.shuffleOptions.

```
rememberSaveable(question.id, options.size, question.hasOpenChoice, question.shuffleOptions) {
```

### <a id="sv-and-f-0074"></a>`SV-AND-F-0074`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/ChoiceOptions.kt#ChoiceOptions (L47-83)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Treats only the last option in question.choices as the open-ended choice when question.hasOpenChoice is true.

```
val isOpenChoice = question.hasOpenChoice && index == options.lastIndex
```

### <a id="sv-and-f-0089"></a>`SV-AND-F-0089`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/ChoiceOptions.kt#surveyChoiceOrder (L176-189)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Excludes the last option index from shuffling when hasOpenChoice is true.

```
val regular = if (hasOpenChoice) indices.dropLast(1) else indices
```

### <a id="sv-and-f-0091"></a>`SV-AND-F-0091`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/ChoiceOptions.kt#surveyChoiceOrder (L176-189)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Appends the open-choice index at the end of the shuffled order when hasOpenChoice is true and options is non-empty.

```
if (hasOpenChoice && options.isNotEmpty()) shuffled.add(options.lastIndex)
```

### <a id="sv-and-f-0099"></a>`SV-AND-F-0099`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/EmojiRating.kt#emojisForScale (L123-135)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Maps a 2-point scale to ThumbsUp and ThumbsDown emojis.

```
2 -> listOf(PostHogEmoji.ThumbsUp, PostHogEmoji.ThumbsDown)
```

### <a id="sv-and-f-0100"></a>`SV-AND-F-0100`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/EmojiRating.kt#emojisForScale (L123-135)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Maps a 3-point scale to Dissatisfied, Neutral, and Satisfied emojis.

```
3 -> listOf(PostHogEmoji.Dissatisfied, PostHogEmoji.Neutral, PostHogEmoji.Satisfied)
```

### <a id="sv-and-f-0101"></a>`SV-AND-F-0101`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/EmojiRating.kt#emojisForScale (L123-135)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Falls back to the 5-emoji scale (VeryDissatisfied, Dissatisfied, Neutral, Satisfied, VerySatisfied) for any scale other than 2 or 3.

```
PostHogEmoji.VeryDissatisfied,
```

### <a id="sv-and-f-0112"></a>`SV-AND-F-0112`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/LinkQuestion.kt#openLink (L33-45)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns early without launching an activity when url is null or blank.

```
if (url.isNullOrBlank()) return
```

### <a id="sv-and-f-0113"></a>`SV-AND-F-0113`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/LinkQuestion.kt#openLink (L33-45)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Starts an Android ACTION_VIEW activity with FLAG_ACTIVITY_NEW_TASK for url.toUri().

```
Intent(Intent.ACTION_VIEW, url.toUri()).apply {
```

### <a id="sv-and-f-0114"></a>`SV-AND-F-0114`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/LinkQuestion.kt#openLink (L33-45)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Silently catches and ignores any exception thrown while creating or starting the view intent.

```
runCatching {
```

### <a id="sv-and-f-0115"></a>`SV-AND-F-0115`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/NumberRating.kt#NumberRating (L32-118)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Renders a horizontal row of clickable rating segments spanning the inclusive range from question.scaleLowerBound to question.scaleUpperBound.

```
val values = (question.scaleLowerBound..question.scaleUpperBound).toList()
```

### <a id="sv-and-f-0122"></a>`SV-AND-F-0122`

- **Surface:** `AND` | **Type:** `control` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/OpenText.kt#OpenText (L44-78)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Renders a 150.dp-tall BasicTextField input bound to value and onValueChange with restoreFocusOnReentry() and appearance.inputTextColor

```
modifier = Modifier.fillMaxSize().restoreFocusOnReentry(),
```

### <a id="sv-and-f-0125"></a>`SV-AND-F-0125`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#SurveySheet (L72-216)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Prevents the modal bottom sheet from transitioning to SheetValue.Hidden via drag/swipe

```
confirmValueChange = { it != SheetValue.Hidden },
```

### <a id="sv-and-f-0126"></a>`SV-AND-F-0126`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#SurveySheet (L72-216)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Shows the intro screen initially only when starting at question index 0, displayIntroScreen is true, and an intro header or description is present

```
survey.initialQuestionIndex == 0 && appearance.displayIntroScreen &&
```

### <a id="sv-and-f-0137"></a>`SV-AND-F-0137`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#QuestionContent (L218-236)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Renders UnsupportedQuestionPlaceholder for unknown question types with a button label defaulting to "Close"

```
else -> UnsupportedQuestionPlaceholder(buttonLabel = question.buttonText ?: "Close", onClose = onClose)
```

### <a id="sv-and-f-0138"></a>`SV-AND-F-0138`

- **Surface:** `AND` | **Type:** `input` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#RatingQuestionDispatch (L238-273)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Allows rating submission only when question.isOptional is true or a rating value is selected

```
val canSubmit = question.isOptional || rating != null
```

### <a id="sv-and-f-0142"></a>`SV-AND-F-0142`

- **Surface:** `AND` | **Type:** `input` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#OpenTextQuestionDispatch (L275-293)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Allows open-text submission only when question.isOptional is true or the entered text is non-blank

```
val canSubmit = question.isOptional || text.isNotBlank()
```

### <a id="sv-and-f-0145"></a>`SV-AND-F-0145`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#LinkQuestionDispatch (L295-310)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Renders an always-enabled BottomSection button falling back to localAppearance().submitButtonText when question.buttonText is null

```
label = question.buttonText ?: localAppearance().submitButtonText,
```

### <a id="sv-and-f-0148"></a>`SV-AND-F-0148`

- **Surface:** `AND` | **Type:** `input` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#SingleChoiceQuestionDispatch (L312-348)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Requires a selected choice (and non-blank openInput when the open choice is selected) unless question.isOptional is true

```
(selected != null && (!hasOpenChoiceSelected || openInput.isNotBlank()))
```

### <a id="sv-and-f-0154"></a>`SV-AND-F-0154`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#MultipleChoiceQuestionDispatch (L350-382)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Enables submission only when question.isOptional is true, or when at least one choice is selected and any selected open choice has non-blank openInput.

```
(selected.isNotEmpty() && (!hasOpenChoiceSelected || openInput.isNotBlank()))
```

### <a id="sv-and-f-0157"></a>`SV-AND-F-0157`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android-surveys-compose/src/main/java/com/posthog/android/surveys/compose/internal/ui/SurveySheet.kt#MultipleChoiceQuestionDispatch (L350-382)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Submits PostHogSurveyResponse.MultipleChoice with null instead of an empty list when no choices are selected.

```
onSubmit(PostHogSurveyResponse.MultipleChoice(responses.ifEmpty { null }))
```

### <a id="sv-and-f-0166"></a>`SV-AND-F-0166`

- **Surface:** `AND` | **Type:** `fact` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration (L42-1296)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Constructs a "survey shown" event with survey properties, language, and distinctId

```
): SurveyEvent = SurveyEvent("survey shown", surveyEventProperties(survey, language = language), distinctId)
```

### <a id="sv-and-f-0168"></a>`SV-AND-F-0168`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration (L42-1296)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Allows showing the next survey only when cachePreferences is available, no activeSurvey is set, and pendingPresentation is null

```
config.cachePreferences?.isAvailable() != false && activeSurvey == null && pendingPresentation == null
```

### <a id="sv-and-f-0169"></a>`SV-AND-F-0169`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration (L42-1296)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Captures survey lifecycle and response events via postHog.capture

```
postHog?.capture(event.name, distinctId = event.distinctId, properties = event.properties)
```

### <a id="sv-and-f-0170"></a>`SV-AND-F-0170`

- **Surface:** `AND` | **Type:** `storage` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration (L42-1296)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Persists the last seen survey date as an ISO-8601 string in cachePreferences

```
config.cachePreferences?.setValue(PostHogPreferences.LAST_SEEN_SURVEY_DATE, formatISO8601Date(date))
```

### <a id="sv-and-f-0171"></a>`SV-AND-F-0171`

- **Surface:** `AND` | **Type:** `wiring` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.install (L103-122)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Installs the surveys integration on PostHog, marks it started, resolves the UI delegate, and attempts to show the next survey

```
public override fun install(postHog: PostHogInterface) {
```

### <a id="sv-and-f-0173"></a>`SV-AND-F-0173`

- **Surface:** `AND` | **Type:** `wiring` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.onSurveysLoaded (L143-156)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Handles loaded surveys by caching them, reconciling progressStore, and rebuilding the event-to-surveys map

```
public override fun onSurveysLoaded(surveys: List<Survey>) {
```

### <a id="sv-and-f-0175"></a>`SV-AND-F-0175`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getActiveMatchingSurveys (L237-290)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns an empty list when config.surveys is disabled

```
if (!config.surveys) {
```

### <a id="sv-and-f-0176"></a>`SV-AND-F-0176`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getActiveMatchingSurveys (L237-290)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns an empty list when postHog is not installed

```
val postHog = postHog ?: return emptyList()
```

### <a id="sv-and-f-0184"></a>`SV-AND-F-0184`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.showSurvey (L349-401)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns early without showing the survey if the integration is not started.

```
if (!synchronized(lifecycleLock) { isStarted }) return
```

### <a id="sv-and-f-0187"></a>`SV-AND-F-0187`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.showSurvey (L349-401)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Renders the display survey via the surveys delegate with shown, response, and closed callbacks when the presentation is still current.

```
getSurveysDelegate().renderSurvey(displaySurvey, onSurveyShown, onSurveyResponse, onSurveyClosed)
```

### <a id="sv-and-f-0188"></a>`SV-AND-F-0188`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.showSurvey (L349-401)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs "Cannot show survey - another survey is already active" and aborts if canShowNextSurvey() returns false.

```
if (!canShowNextSurvey()) {
```

### <a id="sv-and-f-0189"></a>`SV-AND-F-0189`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.showSurvey (L349-401)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Under activeSurveyLock, logs "Cannot show survey - another survey is already pending" and aborts if pendingPresentation is non-null; otherwise increments presentationGeneration and records pendingPresentation.

```
if (pendingPresentation != null) {
```

### <a id="sv-and-f-0193"></a>`SV-AND-F-0193`

- **Surface:** `AND` | **Type:** `control` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.renderIfCurrent (L403-416)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Runs on the main thread and only invokes render() if isStarted is true and generation and presentation still match resetGeneration and presentationGeneration; otherwise releases pendingPresentation and aborts.

```
if (!synchronized(lifecycleLock) { isStarted } || !current) {
```

### <a id="sv-and-f-0207"></a>`SV-AND-F-0207`

- **Surface:** `AND` | **Type:** `branch` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyResponseCallback (L472-503)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Captures a "survey sent" event when shouldSendResponse(originalSurvey, activeSurveyCompleted) is true, then returns nextQuestion.

```
if (shouldSendResponse(originalSurvey, activeSurveyCompleted)) {
```

### <a id="sv-and-f-0208"></a>`SV-AND-F-0208`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyResponseCallback (L472-503)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Handles the survey-response delegate callback, returning the next question state (PostHogNextSurveyQuestion) or null when rejected.

```
onSurveyResponse@{ responseSurvey, questionIndex, response ->
```

### <a id="sv-and-f-0209"></a>`SV-AND-F-0209`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyResponseCallback (L472-503)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs a warning and returns null if the response belongs to a non-active attempt (!isActiveAttempt(responseSurvey.id, responseContext.submissionId)) or !canRecordResponse(responseContext).

```
if (!isActiveAttempt(responseSurvey.id, responseContext.submissionId)) {
```

### <a id="sv-and-f-0210"></a>`SV-AND-F-0210`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyResponseCallback (L472-503)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Captures a "survey sent" event with current responses, completion status, and distinctId when shouldSendResponse(originalSurvey, activeSurveyCompleted) is true.

```
surveySentEvent(responseContext, currentSurveyResponses.toMap(), nextQuestion.isSurveyCompleted, distinctId)
```

### <a id="sv-and-f-0213"></a>`SV-AND-F-0213`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyShownCallback (L505-535)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Activates the survey, updates the last-seen survey date, removes it from eventActivatedSurveys, and captures a "survey shown" event.

```
surveyShownEvent(originalSurvey, responseContext.language, distinctId)
```

### <a id="sv-and-f-0214"></a>`SV-AND-F-0214`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyShownCallback (L505-535)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Handles the survey-shown delegate callback when the UI reports a survey was displayed.

```
private fun surveyShownCallback(
```

### <a id="sv-and-f-0215"></a>`SV-AND-F-0215`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveyShownCallback (L505-535)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Releases pendingPresentation and aborts activation and event emission if !isCurrentContext(responseContext).

```
if (!isCurrentContext(responseContext)) return@synchronized null
```

### <a id="sv-and-f-0219"></a>`SV-AND-F-0219`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.canRecordResponse (L570-577)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Allows recording a response only when hasExpectedProgress(context, activeProgressWasPersisted) is true and context.resetGeneration matches resetGeneration.

```
if (hasExpectedProgress(context, activeProgressWasPersisted) && context.resetGeneration == resetGeneration
```

### <a id="sv-and-f-0223"></a>`SV-AND-F-0223`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getResponseBasedNextQuestion (L687-710)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs an unsupported question type message and returns null for non-SingleChoice, non-Rating responses

```
config.logger.log("[Surveys] Got response based branching for an unsupported question type.")
```

### <a id="sv-and-f-0227"></a>`SV-AND-F-0227`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.handleSingleChoiceResponseBranching (L715-742)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs an error and returns null when no matching response index or branching target is found

```
config.logger.log("[Surveys] Could not find response index for specific question.")
```

### <a id="sv-and-f-0230"></a>`SV-AND-F-0230`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.handleRatingResponseBranching (L747-771)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs an error and returns null when scale, rating, rating bucket, or target index cannot be resolved

```
config.logger.log("[Surveys] Could not get response bucket for rating question.")
```

### <a id="sv-and-f-0231"></a>`SV-AND-F-0231`

- **Surface:** `AND` | **Type:** `limit` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.questionDestination (L773-780)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Clamps target questionIndex to the range 0..maxOf(totalQuestions - 1, 0)

```
questionIndex = index.coerceIn(0, maxOf(totalQuestions - 1, 0)),
```

### <a id="sv-and-f-0235"></a>`SV-AND-F-0235`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.processBranchingStep (L785-801)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns null for any other nextIndex value or type

```
else -> null
```

### <a id="sv-and-f-0240"></a>`SV-AND-F-0240`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getRatingBucketForResponseValue (L807-846)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Maps rating scale (3, 5, 7, or 10) and value to a bucket string, returning null for unsupported scales or out-of-range values

```
return when (scale) {
```

### <a id="sv-and-f-0241"></a>`SV-AND-F-0241`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.showNextSurvey (L879-895)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns early without showing a survey if !canRenderSurveys() or !canShowNextSurvey()

```
if (!canRenderSurveys() || !canShowNextSurvey()) {
```

### <a id="sv-and-f-0245"></a>`SV-AND-F-0245`

- **Surface:** `AND` | **Type:** `storage` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveySentEvent (L977-996)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Includes $set setting the survey responded interaction person property to true

```
getSurveyInteractionProperty(context.survey, "responded") to true,
```

### <a id="sv-and-f-0247"></a>`SV-AND-F-0247`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.surveySentEvent (L977-996)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Constructs a "survey sent" SurveyEvent with response properties, $survey_submission_id, $survey_completed, $set, language, and distinctId

```
return SurveyEvent("survey sent", surveyEventProperties(context.survey, additionalProperties, context.language), distinctId)
```

### <a id="sv-and-f-0253"></a>`SV-AND-F-0253`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.buildSurveyResponseProperties (L1028-1058)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Resolves each question's text in order: recorded questionText, translated question text, then base question.question

```
val effectiveQuestion = questionText[index] ?: translatedText ?: question.question
```

### <a id="sv-and-f-0257"></a>`SV-AND-F-0257`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getSeenSurveyKeys (L1161-1168)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Reads PostHogPreferences.SURVEY_SEEN from config.cachePreferences into the in-memory seenSurveyKeys map when not yet loaded.

```
val storedKeys = config.cachePreferences?.getValue(PostHogPreferences.SURVEY_SEEN) as? Map<String, Boolean>
```

### <a id="sv-and-f-0259"></a>`SV-AND-F-0259`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getSurveySeen (L1174-1183)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Treats a survey as not seen (returns false) if canActivateOrResume(survey) is true

```
if (canActivateOrResume(survey)) {
```

### <a id="sv-and-f-0260"></a>`SV-AND-F-0260`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getSurveySeen (L1174-1183)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Defaults to false when the survey's seen key is absent from seenKeys

```
return seenKeys[key] ?: false
```

### <a id="sv-and-f-0264"></a>`SV-AND-F-0264`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getLastSeenSurveyDate (L1219-1227)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs parse failure and returns null if the stored last seen survey date fails ISO-8601 parsing

```
config.logger.log("Failed to parse last seen survey date: $dateString")
```

### <a id="sv-and-f-0265"></a>`SV-AND-F-0265`

- **Surface:** `AND` | **Type:** `port` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.getLastSeenSurveyDate (L1219-1227)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Reads PostHogPreferences.LAST_SEEN_SURVEY_DATE as a String from config.cachePreferences and returns null if absent or not a String.

```
val dateString = config.cachePreferences?.getValue(PostHogPreferences.LAST_SEEN_SURVEY_DATE) as? String ?: return null
```

### <a id="sv-and-f-0266"></a>`SV-AND-F-0266`

- **Surface:** `AND` | **Type:** `wiring` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.onEvent (L1250-1281)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Receives SDK captured events via onEvent to evaluate event-activated surveys

```
public override fun onEvent(
```

### <a id="sv-and-f-0267"></a>`SV-AND-F-0267`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.onEvent (L1250-1281)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns early without activating surveys when canRenderSurveys() is false

```
if (!canRenderSurveys()) return
```

### <a id="sv-and-f-0274"></a>`SV-AND-F-0274`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.matchPropertyFilters (L1283-1295)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns true by default when propertyFilters is null or empty.

```
if (propertyFilters.isNullOrEmpty()) return true
```

### <a id="sv-and-f-0275"></a>`SV-AND-F-0275`

- **Surface:** `AND` | **Type:** `guard` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/PostHogSurveysIntegration.kt#PostHogSurveysIntegration.matchPropertyFilters (L1283-1295)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Returns false if any filtered propertyName is absent or null in eventProperties, or if filter.operator is unknown in surveyValidationMap.

```
val eventValue = eventProperties?.get(propertyName) ?: return@all false
```

### <a id="sv-and-f-0281"></a>`SV-AND-F-0281`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/SurveyProgressStore.kt#SurveyProgressStore (L50-137)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Keys survey progress entries by "{survey.id}/{survey.currentIteration ?: 0}", defaulting missing currentIteration to 0.

```
private fun key(survey: Survey): String = "${survey.id}/${survey.currentIteration ?: 0}"
```

### <a id="sv-and-f-0286"></a>`SV-AND-F-0286`

- **Surface:** `AND` | **Type:** `storage` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/SurveyProgressStore.kt#SurveyProgressStore.records (L70-75)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Reads stored survey progress map from config.cachePreferences under PostHogPreferences.SURVEY_PROGRESS.

```
val stored = config.cachePreferences?.getValue(PostHogPreferences.SURVEY_PROGRESS) as? Map<*, *>
```

### <a id="sv-and-f-0290"></a>`SV-AND-F-0290`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/SurveyProgressStore.kt#SurveyProgressStore.load (L77-97)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Logs "Discarding invalid saved survey progress" when deserializing saved survey progress throws an Exception.

```
config.logger.log("Discarding invalid saved survey progress")
```

### <a id="sv-and-f-0292"></a>`SV-AND-F-0292`

- **Surface:** `AND` | **Type:** `storage` | **Source:** `posthog-android:posthog-android/src/main/java/com/posthog/android/surveys/SurveyProgressStore.kt#SurveyProgressStore.writeRecords (L129-136)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Persists records to config.cachePreferences under PostHogPreferences.SURVEY_PROGRESS only when generation equals resetGeneration.

```
config.cachePreferences?.setValue(PostHogPreferences.SURVEY_PROGRESS, records)
```

### <a id="sv-and-f-0297"></a>`SV-AND-F-0297`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog/src/main/java/com/posthog/surveys/PostHogDisplaySurvey.kt#PostHogDisplaySurvey.Companion.toDisplaySurvey (L67-90)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** toDisplaySurvey uses surveyTranslation.name when available and falls back to survey.name.

```
name = surveyTranslation?.name ?: survey.name,
```

### <a id="sv-and-f-0311"></a>`SV-AND-F-0311`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog/src/main/java/com/posthog/surveys/PostHogDisplaySurveyQuestion.kt#PostHogDisplaySurveyQuestion.Companion.fromSurveyQuestion (L30-158)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Prefers translation description and buttonText over the base question values when present.

```
val description = translation?.description ?: question.description
```

### <a id="sv-and-f-0325"></a>`SV-AND-F-0325`

- **Surface:** `AND` | **Type:** `default` | **Source:** `posthog-android:posthog/src/main/java/com/posthog/surveys/PostHogSurveysConfig.kt#PostHogSurveysConfig (L6-31)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** surveysDelegate defaults to an instance of PostHogSurveysDefaultDelegate

```
public var surveysDelegate: PostHogSurveysDelegate = PostHogSurveysDefaultDelegate()
```

### <a id="sv-and-f-0329"></a>`SV-AND-F-0329`

- **Surface:** `AND` | **Type:** `error` | **Source:** `posthog-android:posthog/src/main/java/com/posthog/surveys/PostHogSurveysDefaultDelegate.kt#PostHogSurveysDefaultDelegate.renderSurvey (L21-30)` | **Pin:** `4c3c125da08c48fd0928`
- **Claim:** Default renderSurvey is a no-op that does not render UI or trigger survey callbacks, only logging the survey id and name and instructing callers to implement a custom PostHogSurveysDelegate.

```
config?.logger?.log("Implement your own PostHogSurveysDelegate to render surveys")
```

### <a id="sv-api-f-0011"></a>`SV-API-F-0011`

- **Surface:** `API` | **Type:** `port` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_get_surveys_response_counts (L21-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Executes an offline HogQL query against events with query_type 'stop_surveys_reached_target' to count unique survey responses per survey_id.

```
response = execute_hogql_query(
```

### <a id="sv-api-f-0012"></a>`SV-API-F-0012`

- **Surface:** `API` | **Type:** `port` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_get_surveys_response_counts (L21-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Filters the HogQL query to 'survey sent' events at or after earliest_survey_creation_date for the given surveys_ids.

```
WHERE event = 'survey sent'
```

### <a id="sv-api-f-0013"></a>`SV-API-F-0013`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_get_surveys_response_counts (L21-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Counts distinct responses using properties.$survey_submission_id when non-empty, falling back to toString(uuid).

```
coalesce(properties.$survey_submission_id, '') != '',
```

### <a id="sv-api-f-0014"></a>`SV-API-F-0014`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_get_surveys_response_counts (L21-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Caps the HogQL query result rows to the number of survey IDs in surveys_ids.

```
"limit": ast.Constant(value=len(surveys_ids)),
```

### <a id="sv-api-f-0015"></a>`SV-API-F-0015`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_stop_survey_if_reached_limit (L54-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Refreshes the Survey instance from the database before checking its response limit and end_date.

```
survey.refresh_from_db()
```

### <a id="sv-api-f-0016"></a>`SV-API-F-0016`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_stop_survey_if_reached_limit (L54-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns early without stopping the survey if responses_limit is None or end_date is already set.

```
if survey.responses_limit is None or survey.end_date is not None:
```

### <a id="sv-api-f-0018"></a>`SV-API-F-0018`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#_stop_survey_if_reached_limit (L54-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Stops the survey by setting end_date to timezone.now(), clearing responses_limit to None, and saving those fields to the Survey model.

```
survey.save(update_fields=["end_date", "responses_limit"])
```

### <a id="sv-api-f-0020"></a>`SV-API-F-0020`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#stop_surveys_reached_target (L78-100)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries Survey records that have a non-null responses_limit and no end_date set.

```
all_surveys = Survey.objects.exclude(Q(responses_limit__isnull=True) | Q(end_date__isnull=False)).only(
```

### <a id="sv-api-f-0021"></a>`SV-API-F-0021`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/stop_surveys_reached_target.py#stop_surveys_reached_target (L78-100)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Bulk-loads Team records for all teams referenced by active response-limited surveys.

```
teams_by_id = Team.objects.in_bulk({survey.team_id for survey in all_surveys_sorted})
```

### <a id="sv-api-f-0026"></a>`SV-API-F-0026`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/survey_system_stop.py#log_survey_stopped_by_system (L8-22)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Writes an "updated" activity log entry via log_activity for scope="Survey" and item_id=survey.id under survey.team_id and survey.team.organization_id.

```
log_activity(
```

### <a id="sv-api-f-0036"></a>`SV-API-F-0036`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_update_survey_iteration (L17-52)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Refreshes the Survey instance from the database before evaluating or updating its iteration state.

```
survey.refresh_from_db()
```

### <a id="sv-api-f-0040"></a>`SV-API-F-0040`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_update_survey_iteration (L17-52)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Persists the updated end_date field on the Survey model when closing the survey after its final iteration.

```
survey.save(update_fields=["end_date"])
```

### <a id="sv-api-f-0043"></a>`SV-API-F-0043`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_update_survey_iteration (L17-52)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Saves updated current_iteration, current_iteration_start_date, internal_targeting_flag_id, and iteration_start_dates to the Survey model.

```
survey.save(
```

### <a id="sv-api-f-0047"></a>`SV-API-F-0047`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_get_final_iteration_end (L66-78)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Catches OverflowError when adding iteration_frequency_days and returns None.

```
except OverflowError:
```

### <a id="sv-api-f-0048"></a>`SV-API-F-0048`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_was_resumed_by_a_person_after (L81-91)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries ActivityLog for non-system Survey entries created after day where end_date was changed to None.

```
return ActivityLog.objects.filter(
```

### <a id="sv-api-f-0050"></a>`SV-API-F-0050`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_get_targeting_flag (L94-133)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** If survey.internal_targeting_flag already exists, updates its release conditions via update_flag with user=None.

```
if existing_targeting_flag is not None:
```

### <a id="sv-api-f-0051"></a>`SV-API-F-0051`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_get_targeting_flag (L94-133)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When no internal_targeting_flag exists, creates an active FeatureFlag keyed by str(survey.id) with creation_context="surveys" and user=None.

```
flag = create_flag(
```

### <a id="sv-api-f-0052"></a>`SV-API-F-0052`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#_get_targeting_flag (L94-133)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When survey.created_by is set, updates created_by on the newly created FeatureFlag record in the database and on the returned object.

```
FeatureFlag.objects.filter(pk=flag.pk).update(created_by=survey.created_by)
```

### <a id="sv-api-f-0053"></a>`SV-API-F-0053`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#update_survey_iteration (L155-183)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries Survey records with schedule=Survey.Schedule.RECURRING, non-null start_date, null end_date, and either iteration_count set or repeat_until_stopped=True.

```
surveys_with_recurring_schedules = Survey.objects.filter(
```

### <a id="sv-api-f-0054"></a>`SV-API-F-0054`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:posthog/tasks/update_survey_iteration.py#update_survey_iteration (L155-183)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Catches exceptions per survey while iterating recurring schedules and reports them via capture_exception with survey_id and team_id without stopping other surveys.

```
capture_exception(e, additional_properties={"survey_id": str(survey.id), "team_id": survey.team_id})
```

### <a id="sv-api-f-0062"></a>`SV-API-F-0062`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/site_snapshot_serializers.py#SiteSnapshotQuerySerializer (L7-28)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Validates required `device` query parameter against `SnapshotDevice.choices`.

```
device = serializers.ChoiceField(
```

### <a id="sv-api-f-0063"></a>`SV-API-F-0063`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/site_snapshot_serializers.py#SiteSnapshotQuerySerializer (L7-28)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Accepts optional `url` query parameter with `required=False` and `allow_blank=True` to override saved survey URL targeting or match every page when empty.

```
url = serializers.CharField(
```

### <a id="sv-api-f-0064"></a>`SV-API-F-0064`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/site_snapshot_serializers.py#SiteSnapshotQuerySerializer (L7-28)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Accepts optional `url_match_type` query parameter validated against `SurveyMatchType.choices`.

```
url_match_type = serializers.ChoiceField(
```

### <a id="sv-api-f-0066"></a>`SV-API-F-0066`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/api/site_snapshot_serializers.py#url_target_of_query (L74-80)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Constructs `UrlTarget` with `url=query["url"]` and defaults `match_type` to `SurveyMatchType.ICONTAINS` when `url_match_type` is absent or falsy.

```
match_type=SurveyMatchType(query.get("url_match_type") or SurveyMatchType.ICONTAINS),
```

### <a id="sv-api-f-0107"></a>`SV-API-F-0107`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyResponsesQuerySerializer (L1028-1075)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** `limit` has a minimum value of `1`.

```
min_value=1,
```

### <a id="sv-api-f-0108"></a>`SV-API-F-0108`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyResponsesQuerySerializer (L1028-1075)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** `limit` has a maximum value of `500`.

```
max_value=500,
```

### <a id="sv-api-f-0110"></a>`SV-API-F-0110`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyResponsesQuerySerializer (L1028-1075)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** `offset` has a minimum value of `0`.

```
min_value=0,
```

### <a id="sv-api-f-0111"></a>`SV-API-F-0111`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializer (L1087-1200)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Mixes in `SearchMatchTypeSerializerMixin` and `UserAccessControlSerializerMixin` on `SurveySerializer`.

```
class SurveySerializer(SearchMatchTypeSerializerMixin, UserAccessControlSerializerMixin, serializers.ModelSerializer):
```

### <a id="sv-api-f-0114"></a>`SV-API-F-0114`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializer (L1087-1200)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Serializes `Survey` model instances.

```
model = Survey
```

### <a id="sv-api-f-0120"></a>`SV-API-F-0120`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#_free_name_hint (L1203-1211)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries `Survey` names in the given `project_id` starting with `name` to avoid collisions.

```
Survey.objects.filter(team__project_id=project_id, name__startswith=name).values_list("name", flat=True)
```

### <a id="sv-api-f-0124"></a>`SV-API-F-0124`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly (L1214-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** `iteration_count` is constrained between `0` and `MAX_ITERATION_COUNT` (optional and nullable).

```
required=False, allow_null=True, max_value=MAX_ITERATION_COUNT, min_value=0
```

### <a id="sv-api-f-0125"></a>`SV-API-F-0125`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly (L1214-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** `base_language` has a `max_length` of `20` characters.

```
max_length=20,
```

### <a id="sv-api-f-0126"></a>`SV-API-F-0126`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly (L1214-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Creates and updates `Survey` model instances.

```
model = Survey
```

### <a id="sv-api-f-0128"></a>`SV-API-F-0128`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly (L1214-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises `serializers.ValidationError` if `WAIT_PERIOD_DAYS_CONDITION` is present in `conditions` and fails `is_valid_wait_period`.

```
raise serializers.ValidationError(f"{WAIT_PERIOD_DAYS_CONDITION} must be a number of days")
```

### <a id="sv-api-f-0129"></a>`SV-API-F-0129`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly (L1214-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises `serializers.ValidationError("Schedule must be one of: once, recurring, always")` if `schedule` is not `None` and not one of `"once"`, `"recurring"`, or `"always"`.

```
raise serializers.ValidationError("Schedule must be one of: once, recurring, always")
```

### <a id="sv-api-f-0132"></a>`SV-API-F-0132`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_and_sanitize_link (L1221-1245)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when link URL scheme is not in ALLOWED_LINK_URL_SCHEMES

```
if parsed_url.scheme not in ALLOWED_LINK_URL_SCHEMES:
```

### <a id="sv-api-f-0133"></a>`SV-API-F-0133`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_and_sanitize_link (L1221-1245)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when a mailto link does not match EMAIL_REGEX

```
if not re.match(EMAIL_REGEX, link):
```

### <a id="sv-api-f-0134"></a>`SV-API-F-0134`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_and_sanitize_link (L1221-1245)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when an https link has no netloc

```
raise serializers.ValidationError("Invalid HTTPS URL. Please enter a valid HTTPS link.")
```

### <a id="sv-api-f-0136"></a>`SV-API-F-0136`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_and_sanitize_choices (L1247-1260)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when any question choice is not a string

```
raise serializers.ValidationError("Question choices must be strings")
```

### <a id="sv-api-f-0137"></a>`SV-API-F-0137`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_and_sanitize_choices (L1247-1260)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when any question choice is empty or whitespace-only

```
raise serializers.ValidationError("Question choices cannot be empty")
```

### <a id="sv-api-f-0139"></a>`SV-API-F-0139`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.Meta.fields (L1286-1326)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes responses_limit field on SurveySerializerCreateUpdateOnly to configure maximum survey responses

```
"responses_limit",
```

### <a id="sv-api-f-0141"></a>`SV-API-F-0141`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_appearance (L1337-1370)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when appearance is not a dict

```
raise serializers.ValidationError("Appearance must be an object")
```

### <a id="sv-api-f-0143"></a>`SV-API-F-0143`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_appearance (L1337-1370)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when thankYouMessageDescriptionContentType is not 'text' or 'html'

```
raise serializers.ValidationError("thankYouMessageDescriptionContentType must be one of ['text', 'html']")
```

### <a id="sv-api-f-0144"></a>`SV-API-F-0144`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_appearance (L1337-1370)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when introScreenDescriptionContentType is not 'text' or 'html'

```
raise serializers.ValidationError("introScreenDescriptionContentType must be one of ['text', 'html']")
```

### <a id="sv-api-f-0145"></a>`SV-API-F-0145`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_appearance (L1337-1370)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when surveyPopupDelaySeconds is negative

```
raise serializers.ValidationError("Survey popup delay seconds must be a positive integer")
```

### <a id="sv-api-f-0146"></a>`SV-API-F-0146`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_appearance (L1337-1370)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when whiteLabel is provided and not a boolean

```
raise serializers.ValidationError("whiteLabel must be a boolean")
```

### <a id="sv-api-f-0147"></a>`SV-API-F-0147`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_appearance (L1337-1370)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires organization AvailableFeature.WHITE_LABELLING entitlement when whiteLabel is enabled, otherwise raises ValidationError

```
raise serializers.ValidationError("You need to upgrade to PostHog Enterprise to use white labelling")
```

### <a id="sv-api-f-0149"></a>`SV-API-F-0149`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_conditions (L1372-1381)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when conditions is not a dict

```
raise serializers.ValidationError("Conditions must be an object")
```

### <a id="sv-api-f-0152"></a>`SV-API-F-0152`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._require_viewer_access_to_added_actions (L1390-1418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when any action id in conditions is non-integer or outside (0, 2**63)

```
raise serializers.ValidationError("Each action in the survey conditions needs a valid id.")
```

### <a id="sv-api-f-0153"></a>`SV-API-F-0153`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._require_viewer_access_to_added_actions (L1390-1418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Excludes existing instance action IDs from access check when updating a survey

```
action_ids -= set(self.instance.actions.values_list("id", flat=True))
```

### <a id="sv-api-f-0154"></a>`SV-API-F-0154`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._require_viewer_access_to_added_actions (L1390-1418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Loads newly added Action records for the project to verify access

```
added_actions = get_actions(project_id=self.context["project_id"], action_ids=action_ids)
```

### <a id="sv-api-f-0155"></a>`SV-API-F-0155`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._require_viewer_access_to_added_actions (L1390-1418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires viewer access to every newly added action in conditions, raising ValidationError for denied action IDs

```
denied_ids = self.context["view"].user_access_control.denied_object_ids(
```

### <a id="sv-api-f-0157"></a>`SV-API-F-0157`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_targeting_flag_filters (L1420-1427)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when targeting_flag_filters is not a dict

```
raise serializers.ValidationError("targeting_flag_filters must be an object")
```

### <a id="sv-api-f-0159"></a>`SV-API-F-0159`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_form_content (L1429-1436)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when form_content is not a dict

```
raise serializers.ValidationError("form_content must be an object")
```

### <a id="sv-api-f-0163"></a>`SV-API-F-0163`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_translation_language_key (L1451-1477)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError (code translation_language_empty) when translation language key is not a non-empty string

```
if not isinstance(raw_key, str) or not raw_key.strip():
```

### <a id="sv-api-f-0164"></a>`SV-API-F-0164`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_translation_language_key (L1451-1477)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError (code translation_language_reserved) when translation language key is in REJECTED_TRANSLATION_KEYS

```
if normalized in REJECTED_TRANSLATION_KEYS:
```

### <a id="sv-api-f-0165"></a>`SV-API-F-0165`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_translation_language_key (L1451-1477)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError (code translation_language_invalid) when translation language key does not match BCP47_LANGUAGE_CODE_RE

```
if not BCP47_LANGUAGE_CODE_RE.match(normalized):
```

### <a id="sv-api-f-0166"></a>`SV-API-F-0166`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._validate_translation_language_key (L1451-1477)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError (code translation_language_matches_base) when translation language key matches base_language

```
if normalized == base_language:
```

### <a id="sv-api-f-0168"></a>`SV-API-F-0168`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_base_language (L1479-1494)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when base_language is not a string

```
raise serializers.ValidationError("base_language must be a string.")
```

### <a id="sv-api-f-0169"></a>`SV-API-F-0169`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_base_language (L1479-1494)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when normalized base_language is in REJECTED_TRANSLATION_KEYS

```
if normalized in REJECTED_TRANSLATION_KEYS:
```

### <a id="sv-api-f-0170"></a>`SV-API-F-0170`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_base_language (L1479-1494)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when normalized base_language does not match BCP47_LANGUAGE_CODE_RE

```
if not BCP47_LANGUAGE_CODE_RE.match(normalized):
```

### <a id="sv-api-f-0172"></a>`SV-API-F-0172`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_translations (L1496-1554)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when translations is not a dict

```
raise serializers.ValidationError("Translations must be an object")
```

### <a id="sv-api-f-0192"></a>`SV-API-F-0192`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_questions (L1637-1782)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError if questions is not a list

```
raise serializers.ValidationError("Questions must be a list of objects")
```

### <a id="sv-api-f-0195"></a>`SV-API-F-0195`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_questions (L1637-1782)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError if a question's text is missing or empty

```
if not question_text:
```

### <a id="sv-api-f-0207"></a>`SV-API-F-0207`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_questions (L1637-1782)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError if a translation's choices array length differs from the base question's choices length

```
if len(translated_choices) != len(original_choices):
```

### <a id="sv-api-f-0212"></a>`SV-API-F-0212`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_questions (L1637-1782)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError if a validation rule type is not one of min_length, max_length, or email

```
if rule_type not in valid_types:
```

### <a id="sv-api-f-0214"></a>`SV-API-F-0214`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate_questions (L1637-1782)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError if both min_length and max_length rules are present and min_length exceeds max_length

```
if min_val is not None and max_val is not None and min_val > max_val:
```

### <a id="sv-api-f-0221"></a>`SV-API-F-0221`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._reconcile_schedule_with_iterations (L2004-2042)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError on repeat_until_stopped if disabling repeat_until_stopped on an open-ended recurring schedule without setting iteration_count

```
{"repeat_until_stopped": "Set iteration_count to give a recurring schedule a final repeat."}
```

### <a id="sv-api-f-0222"></a>`SV-API-F-0222`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._reconcile_schedule_with_iterations (L2004-2042)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError on iteration_frequency_days if effective repeat_until_stopped is true without iteration_frequency_days

```
{"iteration_frequency_days": "Set a number of days between repeats to repeat until stopped."}
```

### <a id="sv-api-f-0223"></a>`SV-API-F-0223`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When linked_flag_id is provided, loads the FeatureFlag scoped to context['team_id']

```
linked_flag = FeatureFlag.objects.get(pk=linked_flag_id, team_id=self.context["team_id"])
```

### <a id="sv-api-f-0224"></a>`SV-API-F-0224`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError('Feature Flag with this ID does not exist') when linked_flag_id does not match a FeatureFlag in the team

```
raise serializers.ValidationError("Feature Flag with this ID does not exist")
```

### <a id="sv-api-f-0225"></a>`SV-API-F-0225`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When targeting_flag_id is provided, loads the targeting FeatureFlag scoped to context['team_id']

```
targeting_flag = FeatureFlag.objects.get(pk=targeting_flag_id, team_id=self.context["team_id"])
```

### <a id="sv-api-f-0226"></a>`SV-API-F-0226`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError('Targeting Feature Flag with this ID does not exist') when targeting_flag_id does not match a FeatureFlag in the team

```
raise serializers.ValidationError("Targeting Feature Flag with this ID does not exist")
```

### <a id="sv-api-f-0227"></a>`SV-API-F-0227`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** On create or when targeting_flag_id changes, verifies the targeting flag is available for FLAG_OWNER_SURVEY via assert_flag_available_for

```
if self.instance is None or self.instance.targeting_flag_id != targeting_flag_id:
```

### <a id="sv-api-f-0228"></a>`SV-API-F-0228`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When linked_insight_id is provided, verifies an Insight with that pk exists in context['team_id']

```
Insight.objects.get(pk=linked_insight_id, team_id=self.context["team_id"])
```

### <a id="sv-api-f-0229"></a>`SV-API-F-0229`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError('Insight with this ID does not exist') when linked_insight_id does not match an Insight in the team

```
raise serializers.ValidationError("Insight with this ID does not exist")
```

### <a id="sv-api-f-0230"></a>`SV-API-F-0230`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when conditions.linkedFlagVariant (other than 'any') is used with a linked flag whose variants raise ConfigFormatError

```
"linkedFlagVariant cannot be used with this feature flag's configuration format"
```

### <a id="sv-api-f-0231"></a>`SV-API-F-0231`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError listing available variants when conditions.linkedFlagVariant is not among the linked flag's non-empty variant keys

```
f"Feature flag variant '{linked_flag_variant}' does not exist. Available variants: {', '.join(available_variants)}"
```

### <a id="sv-api-f-0232"></a>`SV-API-F-0232`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when conditions.linkedFlagVariant (other than 'any') is specified on a linked flag that has no variants

```
f"Feature flag variant '{linked_flag_variant}' specified but the linked feature flag has no variants"
```

### <a id="sv-api-f-0233"></a>`SV-API-F-0233`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** On create, raises ValidationError if conditions.linkedFlagVariant is set without linked_flag_id

```
"linkedFlagVariant can only be used when a linked_flag_id is specified"
```

### <a id="sv-api-f-0235"></a>`SV-API-F-0235`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** On POST, checks Survey table for an existing survey with the same name in context['project_id'] and raises ValidationError with code='unique' and a free-name hint

```
and Survey.objects.filter(name=data.get("name"), team__project_id=self.context["project_id"]).exists()
```

### <a id="sv-api-f-0236"></a>`SV-API-F-0236`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** On update when renaming, checks Survey table excluding self.instance.id for another survey with the same name in context['project_id'] and raises ValidationError with code='unique'

```
and Survey.objects.filter(name=data.get("name"), team__project_id=self.context["project_id"])
```

### <a id="sv-api-f-0237"></a>`SV-API-F-0237`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Rejects targeting_flag_filters with code='invalid' if any group has rollout_percentage of 100 or None and no properties

```
"Invalid operation: User targeting rolls out to everyone. If you want to roll out to everyone, delete this targeting",
```

### <a id="sv-api-f-0238"></a>`SV-API-F-0238`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError on response_sampling_start_date if it is earlier than today at 00:00:00 UTC

```
if response_sampling_start_date < today_utc:
```

### <a id="sv-api-f-0239"></a>`SV-API-F-0239`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError on response_sampling_interval when provided and <= 0

```
if response_sampling_interval is not None and response_sampling_interval <= 0:
```

### <a id="sv-api-f-0240"></a>`SV-API-F-0240`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError on response_sampling_start_date when response_sampling_limit (defaulting to 0) > 0 and response_sampling_interval > 0 while response_sampling_start_date is None

```
"response_sampling_start_date": "Response sampling start date should be set if response_sampling_start_date is not zero."
```

### <a id="sv-api-f-0242"></a>`SV-API-F-0242`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when effective appearance.shuffleQuestions is truthy and any effective question has non-None branching

```
"Question shuffling and question branching cannot be used together. "
```

### <a id="sv-api-f-0243"></a>`SV-API-F-0243`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** For EXTERNAL_SURVEY, collects per-field ValidationErrors for any non-None field in FIELDS_NOT_APPLICABLE_TO_EXTERNAL_SURVEYS

```
errors[field] = f"{field} is not allowed for external surveys"
```

### <a id="sv-api-f-0244"></a>`SV-API-F-0244`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** For EXTERNAL_SURVEY, rejects non-None CONDITION_FIELDS_NOT_APPLICABLE_TO_EXTERNAL_SURVEYS and non-empty conditions.actions.values or conditions.events.values

```
f"The following condition fields are not allowed for external surveys: {', '.join(condition_errors)}"
```

### <a id="sv-api-f-0245"></a>`SV-API-F-0245`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** For EXTERNAL_SURVEY, rejects non-None appearance.surveyPopupDelaySeconds and raises ValidationError(errors) if any external survey errors exist

```
errors["appearance"] = "surveyPopupDelaySeconds is not allowed for external surveys"
```

### <a id="sv-api-f-0246"></a>`SV-API-F-0246`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.validate (L1789-2002)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** On update, when base_language changes from the normalized existing base_language (defaulting to DEFAULT_BASE_LANGUAGE), raises ValidationError with code='translation_language_matches_base' if it collides with preserved survey or question translation keys

```
and normalized_base != _normalize_language_code(self.instance.base_language or DEFAULT_BASE_LANGUAGE)
```

### <a id="sv-api-f-0249"></a>`SV-API-F-0249`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.create (L2057-2117)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Loads the Team matching context["team_id"] and sets team_id on validated_data.

```
team = Team.objects.get(id=self.context["team_id"])
```

### <a id="sv-api-f-0251"></a>`SV-API-F-0251`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.create (L2057-2117)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires feature flag write scope (survey:write, action survey.create.targeting_flag_id) when targeting_flag_id is provided.

```
if validated_data.get("targeting_flag_id"):
```

### <a id="sv-api-f-0252"></a>`SV-API-F-0252`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.create (L2057-2117)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires feature flag write scope (survey:write, action survey.create) when targeting_flag_filters is provided.

```
action="survey.create",
```

### <a id="sv-api-f-0255"></a>`SV-API-F-0255`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.create (L2057-2117)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets created_by to the requesting user and persists the new Survey inside a database transaction.

```
instance = super().create(validated_data)
```

### <a id="sv-api-f-0257"></a>`SV-API-F-0257`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.create (L2057-2117)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Logs a "created" Survey activity entry with organization, team, user, and impersonation details.

```
log_activity(
```

### <a id="sv-api-f-0258"></a>`SV-API-F-0258`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Fetches the pre-update Survey record from the database for change diffing and lifecycle event checks.

```
before_update = Survey.objects.get(pk=instance.pk)
```

### <a id="sv-api-f-0259"></a>`SV-API-F-0259`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires feature flag write scope (survey:write, action survey.update.targeting_flag_id) when targeting_flag_id is provided.

```
if validated_data.get("targeting_flag_id"):
```

### <a id="sv-api-f-0260"></a>`SV-API-F-0260`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError if iteration_count is positive and less than instance.current_iteration.

```
raise serializers.ValidationError(
```

### <a id="sv-api-f-0261"></a>`SV-API-F-0261`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires feature flag write scope (survey:write, action survey.update.remove_targeting_flag) when remove_targeting_flag is true and a targeting_flag exists.

```
action="survey.update.remove_targeting_flag",
```

### <a id="sv-api-f-0263"></a>`SV-API-F-0263`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires feature flag write scope (survey:write, action survey.update.targeting_flag_filters) when targeting_flag_filters is provided.

```
action="survey.update.targeting_flag_filters",
```

### <a id="sv-api-f-0265"></a>`SV-API-F-0265`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Persists updated Survey fields inside a gated_atomic transaction.

```
instance = super().update(instance, validated_data)
```

### <a id="sv-api-f-0266"></a>`SV-API-F-0266`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Loads the Team matching context["team_id"] after updating the survey.

```
team = Team.objects.get(id=self.context["team_id"])
```

### <a id="sv-api-f-0267"></a>`SV-API-F-0267`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates and saves instance.targeting_flag.active based on _should_survey_flags_be_active(instance) when targeting_flag is present.

```
instance.targeting_flag.save()
```

### <a id="sv-api-f-0269"></a>`SV-API-F-0269`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Logs an "updated" Survey activity entry with computed field changes.

```
activity="updated",
```

### <a id="sv-api-f-0270"></a>`SV-API-F-0270`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits a "survey launched" event when start_date transitions from None to non-null.

```
if before_update.start_date is None and instance.start_date is not None:
```

### <a id="sv-api-f-0271"></a>`SV-API-F-0271`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits a "survey stopped" event when end_date transitions from None to non-null.

```
elif before_update.end_date is None and instance.end_date is not None:
```

### <a id="sv-api-f-0272"></a>`SV-API-F-0272`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly.update (L2119-2292)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits a "survey resumed" event when a previously started and stopped survey has end_date cleared to None.

```
elif before_update.start_date is not None and before_update.end_date is not None and instance.end_date is None:
```

### <a id="sv-api-f-0275"></a>`SV-API-F-0275`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._add_internal_response_sampling_filters (L2294-2313)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Creates a 100%-rollout internal_response_sampling_flag with suffix "-sampling" (active if start_date is set) and saves the Survey.

```
instance.internal_response_sampling_flag = self._create_or_update_targeting_flag(
```

### <a id="sv-api-f-0276"></a>`SV-API-F-0276`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._associate_actions (L2315-2333)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clears all associated actions on the survey and returns when conditions is None.

```
if conditions is None:
```

### <a id="sv-api-f-0277"></a>`SV-API-F-0277`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._associate_actions (L2315-2333)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clears all associated actions on the survey and returns when conditions.actions is None.

```
if actions is None:
```

### <a id="sv-api-f-0278"></a>`SV-API-F-0278`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._associate_actions (L2315-2333)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clears all associated actions on the survey and returns when conditions.actions.values is None or empty.

```
if values is None or len(values) == 0:
```

### <a id="sv-api-f-0279"></a>`SV-API-F-0279`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._associate_actions (L2315-2333)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets survey actions to existing action IDs in context["project_id"] matching conditions.actions.values and saves the Survey.

```
instance.actions.set(existing_action_ids(project_id=self.context["project_id"], action_ids=action_ids))
```

### <a id="sv-api-f-0281"></a>`SV-API-F-0281`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._add_user_survey_interacted_filters (L2338-2371)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates the existing internal_targeting_flag (suffix "-custom") with merged interaction filters and active status when present, otherwise creates a new flag and saves internal_targeting_flag_id on the Survey.

```
if instance.internal_targeting_flag:
```

### <a id="sv-api-f-0283"></a>`SV-API-F-0283`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._create_or_update_targeting_flag (L2373-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When existing_flag is provided, sets request.method to PATCH and updates the flag's filters via update_flag.

```
if existing_flag:
```

### <a id="sv-api-f-0284"></a>`SV-API-F-0284`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._create_or_update_targeting_flag (L2373-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When existing_flag is not provided, sets request.method to POST, generates a slugified key with a 10-char hex ID and optional suffix, and atomically creates a FeatureFlag owned by FLAG_OWNER_SURVEY with creation_context 'surveys'.

```
feature_flag_key = slugify(f"{SURVEY_TARGETING_FLAG_PREFIX}{random_id}{flag_name_suffix or ''}")
```

### <a id="sv-api-f-0285"></a>`SV-API-F-0285`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnly._create_or_update_targeting_flag (L2373-2418)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Catches ApprovalRequired during targeting flag creation and raises serializers.ValidationError stating that a feature flag approval policy blocks saving the survey.

```
except ApprovalRequired:
```

### <a id="sv-api-f-0289"></a>`SV-API-F-0289`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveySerializerCreateUpdateOnlySchema.Meta.extra_kwargs (L2498-2521)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enforces min_length of 1 on the survey name field.

```
"min_length": 1,
```

### <a id="sv-api-f-0291"></a>`SV-API-F-0291`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyFilterSet (L2528-2558)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes query filters on Survey for ids (comma-separated UUIDs, omitting missing IDs), created_by, linked_flag_id, status (draft/running/complete), archived, and type.

```
class SurveyFilterSet(FilterSet):
```

### <a id="sv-api-f-0292"></a>`SV-API-F-0292`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet (L2561-3820)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Wires SurveyViewSet with ApprovalHandlingMixin, TeamAndOrgViewSetMixin, AccessControlViewSetMixin, ModelViewSet, DjangoFilterBackend, and SurveyFilterSet.

```
class SurveyViewSet(ApprovalHandlingMixin, TeamAndOrgViewSetMixin, AccessControlViewSetMixin, viewsets.ModelViewSet):
```

### <a id="sv-api-f-0293"></a>`SV-API-F-0293`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet (L2561-3820)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enforces access control on SurveyViewSet using scope_object = 'survey'.

```
scope_object = "survey"
```

### <a id="sv-api-f-0294"></a>`SV-API-F-0294`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet (L2561-3820)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries Survey with select_related for linked_flag, linked_insight, targeting_flag, and internal_targeting_flag.

```
queryset = Survey.objects.select_related(
```

### <a id="sv-api-f-0297"></a>`SV-API-F-0297`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet (L2561-3820)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Converts ValueError from validate_and_parse_dates into DRF exceptions.ValidationError.

```
raise exceptions.ValidationError(str(exc)) from exc
```

### <a id="sv-api-f-0299"></a>`SV-API-F-0299`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.safely_get_queryset (L2588-2602)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** On list action, raises serializers.ValidationError when the search query parameter exceeds MAX_SEARCH_LENGTH.

```
if len(search) > MAX_SEARCH_LENGTH:
```

### <a id="sv-api-f-0317"></a>`SV-API-F-0317`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.responses_count (L2760-2776)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes list GET action at /responses_count/ requiring survey:read scope.

```
@action(methods=["GET"], detail=False, required_scopes=["survey:read"])
```

### <a id="sv-api-f-0318"></a>`SV-API-F-0318`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.responses_count (L2760-2776)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads optional exclude_archived boolean query param (defaulting to false) and comma-separated survey_ids query param.

```
exclude_archived = request.query_params.get("exclude_archived", "false").lower() == "true"
```

### <a id="sv-api-f-0321"></a>`SV-API-F-0321`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet._get_survey_stats (L2792-2816)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Catches ValueError from get_survey_stats and raises DRF ValidationError.

```
raise exceptions.ValidationError(str(exc)) from exc
```

### <a id="sv-api-f-0322"></a>`SV-API-F-0322`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.survey_stats (L2818-2919)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes detail GET action at /stats/ requiring survey:read scope.

```
@action(methods=["GET"], detail=True, url_path="stats", required_scopes=["survey:read"])
```

### <a id="sv-api-f-0323"></a>`SV-API-F-0323`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.survey_stats (L2818-2919)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads optional query params date_from, date_to, exclude_archived (default false), and include_per_question_stats (default false).

```
include_per_question_stats = request.query_params.get("include_per_question_stats", "false").lower() == "true"
```

### <a id="sv-api-f-0324"></a>`SV-API-F-0324`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.survey_stats (L2818-2919)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises NotFound('Survey not found') when the requested survey does not exist.

```
raise exceptions.NotFound("Survey not found")
```

### <a id="sv-api-f-0339"></a>`SV-API-F-0339`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.site_snapshot (L2989-3014)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET detail action site_snapshot with pagination disabled

```
def site_snapshot(self, request: ValidatedRequest, **kwargs: Any) -> Response:
```

### <a id="sv-api-f-0340"></a>`SV-API-F-0340`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.site_snapshot (L2989-3014)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires survey:read, session_recording:read, and export:read scopes

```
required_scopes=["survey:read", "session_recording:read", "export:read"],
```

### <a id="sv-api-f-0341"></a>`SV-API-F-0341`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.site_snapshot (L2989-3014)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Validates query parameters (device and URL target) via SiteSnapshotQuerySerializer

```
query_serializer=SiteSnapshotQuerySerializer,
```

### <a id="sv-api-f-0343"></a>`SV-API-F-0343`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.archived_response_uuids (L3017-3027)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET detail action at archived-response-uuids requiring survey:read scope

```
@action(methods=["GET"], detail=True, url_path="archived-response-uuids", required_scopes=["survey:read"])
```

### <a id="sv-api-f-0345"></a>`SV-API-F-0345`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.survey_responses_list (L3029-3108)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET detail action at responses requiring survey:read and query:read scopes

```
@action(methods=["GET"], detail=True, url_path="responses", required_scopes=["survey:read", "query:read"])
```

### <a id="sv-api-f-0346"></a>`SV-API-F-0346`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.survey_responses_list (L3029-3108)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Validates query parameters with SurveyResponsesQuerySerializer and raises on invalid input

```
params = SurveyResponsesQuerySerializer(data=request.query_params)
```

### <a id="sv-api-f-0347"></a>`SV-API-F-0347`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.survey_responses_list (L3029-3108)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValidationError when score_lte or score_gte is provided without question_id

```
raise exceptions.ValidationError("score_lte and score_gte require question_id to be specified")
```

### <a id="sv-api-f-0351"></a>`SV-API-F-0351`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.question_labels (L3110-3176)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET list action at question_labels requiring survey:read scope

```
@action(methods=["GET"], detail=False, url_path="question_labels", required_scopes=["survey:read"])
```

### <a id="sv-api-f-0352"></a>`SV-API-F-0352`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.question_labels (L3110-3176)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Filters surveys by the caller's user access level via user_access_control.filter_queryset_by_access_level

```
queryset = self.user_access_control.filter_queryset_by_access_level(self.get_queryset())
```

### <a id="sv-api-f-0353"></a>`SV-API-F-0353`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.question_labels (L3110-3176)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Iterates Survey records (id, name, questions) from the database in chunks of 200

```
for survey in queryset.iterator(chunk_size=200):
```

### <a id="sv-api-f-0358"></a>`SV-API-F-0358`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.global_stats (L3178-3221)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET /api/projects/{}/surveys/stats action requiring survey:read scope.

```
@action(methods=["GET"], detail=False, url_path="stats", required_scopes=["survey:read"])
```

### <a id="sv-api-f-0359"></a>`SV-API-F-0359`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.global_stats (L3178-3221)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads optional date_from query parameter defaulting to None.

```
date_from = request.query_params.get("date_from", None)
```

### <a id="sv-api-f-0360"></a>`SV-API-F-0360`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.global_stats (L3178-3221)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads optional date_to query parameter defaulting to None.

```
date_to = request.query_params.get("date_to", None)
```

### <a id="sv-api-f-0362"></a>`SV-API-F-0362`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.all_activity (L3223-3232)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET /api/projects/{}/surveys/activity action requiring activity_log:read scope.

```
@action(methods=["GET"], url_path="activity", detail=False, required_scopes=["activity_log:read"])
```

### <a id="sv-api-f-0363"></a>`SV-API-F-0363`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.all_activity (L3223-3232)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Parses activity log pagination parameters (limit and page) from the request.

```
page_params = parse_activity_page_params(request)
```

### <a id="sv-api-f-0364"></a>`SV-API-F-0364`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.all_activity (L3223-3232)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Loads paginated Survey-scoped activity log entries for the current team.

```
scope="Survey", team_id=self.team_id, limit=page_params.limit, page=page_params.page
```

### <a id="sv-api-f-0365"></a>`SV-API-F-0365`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.activity (L3234-3251)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes GET /api/projects/{}/surveys/{}/activity detail action requiring activity_log:read scope.

```
@action(methods=["GET"], detail=True, required_scopes=["activity_log:read"])
```

### <a id="sv-api-f-0366"></a>`SV-API-F-0366`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.activity (L3234-3251)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Parses activity log pagination parameters (limit and page) from the request.

```
page_params = parse_activity_page_params(request)
```

### <a id="sv-api-f-0367"></a>`SV-API-F-0367`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.activity (L3234-3251)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Fetches the target Survey instance via self.get_object().

```
item = self.get_object()
```

### <a id="sv-api-f-0368"></a>`SV-API-F-0368`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/api/survey.py#SurveyViewSet.activity (L3234-3251)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Loads paginated Survey-scoped activity log entries filtered to the target survey's ID.

```
item_ids=[str(item.id)],
```

### <a id="sv-api-f-0474"></a>`SV-API-F-0474`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/api/survey.py#create_flag_with_survey_errors (L4224-4249)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Catches ValidationError and, when filters details include BEHAVIOURAL_COHORT_FOUND_ERROR_CODE, re-raises it replacing 'feature flags' with 'surveys' and appending event-trigger/static-cohort guidance; otherwise re-raises the original error

```
if matching_errors:
```

### <a id="sv-api-f-0541"></a>`SV-API-F-0541`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/internal_targeting.py#build_internal_targeting_groups (L32-94)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires the person property $survey_dismissed/{survey_key} to be not set in all generated release groups.

```
"key": f"{SurveyEventProperties.SURVEY_DISMISSED}/{survey_key}",
```

### <a id="sv-api-f-0542"></a>`SV-API-F-0542`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/internal_targeting.py#build_internal_targeting_groups (L32-94)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires the person property $survey_responded/{survey_key} to be not set in all generated release groups.

```
"key": f"{SurveyEventProperties.SURVEY_RESPONDED}/{survey_key}",
```

### <a id="sv-api-f-0544"></a>`SV-API-F-0544`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/internal_targeting.py#build_internal_targeting_groups (L32-94)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When wait_period_days is set, returns two 100%-rollout release groups combining base_properties with SURVEY_LAST_SEEN_DATE either is_not_set or is_date_before '{wait_period_days}d'.

```
"value": f"{wait_period_days}d",
```

### <a id="sv-api-f-0557"></a>`SV-API-F-0557`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#MAX_ITERATION_COUNT (L26-26)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maximum survey iteration count is capped at 500

```
MAX_ITERATION_COUNT = 500
```

### <a id="sv-api-f-0558"></a>`SV-API-F-0558`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#MAX_ITERATION_FREQUENCY_DAYS (L27-27)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maximum survey iteration frequency is capped at 365 days

```
MAX_ITERATION_FREQUENCY_DAYS = 365
```

### <a id="sv-api-f-0559"></a>`SV-API-F-0559`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Survey model inherits FileSystemSyncMixin, RootTeamMixin, and UUIDTModel

```
class Survey(FileSystemSyncMixin, RootTeamMixin, UUIDTModel):
```

### <a id="sv-api-f-0560"></a>`SV-API-F-0560`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Persists Survey records in the posthog_survey database table

```
db_table = "posthog_survey"
```

### <a id="sv-api-f-0561"></a>`SV-API-F-0561`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defines survey types: popover, widget, external_survey, and api

```
class SurveyType(models.TextChoices):
```

### <a id="sv-api-f-0562"></a>`SV-API-F-0562`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defines response sampling interval types: day, week, and month

```
class SurveySamplingIntervalType(models.TextChoices):
```

### <a id="sv-api-f-0563"></a>`SV-API-F-0563`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defines survey schedule modes: once, recurring, and always

```
class Schedule(models.TextChoices):
```

### <a id="sv-api-f-0564"></a>`SV-API-F-0564`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enforces uniqueness of survey name per team

```
constraints = [models.UniqueConstraint(fields=["team", "name"], name="unique survey name for team")]
```

### <a id="sv-api-f-0565"></a>`SV-API-F-0565`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Survey name is limited to 400 characters

```
name = models.CharField(max_length=400)
```

### <a id="sv-api-f-0566"></a>`SV-API-F-0566`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Survey type is limited to 40 characters and constrained to SurveyType choices

```
type = models.CharField(max_length=40, choices=SurveyType)
```

### <a id="sv-api-f-0567"></a>`SV-API-F-0567`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Survey archived flag defaults to False

```
archived = models.BooleanField(default=False)
```

### <a id="sv-api-f-0568"></a>`SV-API-F-0568`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Optional positive integer cap on total survey responses via responses_limit

```
responses_limit = models.PositiveIntegerField(null=True)
```

### <a id="sv-api-f-0569"></a>`SV-API-F-0569`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** response_sampling_interval_type is limited to 6 characters

```
max_length=6,
```

### <a id="sv-api-f-0570"></a>`SV-API-F-0570`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** response_sampling_interval_type defaults to week

```
default=SurveySamplingIntervalType.WEEK,
```

### <a id="sv-api-f-0571"></a>`SV-API-F-0571`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When repeat_until_stopped is true on a recurring schedule, iteration_count is ignored and the survey never closes on its own after a final repeat

```
"When true, iteration_count is ignored and the survey never closes on its own after a final repeat."
```

### <a id="sv-api-f-0572"></a>`SV-API-F-0572`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Survey schedule field is limited to 40 characters

```
max_length=40,
```

### <a id="sv-api-f-0573"></a>`SV-API-F-0573`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Survey schedule defaults to once

```
default=Schedule.ONCE,
```

### <a id="sv-api-f-0574"></a>`SV-API-F-0574`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** enable_partial_responses defaults to False

```
enable_partial_responses = models.BooleanField(default=False, null=True)
```

### <a id="sv-api-f-0575"></a>`SV-API-F-0575`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** enable_iframe_embedding defaults to False

```
enable_iframe_embedding = models.BooleanField(default=False)
```

### <a id="sv-api-f-0576"></a>`SV-API-F-0576`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#Survey (L42-395)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** base_language is limited to 20 characters and defaults to "en"

```
base_language = models.CharField(max_length=20, default="en")
```

### <a id="sv-api-f-0579"></a>`SV-API-F-0579`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#Survey.repeats_on_schedule (L325-332)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns False when iteration_frequency_days is unset or zero.

```
if not self.iteration_frequency_days:
```

### <a id="sv-api-f-0580"></a>`SV-API-F-0580`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/models.py#Survey.repeats_on_schedule (L325-332)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns True when iteration_count is set or when repeat_until_stopped is true and schedule is Schedule.RECURRING.

```
return bool(self.iteration_count) or (
```

### <a id="sv-api-f-0588"></a>`SV-API-F-0588`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#update_response_sampling_limits (L398-443)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clears response_sampling_daily_limits to None and returns when response_sampling_interval_type is None, response_sampling_limit is 0, or response_sampling_start_date is None.

```
instance.response_sampling_interval_type is None
```

### <a id="sv-api-f-0589"></a>`SV-API-F-0589`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#update_response_sampling_limits (L398-443)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Calculates total_days from response_sampling_interval using 1 day for 'day', 7 days for 'week', and 30 days for 'month'.

```
if instance.response_sampling_interval_type == "day":
```

### <a id="sv-api-f-0590"></a>`SV-API-F-0590`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/models.py#update_response_sampling_limits (L398-443)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Builds a daily cumulative response_sampling_limit and rollout_percentage schedule (rounded to 2 decimals, final day forced to 100.0) and stores it as JSON on response_sampling_daily_limits.

```
instance.response_sampling_daily_limits = json.dumps(schedule)
```

### <a id="sv-api-f-0591"></a>`SV-API-F-0591`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/models.py#pre_save_survey (L446-450)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Connects pre_save signal on Survey to run update_survey_iterations, update_response_sampling_limits, and ensure_question_ids.

```
@receiver(pre_save, sender=Survey)
```

### <a id="sv-api-f-0592"></a>`SV-API-F-0592`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#ensure_question_ids (L453-463)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns early without modifying questions when instance.questions is empty or falsy.

```
if not instance.questions:
```

### <a id="sv-api-f-0593"></a>`SV-API-F-0593`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#ensure_question_ids (L453-463)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Generates and assigns a UUID4 string id to any question missing an id.

```
if not question.get("id"):
```

### <a id="sv-api-f-0594"></a>`SV-API-F-0594`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#update_survey_iterations (L466-493)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Resets iteration_start_dates to [] and current_iteration and current_iteration_start_date to None when repeats_on_schedule is False.

```
if not instance.repeats_on_schedule:
```

### <a id="sv-api-f-0595"></a>`SV-API-F-0595`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#update_survey_iterations (L466-493)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets iteration_start_dates to None and returns when start_date is None.

```
if instance.start_date is None:
```

### <a id="sv-api-f-0596"></a>`SV-API-F-0596`

- **Surface:** `API` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/models.py#update_survey_iterations (L466-493)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** For repeat_until_stopped surveys, computes current iteration via open_ended_iteration and generates a 2-date rrule window starting from the current iteration start date.

```
if instance.repeat_until_stopped:
```

### <a id="sv-api-f-0597"></a>`SV-API-F-0597`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#update_survey_iterations (L466-493)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** For fixed-count recurring surveys, caps the generated iteration_start_dates window at MAX_ITERATION_COUNT.

```
window_count = min(instance.iteration_count, MAX_ITERATION_COUNT)
```

### <a id="sv-api-f-0598"></a>`SV-API-F-0598`

- **Surface:** `API` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/models.py#update_survey_iterations (L466-493)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Initializes current_iteration to 1 and current_iteration_start_date to start_date when current_iteration is None or 0.

```
if instance.current_iteration is None or instance.current_iteration == 0:
```

### <a id="sv-api-f-0599"></a>`SV-API-F-0599`

- **Surface:** `API` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/models.py#open_ended_iteration (L496-497)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Computes 1-based iteration index from elapsed days divided by frequency_days, clamped to a minimum of 1.

```
return max((now - start_date).days // frequency_days + 1, 1)
```

### <a id="sv-api-f-0600"></a>`SV-API-F-0600`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/models.py#survey_changed (L514-525)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Registers survey_changed as a Django post_save signal receiver for Survey.

```
@receiver(post_save, sender=Survey)
```

### <a id="sv-api-f-0601"></a>`SV-API-F-0601`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/models.py#survey_changed (L514-525)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Registers survey_changed as a Django post_delete signal receiver for Survey.

```
@receiver(post_delete, sender=Survey)
```

### <a id="sv-api-f-0603"></a>`SV-API-F-0603`

- **Surface:** `API` | **Type:** `wiring` | **Source:** `posthog:products/surveys/backend/models.py#survey_changed (L514-525)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enqueues update_team_flags_cache for instance.team_id on database transaction commit.

```
transaction.on_commit(lambda: update_team_flags_cache.delay(instance.team_id))
```

### <a id="sv-api-f-0671"></a>`SV-API-F-0671`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/responses/fetch_rows.py#RESPONSE_EVENT_FILTER (L125-128)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Filters HogQL events to 'survey sent', or 'survey dismissed' / 'survey abandoned' events.

```
RESPONSE_EVENT_FILTER = """(event = 'survey sent' OR (
```

### <a id="sv-api-f-0722"></a>`SV-API-F-0722`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/responses/stats.py#StatsWindow (L61-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Raises ValueError('date_from must be before date_to') when both date_from and date_to are set and date_from > date_to.

```
raise ValueError("date_from must be before date_to")
```

### <a id="sv-api-f-0728"></a>`SV-API-F-0728`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/responses/stats.py#validate_and_parse_dates (L93-110)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Catches ValueError and OverflowError during date parsing and raises ValueError with an invalid date format message.

```
except (ValueError, OverflowError) as exc:
```

### <a id="sv-api-f-0733"></a>`SV-API-F-0733`

- **Surface:** `API` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/responses/stats.py#_latest_response_event_uuids_subquery (L202-217)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Builds a HogQL subquery selecting the latest response event UUID per (properties.$survey_id, SUBMISSION_GROUPING_KEY) from events filtered by RESPONSE_EVENT_FILTER and scan_conditions.

```
SELECT argMax(uuid, tuple(timestamp, uuid))
```

### <a id="sv-api-f-0734"></a>`SV-API-F-0734`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/responses/stats.py#get_survey_stats (L220-387)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Loads the Team record by primary key team_id.

```
team = Team.objects.get(pk=team_id)
```

### <a id="sv-api-f-0737"></a>`SV-API-F-0737`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/responses/stats.py#get_survey_stats (L220-387)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When survey_id is provided, queries Survey for start_date, created_at, and end_date.

```
Survey.objects.filter(team_id=team_id, id=survey_id).values("start_date", "created_at", "end_date").first()
```

### <a id="sv-api-f-0745"></a>`SV-API-F-0745`

- **Surface:** `API` | **Type:** `storage` | **Source:** `posthog:products/surveys/backend/responses/stats.py#get_survey_stats (L220-387)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When survey_id is omitted, loads IDs of all non-archived surveys for the team.

```
active_survey_ids = list(Survey.objects.filter(team_id=team_id, archived=False).values_list("id", flat=True))
```

### <a id="sv-api-f-0751"></a>`SV-API-F-0751`

- **Surface:** `API` | **Type:** `port` | **Source:** `posthog:products/surveys/backend/responses/stats.py#get_survey_stats (L220-387)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Executes the survey_stats_base HogQL query to aggregate shown, dismissed, and sent survey events.

```
query_type="survey_stats_base",
```

### <a id="sv-api-f-0752"></a>`SV-API-F-0752`

- **Surface:** `API` | **Type:** `port` | **Source:** `posthog:products/surveys/backend/responses/stats.py#get_survey_stats (L220-387)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Executes the survey_stats_dismissed_and_sent HogQL query to count persons who both dismissed and sent a response.

```
query_type="survey_stats_dismissed_and_sent",
```

### <a id="sv-api-f-0855"></a>`SV-API-F-0855`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#surveys_endpoint (L44-94)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Validates and extracts the project token via extract_token, returning its 400 or 401 error response on failure.

```
let token = match extract_token(&params, &method, &body) {
```

### <a id="sv-api-f-0860"></a>`SV-API-F-0860`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#extract_token (L99-135)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads the token from query parameter token or fallback api_key, ignoring empty strings.

```
.or(params.api_key.as_deref())
```

### <a id="sv-api-f-0861"></a>`SV-API-F-0861`

- **Surface:** `API` | **Type:** `input` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#extract_token (L99-135)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** For POST requests without a query token, parses URL-encoded form body for token or fallback api_key (ignoring empty strings) and returns 401 Unauthorized if missing.

```
let owned = form.token.or(form.api_key).filter(|s| !s.is_empty());
```

### <a id="sv-api-f-0862"></a>`SV-API-F-0862`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#extract_token (L99-135)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns 401 Unauthorized with "Token not provided" when no non-empty token is supplied.

```
return Err((StatusCode::UNAUTHORIZED, "Token not provided").into_response());
```

### <a id="sv-api-f-0863"></a>`SV-API-F-0863`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#extract_token (L99-135)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Validates the extracted token string with Token::parse and maps failures via token_error_response.

```
Token::parse(raw).map_err(token_error_response)
```

### <a id="sv-api-f-0864"></a>`SV-API-F-0864`

- **Surface:** `API` | **Type:** `guard` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#token_error_response (L137-143)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maps TokenError::Empty to HTTP 401 Unauthorized with the error message body.

```
TokenError::Empty => StatusCode::UNAUTHORIZED,
```

### <a id="sv-api-f-0865"></a>`SV-API-F-0865`

- **Surface:** `API` | **Type:** `error` | **Source:** `posthog:rust/hypercache-server/src/api/surveys.rs#token_error_response (L137-143)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maps TokenError::TooLong and TokenError::InvalidCharacters to HTTP 400 Bad Request with the error message body.

```
TokenError::TooLong | TokenError::InvalidCharacters => StatusCode::BAD_REQUEST,
```

### <a id="sv-flt-f-0086"></a>`SV-FLT-F-0086`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#surveyPopupDelay (L17-24)` | **Pin:** `df97547d612967e99784`
- **Claim:** Returns Duration.zero (no popup delay) when seconds is null, NaN, or less than or equal to 0.

```
if (seconds == null || seconds.isNaN || seconds <= 0) {
```

### <a id="sv-flt-f-0087"></a>`SV-FLT-F-0087`

- **Surface:** `FLT` | **Type:** `limit` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#surveyPopupDelay (L17-24)` | **Pin:** `df97547d612967e99784`
- **Claim:** Caps the survey popup delay at a maximum of 1 day (Duration(days: 1)).

```
if (seconds >= maxDelay.inSeconds) return maxDelay;
```

### <a id="sv-flt-f-0088"></a>`SV-FLT-F-0088`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#surveyPopupDelay (L17-24)` | **Pin:** `df97547d612967e99784`
- **Claim:** Converts positive popup delay seconds under the 1-day cap to rounded milliseconds.

```
return Duration(milliseconds: (seconds * 1000).round());
```

### <a id="sv-flt-f-0089"></a>`SV-FLT-F-0089`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService (L30-252)` | **Pin:** `df97547d612967e99784`
- **Claim:** SurveyService is a singleton returning a single shared _instance from its factory constructor.

```
static final SurveyService _instance = SurveyService._internal();
```

### <a id="sv-flt-f-0090"></a>`SV-FLT-F-0090`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService (L30-252)` | **Pin:** `df97547d612967e99784`
- **Claim:** onContextAvailable() completes a pending survey show with true only when no popup delay timer is active.

```
if (_popupDelayTimer == null) _endPendingShow(show: true);
```

### <a id="sv-flt-f-0091"></a>`SV-FLT-F-0091`

- **Surface:** `FLT` | **Type:** `guard` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService.showSurvey (L48-88)` | **Pin:** `df97547d612967e99784`
- **Claim:** Logs a debug message and returns early without showing a survey if _isShowingSurvey is already true.

```
if (_isShowingSurvey) {
```

### <a id="sv-flt-f-0092"></a>`SV-FLT-F-0092`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService.showSurvey (L48-88)` | **Pin:** `df97547d612967e99784`
- **Claim:** Computes popup delay from survey.appearance?.surveyPopupDelaySeconds and aborts if the delay wait is cancelled.

```
if (delay > Duration.zero && !await _waitForPopupDelay(survey, delay)) {
```

### <a id="sv-flt-f-0093"></a>`SV-FLT-F-0093`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService.showSurvey (L48-88)` | **Pin:** `df97547d612967e99784`
- **Claim:** When PosthogObserver.currentContext is null, waits for context availability in a loop and aborts if cancelled or another survey starts showing.

```
if (!await _waitForContext() || _isShowingSurvey) return;
```

### <a id="sv-flt-f-0094"></a>`SV-FLT-F-0094`

- **Surface:** `FLT` | **Type:** `guard` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService.showSurvey (L48-88)` | **Pin:** `df97547d612967e99784`
- **Claim:** Aborts showing the survey if the obtained BuildContext is no longer mounted.

```
if (!context.mounted) return;
```

### <a id="sv-flt-f-0095"></a>`SV-FLT-F-0095`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService.showSurvey (L48-88)` | **Pin:** `df97547d612967e99784`
- **Claim:** Delegates displaying the survey to _showSurveyWithNavigator once a mounted PosthogObserver context is available.

```
return _showSurveyWithNavigator(
```

### <a id="sv-flt-f-0096"></a>`SV-FLT-F-0096`

- **Surface:** `FLT` | **Type:** `schedule` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService._waitForPopupDelay (L92-103)` | **Pin:** `df97547d612967e99784`
- **Claim:** Starts a Timer for the popup delay that clears the delayed survey state and completes the pending show with true.

```
_popupDelayTimer = Timer(delay, () {
```

### <a id="sv-flt-f-0097"></a>`SV-FLT-F-0097`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService._startPendingShow (L109-114)` | **Pin:** `df97547d612967e99784`
- **Claim:** Cancels any prior pending show with false before creating and returning a new Completer<bool> future.

```
_endPendingShow(show: false);
```

### <a id="sv-flt-f-0098"></a>`SV-FLT-F-0098`

- **Surface:** `FLT` | **Type:** `port` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService._showSurveyWithNavigator (L129-184)` | **Pin:** `df97547d612967e99784`
- **Claim:** Presents the survey in a non-dismissible, scroll-controlled modal bottom sheet on the root navigator.

```
final modalDismissal = showModalBottomSheet<void>(
```

### <a id="sv-flt-f-0103"></a>`SV-FLT-F-0103`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/survey_service.dart#SurveyService._buildSurveyWidget (L187-200)` | **Pin:** `df97547d612967e99784`
- **Claim:** Builds a SurveyBottomSheet widget configured with survey callbacks and SurveyAppearance.fromPostHog(survey.appearance).

```
appearance: SurveyAppearance.fromPostHog(survey.appearance),
```

### <a id="sv-flt-f-0116"></a>`SV-FLT-F-0116`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState (L44-213)` | **Pin:** `df97547d612967e99784`
- **Claim:** Auto-submit on tap is enabled only when skipSubmitButton is true, isMultipleChoice is false, and hasOpenChoice is false.

```
widget.skipSubmitButton &&
```

### <a id="sv-flt-f-0117"></a>`SV-FLT-F-0117`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState (L44-213)` | **Pin:** `df97547d612967e99784`
- **Claim:** When hasOpenChoice is enabled, the last item in choices is treated as the open-choice option.

```
return widget.hasOpenChoice && choice == widget.choices.last;
```

### <a id="sv-flt-f-0119"></a>`SV-FLT-F-0119`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._createDisplayOrder (L48-64)` | **Pin:** `df97547d612967e99784`
- **Claim:** Preserves the original choice order when shuffleOptions is false.

```
if (!widget.shuffleOptions) return indices;
```

### <a id="sv-flt-f-0120"></a>`SV-FLT-F-0120`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._createDisplayOrder (L48-64)` | **Pin:** `df97547d612967e99784`
- **Claim:** Keeps the open-choice option pinned at the end rather than shuffling it with regular choices when hasOpenChoice is true.

```
final openChoice = widget.hasOpenChoice && indices.isNotEmpty
```

### <a id="sv-flt-f-0121"></a>`SV-FLT-F-0121`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._createDisplayOrder (L48-64)` | **Pin:** `df97547d612967e99784`
- **Claim:** Reverses the shuffled indices if shuffling yields the same choice label order as the unshuffled list.

```
final unchanged = listEquals(
```

### <a id="sv-flt-f-0122"></a>`SV-FLT-F-0122`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._canSubmit (L80-89)` | **Pin:** `df97547d612967e99784`
- **Claim:** Allows submission regardless of selection state when optional is true.

```
if (widget.optional) return true;
```

### <a id="sv-flt-f-0123"></a>`SV-FLT-F-0123`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._canSubmit (L80-89)` | **Pin:** `df97547d612967e99784`
- **Claim:** Blocks submission when no choices are selected and the question is not optional.

```
if (_selectedChoices.isEmpty) return false;
```

### <a id="sv-flt-f-0124"></a>`SV-FLT-F-0124`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._canSubmit (L80-89)` | **Pin:** `df97547d612967e99784`
- **Claim:** Blocks submission when hasOpenChoice is true, the open-choice option is selected, and its trimmed text input is empty.

```
if (widget.hasOpenChoice &&
```

### <a id="sv-flt-f-0125"></a>`SV-FLT-F-0125`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._onSubmit (L95-112)` | **Pin:** `df97547d612967e99784`
- **Claim:** Ignores submission attempts when _canSubmit is false.

```
if (!_canSubmit) return;
```

### <a id="sv-flt-f-0126"></a>`SV-FLT-F-0126`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._onSubmit (L95-112)` | **Pin:** `df97547d612967e99784`
- **Claim:** Substitutes the trimmed open-choice text input for the open-choice label in the submitted result.

```
if (_isOpenChoice(choice)) {
```

### <a id="sv-flt-f-0127"></a>`SV-FLT-F-0127`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState._onSubmit (L95-112)` | **Pin:** `df97547d612967e99784`
- **Claim:** Calls widget.onSubmit with null if no choices were selected, or with the list of selected answer strings.

```
widget.onSubmit(result.isEmpty ? null : result);
```

### <a id="sv-flt-f-0128"></a>`SV-FLT-F-0128`

- **Surface:** `FLT` | **Type:** `limit` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState.build (L121-212)` | **Pin:** `df97547d612967e99784`
- **Claim:** Constrains the scrollable choice list maxHeight to screenHeight - 350 - keyboardHeight - bottomPadding.

```
final headerFooterHeight = 350; //estimated
```

### <a id="sv-flt-f-0130"></a>`SV-FLT-F-0130`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState.build (L121-212)` | **Pin:** `df97547d612967e99784`
- **Claim:** Tapping a choice immediately selects it and submits the question when _shouldAutoSubmit is true.

```
if (_shouldAutoSubmit) {
```

### <a id="sv-flt-f-0131"></a>`SV-FLT-F-0131`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState.build (L121-212)` | **Pin:** `df97547d612967e99784`
- **Claim:** Toggles individual choices in _selectedChoices when isMultipleChoice is true, or toggles single selection otherwise.

```
if (widget.isMultipleChoice) {
```

### <a id="sv-flt-f-0132"></a>`SV-FLT-F-0132`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState.build (L121-212)` | **Pin:** `df97547d612967e99784`
- **Claim:** Omits the submit button when _shouldAutoSubmit is true.

```
if (!_shouldAutoSubmit) ...[
```

### <a id="sv-flt-f-0133"></a>`SV-FLT-F-0133`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/choice_question.dart#_ChoiceQuestionWidgetState.build (L121-212)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders the submit SurveyButton (labeled by surveyQuestionButtonLabel) enabled only when _canSubmit is true.

```
onPressed: _canSubmit ? _onSubmit : null,
```

### <a id="sv-flt-f-0135"></a>`SV-FLT-F-0135`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/confirmation_message.dart#ConfirmationMessage.build (L20-55)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders appearance.thankYouMessageHeader as centered bold 18px text colored with appearance.descriptionTextColor.

```
appearance.thankYouMessageHeader,
```

### <a id="sv-flt-f-0136"></a>`SV-FLT-F-0136`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/confirmation_message.dart#ConfirmationMessage.build (L20-55)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders appearance.thankYouMessageDescription as centered 16px text only when it is non-empty and thankYouMessageDescriptionContentType is PostHogDisplaySurveyTextContentType.text.

```
if (appearance.thankYouMessageDescription?.isNotEmpty == true &&
```

### <a id="sv-flt-f-0137"></a>`SV-FLT-F-0137`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/confirmation_message.dart#ConfirmationMessage.build (L20-55)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders a SurveyButton labeled with appearance.thankYouMessageCloseButtonText that triggers onClose when pressed.

```
text: appearance.thankYouMessageCloseButtonText,
```

### <a id="sv-flt-f-0139"></a>`SV-FLT-F-0139`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/intro_message.dart#IntroMessage.build (L22-58)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders appearance.introScreenHeader as centered bold 18px Text using appearance.descriptionTextColor only when introScreenHeader is non-null and non-empty.

```
if (appearance.introScreenHeader?.isNotEmpty == true)
```

### <a id="sv-flt-f-0140"></a>`SV-FLT-F-0140`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/intro_message.dart#IntroMessage.build (L22-58)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders appearance.introScreenDescription as centered 16px Text using appearance.descriptionTextColor only when introScreenDescription is non-null and non-empty and introScreenDescriptionContentType is PostHogDisplaySurveyTextContentType.text.

```
if (appearance.introScreenDescription?.isNotEmpty == true &&
```

### <a id="sv-flt-f-0141"></a>`SV-FLT-F-0141`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/intro_message.dart#IntroMessage.build (L22-58)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders a SurveyButton labeled with appearance.introScreenButtonText that invokes the onStart callback when pressed.

```
text: appearance.introScreenButtonText,
```

### <a id="sv-flt-f-0142"></a>`SV-FLT-F-0142`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/link_question.dart#LinkQuestion.build (L30-52)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders a QuestionHeader displaying the question, optional description, descriptionContentType, and appearance in a vertically stacked Column.

```
QuestionHeader(
```

### <a id="sv-flt-f-0153"></a>`SV-FLT-F-0153`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState (L32-109)` | **Pin:** `df97547d612967e99784`
- **Claim:** When widget.optional is true, _canSubmit always returns true regardless of whether a response was entered.

```
if (widget.optional) return true;
```

### <a id="sv-flt-f-0154"></a>`SV-FLT-F-0154`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState (L32-109)` | **Pin:** `df97547d612967e99784`
- **Claim:** When the question is required, _canSubmit returns true only if _response is non-empty.

```
return _response?.isNotEmpty == true;
```

### <a id="sv-flt-f-0155"></a>`SV-FLT-F-0155`

- **Surface:** `FLT` | **Type:** `limit` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState.build (L41-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** Caps the text field container maxHeight to 70% of available height (screen height minus bottom viewInsets, bottom padding, and 150.0 reserved space).

```
final textFieldMaxHeight = (availableHeight - reservedSpace) * 0.7;
```

### <a id="sv-flt-f-0156"></a>`SV-FLT-F-0156`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState.build (L41-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders QuestionHeader with widget.question, widget.description, widget.descriptionContentType, and widget.appearance.

```
question: widget.question,
```

### <a id="sv-flt-f-0157"></a>`SV-FLT-F-0157`

- **Surface:** `FLT` | **Type:** `limit` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState.build (L41-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** Enforces a minimum height of 80 on the text input container.

```
minHeight: 80,
```

### <a id="sv-flt-f-0158"></a>`SV-FLT-F-0158`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState.build (L41-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders a scrollable multi-line TextField (minLines: 2, maxLines: null) with placeholder hintText from widget.appearance.inputPlaceholder.

```
hintText: widget.appearance.inputPlaceholder,
```

### <a id="sv-flt-f-0159"></a>`SV-FLT-F-0159`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState.build (L41-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** Updates _response via setState whenever the TextField value changes.

```
_response = value;
```

### <a id="sv-flt-f-0160"></a>`SV-FLT-F-0160`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/open_text_question.dart#_OpenTextQuestionState.build (L41-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders SurveyButton labeled widget.buttonText, enabled by _canSubmit, which trims _response (defaulting to "") and invokes widget.onSubmit on press.

```
onPressed: () => widget.onSubmit(_response?.trim() ?? ""),
```

### <a id="sv-flt-f-0161"></a>`SV-FLT-F-0161`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/question_header.dart#QuestionHeader.build (L20-48)` | **Pin:** `df97547d612967e99784`
- **Claim:** When question is non-null, renders the question text in bold 18pt font colored with appearance.questionTextColor inside a start-aligned Column.

```
if (question != null) ...[
```

### <a id="sv-flt-f-0162"></a>`SV-FLT-F-0162`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/question_header.dart#QuestionHeader.build (L20-48)` | **Pin:** `df97547d612967e99784`
- **Claim:** When description is non-empty and descriptionContentType equals PostHogDisplaySurveyTextContentType.text, renders an 8px vertical spacer followed by the description text in 16pt font colored with appearance.descriptionTextColor.

```
if (description?.isNotEmpty == true &&
```

### <a id="sv-flt-f-0176"></a>`SV-FLT-F-0176`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState (L46-251)` | **Pin:** `df97547d612967e99784`
- **Claim:** _canSubmit returns true when widget.optional is true; otherwise requires _rating != null

```
if (widget.optional) return true;
```

### <a id="sv-flt-f-0177"></a>`SV-FLT-F-0177`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._ratingRange (L54-60)` | **Pin:** `df97547d612967e99784`
- **Claim:** _ratingRange generates an inclusive integer list from scaleLowerBound to scaleUpperBound

```
widget.scaleUpperBound - widget.scaleLowerBound + 1,
```

### <a id="sv-flt-f-0178"></a>`SV-FLT-F-0178`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._getRatingIconType (L62-107)` | **Pin:** `df97547d612967e99784`
- **Claim:** For a 2-point scale, maps index 0 to thumbsUp and index 1 to thumbsDown (defaulting to thumbsUp)

```
if (range == 2) {
```

### <a id="sv-flt-f-0179"></a>`SV-FLT-F-0179`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._getRatingIconType (L62-107)` | **Pin:** `df97547d612967e99784`
- **Claim:** For a 3-point scale, maps indices 0-2 to dissatisfied, neutral, and satisfied (defaulting to neutral)

```
} else if (range == 3) {
```

### <a id="sv-flt-f-0180"></a>`SV-FLT-F-0180`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._getRatingIconType (L62-107)` | **Pin:** `df97547d612967e99784`
- **Claim:** For a 5-point scale, maps indices 0-4 to veryDissatisfied through verySatisfied (defaulting to neutral)

```
} else if (range == 5) {
```

### <a id="sv-flt-f-0181"></a>`SV-FLT-F-0181`

- **Surface:** `FLT` | **Type:** `default` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._getRatingIconType (L62-107)` | **Pin:** `df97547d612967e99784`
- **Claim:** Falls back to RatingIconType.neutral for any rating range other than 2, 3, or 5

```
return RatingIconType.neutral;
```

### <a id="sv-flt-f-0182"></a>`SV-FLT-F-0182`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._buildRatingButton (L109-160)` | **Pin:** `df97547d612967e99784`
- **Claim:** When widget.skipSubmitButton is true, tapping a rating button sets _rating and immediately calls widget.onSubmit(value)

```
if (widget.skipSubmitButton) {
```

### <a id="sv-flt-f-0183"></a>`SV-FLT-F-0183`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._buildRatingButton (L109-160)` | **Pin:** `df97547d612967e99784`
- **Claim:** When skipSubmitButton is false, tapping the currently selected rating clears _rating to null; otherwise selects it

```
if (_rating == value) {
```

### <a id="sv-flt-f-0184"></a>`SV-FLT-F-0184`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._buildRatingButton (L109-160)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders an InkWell with RatingIcon (dimmed to alpha 128 when unselected) when type is emoji and range is 2, 3, or 5

```
if (widget.type == PostHogDisplaySurveyRatingType.emoji &&
```

### <a id="sv-flt-f-0185"></a>`SV-FLT-F-0185`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState._buildRatingButton (L109-160)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders NumberRatingButton for number rating type or unsupported emoji ranges

```
return NumberRatingButton(
```

### <a id="sv-flt-f-0186"></a>`SV-FLT-F-0186`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState.build (L163-250)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders QuestionHeader with question, description, descriptionContentType, and appearance

```
QuestionHeader(
```

### <a id="sv-flt-f-0187"></a>`SV-FLT-F-0187`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState.build (L163-250)` | **Pin:** `df97547d612967e99784`
- **Claim:** Lays out emoji rating buttons in a Row (spaceAround for 2-point thumb surveys, spaceBetween otherwise), or number buttons inside a bordered Container

```
if (widget.type == PostHogDisplaySurveyRatingType.emoji)
```

### <a id="sv-flt-f-0188"></a>`SV-FLT-F-0188`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState.build (L163-250)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders lower/upper bound label row only when at least one bound label is non-null and it is not a 2-point thumb survey

```
if ((widget.lowerBoundLabel != null ||
```

### <a id="sv-flt-f-0191"></a>`SV-FLT-F-0191`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState.build (L163-250)` | **Pin:** `df97547d612967e99784`
- **Claim:** Renders the submit SurveyButton only when !widget.skipSubmitButton

```
if (!widget.skipSubmitButton) ...[
```

### <a id="sv-flt-f-0192"></a>`SV-FLT-F-0192`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/rating_question.dart#_RatingQuestionState.build (L163-250)` | **Pin:** `df97547d612967e99784`
- **Claim:** Submit SurveyButton (labeled via surveyQuestionButtonLabel from buttonText or appearance.submitButtonText) invokes widget.onSubmit(rating) when _canSubmit is true and is disabled otherwise

```
onPressed: _canSubmit ? () => widget.onSubmit(rating) : null,
```

### <a id="sv-flt-f-0193"></a>`SV-FLT-F-0193`

- **Surface:** `FLT` | **Type:** `default` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState (L42-288)` | **Pin:** `df97547d612967e99784`
- **Claim:** Initializes _currentIndex to widget.survey.initialQuestionIndex.

```
late int _currentIndex = widget.survey.initialQuestionIndex;
```

### <a id="sv-flt-f-0194"></a>`SV-FLT-F-0194`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState (L42-288)` | **Pin:** `df97547d612967e99784`
- **Claim:** Shows the intro screen initially only when initialQuestionIndex is 0, appearance.displayIntroScreen is true, and either introScreenHeader or introScreenDescription is non-empty.

```
late bool _showingIntroScreen = widget.survey.initialQuestionIndex == 0 &&
```

### <a id="sv-flt-f-0195"></a>`SV-FLT-F-0195`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState (L42-288)` | **Pin:** `df97547d612967e99784`
- **Claim:** Calls widget.onShown(widget.survey) in initState when the survey bottom sheet is initialized.

```
widget.onShown(widget.survey);
```

### <a id="sv-flt-f-0197"></a>`SV-FLT-F-0197`

- **Surface:** `FLT` | **Type:** `guard` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** Ignores _submitResponse calls if a submission is already in progress (_isSubmitting) or the survey is already completed (_isCompleted).

```
if (_isSubmitting || _isCompleted) return;
```

### <a id="sv-flt-f-0198"></a>`SV-FLT-F-0198`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** Awaits widget.onResponse(widget.survey, _currentIndex, response) and aborts early if the state is unmounted or nextQuestion is null.

```
if (!mounted || nextQuestion == null) return;
```

### <a id="sv-flt-f-0199"></a>`SV-FLT-F-0199`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** When link is non-null and non-empty, opens the URL via PosthogFlutterPlatformInterface.instance.openUrl(link).

```
await PosthogFlutterPlatformInterface.instance.openUrl(link);
```

### <a id="sv-flt-f-0200"></a>`SV-FLT-F-0200`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** Returns early if the widget is no longer mounted after opening the link.

```
if (!mounted) return;
```

### <a id="sv-flt-f-0201"></a>`SV-FLT-F-0201`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** If nextQuestion.isSurveyCompleted is true and widget.appearance.displayThankYouMessage is false, closes the bottom sheet immediately via _handleClose() instead of showing the confirmation message.

```
if (nextQuestion.isSurveyCompleted &&
```

### <a id="sv-flt-f-0202"></a>`SV-FLT-F-0202`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** Updates _currentIndex to nextQuestion.questionIndex and _isCompleted to nextQuestion.isSurveyCompleted via setState.

```
_currentIndex = nextQuestion.questionIndex;
```

### <a id="sv-flt-f-0203"></a>`SV-FLT-F-0203`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._submitResponse (L67-97)` | **Pin:** `df97547d612967e99784`
- **Claim:** Resets _isSubmitting to false in the finally block if the widget is still mounted.

```
if (mounted) setState(() => _isSubmitting = false);
```

### <a id="sv-flt-f-0204"></a>`SV-FLT-F-0204`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildQuestion (L99-178)` | **Pin:** `df97547d612967e99784`
- **Claim:** Switches on currentQuestion.type to render OpenTextQuestion, LinkQuestion, RatingQuestion, or ChoiceQuestionWidget.

```
switch (currentQuestion.type) {
```

### <a id="sv-flt-f-0205"></a>`SV-FLT-F-0205`

- **Surface:** `FLT` | **Type:** `default` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildQuestion (L99-178)` | **Pin:** `df97547d612967e99784`
- **Claim:** For openText questions, resolves buttonText using surveyQuestionButtonLabel with appearance.submitButtonText as fallback.

```
buttonText: surveyQuestionButtonLabel(
```

### <a id="sv-flt-f-0206"></a>`SV-FLT-F-0206`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildQuestion (L99-178)` | **Pin:** `df97547d612967e99784`
- **Claim:** For link questions, pressing the button submits true as a boolean response along with linkQuestion.link.

```
onPressed: () => _submitResponse(
```

### <a id="sv-flt-f-0207"></a>`SV-FLT-F-0207`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildQuestion (L99-178)` | **Pin:** `df97547d612967e99784`
- **Claim:** For rating questions, renders RatingQuestion with scale bounds, bound labels, ratingType, optional, skipSubmitButton, and _submitResponse.

```
return RatingQuestion(
```

### <a id="sv-flt-f-0208"></a>`SV-FLT-F-0208`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildQuestion (L99-178)` | **Pin:** `df97547d612967e99784`
- **Claim:** For singleChoice and multipleChoice questions, renders ChoiceQuestionWidget with isMultipleChoice set to true only for multipleChoice.

```
return ChoiceQuestionWidget(
```

### <a id="sv-flt-f-0209"></a>`SV-FLT-F-0209`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState.build (L181-201)` | **Pin:** `df97547d612967e99784`
- **Claim:** Returns the built sheet directly without overriding theme typography when widget.appearance.fontFamily is null.

```
if (fontFamily == null) {
```

### <a id="sv-flt-f-0210"></a>`SV-FLT-F-0210`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState.build (L181-201)` | **Pin:** `df97547d612967e99784`
- **Claim:** When widget.appearance.fontFamily is non-null, wraps the sheet in Theme and DefaultTextStyle.merge applying fontFamily.

```
textTheme: theme.textTheme.apply(fontFamily: fontFamily),
```

### <a id="sv-flt-f-0212"></a>`SV-FLT-F-0212`

- **Surface:** `FLT` | **Type:** `fact` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildSheet (L203-287)` | **Pin:** `df97547d612967e99784`
- **Claim:** Pads the bottom of the sheet by mediaQuery.viewInsets.bottom to accommodate the on-screen keyboard.

```
padding: EdgeInsets.only(bottom: mediaQuery.viewInsets.bottom),
```

### <a id="sv-flt-f-0214"></a>`SV-FLT-F-0214`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildSheet (L203-287)` | **Pin:** `df97547d612967e99784`
- **Claim:** When _isCompleted is true, renders ConfirmationMessage with thankYouMessageDescriptionContentType defaulting to PostHogDisplaySurveyTextContentType.text.

```
if (_isCompleted)
```

### <a id="sv-flt-f-0215"></a>`SV-FLT-F-0215`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildSheet (L203-287)` | **Pin:** `df97547d612967e99784`
- **Claim:** When not completed and _showingIntroScreen is true, renders IntroMessage whose onStart sets _showingIntroScreen to false and defaults introScreenDescriptionContentType to PostHogDisplaySurveyTextContentType.text.

```
else if (_showingIntroScreen)
```

### <a id="sv-flt-f-0216"></a>`SV-FLT-F-0216`

- **Surface:** `FLT` | **Type:** `guard` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_bottom_sheet.dart#_SurveyBottomSheetState._buildSheet (L203-287)` | **Pin:** `df97547d612967e99784`
- **Claim:** Wraps the active question in AbsorbPointer with absorbing set to _isSubmitting to block input while a response is submitting.

```
absorbing: _isSubmitting,
```

### <a id="sv-flt-f-0226"></a>`SV-FLT-F-0226`

- **Surface:** `FLT` | **Type:** `limit` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_choice_button.dart#SurveyChoiceButton.build (L27-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** The choice button container enforces a minimum height of 48.

```
constraints: const BoxConstraints(minHeight: 48),
```

### <a id="sv-flt-f-0228"></a>`SV-FLT-F-0228`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_choice_button.dart#SurveyChoiceButton.build (L27-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** When isOpenChoice is true, renders a Column with '$label:' and an editable TextFormField; otherwise renders a plain Text(label).

```
child: isOpenChoice
```

### <a id="sv-flt-f-0229"></a>`SV-FLT-F-0229`

- **Surface:** `FLT` | **Type:** `branch` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_choice_button.dart#SurveyChoiceButton.build (L27-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** In open-choice mode, the TextFormField is enabled and autofocused only when isSelected is true.

```
enabled: isSelected,
```

### <a id="sv-flt-f-0230"></a>`SV-FLT-F-0230`

- **Surface:** `FLT` | **Type:** `control` | **Source:** `posthog-flutter:posthog_flutter/lib/src/surveys/widgets/survey_choice_button.dart#SurveyChoiceButton.build (L27-108)` | **Pin:** `df97547d612967e99784`
- **Claim:** In open-choice mode, the TextFormField initializes with openChoiceInput and notifies onOpenChoiceChanged on text changes.

```
initialValue: openChoiceInput,
```

### <a id="sv-ios-f-0025"></a>`SV-IOS-F-0025`

- **Surface:** `IOS` | **Type:** `input` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyEnums.swift#PostHogSurveyType.init (L28-37)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Decodes survey type strings "popover", "api", and "widget", preserving unrecognized values as .unknown(type:).

```
) { .unknown(type: $0) }
```

### <a id="sv-ios-f-0026"></a>`SV-IOS-F-0026`

- **Surface:** `IOS` | **Type:** `input` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyEnums.swift#PostHogSurveyQuestionType.init (L48-59)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Decodes question type strings "open", "link", "rating", "multiple_choice", and "single_choice", preserving unrecognized values as .unknown(type:).

```
"multiple_choice": .multipleChoice,
```

### <a id="sv-ios-f-0036"></a>`SV-IOS-F-0036`

- **Surface:** `IOS` | **Type:** `default` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyEnums.swift#PostHogSurveySchedule.init (L298-307)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Falls back to .unknown(schedule:) preserving the raw string when decoding an unrecognized survey schedule value.

```
) { .unknown(schedule: $0) }
```

### <a id="sv-ios-f-0037"></a>`SV-IOS-F-0037`

- **Surface:** `IOS` | **Type:** `input` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyEnums.swift#PostHogSurveyQuestionBranchingType.init (L317-327)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Decodes survey question branching type strings "next_question", "end", "response_based", and "specific_question" into PostHogSurveyQuestionBranchingType cases (.nextQuestion, .end, .responseBased, .specificQuestion) via decodeSurveyStringValue.

```
self = try decodeSurveyStringValue(
```

### <a id="sv-ios-f-0042"></a>`SV-IOS-F-0042`

- **Surface:** `IOS` | **Type:** `default` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyQuestion.swift#PostHogSurveyQuestion.init (L42-60)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Preserves unrecognized survey question types as .unknown(type:) instead of failing decoding

```
case let .unknown(type):
```

### <a id="sv-ios-f-0045"></a>`SV-IOS-F-0045`

- **Surface:** `IOS` | **Type:** `input` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyQuestion.swift#PostHogSurveyQuestionBranching.init (L190-224)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Decodes integer target question index for specificQuestion branching

```
self = try .specificQuestion(index: container.decode(Int.self, forKey: .index))
```

### <a id="sv-ios-f-0046"></a>`SV-IOS-F-0046`

- **Surface:** `IOS` | **Type:** `default` | **Source:** `posthog-ios:PostHog/Models/Surveys/PostHogSurveyQuestion.swift#PostHogSurveyQuestionBranching.init (L190-224)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Preserves unrecognized branching types as .unknown(type:) instead of failing decoding

```
case let .unknown(type):
```

### <a id="sv-ios-f-0093"></a>`SV-IOS-F-0093`

- **Surface:** `IOS` | **Type:** `wiring` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.install (L81-87)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Installs the survey integration once via integrationInstallState, binds the PostHogSDK reference, initializes SurveyProgressStore if storage is available, and calls start().

```
installIfNeeded(using: Self.integrationInstallState) {
```

### <a id="sv-ios-f-0095"></a>`SV-IOS-F-0095`

- **Surface:** `IOS` | **Type:** `wiring` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.start (L96-116)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Subscribes to remoteConfig.onPersonPropertiesForFlagsChanged to refresh active survey translations.

```
personPropertiesChangedToken = postHog?.remoteConfig?.onPersonPropertiesForFlagsChanged.subscribe { [weak self] _ in
```

### <a id="sv-ios-f-0096"></a>`SV-IOS-F-0096`

- **Surface:** `IOS` | **Type:** `wiring` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.start (L96-116)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** On iOS, subscribes to postHog.onEventCaptured to route captured events to onEvent(event:).

```
eventCapturedToken = postHog?.onEventCaptured.subscribe { [weak self] event in
```

### <a id="sv-ios-f-0097"></a>`SV-IOS-F-0097`

- **Surface:** `IOS` | **Type:** `schedule` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.start (L96-116)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** On iOS, subscribes to runLoopPublisher.onOpportunity throttled to 5 seconds to attempt showing the next survey.

```
runLoopOpportunityToken = DI.main.runLoopPublisher.onOpportunity.subscribe(throttle: 5) { [weak self] in
```

### <a id="sv-ios-f-0098"></a>`SV-IOS-F-0098`

- **Surface:** `IOS` | **Type:** `wiring` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.start (L96-116)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** On iOS, subscribes to appLifecyclePublisher.onDidBecomeActive to attempt showing the next survey.

```
didBecomeActiveToken = DI.main.appLifecyclePublisher.onDidBecomeActive.subscribe { [weak self] in
```

### <a id="sv-ios-f-0106"></a>`SV-IOS-F-0106`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.getSurveys (L202-225)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Returns early without loading surveys if remoteConfig is nil

```
guard let remoteConfig else {
```

### <a id="sv-ios-f-0107"></a>`SV-IOS-F-0107`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.getSurveys (L202-225)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Logs that surveys are disabled and invokes callback with an empty array when config is nil or config._surveys is false

```
guard let config = config, config._surveys else {
```

### <a id="sv-ios-f-0110"></a>`SV-IOS-F-0110`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.getRemoteConfig (L227-238)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Reads cached remote config or triggers remoteConfig.reloadRemoteConfig when missing or forceReload is true

```
reload: { remoteConfig.reloadRemoteConfig(callback: $0) },
```

### <a id="sv-ios-f-0111"></a>`SV-IOS-F-0111`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.getFeatureFlags (L240-253)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Reads cached feature flags or triggers remoteConfig.reloadFeatureFlags and extracts featureFlags dictionary

```
remoteConfig.reloadFeatureFlags(callback: callback.map { callback in { callback($0.featureFlags) } })
```

### <a id="sv-ios-f-0127"></a>`SV-IOS-F-0127`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.setSurveySeen (L404-412)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Skips marking the survey as seen if a non-nil generation does not match currentGeneration

```
guard generation == nil || generation == currentGeneration else { return }
```

### <a id="sv-ios-f-0128"></a>`SV-IOS-F-0128`

- **Surface:** `IOS` | **Type:** `storage` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.setSurveySeen (L404-412)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Persists true for the survey's seen key in the .surveySeen storage dictionary

```
storage?.setDictionary(forKey: .surveySeen, contents: seenKeys)
```

### <a id="sv-ios-f-0136"></a>`SV-IOS-F-0136`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.withActiveSurveyAttempt (L519-535)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Returns nil if attemptId is nil, does not match activeSurveyAttemptId, or storage is nil

```
guard let attemptId, attemptId == activeSurveyAttemptId, let storage else { return nil }
```

### <a id="sv-ios-f-0137"></a>`SV-IOS-F-0137`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.withActiveSurveyAttempt (L519-535)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Clears the active survey and returns nil if activeSurveyGeneration does not match current storage generation

```
guard activeSurveyGeneration == generation else {
```

### <a id="sv-ios-f-0138"></a>`SV-IOS-F-0138`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.withActiveSurveyAttempt (L519-535)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Clears the active survey and returns nil if storage survey generation changed during operation execution

```
guard storage.isSurveyGenerationCurrent(generation) else {
```

### <a id="sv-ios-f-0146"></a>`SV-IOS-F-0146`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.handleSurveyResponse (L595-655)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Ignores the response and logs a message if there is no active survey or survey.id does not match activeSurvey.id.

```
guard let activeSurvey, survey.id == activeSurvey.id else {
```

### <a id="sv-ios-f-0147"></a>`SV-IOS-F-0147`

- **Surface:** `IOS` | **Type:** `input` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.handleSurveyResponse (L595-655)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Rejects the response and returns nil if the survey is already completed, index is negative, index does not match activeSurveyQuestionIndex, or index is out of bounds.

```
guard !activeSurveyCompleted, index >= 0, index == activeSurveyQuestionIndex, activeSurvey.questions.indices.contains(index) else { return nil }
```

### <a id="sv-ios-f-0151"></a>`SV-IOS-F-0151`

- **Surface:** `IOS` | **Type:** `branch` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.handleSurveyResponse (L595-655)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Sends a 'survey sent' event when enablePartialResponses is true or the survey is completed.

```
if activeSurvey.enablePartialResponses == true || isCompleted {
```

### <a id="sv-ios-f-0152"></a>`SV-IOS-F-0152`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.handleSurveyResponse (L595-655)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** If activeProgressWasPersisted is true, verifies the persisted submissionId in progressStore matches the active submissionId; otherwise clears the active survey and returns nil.

```
guard !activeProgressWasPersisted || progressStore?.load(activeSurvey)?.submissionId == submissionId else {
```

### <a id="sv-ios-f-0160"></a>`SV-IOS-F-0160`

- **Surface:** `IOS` | **Type:** `storage` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.sendSurveySentEvent (L719-751)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Sets person property $survey_responded/{id} to true via $set along with $survey_completed and $survey_submission_id, and marks the survey as seen.

```
"$set": [survey.interactionProperty("responded"): true],
```

### <a id="sv-ios-f-0161"></a>`SV-IOS-F-0161`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.sendSurveySentEvent (L719-751)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Emits the 'survey sent' event with survey response properties, language, and generation.

```
event: "survey sent",
```

### <a id="sv-ios-f-0164"></a>`SV-IOS-F-0164`

- **Surface:** `IOS` | **Type:** `default` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.buildSurveyResponseProperties (L788-815)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Resolves each question's text from responseQuestionText[key], falling back to translatedQuestionText and then question.question.

```
let effectiveQuestion = responseQuestionText[key]
```

### <a id="sv-ios-f-0169"></a>`SV-IOS-F-0169`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.sendSurveyEvent (L831-848)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Captures the survey event with merged survey.eventProperties and additionalProperties via postHog.capture.

```
postHog.capture(event, distinctId: distinctId, properties: properties)
```

### <a id="sv-ios-f-0177"></a>`SV-IOS-F-0177`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.persistActiveProgressLocked (L897-910)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** No-ops when activeSurvey, activeSurveySubmissionId, or activeSurveyGeneration is nil.

```
guard let survey = activeSurvey, let submissionId = activeSurveySubmissionId, let resetEpoch = activeSurveyGeneration else { return }
```

### <a id="sv-ios-f-0178"></a>`SV-IOS-F-0178`

- **Surface:** `IOS` | **Type:** `storage` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.persistActiveProgressLocked (L897-910)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Removes saved progress from progressStore and returns when activeSurveyCompleted is true.

```
progressStore?.remove(survey)
```

### <a id="sv-ios-f-0179"></a>`SV-IOS-F-0179`

- **Surface:** `IOS` | **Type:** `storage` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.persistActiveProgressLocked (L897-910)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Saves current survey progress (questionIndex, responses, questionText, language) to progressStore.

```
progressStore?.save(progress, for: survey)
```

### <a id="sv-ios-f-0180"></a>`SV-IOS-F-0180`

- **Surface:** `IOS` | **Type:** `storage` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.persistActiveProgressLocked (L897-910)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Verifies whether progress exists in progressStore to update activeProgressWasPersisted.

```
activeProgressWasPersisted = progressStore?.load(survey) != nil
```

### <a id="sv-ios-f-0187"></a>`SV-IOS-F-0187`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/PostHogSurveyIntegration.swift#PostHogSurveyIntegration.getNextQuestion (L1062-1069)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Returns nil if activeSurvey is nil.

```
guard let activeSurvey else { return nil }
```

### <a id="sv-ios-f-0213"></a>`SV-IOS-F-0213`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/QuestionTypes.swift#OpenTextQuestionView.body (L19-57)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Submit button labeled with question.buttonText (falling back to appearance.submitButtonText) that calls onNextQuestion with nil if trimmed text is empty or raw text otherwise, disabled when !canSubmit.

```
BottomSection(label: question.buttonText ?? appearance.submitButtonText) {
```

### <a id="sv-ios-f-0215"></a>`SV-IOS-F-0215`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/QuestionTypes.swift#LinkQuestionView (L83-112)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Action button labeled with question.buttonText (falling back to appearance.submitButtonText) that invokes onNextQuestion(true).

```
BottomSection(label: question.buttonText ?? appearance.submitButtonText) {
```

### <a id="sv-ios-f-0216"></a>`SV-IOS-F-0216`

- **Surface:** `IOS` | **Type:** `branch` | **Source:** `posthog-ios:PostHog/Surveys/QuestionTypes.swift#LinkQuestionView (L83-112)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Opens question.link via UIApplication.shared.open when the URL is valid and UIApplication.shared.canOpenURL returns true.

```
if let link, UIApplication.shared.canOpenURL(link) {
```

### <a id="sv-ios-f-0218"></a>`SV-IOS-F-0218`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/QuestionTypes.swift#MultipleChoiceQuestionView.body (L248-270)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Submit button labeled with question.buttonText (falling back to appearance.submitButtonText) that calls onNextQuestion(response), disabled when !canSubmit.

```
BottomSection(label: question.buttonText ?? appearance.submitButtonText) {
```

### <a id="sv-ios-f-0230"></a>`SV-IOS-F-0230`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/SurveyDisplayController.swift#SurveyDisplayController.onNextQuestion (L61-75)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Returns early without action if no survey is currently displayed.

```
guard let displayedSurvey else { return }
```

### <a id="sv-ios-f-0231"></a>`SV-IOS-F-0231`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/SurveyDisplayController.swift#SurveyDisplayController.onNextQuestion (L61-75)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Emits onSurveyResponse with the displayed survey, question index, and response, and dismisses the survey if the callback returns nil.

```
guard let next = onSurveyResponse?(displayedSurvey, index, response) else {
```

### <a id="sv-ios-f-0238"></a>`SV-IOS-F-0238`

- **Surface:** `IOS` | **Type:** `storage` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgress (L4-28)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** SurveyProgress persists Codable survey progress state including resetEpoch, submissionId, questionIndex, responses, questionText, and optional language.

```
struct SurveyProgress: Codable {
```

### <a id="sv-ios-f-0254"></a>`SV-IOS-F-0254`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.save (L87-97)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Aborts save if progress.resetEpoch does not match the current generation or JSON encoding fails.

```
guard progress.resetEpoch == generation,
```

### <a id="sv-ios-f-0255"></a>`SV-IOS-F-0255`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.save (L87-97)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Aborts save without writing if the survey state generation is no longer current.

```
guard storage.isSurveyGenerationCurrent(generation) else { return }
```

### <a id="sv-ios-f-0257"></a>`SV-IOS-F-0257`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.save (L87-97)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** SurveyProgressStore.save reads existing survey progress records from .surveyProgress before updating the survey's entry.

```
var records = storage.getDictionary(forKey: .surveyProgress) ?? [:]
```

### <a id="sv-ios-f-0258"></a>`SV-IOS-F-0258`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.save (L87-97)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** SurveyProgressStore.save writes the updated survey progress dictionary to .surveyProgress storage when the generation is still current.

```
storage.setDictionary(forKey: .surveyProgress, contents: records)
```

### <a id="sv-ios-f-0259"></a>`SV-IOS-F-0259`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.reconcile (L99-108)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Aborts reconciliation without writing if the survey state generation is no longer current.

```
guard storage.isSurveyGenerationCurrent(generation) else { return }
```

### <a id="sv-ios-f-0260"></a>`SV-IOS-F-0260`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.reconcile (L99-108)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** SurveyProgressStore.reconcile reads existing progress records from .surveyProgress storage.

```
let records = storage.getDictionary(forKey: .surveyProgress) ?? [:]
```

### <a id="sv-ios-f-0261"></a>`SV-IOS-F-0261`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.reconcile (L99-108)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** SurveyProgressStore.reconcile aborts without modifying storage if storage.isSurveyGenerationCurrent(generation) is false.

```
guard storage.isSurveyGenerationCurrent(generation) else { return }
```

### <a id="sv-ios-f-0262"></a>`SV-IOS-F-0262`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/SurveyProgressStore.swift#SurveyProgressStore.reconcile (L99-108)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** SurveyProgressStore.reconcile prunes .surveyProgress storage to keep only entries matching active surveys (surveys.filter(\.isActive)).

```
storage.setDictionary(forKey: .surveyProgress, contents: records.filter { entry in
```

### <a id="sv-ios-f-0332"></a>`SV-IOS-F-0332`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/Utils/EmojiRating.swift#EmojiRating.body (L19-56)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Emoji rating button toggles selectedValue between the tapped value and nil with a 0.25s linear animation

```
selectedValue = selectedValue == value ? nil : value
```

### <a id="sv-ios-f-0343"></a>`SV-IOS-F-0343`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/Utils/MultipleChoiceOptions.swift#MultipleChoiceOptions.body (L47-88)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Each survey choice is a Button that toggles its selection via setSelected(!isSelected, index: index) with a 0.15s linear animation.

```
setSelected(!isSelected, index: index)
```

### <a id="sv-ios-f-0346"></a>`SV-IOS-F-0346`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/Utils/MultipleChoiceOptions.swift#MultipleChoiceOptions.openChoiceField (L116-130)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** When isOpenChoice(index) is true, renders a TextField bound to $openChoiceInput and $isTextFieldFocused, positioned and sized using textFieldRect.

```
TextField("", text: $openChoiceInput)
```

### <a id="sv-ios-f-0347"></a>`SV-IOS-F-0347`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/Utils/MultipleChoiceOptions.swift#MultipleChoiceOptions.openChoiceField (L116-130)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Disables the open-choice TextField and dims its foreground color to inputTextColor.opacity(0.5) when !isSelected(index).

```
.disabled(!isSelected(index))
```

### <a id="sv-ios-f-0351"></a>`SV-IOS-F-0351`

- **Surface:** `IOS` | **Type:** `control` | **Source:** `posthog-ios:PostHog/Surveys/Utils/NumberRating.swift#NumberRating.body (L19-64)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Renders a SegmentedControl of height 45 over scale.range bound to selectedValue, displaying each integer option.

```
selectedValue: $selectedValue
```

### <a id="sv-ios-f-0356"></a>`SV-IOS-F-0356`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration (L48-432)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** canEvaluateSurveyFeatureFlags returns true only when not awaiting a feature flags generation, survey feature flags are not unavailable, and remoteConfig.isLoadingFeatureFlags is not true.

```
surveyAwaitingFeatureFlagsGeneration == nil && !surveyFeatureFlagsUnavailable
```

### <a id="sv-ios-f-0359"></a>`SV-IOS-F-0359`

- **Surface:** `IOS` | **Type:** `guard` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.decodeAndSetSurveys (L49-63)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Aborts without updating the survey cache or invoking callback if beforeCacheUpdate(loadedSurveys) returns false.

```
guard beforeCacheUpdate?(loadedSurveys) != false else { return }
```

### <a id="sv-ios-f-0361"></a>`SV-IOS-F-0361`

- **Surface:** `IOS` | **Type:** `input` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.decodeSurveys (L78-94)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Returns an empty array if remoteConfig["surveys"] is missing or not an array of dictionaries.

```
guard let surveysJSON = remoteConfig["surveys"] as? [[String: Any]] else {
```

### <a id="sv-ios-f-0362"></a>`SV-IOS-F-0362`

- **Surface:** `IOS` | **Type:** `error` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.decodeSurveys (L78-94)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Decodes each survey dictionary with PostHogApi.jsonDecoder, logging decode errors via hedgeLog and skipping invalid survey entries.

```
hedgeLog("Error decoding Survey: \(error)")
```

### <a id="sv-ios-f-0365"></a>`SV-IOS-F-0365`

- **Surface:** `IOS` | **Type:** `wiring` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.subscribeToRemoteConfigUpdates (L114-140)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Subscribes to onRemoteConfigLoaded; when non-nil config arrives, acquires surveyRefreshProcessingLock, begins a refresh generation, decodes and caches surveys if still current, refreshes feature flags if needed, and calls showNextSurvey().

```
remoteConfigLoadedToken = postHog?.remoteConfig?.onRemoteConfigLoaded.subscribe { [weak self] remoteConfig in
```

### <a id="sv-ios-f-0366"></a>`SV-IOS-F-0366`

- **Surface:** `IOS` | **Type:** `wiring` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.subscribeToRemoteConfigUpdates (L114-140)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Subscribes to onFeatureFlagsLoaded; when non-nil flags arrive and no survey refresh is awaiting flags, clears surveyFeatureFlagsUnavailable and calls showNextSurvey().

```
featureFlagsLoadedToken = postHog?.remoteConfig?.onFeatureFlagsLoaded.subscribe { [weak self] flags in
```

### <a id="sv-ios-f-0368"></a>`SV-IOS-F-0368`

- **Surface:** `IOS` | **Type:** `port` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.refreshFeatureFlagsIfNeeded (L164-185)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Calls reloadFeatureFlagsForSurvey when flags are needed; on completion for the matching generation, clears surveyAwaitingFeatureFlagsGeneration, sets surveyFeatureFlagsUnavailable when flags is nil, and calls showNextSurvey().

```
postHog?.remoteConfig?.reloadFeatureFlagsForSurvey { [weak self] flags in
```

### <a id="sv-ios-f-0374"></a>`SV-IOS-F-0374`

- **Surface:** `IOS` | **Type:** `limit` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#processBranchingStep (L260-268)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** When nextIndex is an Int, clamps the returned .index to at most totalQuestions - 1.

```
return .index(min(nextIndex, totalQuestions - 1))
```

### <a id="sv-ios-f-0378"></a>`SV-IOS-F-0378`

- **Surface:** `IOS` | **Type:** `error` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#getRatingBucketForResponseValue (L272-296)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Logs an invalid scale error via hedgeLog and returns nil when scale or value is outside valid ranges (3: 1-3, 5: 1-5, 7: 1-7, 10: 0-10).

```
hedgeLog("[Surveys] Cannot get rating bucket for invalid scale: \(scale). The scale must be one of: 3 (1-3), 5 (1-5), 7 (1-7), 10 (0-10).")
```

### <a id="sv-ios-f-0379"></a>`SV-IOS-F-0379`

- **Surface:** `IOS` | **Type:** `limit` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#RatingBucket (L304-318)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Defines the 3-point rating scale range as 1...3.

```
static let threePointRange = 1 ... 3
```

### <a id="sv-ios-f-0380"></a>`SV-IOS-F-0380`

- **Surface:** `IOS` | **Type:** `limit` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#RatingBucket (L304-318)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Defines the 5-point rating scale range as 1...5.

```
static let fivePointRange = 1 ... 5
```

### <a id="sv-ios-f-0381"></a>`SV-IOS-F-0381`

- **Surface:** `IOS` | **Type:** `limit` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#RatingBucket (L304-318)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Defines the 7-point rating scale range as 1...7.

```
static let sevenPointRange = 1 ... 7
```

### <a id="sv-ios-f-0382"></a>`SV-IOS-F-0382`

- **Surface:** `IOS` | **Type:** `limit` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#RatingBucket (L304-318)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Defines the 10-point (NPS) rating scale range as 0...10.

```
static let tenPointRange = 0 ... 10
```

### <a id="sv-ios-f-0388"></a>`SV-IOS-F-0388`

- **Surface:** `IOS` | **Type:** `error` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.getResponseBasedNextQuestionIndex (L389-431)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Logs a warning and returns nil if response-based branching is evaluated with a nil response.

```
guard let response else {
```

### <a id="sv-ios-f-0393"></a>`SV-IOS-F-0393`

- **Surface:** `IOS` | **Type:** `error` | **Source:** `posthog-ios:PostHog/Surveys/Utils/PostHogSurveyMatching.swift#PostHogSurveyIntegration.getResponseBasedNextQuestionIndex (L389-431)` | **Pin:** `ba9fa9e4c09230afa1c6`
- **Claim:** Logs a warning and returns nil for unsupported (question, response.type) combinations in response-based branching.

```
hedgeLog("[Surveys] Got response based branching for an unsupported question type.")
```

### <a id="sv-mcp-f-0005"></a>`SV-MCP-F-0005`

- **Surface:** `MCP` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/max_tools.py#get_team_survey_config (L84-90)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns team survey_config.appearance (defaulting to {}) and default_settings with type 'popover' and enable_partial_responses=True.

```
"default_settings": {"type": "popover", "enable_partial_responses": True},
```

### <a id="sv-mcp-f-0024"></a>`SV-MCP-F-0024`

- **Surface:** `MCP` | **Type:** `input` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Ignores extra unknown fields in CreateSurveyToolArgs.

```
model_config = ConfigDict(extra="ignore")
```

### <a id="sv-mcp-f-0025"></a>`SV-MCP-F-0025`

- **Surface:** `MCP` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults survey_type to "popover".

```
default="popover", description="Survey display type"
```

### <a id="sv-mcp-f-0026"></a>`SV-MCP-F-0026`

- **Surface:** `MCP` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults should_launch to False.

```
should_launch: bool = Field(default=False, description="Launch immediately after creation")
```

### <a id="sv-mcp-f-0027"></a>`SV-MCP-F-0027`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires linked_flag_id to be >= 1 when provided.

```
linked_flag_id: int | None = Field(default=None, ge=1, description="Feature flag ID for targeting")
```

### <a id="sv-mcp-f-0028"></a>`SV-MCP-F-0028`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Accepts an optional responses_limit specifying the maximum number of responses to collect.

```
responses_limit: int | None = Field(default=None, description="Maximum number of responses to collect")
```

### <a id="sv-mcp-f-0029"></a>`SV-MCP-F-0029`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Constrains iteration_count between 1 and MAX_ITERATION_COUNT.

```
le=MAX_ITERATION_COUNT,
```

### <a id="sv-mcp-f-0030"></a>`SV-MCP-F-0030`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyToolArgs (L407-439)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Constrains iteration_frequency_days between 1 and MAX_ITERATION_FREQUENCY_DAYS.

```
le=MAX_ITERATION_FREQUENCY_DAYS,
```

### <a id="sv-mcp-f-0031"></a>`SV-MCP-F-0031`

- **Surface:** `MCP` | **Type:** `port` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool (L442-584)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exposes the create_survey MaxTool.

```
name: str = "create_survey"
```

### <a id="sv-mcp-f-0032"></a>`SV-MCP-F-0032`

- **Surface:** `MCP` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool (L442-584)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Requires ("survey", "editor") resource access to invoke CreateSurveyTool.

```
return [("survey", "editor")]
```

### <a id="sv-mcp-f-0033"></a>`SV-MCP-F-0033`

- **Surface:** `MCP` | **Type:** `guard` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool (L442-584)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Treats create_survey as a dangerous operation requiring confirmation when should_launch is True.

```
async def is_dangerous_operation(self, should_launch: bool = False, **kwargs) -> bool:
```

### <a id="sv-mcp-f-0034"></a>`SV-MCP-F-0034`

- **Surface:** `MCP` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool (L442-584)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Formats dangerous operation preview with the survey name, question count, and a notice that it will immediately start collecting responses.

```
return f"**Create and launch** survey '{name}' with {question_count} question(s). It will immediately start collecting responses."
```

### <a id="sv-mcp-f-0035"></a>`SV-MCP-F-0035`

- **Surface:** `MCP` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets archived=False and enable_partial_responses=True by default in survey_data.

```
"enable_partial_responses": True,
```

### <a id="sv-mcp-f-0036"></a>`SV-MCP-F-0036`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets responses_limit on survey_data when responses_limit is not None.

```
if responses_limit is not None:
```

### <a id="sv-mcp-f-0037"></a>`SV-MCP-F-0037`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets linked_flag_id on non-external surveys when linked_flag_id is not None.

```
if linked_flag_id is not None:
```

### <a id="sv-mcp-f-0038"></a>`SV-MCP-F-0038`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Attaches targeting conditions on non-external surveys when _build_targeting_conditions returns a non-empty dict.

```
if conditions:
```

### <a id="sv-mcp-f-0039"></a>`SV-MCP-F-0039`

- **Surface:** `MCP` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Merges _build_schedule_fields output into survey_data for non-external surveys.

```
survey_data.update(_build_schedule_fields(schedule, iteration_count, iteration_frequency_days))
```

### <a id="sv-mcp-f-0040"></a>`SV-MCP-F-0040`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Removes surveyPopupDelaySeconds from appearance and skips targeting/schedule for external_survey.

```
survey_data["appearance"].pop("surveyPopupDelaySeconds", None)
```

### <a id="sv-mcp-f-0041"></a>`SV-MCP-F-0041`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._build_survey_data (L459-508)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets linked_insight_id from self.context.get("insight_id") when present.

```
if self.context.get("insight_id"):
```

### <a id="sv-mcp-f-0042"></a>`SV-MCP-F-0042`

- **Surface:** `MCP` | **Type:** `error` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._arun_impl (L510-584)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns validation_failed error when no questions are provided.

```
if not questions:
```

### <a id="sv-mcp-f-0043"></a>`SV-MCP-F-0043`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#CreateSurveyTool._arun_impl (L510-584)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets start_date to django.utils.timezone.now() to launch immediately when should_launch is True.

```
if should_launch:
```

### <a id="sv-mcp-f-0056"></a>`SV-MCP-F-0056`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#EditSurveyToolArgs (L622-660)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Accepts an optional responses_limit integer capping the maximum number of responses.

```
responses_limit: int | None = Field(default=None, description="Maximum number of responses")
```

### <a id="sv-mcp-f-0079"></a>`SV-MCP-F-0079`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#EditSurveyTool._arun_impl (L734-859)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates responses_limit on the survey when responses_limit is provided.

```
if responses_limit is not None:
```

### <a id="sv-mcp-f-0097"></a>`SV-MCP-F-0097`

- **Surface:** `MCP` | **Type:** `port` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool (L871-1044)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Registers the analyze_survey_responses MaxTool for retrieving and analyzing survey responses.

```
name: str = "analyze_survey_responses"
```

### <a id="sv-mcp-f-0100"></a>`SV-MCP-F-0100`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._format_responses_for_analysis (L893-917)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Formats each response in group.responses as a bulleted quoted line when responses is non-empty.

```
if responses:
```

### <a id="sv-mcp-f-0101"></a>`SV-MCP-F-0101`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._format_responses_for_analysis (L893-917)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Outputs "Responses:" followed by bulleted response lines when response_texts is non-empty, otherwise outputs "Responses: (none)".

```
if response_texts:
```

### <a id="sv-mcp-f-0102"></a>`SV-MCP-F-0102`

- **Surface:** `MCP` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._fetch_survey_responses (L919-967)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults start_date to survey.start_date, then survey.created_at, or 365 days before the current time if both are unset.

```
start_date = survey.start_date or survey.created_at or (django.utils.timezone.now() - timedelta(days=365))
```

### <a id="sv-mcp-f-0103"></a>`SV-MCP-F-0103`

- **Surface:** `MCP` | **Type:** `default` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._fetch_survey_responses (L919-967)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults end_date to survey.end_date or the current timestamp if unset.

```
end_date = survey.end_date or django.utils.timezone.now()
```

### <a id="sv-mcp-f-0104"></a>`SV-MCP-F-0104`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._fetch_survey_responses (L919-967)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Skips survey questions whose type is not "open" when fetching responses.

```
if q_type != "open":
```

### <a id="sv-mcp-f-0105"></a>`SV-MCP-F-0105`

- **Surface:** `MCP` | **Type:** `limit` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._fetch_survey_responses (L919-967)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Caps fetched responses at 50 per open-ended question via fetch_responses.

```
limit=50,  # Limit responses per question
```

### <a id="sv-mcp-f-0106"></a>`SV-MCP-F-0106`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._fetch_survey_responses (L919-967)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Filters out empty or whitespace-only response strings before constructing SurveyAnalysisResponseItem entries.

```
if text and text.strip()
```

### <a id="sv-mcp-f-0107"></a>`SV-MCP-F-0107`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._fetch_survey_responses (L919-967)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Appends a SurveyAnalysisQuestionGroup only when response_items is non-empty, falling back to the stringified question index when question_id is missing.

```
if response_items:
```

### <a id="sv-mcp-f-0112"></a>`SV-MCP-F-0112`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._arun_impl (L969-1044)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns an empty-responses message with response_count 0 when _fetch_survey_responses returns no groups.

```
if not responses:
```

### <a id="sv-mcp-f-0113"></a>`SV-MCP-F-0113`

- **Surface:** `MCP` | **Type:** `branch` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._arun_impl (L969-1044)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns an empty-responses message with response_count 0 when total_response_count across all groups is 0.

```
if total_response_count == 0:
```

### <a id="sv-mcp-f-0114"></a>`SV-MCP-F-0114`

- **Surface:** `MCP` | **Type:** `fact` | **Source:** `posthog:products/surveys/backend/max_tools.py#SurveyAnalysisTool._arun_impl (L969-1044)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns formatted open-ended responses with analysis instructions and metadata containing survey_id, survey_name, and response_count.

```
"response_count": total_response_count,
```

### <a id="sv-mcp-f-0117"></a>`SV-MCP-F-0117`

- **Surface:** `MCP` | **Type:** `wiring` | **Source:** `posthog:products/surveys/mcp/tools.yaml#ui_apps.survey-global-stats` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** The survey-global-stats MCP UI app binds view_prop `data` of type `SurveyStatsData` to `SurveyStatsView`.

```
view_component: SurveyStatsView
```

### <a id="sv-rn-f-0001"></a>`SV-RN-F-0001`

- **Surface:** `RN` | **Type:** `default` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Uses props.client as the PostHog client when provided, falling back to the client from usePostHog().

```
const posthog = props.client ?? posthogFromHook
```

### <a id="sv-rn-f-0002"></a>`SV-RN-F-0002`

- **Surface:** `RN` | **Type:** `wiring` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to PostHog 'personProperties' events to recompute userLanguage via detectUserLanguage(posthog) and unsubscribes on cleanup.

```
const unsubscribe = posthog.on('personProperties', updateLanguage)
```

### <a id="sv-rn-f-0003"></a>`SV-RN-F-0003`

- **Surface:** `RN` | **Type:** `wiring` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to PostHog 'surveysReset' events to invalidate the current survey session refs and clear activeSurvey.

```
posthog.on('surveysReset', () => {
```

### <a id="sv-rn-f-0004"></a>`SV-RN-F-0004`

- **Surface:** `RN` | **Type:** `port` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Waits for posthog.ready() and posthog._onSurveysReady(), refreshes userLanguage, and fetches surveys via posthog.getSurveys().

```
return posthog.getSurveys()
```

### <a id="sv-rn-f-0005"></a>`SV-RN-F-0005`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips progress reconciliation and setting surveys state if the provider unmounted before posthog.getSurveys() resolved.

```
if (!mounted) return
```

### <a id="sv-rn-f-0006"></a>`SV-RN-F-0006`

- **Surface:** `RN` | **Type:** `error` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Silently ignores errors thrown during the async survey initialization and loading chain.

```
.catch(() => {})
```

### <a id="sv-rn-f-0007"></a>`SV-RN-F-0007`

- **Surface:** `RN` | **Type:** `guard` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Clears pendingSurveyRef and aborts survey matching and scheduling when survey storage is not ready or canCaptureSurvey(posthog) returns false.

```
if (!isReady || !canCaptureSurvey(posthog)) {
```

### <a id="sv-rn-f-0008"></a>`SV-RN-F-0008`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns early without re-evaluating or replacing activeSurvey while the active survey is already marked as shown.

```
if (isShown) {
```

### <a id="sv-rn-f-0009"></a>`SV-RN-F-0009`

- **Surface:** `RN` | **Type:** `default` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Defaults defaultSurveyAppearance.surveyPopupDelaySeconds to 0, clamps computed popup delays to at least 0 seconds, and uses defaultDelay when overrideAppearanceWithDefault is true.

```
const defaultDelay = defaultSurveyAppearance.surveyPopupDelaySeconds ?? 0
```

### <a id="sv-rn-f-0010"></a>`SV-RN-F-0010`

- **Surface:** `RN` | **Type:** `fact` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Filters active matching surveys to SurveyType.Popover and sorts in-progress surveys ahead of new surveys, breaking ties by ascending popup delay.

```
.filter((survey) => survey.type === SurveyType.Popover)
```

### <a id="sv-rn-f-0011"></a>`SV-RN-F-0011`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Preserves the current activeSurvey without rescheduling if it remains present in the eligible popoverSurveys list.

```
if (activeSurvey && popoverSurveys.some((survey) => survey.id === activeSurvey.id)) {
```

### <a id="sv-rn-f-0012"></a>`SV-RN-F-0012`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Clears activeSurvey and pendingSurveyRef and exits when no matching popover survey is found.

```
if (!nextSurvey) {
```

### <a id="sv-rn-f-0013"></a>`SV-RN-F-0013`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Records a new pendingSurveyRef with surveyKey and startedAt timestamp when the next candidate survey iteration changes.

```
if (pendingSurveyRef.current?.surveyKey !== surveyKey) {
```

### <a id="sv-rn-f-0014"></a>`SV-RN-F-0014`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Immediately clears pendingSurveyRef and sets activeSurvey to nextSurvey when remainingDelay is 0 ms.

```
if (remainingDelay === 0) {
```

### <a id="sv-rn-f-0015"></a>`SV-RN-F-0015`

- **Surface:** `RN` | **Type:** `schedule` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Schedules a setTimeout for remainingDelay ms to activate nextSurvey after its popup delay elapses, clearing the timeout on cleanup.

```
const surveyActivationTimeoutId = setTimeout(() => {
```

### <a id="sv-rn-f-0016"></a>`SV-RN-F-0016`

- **Surface:** `RN` | **Type:** `guard` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Aborts delayed survey activation when the timer fires if canCaptureSurvey(posthog) returns false.

```
if (!canCaptureSurvey(posthog)) return
```

### <a id="sv-rn-f-0017"></a>`SV-RN-F-0017`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Re-evaluates active matching surveys when the delay timer fires and activates nextSurvey only if it is still eligible.

```
if (nextSurveyEligible) {
```

### <a id="sv-rn-f-0018"></a>`SV-RN-F-0018`

- **Surface:** `RN` | **Type:** `fact` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Translates activeSurvey for userLanguage via applySurveyTranslationForUser when an activeSurvey is selected.

```
return activeSurvey ? applySurveyTranslationForUser(activeSurvey, posthog, userLanguage) : undefined
```

### <a id="sv-rn-f-0019"></a>`SV-RN-F-0019`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Uses only builtInSurveyAppearance merged with defaultSurveyAppearance when overrideAppearanceWithDefault is true or translatedActiveSurvey is absent; otherwise merges survey-specific appearance on top.

```
if (overrideAppearanceWithDefault || !translatedActiveSurvey) {
```

### <a id="sv-rn-f-0020"></a>`SV-RN-F-0020`

- **Surface:** `RN` | **Type:** `default` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Derives submitButtonTextColor via getContrastingTextColor(submitButtonColor) when submitButtonColor is set on survey appearance without an explicit submitButtonTextColor.

```
getContrastingTextColor(translatedActiveSurvey.survey.appearance.submitButtonColor),
```

### <a id="sv-rn-f-0023"></a>`SV-RN-F-0023`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Deduplicates onShow calls by returning early when shownSurveyIdRef.current already equals activeSurvey.id.

```
if (shownSurveyIdRef.current === activeSurvey.id) {
```

### <a id="sv-rn-f-0024"></a>`SV-RN-F-0024`

- **Surface:** `RN` | **Type:** `port` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Emits the 'survey shown' event with the translated survey and language via sendSurveyShownEvent when a survey is shown.

```
sendSurveyShownEvent(translatedActiveSurvey.survey, posthog, translatedActiveSurvey.language)
```

### <a id="sv-rn-f-0026"></a>`SV-RN-F-0026`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips sending a survey dismissed event on close when submitted is true or activeSession.completed is true.

```
if (submitted || activeSession.completed) return
```

### <a id="sv-rn-f-0027"></a>`SV-RN-F-0027`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Emits the 'survey dismissed' event via dismissedSurveyEvent on non-completed close only when canCaptureSurvey(posthog) returns true.

```
if (canCaptureSurvey(posthog)) {
```

### <a id="sv-rn-f-0028"></a>`SV-RN-F-0028`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Passes activeSession.progress.surveyLanguage to dismissedSurveyEvent when at least one response exists, otherwise falls back to translatedActiveSurvey.language.

```
Object.keys(activeSession.progress.responses).length > 0
```

### <a id="sv-rn-f-0029"></a>`SV-RN-F-0029`

- **Surface:** `RN` | **Type:** `default` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Defaults autoPresent to true unless props.autoPresentSurveys is explicitly false.

```
const autoPresent = props.autoPresentSurveys !== false
```

### <a id="sv-rn-f-0030"></a>`SV-RN-F-0030`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Renders SurveyModal (keyed by submissionId) only when activeContext is defined, the survey type is SurveyType.Popover, and either autoPresent is enabled or the survey is already shown.

```
activeContext && activeContext.survey.type === SurveyType.Popover && (autoPresent || isAlreadyShown)
```

### <a id="sv-rn-f-0039"></a>`SV-RN-F-0039`

- **Surface:** `RN` | **Type:** `storage` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Sets activeSessionRef and shownSurveyIdRef and saves initial activeSession.progress to SurveyProgressStore on show.

```
progressStore.save(activeSurvey, activeSession.progress)
```

### <a id="sv-rn-f-0040"></a>`SV-RN-F-0040`

- **Surface:** `RN` | **Type:** `storage` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Updates lastSeenSurveyDate in survey storage to the current Date when a survey is shown.

```
setLastSeenSurveyDate(new Date())
```

### <a id="sv-rn-f-0041"></a>`SV-RN-F-0041`

- **Surface:** `RN` | **Type:** `storage` | **Source:** `posthog-js:packages/react-native/src/surveys/PostHogSurveyProvider.tsx#PostHogSurveyProvider (L114-417)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** On close, clears activeSessionRef, shownSurveyIdRef, and activeSurvey, and removes saved progress from SurveyProgressStore.

```
progressStore.remove(activeSurvey)
```

### <a id="sv-rn-f-0043"></a>`SV-RN-F-0043`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/components/BottomSection.tsx#BottomSection (L7-55)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** When `skipSubmitButton` is truthy, renders an empty fragment (`<></>`) instead of the submit button.

```
if (skipSubmitButton) {
```

### <a id="sv-rn-f-0044"></a>`SV-RN-F-0044`

- **Surface:** `RN` | **Type:** `control` | **Source:** `posthog-js:packages/react-native/src/surveys/components/BottomSection.tsx#BottomSection (L7-55)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Renders a `TouchableOpacity` submit button displaying `text` (with `getMaxFontSizeMultiplier(appearance, 'button')`), disabled when `submitDisabled` is true, that calls `onSubmit()` when pressed.

```
disabled={submitDisabled}
```

### <a id="sv-rn-f-0047"></a>`SV-RN-F-0047`

- **Surface:** `RN` | **Type:** `branch` | **Source:** `posthog-js:packages/react-native/src/surveys/components/ConfirmationMessage.tsx#ConfirmationMessage (L16-65)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** When isModal is true, renders BottomSection with label appearance.thankYouMessageCloseButtonText, submitDisabled={false}, and onSubmit={onClose}.

```
text={appearance.thankYouMessageCloseButtonText}
```

### <a id="sv-rn-f-0052"></a>`SV-RN-F-0052`

- **Surface:** `RN` | **Type:** `control` | **Source:** `posthog-js:packages/react-native/src/surveys/components/IntroMessage.tsx#IntroMessage (L16-59)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Renders an enabled BottomSection button (labeled appearance.introScreenButtonText or 'Get started') that invokes onStart when submitted.

```
onSubmit={onStart}
```

### <a id="sv-rn-f-0053"></a>`SV-RN-F-0053`

- **Surface:** `RN` | **Type:** `default` | **Source:** `posthog-js:packages/react-native/src/surveys/components/QuestionHeader.tsx#QuestionHeader (L15-47)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Falls back to getContrastingTextColor(appearance.backgroundColor) when appearance.textColor is nullish

```
const textColor = appearance.textColor ?? getContrastingTextColor(appearance.backgroundColor)
```

### <a id="sv-rn-f-0054"></a>`SV-RN-F-0054`

- **Surface:** `RN` | **Type:** `limit` | **Source:** `posthog-js:packages/react-native/src/surveys/components/QuestionHeader.tsx#QuestionHeader (L15-47)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Caps question text font scaling using getMaxFontSizeMultiplier(appearance, 'question')

```
maxFontSizeMultiplier={getMaxFontSizeMultiplier(appearance, 'question')}
```

### <a id="sv-ui-f-0096"></a>`SV-UI-F-0096`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/HostedSurveyEdit.tsx#HostedSurveyEdit (L431-724)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enables survey translations UI and sets activeLanguage to editingLanguage only when FEATURE_FLAGS.SURVEYS_TRANSLATIONS is enabled; otherwise activeLanguage is null.

```
const surveyTranslationsEnabled = !!featureFlags[FEATURE_FLAGS.SURVEYS_TRANSLATIONS]
```

### <a id="sv-ui-f-0117"></a>`SV-UI-F-0117`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/HostedSurveyEdit.tsx#HostedSurveyEdit (L431-724)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When hasBranchingLogic is true, renders the 'Preview branching flow' button (data-attr='preview-survey-branching') that opens SurveyBranchingFlowModal.

```
data-attr="preview-survey-branching"
```

### <a id="sv-ui-f-0150"></a>`SV-UI-F-0150`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/QuickSurveyModal.tsx#QuickSurveyForm (L40-274)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to currentTeam from Kea teamLogic.

```
const { currentTeam } = useValues(teamLogic)
```

### <a id="sv-ui-f-0152"></a>`SV-UI-F-0152`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/QuickSurveyModal.tsx#QuickSurveyForm (L40-274)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Checks !currentTeam?.surveys_opt_in once on mount via useRef and renders a warning notice with SurveyEnableToggle when surveys are disabled for the project.

```
const shouldShowSurveyToggle = useRef(!currentTeam?.surveys_opt_in).current
```

### <a id="sv-ui-f-0158"></a>`SV-UI-F-0158`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/Survey.tsx#scene (L30-34)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Exports the survey scene binding SurveyComponent to surveyLogic and mapping the route param id to props.

```
export const scene: SceneExport<SurveyLogicProps> = {
```

### <a id="sv-ui-f-0159"></a>`SV-UI-F-0159`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/Survey.tsx#SurveyComponent (L36-98)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds editingSurvey, setSelectedPageIndex, and loadSurvey actions from surveyLogic({ id }).

```
const { editingSurvey, setSelectedPageIndex, loadSurvey } = useActions(surveyLogic({ id }))
```

### <a id="sv-ui-f-0161"></a>`SV-UI-F-0161`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/Survey.tsx#SurveyComponent (L36-98)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Registers the 'edit_survey' MaxTool when id is present and not 'new', with initial prompt 'Change this survey: '.

```
identifier: 'edit_survey',
```

### <a id="sv-ui-f-0170"></a>`SV-UI-F-0170`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/Survey.tsx#SurveyForm (L100-124)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Gates the hosted survey editor behind the SURVEYS_HOSTED_EDITOR feature flag and SurveyType.ExternalSurvey.

```
const isHostedEditorEnabled = useFeatureFlag('SURVEYS_HOSTED_EDITOR')
```

### <a id="sv-ui-f-0171"></a>`SV-UI-F-0171`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/Survey.tsx#SurveyForm (L100-124)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds the 'survey' Kea Form to surveyLogic with enableFormOnSubmit.

```
formKey="survey"
```

### <a id="sv-ui-f-0189"></a>`SV-UI-F-0189`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyActionTrigger.tsx#SurveyActionTrigger (L14-49)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to the current survey state from surveyLogic via Kea useValues.

```
const { survey } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0194"></a>`SV-UI-F-0194`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyActionTrigger.tsx#SurveyActionTrigger (L14-49)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates survey.conditions.actions.values with each selected action's id and name via surveyLogic.setSurveyValue when the selection changes.

```
setSurveyValue('conditions', {
```

### <a id="sv-ui-f-0216"></a>`SV-UI-F-0216`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyCompletionConditions (L93-249)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to survey, dataCollectionType, and isAdaptiveLimitFFEnabled from surveyLogic via Kea useValues.

```
const { survey, dataCollectionType, isAdaptiveLimitFFEnabled } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0217"></a>`SV-UI-F-0217`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyCompletionConditions (L93-249)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds setSurveyValue, resetSurveyResponseLimits, resetSurveyAdaptiveSampling, and setDataCollectionType actions from surveyLogic via Kea useActions.

```
useActions(surveyLogic)
```

### <a id="sv-ui-f-0246"></a>`SV-UI-F-0246`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Gates survey translation UI, activeEditingLanguage, and translation validation error banners behind FEATURE_FLAGS.SURVEYS_TRANSLATIONS.

```
const surveyTranslationsEnabled = !!featureFlags[FEATURE_FLAGS.SURVEYS_TRANSLATIONS]
```

### <a id="sv-ui-f-0247"></a>`SV-UI-F-0247`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Allows converting a survey to hosted (ExternalSurvey) only when FEATURE_FLAGS.SURVEYS_HOSTED_EDITOR is enabled and survey.type is not already SurveyType.ExternalSurvey.

```
const canConvertToHosted = hostedEditorEnabled && survey.type !== SurveyType.ExternalSurvey
```

### <a id="sv-ui-f-0283"></a>`SV-UI-F-0283`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders 'Preview branching flow' button when hasBranchingLogic is true, opening SurveyBranchingFlowModal.

```
data-attr="preview-survey-branching"
```

### <a id="sv-ui-f-0292"></a>`SV-UI-F-0292`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Fetches the linked feature flag via api.featureFlags.get(survey.linked_flag_id) when linked_flag_id is set without a loaded linked_flag object.

```
.get(survey.linked_flag_id)
```

### <a id="sv-ui-f-0301"></a>`SV-UI-F-0301`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders SurveyActionTrigger alongside SurveyEventTrigger under 'Activation triggers' only when FEATURE_FLAGS.SURVEYS_ACTIONS is enabled.

```
{featureFlags[FEATURE_FLAGS.SURVEYS_ACTIONS] ? (
```

### <a id="sv-ui-f-0348"></a>`SV-UI-F-0348`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders SurveyBranchingFlowModal controlled by showFlowModal and closes it via setShowFlowModal(false) on onClose.

```
<SurveyBranchingFlowModal survey={survey} isOpen={showFlowModal} onClose={() => setShowFlowModal(false)} />
```

### <a id="sv-ui-f-0353"></a>`SV-UI-F-0353`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEdit.tsx#SurveyEdit (L279-2006)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When FlagSelector changes and survey.linked_flag_id is set without survey.linked_flag loaded, fetches the flag via api.featureFlags.get(survey.linked_flag_id) to populate linked_flag (and on failure clears linked_flag_id to null and removes linkedFlagVariant from conditions); otherwise stores the selected flag in linked_flag and removes linkedFlagVariant from conditions.

```
survey.linked_flag_id &&
```

### <a id="sv-ui-f-0375"></a>`SV-UI-F-0375`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEditIntroScreenHeader.tsx#SurveyEditIntroScreenHeader (L9-38)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to `survey` and `translationErrorsForField` values from `surveyLogic` via Kea `useValues`.

```
const { survey, translationErrorsForField } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0376"></a>`SV-UI-F-0376`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEditIntroScreenHeader.tsx#SurveyEditIntroScreenHeader (L9-38)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds `setSurveyValue` and `setSelectedPageIndex` actions from `surveyLogic` via Kea `useActions`.

```
const { setSurveyValue, setSelectedPageIndex } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0388"></a>`SV-UI-F-0388`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEditQuestionRow.tsx#SurveyEditQuestionGroup (L123-904)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey, descriptionContentType, editingLanguage, translationErrorsForField, and aiGeneratedTranslationFields from surveyLogic via Kea useValues.

```
useValues(surveyLogic)
```

### <a id="sv-ui-f-0389"></a>`SV-UI-F-0389`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEditQuestionRow.tsx#SurveyEditQuestionGroup (L123-904)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds setDefaultForQuestionType, setSurveyValue, resetBranchingForQuestion, setMultipleSurveyQuestion, and clearAiGeneratedTranslationField actions from surveyLogic via Kea useActions.

```
} = useActions(surveyLogic)
```

### <a id="sv-ui-f-0469"></a>`SV-UI-F-0469`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEventTrigger.tsx#SurveyEventSelector (L102-251)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey and surveyRepeatedActivationAvailable from surveyLogic via Kea useValues.

```
const { survey, surveyRepeatedActivationAvailable } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0470"></a>`SV-UI-F-0470`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEventTrigger.tsx#SurveyEventSelector (L102-251)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Wires setSurveyValue action from surveyLogic via Kea useActions to update survey.conditions.

```
const { setSurveyValue } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0478"></a>`SV-UI-F-0478`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEventTrigger.tsx#SurveyEventTrigger (L253-264)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Documents and scopes property filtering on event triggers to posthog-js 1.268+ or posthog-react-native 4.15+.

```
info="Triggers fire when the event is captured in the current user session via a PostHog SDK. Property filtering requires posthog-js 1.268+ or posthog-react-native 4.15+."
```

### <a id="sv-ui-f-0479"></a>`SV-UI-F-0479`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEventTrigger.tsx#SurveyCancelEventTrigger (L266-280)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey from surveyLogic via Kea useValues.

```
const { survey } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0481"></a>`SV-UI-F-0481`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyEventTrigger.tsx#SurveyCancelEventTrigger (L266-280)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Restricts cancel-on-event triggers to web surveys on posthog-js 1.299+.

```
info="Triggers fire when the event is captured in the current user session via a PostHog SDK. Requires posthog-js 1.299+. Web surveys only."
```

### <a id="sv-ui-f-0498"></a>`SV-UI-F-0498`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#getBannerConfig (L37-72)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When variant is 'filtered', returns title 'No responses match current filters', description 'Try adjusting your filters to see matching responses.', bubbleText from getFilterBubbleText(activeFilterTypes), and HedgehogMagnifyingGlass with a 1.2s animation.

```
title: 'No responses match current filters',
```

### <a id="sv-ui-f-0502"></a>`SV-UI-F-0502`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#SurveyNoResponsesBanner (L89-178)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Selects banner variant 'filtered' when isFiltered is true, 'live' when type is 'survey', and 'empty' otherwise.

```
const variant: BannerVariant = isFiltered ? 'filtered' : type === 'survey' ? 'live' : 'empty'
```

### <a id="sv-ui-f-0504"></a>`SV-UI-F-0504`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#SurveyNoResponsesBanner (L89-178)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a 'Clear current filters' LemonButton that calls onClearFilters when both isFiltered and onClearFilters are truthy.

```
<LemonButton type="secondary" size="small" onClick={onClearFilters}>
```

### <a id="sv-ui-f-0506"></a>`SV-UI-F-0506`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#getFilterBubbleText (L180-199)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns 'Try adjusting your filters' when more than one filter type among dateRange, answerFilters, and propertyFilters is active.

```
if (activeCount > 1) {
```

### <a id="sv-ui-f-0507"></a>`SV-UI-F-0507`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#getFilterBubbleText (L180-199)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns 'Try a wider date range' when only dateRange is active.

```
if (dateRange) {
```

### <a id="sv-ui-f-0508"></a>`SV-UI-F-0508`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#getFilterBubbleText (L180-199)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns 'Try different answer filters' when only answerFilters is active.

```
if (answerFilters) {
```

### <a id="sv-ui-f-0509"></a>`SV-UI-F-0509`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyNoResponsesBanner.tsx#getFilterBubbleText (L180-199)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns 'Try fewer property filters' when only propertyFilters is active.

```
if (propertyFilters) {
```

### <a id="sv-ui-f-0519"></a>`SV-UI-F-0519`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyRepeatSchedule.tsx#SurveyIterationOptions (L132-239)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to showSurveyRepeatSchedule and survey from surveyLogic.

```
const { showSurveyRepeatSchedule, survey } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0520"></a>`SV-UI-F-0520`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyRepeatSchedule.tsx#SurveyIterationOptions (L132-239)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Connects to setSurveyValue action from surveyLogic.

```
const { setSurveyValue } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0534"></a>`SV-UI-F-0534`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResponseFilters.tsx#OPERATOR_OPTIONS (L35-65)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** SurveyQuestionType.Open supports filter operators IContains, NotIContains, Regex, NotRegex, Exact, and IsNot.

```
[SurveyQuestionType.Open]: [
```

### <a id="sv-ui-f-0535"></a>`SV-UI-F-0535`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResponseFilters.tsx#OPERATOR_OPTIONS (L35-65)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** SurveyQuestionType.Rating supports filter operators Exact, IsNot, Regex, and NotRegex.

```
[SurveyQuestionType.Rating]: [
```

### <a id="sv-ui-f-0536"></a>`SV-UI-F-0536`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResponseFilters.tsx#OPERATOR_OPTIONS (L35-65)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** SurveyQuestionType.SingleChoice supports filter operators IContains, NotIContains, Regex, NotRegex, Exact, and IsNot.

```
[SurveyQuestionType.SingleChoice]: [
```

### <a id="sv-ui-f-0546"></a>`SV-UI-F-0546`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResponseFilters.tsx#SurveyResponseFilters (L81-270)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders the 'Get SQL Query' button which opens the SurveySQLHelper modal

```
<LemonButton size="small" type="secondary" icon={<IconCode />} onClick={() => setSqlHelperOpen(true)}>
```

### <a id="sv-ui-f-0559"></a>`SV-UI-F-0559`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResponseFilters.tsx#SurveyResponseFilters (L81-270)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders SurveySQLHelper modal controlled by sqlHelperOpen and closed via onClose

```
<SurveySQLHelper isOpen={sqlHelperOpen} onClose={() => setSqlHelperOpen(false)} />
```

### <a id="sv-ui-f-0567"></a>`SV-UI-F-0567`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResultDemo.tsx#SurveyResponsesByQuestionV2Demo (L16-41)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey and surveyDemoData from Kea surveyLogic

```
const { survey, surveyDemoData } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0570"></a>`SV-UI-F-0570`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResultDemo.tsx#DemoStatsSummary (L43-52)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads surveyDemoData from Kea surveyLogic and passes demoStats and demoRates to SurveyStatsSummaryWithData

```
const { surveyDemoData } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0571"></a>`SV-UI-F-0571`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResultDemo.tsx#DemoDataTable (L54-161)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey and surveyDemoData from Kea surveyLogic

```
const { survey, surveyDemoData } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0573"></a>`SV-UI-F-0573`

- **Surface:** `UI` | **Type:** `error` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResultDemo.tsx#DemoDataTable (L54-161)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Silently ignores invalid JSON when parsing demo row person properties, falling back to {}

```
parsedPersonProps = JSON.parse(personProps as string)
```

### <a id="sv-ui-f-0574"></a>`SV-UI-F-0574`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResultDemo.tsx#DemoDataTable (L54-161)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Uses hardcoded example URL 'https://app.example.com/dashboard' for every demo table row

```
url: 'https://app.example.com/dashboard',
```

### <a id="sv-ui-f-0578"></a>`SV-UI-F-0578`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyResultDemo.tsx#SurveyResultDemo (L163-194)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders disabled 'Explore results' primary button with data-attr 'survey-results-explore-demo', tooltip 'This will be available after launching your survey', and disabledReason 'Demo mode - launch survey to explore real results'

```
disabledReason="Demo mode - launch survey to explore real results"
```

### <a id="sv-ui-f-0601"></a>`SV-UI-F-0601`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyEnableToggle (L23-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads currentTeam and currentTeamLoading from teamLogic via useValues

```
const { currentTeam, currentTeamLoading } = useValues(teamLogic)
```

### <a id="sv-ui-f-0602"></a>`SV-UI-F-0602`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyEnableToggle (L23-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds updateCurrentTeam action from teamLogic via useActions

```
const { updateCurrentTeam } = useActions(teamLogic)
```

### <a id="sv-ui-f-0604"></a>`SV-UI-F-0604`

- **Surface:** `UI` | **Type:** `storage` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyEnableToggle (L23-68)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates currentTeam.surveys_opt_in when the toggle is changed

```
surveys_opt_in: checked,
```

### <a id="sv-ui-f-0610"></a>`SV-UI-F-0610`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyDefaultAppearance (L70-188)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads currentTeam from teamLogic via useValues

```
const { currentTeam } = useValues(teamLogic)
```

### <a id="sv-ui-f-0611"></a>`SV-UI-F-0611`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyDefaultAppearance (L70-188)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds updateCurrentTeam action from teamLogic via useActions

```
const { updateCurrentTeam } = useActions(teamLogic)
```

### <a id="sv-ui-f-0612"></a>`SV-UI-F-0612`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyDefaultAppearance (L70-188)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads globalSurveyAppearanceConfigAvailable from surveysLogic via useValues

```
const { globalSurveyAppearanceConfigAvailable } = useValues(surveysLogic)
```

### <a id="sv-ui-f-0617"></a>`SV-UI-F-0617`

- **Surface:** `UI` | **Type:** `storage` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveyDefaultAppearance (L70-188)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Persists sanitizedAppearance into currentTeam.survey_config.appearance via updateCurrentTeam

```
appearance: sanitizedAppearance,
```

### <a id="sv-ui-f-0623"></a>`SV-UI-F-0623`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#openSurveysSettingsDialog (L203-212)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Opens a LemonDialog modal titled 'Surveys settings' (width 600, primary button 'Done') containing SurveySettings with isModal

```
LemonDialog.open({
```

### <a id="sv-ui-f-0624"></a>`SV-UI-F-0624`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveySettings.tsx#SurveysDisabledBanner (L214-247)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads showSurveysDisabledBanner from surveysLogic via useValues

```
const { showSurveysDisabledBanner } = useValues(surveysLogic)
```

### <a id="sv-ui-f-0629"></a>`SV-UI-F-0629`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#UsersCount (L62-100)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads unique_persons count from survey shown stats to display Shown count and compute response rate.

```
const uniqueUsersShown = stats[SurveyEventName.SHOWN].unique_persons
```

### <a id="sv-ui-f-0630"></a>`SV-UI-F-0630`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#UsersCount (L62-100)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads unique_persons count from survey sent stats to display Responses count.

```
const uniqueUsersSent = stats[SurveyEventName.SENT].unique_persons
```

### <a id="sv-ui-f-0634"></a>`SV-UI-F-0634`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#ResponsesCount (L102-141)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads total_count from survey shown stats to display Shown impressions.

```
const impressions = stats[SurveyEventName.SHOWN].total_count
```

### <a id="sv-ui-f-0635"></a>`SV-UI-F-0635`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#ResponsesCount (L102-141)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads total_count from survey sent stats to display Responses count.

```
const sent = stats[SurveyEventName.SENT].total_count
```

### <a id="sv-ui-f-0639"></a>`SV-UI-F-0639`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsStackedBar (L164-206)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey shown total_count / unique_persons for the stacked bar denominator.

```
: stats[SurveyEventName.SHOWN].unique_persons
```

### <a id="sv-ui-f-0640"></a>`SV-UI-F-0640`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsStackedBar (L164-206)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey shown only_seen counts for the 'Unanswered' stacked bar segment.

```
: stats[SurveyEventName.SHOWN].unique_persons_only_seen
```

### <a id="sv-ui-f-0641"></a>`SV-UI-F-0641`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsStackedBar (L164-206)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey dismissed counts for the 'Dismissed without answers' stacked bar segment.

```
: stats[SurveyEventName.DISMISSED].unique_persons
```

### <a id="sv-ui-f-0642"></a>`SV-UI-F-0642`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsStackedBar (L164-206)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey sent counts for the 'Responses' stacked bar segment.

```
: stats[SurveyEventName.SENT].unique_persons
```

### <a id="sv-ui-f-0651"></a>`SV-UI-F-0651`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsSummary (L280-337)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to filterSurveyStatsByDistinctId, processedSurveyStats, surveyRates, surveyBaseStatsLoading, surveyDismissedAndSentCountLoading, resultsRequeryInProgress, and surveyResponseOutcomes from surveyLogic via Kea useValues.

```
} = useValues(surveyLogic)
```

### <a id="sv-ui-f-0652"></a>`SV-UI-F-0652`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsSummary (L280-337)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds the setFilterSurveyStatsByDistinctId action from surveyLogic via Kea useActions.

```
const { setFilterSurveyStatsByDistinctId } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0656"></a>`SV-UI-F-0656`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyStatsSummary.tsx#SurveyStatsSummary (L280-337)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a LemonSwitch labeled 'Count each person once' (data-attr 'survey-stats-count-person-once') bound to filterSurveyStatsByDistinctId that calls setFilterSurveyStatsByDistinctId(checked) on change, wrapped in a tooltip explaining unique-person vs individual-submission counting.

```
label="Count each person once"
```

### <a id="sv-ui-f-0677"></a>`SV-UI-F-0677`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyTranslationFields.tsx#SurveyTranslationFields (L70-307)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to `survey` and `aiGeneratedTranslationFields` from `surveyLogic` via Kea `useValues`.

```
const { survey, aiGeneratedTranslationFields } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0678"></a>`SV-UI-F-0678`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyTranslationFields.tsx#SurveyTranslationFields (L70-307)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds `setSurveyValue` and `clearAiGeneratedTranslationField` actions from `surveyLogic` via Kea `useActions`.

```
const { setSurveyValue, clearAiGeneratedTranslationField } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0713"></a>`SV-UI-F-0713`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyTranslations.tsx#SurveyTranslations (L17-170)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to survey, editingLanguage, aiGeneratedTranslationFields, generatingTranslationDrafts, and dataProcessingAccepted from surveyLogic

```
} = useValues(surveyLogic)
```

### <a id="sv-ui-f-0714"></a>`SV-UI-F-0714`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyTranslations.tsx#SurveyTranslations (L17-170)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds setEditingLanguage and generateTranslationDrafts actions from surveyLogic

```
const { setEditingLanguage, generateTranslationDrafts } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0732"></a>`SV-UI-F-0732`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#CopyResponseKeyButton (L31-43)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a 'Copy survey response key' button that copies `${SurveyEventProperties.SURVEY_RESPONSE}_${questionId}` to the clipboard with label 'survey response key' on click.

```
void copyToClipboard(`${SurveyEventProperties.SURVEY_RESPONSE}_${questionId}`, 'survey response key')
```

### <a id="sv-ui-f-0733"></a>`SV-UI-F-0733`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Wires setAnswerFilters, setPropertyFilters, setDateRange, setShowArchivedResponses, clearFilters, and setResultsFiltersExpanded actions from surveyLogic.

```
} = useActions(surveyLogic)
```

### <a id="sv-ui-f-0734"></a>`SV-UI-F-0734`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates an existing Event answer filter for the question when found by getSurveyIdBasedResponseKey, otherwise appends a new Event filter and calls setAnswerFilters.

```
if (filterIndex >= 0) {
```

### <a id="sv-ui-f-0735"></a>`SV-UI-F-0735`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults newly created question answer filters to PropertyOperator.Exact when no filter exists for that question.

```
operator: PropertyOperator.Exact,
```

### <a id="sv-ui-f-0737"></a>`SV-UI-F-0737`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** DateFilter control updates the survey results date range via setDateRange.

```
onChange={(dateFrom, dateTo) => setDateRange({ date_from: dateFrom, date_to: dateTo })}
```

### <a id="sv-ui-f-0738"></a>`SV-UI-F-0738`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** "Filters" toggle button flips resultsFiltersExpanded and displays a LemonBadge.Number with activeResultsFilterCount when greater than 0.

```
data-attr="survey-results-filters-toggle"
```

### <a id="sv-ui-f-0740"></a>`SV-UI-F-0740`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** "Clear filters" button resets active filters via clearFilters.

```
data-attr="survey-results-clear-filters"
```

### <a id="sv-ui-f-0745"></a>`SV-UI-F-0745`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** "Add filter" PropertyFilters control updates propertyFilters across EventProperties, PersonProperties, EventFeatureFlags, Cohorts, HogQLExpression, and groupsTaxonomicTypes.

```
buttonText="Add filter"
```

### <a id="sv-ui-f-0746"></a>`SV-UI-F-0746`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** "Show archived" switch toggles showArchivedResponses via setShowArchivedResponses.

```
label="Show archived"
```

### <a id="sv-ui-f-0749"></a>`SV-UI-F-0749`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** CopyResponseKeyButton control allows copying the response property key for the question.

```
{question.id && <CopyResponseKeyButton questionId={question.id} />}
```

### <a id="sv-ui-f-0750"></a>`SV-UI-F-0750`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Operator LemonSelect dropdown sets the filter operator for a question from OPERATOR_OPTIONS[question.type] and updates answerFilters.

```
handleUpdateFilter(question.id ?? '', 'operator', val)
```

### <a id="sv-ui-f-0752"></a>`SV-UI-F-0752`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** PropertyValue input edits the event filter value for SurveyEventProperties.SURVEY_RESPONSE_<question.id> and updates answerFilters.

```
propertyKey={`${SurveyEventProperties.SURVEY_RESPONSE}_${question.id}`}
```

### <a id="sv-ui-f-0754"></a>`SV-UI-F-0754`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Scopes response PropertyValue lookups to the "survey sent" event (SurveyEventName.SENT).

```
eventNames={[SurveyEventName.SENT]}
```

### <a id="sv-ui-f-0755"></a>`SV-UI-F-0755`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey, answerFilters, propertyFilters, hasActiveFilters, activeResultsFilterCount, resultsFiltersExpanded, dateRange, showArchivedResponses, and surveyAsInsightURL from surveyLogic.

```
} = useValues(surveyLogic)
```

### <a id="sv-ui-f-0756"></a>`SV-UI-F-0756`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyFilters.tsx#SurveyResultsFiltersBar (L45-255)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads groupsTaxonomicTypes from groupsModel for property filter taxonomic group options.

```
const { groupsTaxonomicTypes } = useValues(groupsModel)
```

### <a id="sv-ui-f-0794"></a>`SV-UI-F-0794`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveySummaryContent (L539-650)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a "Looking for all responses?" button (data-attr="survey-results-view-responses") that triggers onViewResponses.

```
data-attr="survey-results-view-responses"
```

### <a id="sv-ui-f-0837"></a>`SV-UI-F-0837`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveyViewRedesign (L90-471)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Configures the 'Edit' button to link to the guided survey wizard route (urls.surveyWizard(survey.id)) when canUseSurveyWizard(survey) and preferredEditor === 'guided', or to invoke editingSurvey(true) otherwise.

```
render: <LinkPrimitive to={urls.surveyWizard(survey.id)} />,
```

### <a id="sv-ui-f-0848"></a>`SV-UI-F-0848`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveyResponsesContent (L664-757)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey, query, loading, archivedResponseUuids, and expandedResponseUuids state from surveyLogic via useValues

```
} = useValues(surveyLogic)
```

### <a id="sv-ui-f-0849"></a>`SV-UI-F-0849`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveyResponsesContent (L664-757)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds setResponseExpanded and toggleResponseExpansion actions from surveyLogic via useActions

```
const { setResponseExpanded, toggleResponseExpansion } = useActions(surveyLogic)
```

### <a id="sv-ui-f-0858"></a>`SV-UI-F-0858`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveyResponsesContent (L664-757)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Skips toggling row expansion when a click originates inside INTERACTIVE_SELECTOR; otherwise calls toggleResponseExpansion(uuid)

```
if ((e.target as HTMLElement).closest(INTERACTIVE_SELECTOR)) {
```

### <a id="sv-ui-f-0859"></a>`SV-UI-F-0859`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveyResponsesContent (L664-757)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Calls setResponseExpanded(uuid, true) on row expand when the record has a uuid

```
setResponseExpanded(uuid, true)
```

### <a id="sv-ui-f-0860"></a>`SV-UI-F-0860`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyViewRedesign/SurveyViewRedesign.tsx#SurveyResponsesContent (L664-757)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Calls setResponseExpanded(uuid, false) on row collapse when the record has a uuid

```
setResponseExpanded(uuid, false)
```

### <a id="sv-ui-f-0861"></a>`SV-UI-F-0861`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyWidgetCustomization.tsx#SurveyWidgetCustomization (L35-149)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey and surveyErrors state from surveyLogic via Kea useValues.

```
const { survey, surveyErrors } = useValues(surveyLogic)
```

### <a id="sv-ui-f-0872"></a>`SV-UI-F-0872`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyWidgetCustomization.tsx#SurveyWidgetCustomization (L35-149)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Disables both the SurveyTabPositionSelector and the tab position dropdown when surveysStylingAvailable is false.

```
disabled={!surveysStylingAvailable}
```

### <a id="sv-ui-f-0876"></a>`SV-UI-F-0876`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyWidgetCustomization.tsx#SurveyWidgetCustomization (L35-149)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Paywalls "Button position" customization behind AvailableFeature.SURVEYS_STYLING.

```
<PayGateIcon feature={AvailableFeature.SURVEYS_STYLING} />
```

### <a id="sv-ui-f-0877"></a>`SV-UI-F-0877`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/SurveyWidgetCustomization.tsx#SurveyWidgetCustomization (L35-149)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Documents that Button position requires at least version 1.294.0 of posthog-js.

```
info="Requires at least version 1.294.0 of posthog-js"
```

### <a id="sv-ui-f-0878"></a>`SV-UI-F-0878`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/Surveys.tsx#NewSurveyButton (L39-84)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds addProductIntent action from surveysLogic

```
const { addProductIntent } = useActions(surveysLogic)
```

### <a id="sv-ui-f-0879"></a>`SV-UI-F-0879`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/Surveys.tsx#NewSurveyButton (L39-84)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Checks TODAY_RAIL_NAV feature flag to choose between Button and LemonButton rendering

```
const todayRailNav = useFeatureFlag('TODAY_RAIL_NAV')
```

### <a id="sv-ui-f-0880"></a>`SV-UI-F-0880`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/Surveys.tsx#NewSurveyButton (L39-84)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Records SURVEY_ADD_NEW product intent for ProductKey.SURVEYS when the New survey button is clicked

```
intent_context: ProductIntentContext.SURVEY_ADD_NEW,
```

### <a id="sv-ui-f-0903"></a>`SV-UI-F-0903`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/branching-flow/SurveyBranchingFlowModal.tsx#SurveyBranchingFlowModal (L14-22)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a 90vw-wide LemonModal titled "Survey flow" controlled by isOpen and onClose

```
<LemonModal title="Survey flow" isOpen={isOpen} onClose={onClose} width="90vw">
```

### <a id="sv-ui-f-0926"></a>`SV-UI-F-0926`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/components/ArchiveSurveyButton.tsx#ArchiveSurveyButton (L13-33)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds archiveResponse and unarchiveResponse actions from Kea surveyLogic keyed by surveyId

```
const { archiveResponse, unarchiveResponse } = useActions(surveyLogic({ id: surveyId }))
```

### <a id="sv-ui-f-0927"></a>`SV-UI-F-0927`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/components/ArchiveSurveyButton.tsx#ArchiveSurveyButton (L13-33)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults isArchived to false when archivedResponseUuids is undefined or does not contain responseUuid

```
const isArchived = archivedResponseUuids?.has(responseUuid) ?? false
```

### <a id="sv-ui-f-0928"></a>`SV-UI-F-0928`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/components/ArchiveSurveyButton.tsx#ArchiveSurveyButton (L13-33)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When clicked with a truthy responseUuid, dispatches unarchiveResponse(responseUuid) if isArchived is true or archiveResponse(responseUuid) otherwise; no-ops if responseUuid is falsy

```
isArchived ? unarchiveResponse(responseUuid) : archiveResponse(responseUuid)
```

### <a id="sv-ui-f-0929"></a>`SV-UI-F-0929`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/ArchiveSurveyButton.tsx#ArchiveSurveyButton (L13-33)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a full-width LemonButton with data-attr="events-table-archive-survey" labeled "Unarchive response" when archived or "Archive response" when not archived

```
{isArchived ? 'Unarchive' : 'Archive'} response
```

### <a id="sv-ui-f-1125"></a>`SV-UI-F-1125`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/components/SurveyUrlAudienceEstimate.tsx#SurveyUrlAudienceEstimate (L71-174)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey state from Kea surveyLogic via useValues

```
const { survey } = useValues(surveyLogic)
```

### <a id="sv-ui-f-1128"></a>`SV-UI-F-1128`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/components/SurveyUrlAudienceEstimate.tsx#SurveyUrlAudienceEstimate (L71-174)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries HogQL events table for unique person_id count of $pageview events in the last URL_AUDIENCE_ESTIMATE_DAYS matching $current_url

```
const response = await api.queryHogQL<[[number]]>(query, URL_AUDIENCE_ESTIMATE_TAGS, {
```

### <a id="sv-ui-f-1204"></a>`SV-UI-F-1204`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-branching/QuestionBranchingInput.tsx#QuestionBranchingInput (L41-141)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey, getBranchingDropdownValue, and editingLanguage from surveyLogic via useValues.

```
const { survey, getBranchingDropdownValue, editingLanguage } = useValues(surveyLogic)
```

### <a id="sv-ui-f-1228"></a>`SV-UI-F-1228`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/MultipleChoiceBarChart.tsx#CATEGORY_LABEL_WIDTH (L15-15)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Caps category label width in the multiple choice bar chart to 280 pixels

```
const CATEGORY_LABEL_WIDTH = 280
```

### <a id="sv-ui-f-1232"></a>`SV-UI-F-1232`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/MultipleChoiceBarChart.tsx#MultipleChoiceBarChart (L31-114)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Limits category label width to CATEGORY_LABEL_WIDTH (280px)

```
maxCategoryLabelWidth: CATEGORY_LABEL_WIDTH,
```

### <a id="sv-ui-f-1234"></a>`SV-UI-F-1234`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/MultipleChoiceBarChart.tsx#MultipleChoiceBarChart (L31-114)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enforces a minimum bar band size of 32 pixels

```
minBandSize: 32,
```

### <a id="sv-ui-f-1235"></a>`SV-UI-F-1235`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/MultipleChoiceBarChart.tsx#MultipleChoiceBarChart (L31-114)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Constrains the bar chart value domain to 0 through 100 percent

```
valueDomain: { min: 0, max: 100 },
```

### <a id="sv-ui-f-1238"></a>`SV-UI-F-1238`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/MultipleChoiceBarChart.tsx#MultipleChoiceBarChart (L31-114)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders ChoiceTooltip on bar hover with chartData, tooltipContextByIndex, and activeChoiceLabel

```
tooltip={renderTooltip}
```

### <a id="sv-ui-f-1239"></a>`SV-UI-F-1239`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/MultipleChoiceBarChart.tsx#MultipleChoiceBarChart (L31-114)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clicking a bar invokes onBarClick with the clicked bar's dataIndex

```
onPointClick={({ dataIndex }) => onBarClick(dataIndex)}
```

### <a id="sv-ui-f-1301"></a>`SV-UI-F-1301`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/RatingBarChart.tsx#RatingBarChart (L26-92)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults each rating bar's value to 0 when data[i]?.value is nullish.

```
data: chartLabels.map((_, i) => data[i]?.value ?? 0),
```

### <a id="sv-ui-f-1304"></a>`SV-UI-F-1304`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/RatingBarChart.tsx#RatingBarChart (L26-92)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a RatingTooltip for chart bars with chartLabels, tooltipContextByIndex, npsBucketByIndex, and activeRatingLabel.

```
tooltip={renderTooltip}
```

### <a id="sv-ui-f-1305"></a>`SV-UI-F-1305`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/RatingBarChart.tsx#RatingBarChart (L26-92)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clicking a bar point invokes onBarClick with the clicked bar's dataIndex.

```
onPointClick={({ dataIndex }) => onBarClick(dataIndex)}
```

### <a id="sv-ui-f-1396"></a>`SV-UI-F-1396`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/VirtualizedResponseList.tsx#MAX_STATIC_RESPONSES (L22-22)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Caps non-virtualized static grid rendering at 24 responses.

```
const MAX_STATIC_RESPONSES = 24
```

### <a id="sv-ui-f-1398"></a>`SV-UI-F-1398`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/VirtualizedResponseList.tsx#ResponseListItem (L24-102)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Treats responses longer than 100 characters as long responses that can be expanded.

```
const isLongResponse = responseText.length > 100
```

### <a id="sv-ui-f-1399"></a>`SV-UI-F-1399`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/VirtualizedResponseList.tsx#ResponseListItem (L24-102)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Clicking a long response card toggles its expanded popover state.

```
onClick={isLongResponse ? () => setIsExpanded(!isExpanded) : undefined}
```

### <a id="sv-ui-f-1402"></a>`SV-UI-F-1402`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/components/question-visualizations/VirtualizedResponseList.tsx#ResponseListItem (L24-102)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a ViewRecordingButton for response.sessionId and response.timestamp with recording existence checking enabled.

```
sessionId={response.sessionId}
```

### <a id="sv-ui-f-1483"></a>`SV-UI-F-1483`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/hooks/useSurveyResponseColumns.tsx#getTraceIdFromRecord (L22-28)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads $ai_trace_id from record[0].properties and defaults to null when missing.

```
return event?.properties?.$ai_trace_id ?? null
```

### <a id="sv-ui-f-1485"></a>`SV-UI-F-1485`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/hooks/useSurveyResponseColumns.tsx#useSurveyResponseColumns (L46-124)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Status column displays getSurveyResponseStatus(event.event, event.properties) in a warning tag, or 'Completed' in a success tag when falsy.

```
{status || 'Completed'}
```

### <a id="sv-ui-f-1493"></a>`SV-UI-F-1493`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:frontend/src/scenes/surveys/hooks/useSurveyResponseColumns.tsx#useSurveyResponseColumns (L46-124)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a 'View LLM trace' button linking to the AI observability trace route when traceId is present.

```
to={urls.aiObservabilityTrace(traceId)}
```

### <a id="sv-ui-f-1903"></a>`SV-UI-F-1903`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/surveyLogic.tsx#surveyLogic (L1487-4198)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Updates a survey via api.surveys.update, refreshes its project tree item, and records product intent if surveyPayload.intentContext is set.

```
const response = await api.surveys.update(props.id, surveyPayload)
```

### <a id="sv-ui-f-1934"></a>`SV-UI-F-1934`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/surveyLogic.tsx#surveyLogic (L1487-4198)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Generates survey translation drafts via surveysGenerateTranslationsCreate, merges returned survey and question translation patches, and marks generated_field_paths.

```
const result = await surveysGenerateTranslationsCreate(String(teamId), values.survey.id, {
```

### <a id="sv-ui-f-2146"></a>`SV-UI-F-2146`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/surveyLogic.tsx#surveyLogic (L1487-4198)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Enables editing mode via editingSurvey(true) whenever searchParams.edit is truthy on route navigation.

```
if (searchParams.edit) {
```

### <a id="sv-ui-f-2148"></a>`SV-UI-F-2148`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/surveyLogic.tsx#surveyLogic (L1487-4198)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** editingSurvey updates the URL search params by setting edit=true when editing or deleting 'edit' otherwise.

```
if (editing) {
```

### <a id="sv-ui-f-2434"></a>`SV-UI-F-2434`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#buildSurveyResponseEventFilter (L721-726)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Matches SurveyEventName.SENT events in the HogQL survey response filter

```
return `(event = '${SurveyEventName.SENT}' OR (
```

### <a id="sv-ui-f-2435"></a>`SV-UI-F-2435`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#buildSurveyResponseEventFilter (L721-726)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Matches SurveyEventName.DISMISSED or SurveyEventName.ABANDONED events when $survey_partially_completed is 'true'

```
event IN ('${SurveyEventName.DISMISSED}', '${SurveyEventName.ABANDONED}')
```

### <a id="sv-ui-f-2446"></a>`SV-UI-F-2446`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#getSurveyResponseStatus (L877-893)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns 'Dismissed' when eventName is SurveyEventName.DISMISSED and 'Abandoned' otherwise

```
if (eventName === SurveyEventName.DISMISSED) {
```

### <a id="sv-ui-f-2452"></a>`SV-UI-F-2452`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#transformSurveyResponseRows (L906-951)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Sets $survey_completed / $survey_partially_completed from outcome and maps event name to SurveyEventName.SENT when outcome is 'completed' or latestEvent otherwise

```
event: outcome === 'completed' ? SurveyEventName.SENT : latestEvent,
```

### <a id="sv-ui-f-2455"></a>`SV-UI-F-2455`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#buildSurveyResponsesQuery (L1021-1051)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Builds a HogQL query selecting response metadata, answers, status, timestamp, respondent, chosen columns, and actions ordered by submitted_at DESC.

```
return `SELECT ${columns.join(',\n')} FROM (${merged}) ORDER BY submitted_at DESC`
```

### <a id="sv-ui-f-2462"></a>`SV-UI-F-2462`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#buildSurveyResponseSQLQuery (L1121-1138)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Replaces the '{filters}' placeholder with '1 = 1' in standalone survey response SQL queries.

```
}).replaceAll('{filters}', '1 = 1')
```

### <a id="sv-ui-f-2464"></a>`SV-UI-F-2464`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/utils.ts#buildSurveyResponseSQLQuery (L1121-1138)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Caps standalone survey response SQL query results at 100 rows ordered by submitted_at DESC.

```
FROM (${merged}) ORDER BY submitted_at DESC LIMIT 100`
```

### <a id="sv-ui-f-2501"></a>`SV-UI-F-2501`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/utils/demoDataGenerator.ts#generateRatingResponse (L142-176)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults non-NPS rating scale to 5 when ratingQuestion.scale is falsy

```
const scale = ratingQuestion.scale || 5
```

### <a id="sv-ui-f-2510"></a>`SV-UI-F-2510`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/utils/demoDataGenerator.ts#generateSingleChoiceResponse (L178-218)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Falls back to choices[0] if weighted selection loop completes without returning

```
return choices[0]
```

### <a id="sv-ui-f-2512"></a>`SV-UI-F-2512`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:frontend/src/scenes/surveys/utils/demoDataGenerator.ts#generateMultipleChoiceResponse (L220-248)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Selects between MIN_SELECTIONS (1) and Math.min(MAX_SELECTIONS, availableChoices.length) choices, excluding the trailing open choice if present

```
Math.min(DEMO_CONFIG.MAX_SELECTIONS, availableChoices.length)
```

### <a id="sv-ui-f-2514"></a>`SV-UI-F-2514`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:frontend/src/scenes/surveys/utils/demoDataGenerator.ts#generateDemoSurveyResults (L272-329)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Defaults responseCount to DEMO_CONFIG.DEFAULT_RESPONSE_COUNT (76)

```
responseCount: number = DEMO_CONFIG.DEFAULT_RESPONSE_COUNT
```

### <a id="sv-ui-f-2591"></a>`SV-UI-F-2591`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/SurveyWizard.tsx#SurveyWizardComponent (L62-70)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds surveyLogic with the survey id prop for the SurveyWizard subtree.

```
<BindLogic logic={surveyLogic} props={{ id }}>
```

### <a id="sv-ui-f-2594"></a>`SV-UI-F-2594`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/SurveyWizard.tsx#SurveyWizard (L72-558)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads survey and surveyWarnings from surveyLogic

```
const { survey, surveyWarnings } = useValues(surveyLogic)
```

### <a id="sv-ui-f-2606"></a>`SV-UI-F-2606`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/SurveyWizard.tsx#SurveyWizard (L72-558)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Switching to the full editor sets preferred editor to 'full' and navigates to ?edit=true when editing or #fromTemplate=true when creating

```
const target = isEditing ? `${urls.survey(id)}?edit=true` : `${urls.survey(id)}#fromTemplate=true`
```

### <a id="sv-ui-f-2642"></a>`SV-UI-F-2642`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/steps/AppearanceStep.tsx#AppearanceStep (L37-323)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads guardAvailableFeature helper from Kea upgradeModalLogic

```
const { guardAvailableFeature } = useValues(upgradeModalLogic)
```

### <a id="sv-ui-f-2662"></a>`SV-UI-F-2662`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/steps/AppearanceStep.tsx#AppearanceStep (L37-323)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Guards checking 'Hide PostHog branding' with AvailableFeature.WHITE_LABELLING via guardAvailableFeature while allowing unchecking directly

```
guardAvailableFeature(AvailableFeature.WHITE_LABELLING, () =>
```

### <a id="sv-ui-f-2762"></a>`SV-UI-F-2762`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/steps/TemplateStep.tsx#TemplateStep (L96-245)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Configures useMaxTool for create_survey with the trimmed prompt prefixed by ! and invokes handleMaxSurveyCreated with SURVEY_CREATED_SOURCE.SURVEY_WIZARD on callback

```
identifier: 'create_survey',
```

### <a id="sv-ui-f-2884"></a>`SV-UI-F-2884`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/surveyWizardLogic.ts#surveyWizardLogic (L194-610)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When templateMode is 'hosted', selectTemplate redirects to urls.survey('new') for new surveys or '${urls.survey(props.id)}?edit=true' for existing surveys.

```
router.actions.push(props.id === 'new' ? urls.survey('new') : `${urls.survey(props.id)}?edit=true`)
```

### <a id="sv-ui-f-2886"></a>`SV-UI-F-2886`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/surveyWizardLogic.ts#surveyWizardLogic (L194-610)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** launchSurvey creates the survey via api.surveys.create with start_date set to the current ISO timestamp and dispatches launchSurveySuccess.

```
const createdSurvey = await api.surveys.create(surveyData)
```

### <a id="sv-ui-f-2889"></a>`SV-UI-F-2889`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:frontend/src/scenes/surveys/wizard/surveyWizardLogic.ts#surveyWizardLogic (L194-610)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** saveDraft creates a draft survey via api.surveys.create with values.survey (without start_date) and dispatches saveDraftSuccess.

```
const createdSurvey = await api.surveys.create(values.survey)
```

### <a id="sv-ui-f-2903"></a>`SV-UI-F-2903`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResponsesByQuestionV2 (L26-44)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to survey state from surveyLogic via Kea useValues.

```
const { survey } = useValues(surveyLogic)
```

### <a id="sv-ui-f-2904"></a>`SV-UI-F-2904`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResponsesByQuestionV2 (L26-44)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Skips rendering visualization for questions without an id or with type SurveyQuestionType.Link.

```
if (!question.id || question.type === SurveyQuestionType.Link) {
```

### <a id="sv-ui-f-2905"></a>`SV-UI-F-2905`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResponsesByQuestionV2 (L26-44)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders SurveyQuestionVisualization and a LemonDivider for each eligible survey question.

```
<SurveyQuestionVisualization question={question} questionIndex={i} />
```

### <a id="sv-ui-f-2906"></a>`SV-UI-F-2906`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Subscribes to survey, query, loading, stats, filter, and archived response state from surveyLogic via Kea useValues.

```
} = useValues(surveyLogic)
```

### <a id="sv-ui-f-2907"></a>`SV-UI-F-2907`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds clearFilters action from surveyLogic via Kea useActions.

```
const { clearFilters } = useActions(surveyLogic)
```

### <a id="sv-ui-f-2908"></a>`SV-UI-F-2908`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Determines whether the survey has at least one response from processedSurveyStats for the 'survey sent' event.

```
const atLeastOneResponse = !!processedSurveyStats?.[SurveyEventName.SENT].total_count
```

### <a id="sv-ui-f-2909"></a>`SV-UI-F-2909`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Conditionally renders SurveyHeadline only when isSurveyHeadlineEnabled is truthy.

```
{isSurveyHeadlineEnabled && <SurveyHeadline />}
```

### <a id="sv-ui-f-2910"></a>`SV-UI-F-2910`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders stats summary, per-question visualizations, Explore results button, and events table when results are refreshing or at least one response exists; otherwise renders stats summary and SurveyNoResponsesBanner.

```
{isRefreshingResults || atLeastOneResponse ? (
```

### <a id="sv-ui-f-2911"></a>`SV-UI-F-2911`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders primary 'Explore results' LemonButton linking to surveyAsInsightURL (data-attr 'survey-results-explore').

```
data-attr="survey-results-explore"
```

### <a id="sv-ui-f-2912"></a>`SV-UI-F-2912`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Hides the response events table when disableEventsTable is true, and renders LemonSkeleton during initial survey load.

```
(isInitialSurveyLoad ? (
```

### <a id="sv-ui-f-2913"></a>`SV-UI-F-2913`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders Query with dataTableQuery, survey column renderers, expandable response rows, export query responsesExportQuery excluding 'response' and 'actions' columns, and transformSurveyResponseRows.

```
dataTableExportExcludedColumns: ['response', 'actions'],
```

### <a id="sv-ui-f-2914"></a>`SV-UI-F-2914`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns empty rowProps when a table row record is not an object or lacks a 'result' property.

```
if (typeof record !== 'object' || !record || !('result' in record)) {
```

### <a id="sv-ui-f-2915"></a>`SV-UI-F-2915`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Returns empty rowProps when record.result is not an array; otherwise applies 'opacity-50' className if result[0]?.uuid is in archivedResponseUuids.

```
if (!Array.isArray(result)) {
```

### <a id="sv-ui-f-2916"></a>`SV-UI-F-2916`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/SurveyResult.tsx#SurveyResult (L46-156)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Passes clearFilters handler to SurveyNoResponsesBanner when hasActiveFilters is true so users can clear active filters.

```
onClearFilters={hasActiveFilters ? clearFilters : undefined}
```

### <a id="sv-ui-f-2917"></a>`SV-UI-F-2917`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#OUTCOME_LABELS (L6-10)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maps the 'Completed' survey outcome to the label 'Completed' and tooltip description 'Finished the survey.'

```
Completed: { label: 'Completed', description: 'Finished the survey.' },
```

### <a id="sv-ui-f-2918"></a>`SV-UI-F-2918`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#OUTCOME_LABELS (L6-10)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maps the 'Dismissed' survey outcome to the label 'Partial · dismissed' and tooltip description 'Saved answers before closing the survey.'

```
Dismissed: { label: 'Partial · dismissed', description: 'Saved answers before closing the survey.' },
```

### <a id="sv-ui-f-2919"></a>`SV-UI-F-2919`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#OUTCOME_LABELS (L6-10)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Maps the 'Abandoned' survey outcome to the label 'Partial · abandoned' and tooltip description 'Saved answers without finishing the survey.'

```
Abandoned: { label: 'Partial · abandoned', description: 'Saved answers without finishing the survey.' },
```

### <a id="sv-ui-f-2920"></a>`SV-UI-F-2920`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#SurveyResponseBreakdown (L12-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a 'Response completion' section with the heading 'Response completion'

```
<h4 className="text-sm font-semibold m-0">Response completion</h4>
```

### <a id="sv-ui-f-2921"></a>`SV-UI-F-2921`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#SurveyResponseBreakdown (L12-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Displays an 'Of response submissions' label with a tooltip stating that each response submission counts once (including partial answers) and that the person-counting toggle does not change this breakdown

```
<Tooltip title="Each response submission counts once, including submissions with partial answers. The person-counting toggle does not change this breakdown.">
```

### <a id="sv-ui-f-2922"></a>`SV-UI-F-2922`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#SurveyResponseBreakdown (L12-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders a green ('bg-success') status dot when outcome.label is 'Completed' and a muted ('bg-muted') dot for all other outcomes

```
className={`size-2 shrink-0 rounded-full ${outcome.label === 'Completed' ? 'bg-success' : 'bg-muted'}`}
```

### <a id="sv-ui-f-2923"></a>`SV-UI-F-2923`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#SurveyResponseBreakdown (L12-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Displays the mapped label from OUTCOME_LABELS with its description tooltip, falling back to the raw outcome.label when no mapping exists

```
<span>{content?.label ?? outcome.label}</span>
```

### <a id="sv-ui-f-2924"></a>`SV-UI-F-2924`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#SurveyResponseBreakdown (L12-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Formats and displays each outcome's count using humanFriendlyNumber

```
<span className="font-semibold">{humanFriendlyNumber(outcome.count)}</span>{' '}
```

### <a id="sv-ui-f-2925"></a>`SV-UI-F-2925`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseBreakdown.tsx#SurveyResponseBreakdown (L12-51)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Formats and displays each outcome's percentage to 1 decimal place

```
<span className="text-secondary ml-2">{percentage(outcome.percentage, 1)}</span>
```

### <a id="sv-ui-f-2926"></a>`SV-UI-F-2926`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads responseColumns from surveyLogic via Kea useValues.

```
const { responseColumns } = useValues(surveyLogic)
```

### <a id="sv-ui-f-2927"></a>`SV-UI-F-2927`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Binds addResponseColumn and removeResponseColumn actions from surveyLogic via Kea useActions.

```
const { addResponseColumn, removeResponseColumn } = useActions(surveyLogic)
```

### <a id="sv-ui-f-2928"></a>`SV-UI-F-2928`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Resets picking to false when the LemonDropdown closes (visible becomes false).

```
onVisibilityChange={(visible) => !visible && setPicking(false)}
```

### <a id="sv-ui-f-2929"></a>`SV-UI-F-2929`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Scopes the TaxonomicFilter event property picker to the SurveyEventName.SENT ('survey sent') event.

```
eventNames={[SurveyEventName.SENT]}
```

### <a id="sv-ui-f-2930"></a>`SV-UI-F-2930`

- **Surface:** `UI` | **Type:** `fact` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Excludes already-added EventProperties and PersonProperties keys from TaxonomicFilter via excludedProperties.

```
excludedProperties={addedPropertyKeys}
```

### <a id="sv-ui-f-2931"></a>`SV-UI-F-2931`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** When a property is selected in TaxonomicFilter and group.type is EventProperties or PersonProperties, calls addResponseColumn with { type: group.type, key: String(value) } and closes the picker.

```
addResponseColumn({ type: group.type, key: String(value) })
```

### <a id="sv-ui-f-2932"></a>`SV-UI-F-2932`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** LemonSwitch labeled "Person ID" (data-attr "survey-response-column-person_id") that calls addResponseColumn or removeResponseColumn for { type: 'person_id' }.

```
label="Person ID"
```

### <a id="sv-ui-f-2933"></a>`SV-UI-F-2933`

- **Surface:** `UI` | **Type:** `branch` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Omits the removable column row when column.type is 'person_id', rendering rows only for property columns.

```
column.type === 'person_id' ? null : (
```

### <a id="sv-ui-f-2934"></a>`SV-UI-F-2934`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Button with tooltip "Remove column" (data-attr "survey-response-column-remove") that calls removeResponseColumn(column).

```
tooltip="Remove column"
```

### <a id="sv-ui-f-2935"></a>`SV-UI-F-2935`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Button labeled "Add column" (data-attr "survey-response-column-add") that switches the dropdown overlay to the TaxonomicFilter picker.

```
data-attr="survey-response-column-add"
```

### <a id="sv-ui-f-2936"></a>`SV-UI-F-2936`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/components/SurveyResponseColumnsMenu.tsx#SurveyResponseColumnsMenu (L17-103)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Secondary button labeled "Columns" (data-attr "survey-response-columns-menu") with tooltip "Choose which columns appear in the table and in exports" that toggles the columns dropdown.

```
tooltip="Choose which columns appear in the table and in exports"
```

### <a id="sv-ui-f-3113"></a>`SV-UI-F-3113`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads isMaxAvailable from maxGlobalLogic via Kea useValues

```
const { isMaxAvailable } = useValues(maxGlobalLogic)
```

### <a id="sv-ui-f-3114"></a>`SV-UI-F-3114`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Registers the 'create_survey' Max AI tool with initial prompt 'Create a survey to collect ' when isMaxAvailable is true

```
identifier: 'create_survey',
```

### <a id="sv-ui-f-3115"></a>`SV-UI-F-3115`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Invokes surveysLogic.handleMaxSurveyCreated with toolOutput and SURVEY_CREATED_SOURCE.SURVEY_EMPTY_STATE when the 'create_survey' Max tool callback fires

```
?.actions.handleMaxSurveyCreated(toolOutput, SURVEY_CREATED_SOURCE.SURVEY_EMPTY_STATE)
```

### <a id="sv-ui-f-3116"></a>`SV-UI-F-3116`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Gates both survey creation buttons behind AccessControlAction requiring AccessControlLevel.Editor on AccessControlResourceType.Survey

```
minAccessLevel={AccessControlLevel.Editor}
```

### <a id="sv-ui-f-3117"></a>`SV-UI-F-3117`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders primary button labeled 'Create your first survey' (data-attr='new-survey') that navigates to urls.surveyWizard()

```
to={urls.surveyWizard()}
```

### <a id="sv-ui-f-3118"></a>`SV-UI-F-3118`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Dispatches teamLogic.addProductIntent with ProductKey.SURVEYS and ProductIntentContext.SURVEY_ADD_NEW when 'Create your first survey' is clicked

```
teamLogic.findMounted()?.actions.addProductIntent({
```

### <a id="sv-ui-f-3120"></a>`SV-UI-F-3120`

- **Surface:** `UI` | **Type:** `control` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Renders secondary button labeled 'Create with AI' (data-attr='surveys-empty-state-create-with-ai') that calls openMax() on click

```
data-attr="surveys-empty-state-create-with-ai"
```

### <a id="sv-ui-f-3121"></a>`SV-UI-F-3121`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:products/surveys/frontend/emptyState/NewSurveyActions.tsx#NewSurveyActions (L22-75)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Disables the 'Create with AI' button with reason 'PostHog AI is unavailable here' when openMax is unavailable

```
disabledReason={openMax ? undefined : 'PostHog AI is unavailable here'}
```

### <a id="sv-ui-f-3126"></a>`SV-UI-F-3126`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/surveysEmptyState.tsx#surveysEmptyState (L15-37)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Wires the Surveys scene empty state to surveysSetupLogic to determine setup status.

```
statusLogic: surveysSetupLogic,
```

### <a id="sv-ui-f-3130"></a>`SV-UI-F-3130`

- **Surface:** `UI` | **Type:** `guard` | **Source:** `posthog:products/surveys/frontend/emptyState/surveysEmptyState.tsx#surveysEmptyState (L15-37)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Disables skipping the Surveys empty state (skippable is false).

```
skippable: false,
```

### <a id="sv-ui-f-3131"></a>`SV-UI-F-3131`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/surveysSetupLogic.ts#surveysSetupLogic (L13-29)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Registers setup detection logic for ProductKey.SURVEYS using createSetupDetectionLogic.

```
productKey: ProductKey.SURVEYS,
```

### <a id="sv-ui-f-3133"></a>`SV-UI-F-3133`

- **Surface:** `UI` | **Type:** `wiring` | **Source:** `posthog:products/surveys/frontend/emptyState/surveysSetupLogic.ts#surveysSetupLogic (L13-29)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Reads currentProjectId from the mounted Kea projectLogic.

```
const projectId = String(projectLogic.findMounted()?.values.currentProjectId)
```

### <a id="sv-ui-f-3134"></a>`SV-UI-F-3134`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:products/surveys/frontend/emptyState/surveysSetupLogic.ts#surveysSetupLogic (L13-29)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries at most 1 non-archived survey via surveysList to check whether any live survey exists.

```
surveysList(projectId, { limit: 1 }),
```

### <a id="sv-ui-f-3135"></a>`SV-UI-F-3135`

- **Surface:** `UI` | **Type:** `limit` | **Source:** `posthog:products/surveys/frontend/emptyState/surveysSetupLogic.ts#surveysSetupLogic (L13-29)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Queries at most 1 archived survey via surveysList to check whether any archived survey exists.

```
surveysList(projectId, { limit: 1, archived: true }),
```

### <a id="sv-ui-f-3138"></a>`SV-UI-F-3138`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#getSurveyConfigurationProperties (L33-59)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** recurring_survey_iteration_count defaults to 0 when survey.iteration_count is undefined.

```
recurring_survey_iteration_count: survey.iteration_count == undefined ? 0 : survey.iteration_count,
```

### <a id="sv-ui-f-3139"></a>`SV-UI-F-3139`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#getSurveyConfigurationProperties (L33-59)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** recurring_survey_iteration_interval defaults to 0 when survey.iteration_frequency_days is undefined.

```
survey.iteration_frequency_days == undefined ? 0 : survey.iteration_frequency_days,
```

### <a id="sv-ui-f-3142"></a>`SV-UI-F-3142`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#getSurveyCreatedProperties (L61-83)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** is_duplicate defaults to false when isDuplicate is omitted or nullish.

```
is_duplicate: isDuplicate ?? false,
```

### <a id="sv-ui-f-3143"></a>`SV-UI-F-3143`

- **Surface:** `UI` | **Type:** `default` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#getSurveyCreatedProperties (L61-83)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** creation_source defaults to 'full_editor' when creationSource is omitted or nullish.

```
creation_source: creationSource ?? 'full_editor',
```

### <a id="sv-ui-f-3145"></a>`SV-UI-F-3145`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#reportSurveyEdited (L85-93)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits PostHog event 'survey edited' with name, id, created_at, start_date, and getSurveyConfigurationProperties(survey).

```
posthog.capture('survey edited', {
```

### <a id="sv-ui-f-3146"></a>`SV-UI-F-3146`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#reportSurveyCreated (L95-102)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits PostHog event 'survey created' with properties from getSurveyCreatedProperties(survey, isDuplicate, creationSource, createdFrom).

```
posthog.capture('survey created', getSurveyCreatedProperties(survey, isDuplicate, creationSource, createdFrom))
```

### <a id="sv-ui-f-3147"></a>`SV-UI-F-3147`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#reportSurveyThemeSelected (L108-110)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits PostHog event 'survey theme selected' with survey_id (null when survey.id === 'new') and theme_id.

```
posthog.capture('survey theme selected', { survey_id: survey.id === 'new' ? null : survey.id, theme_id: themeId })
```

### <a id="sv-ui-f-3148"></a>`SV-UI-F-3148`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#reportSiteThemeCardViewed (L112-114)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits PostHog event 'survey site theme card viewed' with survey_id and card_state.

```
posthog.capture('survey site theme card viewed', { survey_id: surveyId, card_state: state })
```

### <a id="sv-ui-f-3149"></a>`SV-UI-F-3149`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#reportSiteThemeRequested (L116-118)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits PostHog event 'survey site theme requested' with survey_id.

```
posthog.capture('survey site theme requested', { survey_id: surveyId })
```

### <a id="sv-ui-f-3150"></a>`SV-UI-F-3150`

- **Surface:** `UI` | **Type:** `port` | **Source:** `posthog:products/surveys/frontend/surveyUsage.ts#reportSiteThemeLoaded (L120-122)` | **Pin:** `97bf79d0cc3bed5e7815`
- **Claim:** Emits PostHog event 'survey site theme loaded' with survey_id and outcome.

```
posthog.capture('survey site theme loaded', { survey_id: surveyId, outcome })
```

### <a id="sv-web-f-0001"></a>`SV-WEB-F-0001`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/survey-action-matcher.ts#ActionMatcher.init (L17-24)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** If _host.subscribeCapture is defined, init() subscribes this.on(eventName, eventPayload) and stores the unsubscribe callback.

```
if (!isUndefined(this._host?.subscribeCapture)) {
```

### <a id="sv-web-f-0004"></a>`SV-WEB-F-0004`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/survey-action-matcher.ts#ActionMatcher.register (L33-56)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** register() adds each action to _actionRegistry and indexes each step's event name (defaulting to '') in _actionEvents.

```
this._actionEvents.add(step?.event || '')
```

### <a id="sv-web-f-0005"></a>`SV-WEB-F-0005`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser-common/src/survey-action-matcher.ts#ActionMatcher.register (L33-56)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** If _host.setElementSelectors is present, register() collects all non-empty step.selector values and registers them on _host.

```
if (this._host?.setElementSelectors) {
```

### <a id="sv-web-f-0027"></a>`SV-WEB-F-0027`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver-base.ts#EventReceiver._register (L193-200)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips registering event-based and action-based items if `_host.subscribeCapture` is undefined.

```
if (isUndefined(this._host.subscribeCapture)) {
```

### <a id="sv-web-f-0031"></a>`SV-WEB-F-0031`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver-base.ts#EventReceiver._setupEventBasedItems (L253-284)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes `onEvent` to `_host.subscribeCapture` once (guarded by `??=`) so captured events trigger item matching.

```
this._captureHookUnsubscribe ??= this._host.subscribeCapture?.(matchEventToItem)
```

### <a id="sv-web-f-0035"></a>`SV-WEB-F-0035`

- **Surface:** `WEB` | **Type:** `fact` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver (L19-154)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Uses SurveyEventName.SHOWN ('survey shown') as the shown event name for survey activation tracking.

```
return SurveyEventName.SHOWN
```

### <a id="sv-web-f-0038"></a>`SV-WEB-F-0038`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver (L19-154)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Writes survey activation timestamps to the client key-value store under SURVEYS_ACTIVATED_TIMESTAMPS.

```
this._client.kv.set({ [SURVEYS_ACTIVATED_TIMESTAMPS]: timestamps })
```

### <a id="sv-web-f-0040"></a>`SV-WEB-F-0040`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver (L19-154)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Persists eligible activated survey IDs to the client key-value store under SURVEYS_ACTIVATED.

```
this._client.kv.set({ [SURVEYS_ACTIVATED]: eligibleItems })
```

### <a id="sv-web-f-0041"></a>`SV-WEB-F-0041`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver (L19-154)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Persists the activated session ID to the client key-value store under SURVEYS_ACTIVATED_SESSION.

```
this._client.kv.set({ [SURVEYS_ACTIVATED_SESSION]: sessionId })
```

### <a id="sv-web-f-0043"></a>`SV-WEB-F-0043`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver.constructor (L20-53)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to client capture events via _client.onEvent and forwards event name and payload to the receiver listener.

```
const subscription = _client.onEvent((event) => listener(event.event, event))
```

### <a id="sv-web-f-0044"></a>`SV-WEB-F-0044`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver.constructor (L20-53)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to client session changes via _client.onSession.

```
const subscription = _client.onSession(listener)
```

### <a id="sv-web-f-0045"></a>`SV-WEB-F-0045`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver.constructor (L20-53)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Falls back to the autocapture extension's setElementSelectors when actions.setElementSelectors is not provided.

```
actions?.setElementSelectors ??
```

### <a id="sv-web-f-0046"></a>`SV-WEB-F-0046`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver.constructor (L20-53)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes the session subscription upon construction via _subscribeSession.

```
this._subscribeSession()
```

### <a id="sv-web-f-0047"></a>`SV-WEB-F-0047`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver._activationOutcome (L123-143)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Ignores 'survey sent' events when $survey_completed is false so partial responses do not consume the activation.

```
if (event === SurveyEventName.SENT && eventPayload?.properties?.$survey_completed === false) {
```

### <a id="sv-web-f-0048"></a>`SV-WEB-F-0048`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver._activationOutcome (L123-143)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** When the survey is missing or canSurveyActivateRepeatedly is true, consumes 'survey shown' events and ignores all other events.

```
return event === SurveyEventName.SHOWN ? 'consume' : 'ignore'
```

### <a id="sv-web-f-0049"></a>`SV-WEB-F-0049`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver._activationOutcome (L123-143)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For non-repeating surveys, returns 'persist' when the 'survey shown' event is received.

```
if (event === SurveyEventName.SHOWN) {
```

### <a id="sv-web-f-0050"></a>`SV-WEB-F-0050`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser-common/src/survey-event-receiver.ts#SurveyEventReceiver._activationOutcome (L123-143)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For non-repeating surveys, consumes the activation on 'survey dismissed' or 'survey sent' events and ignores other events.

```
return event === SurveyEventName.DISMISSED || event === SurveyEventName.SENT ? 'consume' : 'ignore'
```

### <a id="sv-web-f-0051"></a>`SV-WEB-F-0051`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/browser-common/src/surveys-config.ts#SURVEYS_CACHE_TTL_MS (L10-10)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Surveys cache TTL is 5 minutes (5 * 60 * 1000 ms).

```
export const SURVEYS_CACHE_TTL_MS = 5 * 60 * 1000 // 5 minutes
```

### <a id="sv-web-f-0052"></a>`SV-WEB-F-0052`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/browser-common/src/surveys-config.ts#SURVEYS_REFRESH_BACKOFF_MS (L13-13)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Surveys refresh backoff duration is set to SURVEYS_CACHE_TTL_MS (5 minutes).

```
export const SURVEYS_REFRESH_BACKOFF_MS = SURVEYS_CACHE_TTL_MS
```

### <a id="sv-web-f-0065"></a>`SV-WEB-F-0065`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager (L141-1169)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns early from renderPopover without rendering if this._host.client?.canCapture is falsy.

```
if (!this._host.client?.canCapture) {
```

### <a id="sv-web-f-0071"></a>`SV-WEB-F-0071`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.constructor (L163-176)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Registers a window languagechange event listener to trigger _onLanguageChange.

```
addEventListener(window, 'languagechange', this._languageChangeListener)
```

### <a id="sv-web-f-0072"></a>`SV-WEB-F-0072`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.constructor (L163-176)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to FeatureFlagsCommonExtension.onFeatureFlags to trigger _onLanguageChange when feature flags update.

```
posthog.client?.getExtension(FeatureFlagsCommonExtension)?.onFeatureFlags(() => this._onLanguageChange()) ??
```

### <a id="sv-web-f-0073"></a>`SV-WEB-F-0073`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.startAutomaticDisplay (L279-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns early if manager is disposed, automatic display is already started, or config.automaticDisplay is false.

```
if (this._disposed || this._automaticDisplayDispose || this._host.config.automaticDisplay === false) return
```

### <a id="sv-web-f-0075"></a>`SV-WEB-F-0075`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.handlePopoverSurvey (L320-454)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns early without displaying a popover survey when the host client cannot capture events.

```
if (!this._host.client?.canCapture) {
```

### <a id="sv-web-f-0079"></a>`SV-WEB-F-0079`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.handlePopoverSurvey (L320-454)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Defaults surveyPopupDelaySeconds to 0 when unset on survey.appearance.

```
const delaySeconds = survey.appearance?.surveyPopupDelaySeconds || 0
```

### <a id="sv-web-f-0081"></a>`SV-WEB-F-0081`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.handlePopoverSurvey (L320-454)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Re-checks client.canCapture and _shouldDisplaySurvey(survey) after the display delay; if ineligible, logs info and removes the survey from focus without rendering.

```
if (!this._host.client?.canCapture || !this._shouldDisplaySurvey(survey)) {
```

### <a id="sv-web-f-0083"></a>`SV-WEB-F-0083`

- **Surface:** `WEB` | **Type:** `schedule` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.handlePopoverSurvey (L320-454)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Schedules a window.setTimeout for remainingMs that deletes survey.id from _surveyTimeouts and calls renderIfStillEligible, storing the timeout ID in _surveyTimeouts.

```
const timeoutId = window.setTimeout(() => {
```

### <a id="sv-web-f-0092"></a>`SV-WEB-F-0092`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager._processPrefillData (L747-799)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Persists prefilled survey submissionId, responses, lastQuestionIndex, questionOrder, visitedIndices, and surveyLanguage to localStorage via setInProgressSurveyState.

```
setInProgressSurveyState(survey, {
```

### <a id="sv-web-f-0094"></a>`SV-WEB-F-0094`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager._processPrefillData (L747-799)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Catches exceptions during prefill processing, logs an error, and returns null.

```
logger.error('[Survey Prefill] Error handling prefill:', error)
```

### <a id="sv-web-f-0096"></a>`SV-WEB-F-0096`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager._isSurveyFeatureFlagEnabled (L801-819)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Evaluates flagKey via FeatureFlagsCommonExtension.isFeatureEnabled, suppressing feature flag call events for internal survey targeting flags.

```
send_event: !flagKey.startsWith(SURVEY_TARGETING_FLAG_PREFIX),
```

### <a id="sv-web-f-0099"></a>`SV-WEB-F-0099`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager._warnIfFeatureFlagsDisabled (L821-832)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips logging if the warning was already emitted or host config featureFlagEvaluation is not false.

```
if (this._loggedFeatureFlagsDisabledWarning || this._host.config.featureFlagEvaluation !== false) {
```

### <a id="sv-web-f-0107"></a>`SV-WEB-F-0107`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.checkSurveyEligibility (L883-937)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Marks survey ineligible if getSurveySeen indicates the survey has already been seen and cannot be activated again.

```
if (getSurveySeen(survey)) {
```

### <a id="sv-web-f-0108"></a>`SV-WEB-F-0108`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.checkSurveyCaptureEligibility (L952-957)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns ineligible with SURVEY_CAPTURING_DISABLED when host client.canCapture is falsy.

```
if (!this._host.client?.canCapture) {
```

### <a id="sv-web-f-0109"></a>`SV-WEB-F-0109`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyManager.callSurveysAndEvaluateDisplayLogic (L1047-1116)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Filters active matching surveys to Popover and Widget types only when `this._host.client?.canCapture` is truthy.

```
canCaptureResponse && (survey.type === SurveyType.Popover || survey.type === SurveyType.Widget)
```

### <a id="sv-web-f-0125"></a>`SV-WEB-F-0125`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#generateSurveys (L1247-1268)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns `undefined` early when `document` or `window` is undefined.

```
if (typeof document === 'undefined' || typeof window === 'undefined') {
```

### <a id="sv-web-f-0126"></a>`SV-WEB-F-0126`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#generateSurveys (L1247-1268)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips automatic survey display and returns `surveyManager` when `posthog.config.automaticDisplay` is `false`.

```
if (posthog.config.automaticDisplay === false) {
```

### <a id="sv-web-f-0127"></a>`SV-WEB-F-0127`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#generateSurveys (L1247-1268)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips automatic survey display and returns `surveyManager` when `isSurveysEnabled` is `false`.

```
if (isSurveysEnabled === false) {
```

### <a id="sv-web-f-0130"></a>`SV-WEB-F-0130`

- **Surface:** `WEB` | **Type:** `schedule` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#startAutomaticDisplay (L1270-1306)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Polls `surveyManager.callSurveysAndEvaluateDisplayLogic(false)` every 1000ms while avoiding duplicate intervals.

```
intervalId = setInterval(() => {
```

### <a id="sv-web-f-0137"></a>`SV-WEB-F-0137`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#usePopupVisibility (L1389-1544)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes popup visibility to true when isPreviewMode is enabled, millisecondDelay is 0, or survey.type is SurveyType.ExternalSurvey.

```
isPreviewMode || millisecondDelay === 0 || survey.type === SurveyType.ExternalSurvey
```

### <a id="sv-web-f-0146"></a>`SV-WEB-F-0146`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#usePopupVisibility (L1389-1544)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** When survey.appearance.autoDisappear is enabled after submission, clears any prior timeout and hides the popup after 5000 ms.

```
autoDisappearTimeout = setTimeout(hidePopupWithAnimation, 5000)
```

### <a id="sv-web-f-0148"></a>`SV-WEB-F-0148`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#usePopupVisibility (L1389-1544)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Writes the current ISO timestamp to localStorage under 'lastSeenSurveyDate' when showing the survey, ignoring storage errors.

```
localStorage.setItem('lastSeenSurveyDate', new Date().toISOString())
```

### <a id="sv-web-f-0150"></a>`SV-WEB-F-0150`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#usePopupVisibility (L1389-1544)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Captures SurveyEventName.SHOWN via posthog.client?.capture with survey metadata, optional language, and sessionRecordingUrl unless skipShownEvent is true.

```
posthog.client?.capture(SurveyEventName.SHOWN, {
```

### <a id="sv-web-f-0157"></a>`SV-WEB-F-0157`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyPopup (L1588-1724)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Converts survey.appearance.surveyPopupDelaySeconds to milliseconds and defaults to 0 when unset.

```
const surveyPopupDelayMilliseconds = survey.appearance?.surveyPopupDelaySeconds
```

### <a id="sv-web-f-0160"></a>`SV-WEB-F-0160`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#SurveyPopup (L1588-1724)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Reuses the in-progress surveySubmissionId or falls back to a newly generated uuidv7() or empty string.

```
const surveySubmissionId = getInProgressSurvey?.surveySubmissionId || uuidv7() || ''
```

### <a id="sv-web-f-0180"></a>`SV-WEB-F-0180`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#Questions (L1726-1979)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes currentQuestionIndex to previewPageIndex if it is a non-negative number, otherwise to saved lastQuestionIndex if within [0, survey.questions.length), or 0.

```
return isNumber(previewPageIndex) && previewPageIndex >= 0 ? previewPageIndex : validSavedIndex
```

### <a id="sv-web-f-0181"></a>`SV-WEB-F-0181`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#Questions (L1726-1979)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes visitedIndices from initialInProgressState.visitedIndices filtered to indices within [0, survey.questions.length), defaulting to an empty array.

```
return (initialInProgressState?.visitedIndices ?? []).filter(
```

### <a id="sv-web-f-0182"></a>`SV-WEB-F-0182`

- **Surface:** `WEB` | **Type:** `default` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#Questions (L1726-1979)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes questionSnapshots from getInProgressSurveyState(survey)?.questionSnapshots, defaulting to an empty object.

```
return inProgressSurveyData?.questionSnapshots ?? {}
```

### <a id="sv-web-f-0185"></a>`SV-WEB-F-0185`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#Questions (L1726-1979)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs 'onNextButtonClick called without a PostHog instance.' and returns early without submitting when posthog is missing.

```
if (!posthog) {
```

### <a id="sv-web-f-0187"></a>`SV-WEB-F-0187`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys-renderer.tsx#Questions (L1726-1979)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs 'onNextButtonClick called without a questionId.' and returns early without recording an answer when questionId is missing.

```
if (!questionId) {
```

### <a id="sv-web-f-0206"></a>`SV-WEB-F-0206`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys (L63-792)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** _isSurveyCacheStale() reads SURVEYS_LOADED_AT from client.kv and considers cache stale when elapsed time exceeds SURVEYS_CACHE_TTL_MS

```
const surveysLoadedAt = this._client?.kv.get(SURVEYS_LOADED_AT)
```

### <a id="sv-web-f-0208"></a>`SV-WEB-F-0208`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.setup (L89-108)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes client.kv before completing survey setup

```
return continueWith(client.kv.initialize(), () => {
```

### <a id="sv-web-f-0210"></a>`SV-WEB-F-0210`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.setup (L89-108)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to remote config updates via client.onRemoteConfig(this.onRemoteConfig)

```
const subscription = client.onRemoteConfig(this.onRemoteConfig)
```

### <a id="sv-web-f-0214"></a>`SV-WEB-F-0214`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.onRemoteConfig (L155-180)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Ignores remote config updates when config.disableSurveys is true

```
if (this._config.disableSurveys) {
```

### <a id="sv-web-f-0215"></a>`SV-WEB-F-0215`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.onRemoteConfig (L155-180)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** When remote config result is not ok, sets _surveyInitializationFailed = true, notifies active matching survey callbacks with error 'Remote config unavailable. Not loading surveys.', and logs a warning

```
error: 'Remote config unavailable. Not loading surveys.',
```

### <a id="sv-web-f-0223"></a>`SV-WEB-F-0223`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips loading surveys when config.disableSurveys is true

```
if (config.disableSurveys) {
```

### <a id="sv-web-f-0224"></a>`SV-WEB-F-0224`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips loading surveys in cookielessMode when client is opted out

```
if (config.cookielessMode && this._client.isOptedOut) {
```

### <a id="sv-web-f-0225"></a>`SV-WEB-F-0225`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Calls _handleSurveyLoadError('PostHog Extensions not found.') if getExtensions() returns nullish

```
this._handleSurveyLoadError('PostHog Extensions not found.')
```

### <a id="sv-web-f-0226"></a>`SV-WEB-F-0226`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns early without loading if _isSurveysEnabled is undefined and config.advancedEnableSurveys is falsy

```
if (isUndefined(this._isSurveysEnabled) && !config.advancedEnableSurveys) {
```

### <a id="sv-web-f-0228"></a>`SV-WEB-F-0228`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Calls _handleSurveyLoadError(LOAD_EXT_NOT_FOUND) if loadExternalDependency is unavailable

```
this._handleSurveyLoadError(LOAD_EXT_NOT_FOUND)
```

### <a id="sv-web-f-0229"></a>`SV-WEB-F-0229`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Loads external survey script via loadExternalDependency, completing initialization on success or calling _handleSurveyLoadError('Could not load surveys script', err) on failure

```
loadExternalDependency((err) => {
```

### <a id="sv-web-f-0230"></a>`SV-WEB-F-0230`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.loadIfEnabled (L206-285)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** On thrown exception during initialization, resets _isInitializingSurveys = false, calls _handleSurveyLoadError('Error initializing surveys', e), and rethrows

```
this._handleSurveyLoadError('Error initializing surveys', e)
```

### <a id="sv-web-f-0234"></a>`SV-WEB-F-0234`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys._completeSurveyInitialization (L288-307)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Reads cached surveys from client.kv under SURVEYS and registers event- or action-based surveys if present

```
const cachedSurveys = this._client?.kv.get<Survey[]>(SURVEYS)
```

### <a id="sv-web-f-0235"></a>`SV-WEB-F-0235`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys._handleSurveyLoadError (L314-319)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Sets _surveyInitializationFailed = true, logs the error via logger.error, and notifies active matching survey callbacks and survey callbacks with { isLoaded: false, error: message }

```
this._surveyInitializationFailed = true
```

### <a id="sv-web-f-0238"></a>`SV-WEB-F-0238`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.getSurveys (L355-425)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** If disableSurveys is enabled in config, logs SURVEY_DISABLED and invokes callback with an empty array.

```
if (this._config.disableSurveys) {
```

### <a id="sv-web-f-0241"></a>`SV-WEB-F-0241`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.getSurveys (L355-425)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Fetches surveys via GET /api/surveys/ with project token, sentAt query param, and configured requestTimeoutMs.

```
.sendRequest('/api/surveys/', {
```

### <a id="sv-web-f-0242"></a>`SV-WEB-F-0242`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.getSurveys (L355-425)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs 'Error processing surveys response' if _handleSurveyResponse throws and falls back to handling a statusCode 0 error response.

```
logger.error('Error processing surveys response', error)
```

### <a id="sv-web-f-0244"></a>`SV-WEB-F-0244`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.getSurveys (L355-425)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs 'Error in survey callback' if invoking a survey callback rejects.

```
.catch((error) => logger.error('Error in survey callback', error))
```

### <a id="sv-web-f-0247"></a>`SV-WEB-F-0247`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys._handleSurveyResponse (L427-452)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** On non-200 status or missing JSON, logs an error (if statusCode !== 0) or warning (if no response.error), sets _lastSurveyRefreshFailedAt, and returns { isLoaded: false, error }.

```
if (statusCode !== 200 || !response.json) {
```

### <a id="sv-web-f-0249"></a>`SV-WEB-F-0249`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys._isSurveyRefreshBackingOff (L485-490)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns true while elapsed time since _lastSurveyRefreshFailedAt is less than SURVEYS_REFRESH_BACKOFF_MS.

```
Date.now() - this._lastSurveyRefreshFailedAt < SURVEYS_REFRESH_BACKOFF_MS
```

### <a id="sv-web-f-0252"></a>`SV-WEB-F-0252`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys._notifySurveyCallbacks (L518-531)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Catches synchronous callback errors and logs 'Error in survey callback'.

```
logger.error('Error in survey callback', error)
```

### <a id="sv-web-f-0254"></a>`SV-WEB-F-0254`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys.onActiveMatchingSurveysChanged (L541-569)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Marks surveys unavailable and notifies the subscription with { isLoaded: false } if disableSurveys is set, survey initialization failed, or no cached surveys exist after a refresh failure with no in-flight request; otherwise notifies if _surveyManager is ready.

```
if (unavailable) {
```

### <a id="sv-web-f-0260"></a>`SV-WEB-F-0260`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys.ts#PostHogSurveys._notifyActiveMatchingSurveyCallback (L579-617)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Catches errors during callback delivery or getActiveMatchingSurveys and logs 'Error in active matching surveys callback'.

```
logger.error('Error in active matching surveys callback', error)
```

### <a id="sv-web-f-0353"></a>`SV-WEB-F-0353`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getRatingBucketForResponseValue (L12-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For scale 2, throws Error('The response must be in range 1-2') if responseValue is < 1 or > 2.

```
throw new Error('The response must be in range 1-2')
```

### <a id="sv-web-f-0355"></a>`SV-WEB-F-0355`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getRatingBucketForResponseValue (L12-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For scale 3, throws Error('The response must be in range 1-3') if responseValue is < 1 or > 3.

```
throw new Error('The response must be in range 1-3')
```

### <a id="sv-web-f-0357"></a>`SV-WEB-F-0357`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getRatingBucketForResponseValue (L12-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For scale 5, throws Error('The response must be in range 1-5') if responseValue is < 1 or > 5.

```
throw new Error('The response must be in range 1-5')
```

### <a id="sv-web-f-0359"></a>`SV-WEB-F-0359`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getRatingBucketForResponseValue (L12-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For scale 7, throws Error('The response must be in range 1-7') if responseValue is < 1 or > 7.

```
throw new Error('The response must be in range 1-7')
```

### <a id="sv-web-f-0361"></a>`SV-WEB-F-0361`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getRatingBucketForResponseValue (L12-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For scale 10, throws Error('The response must be in range 0-10') if responseValue is < 0 or > 10.

```
throw new Error('The response must be in range 0-10')
```

### <a id="sv-web-f-0363"></a>`SV-WEB-F-0363`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getRatingBucketForResponseValue (L12-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Throws Error('The scale must be one of: 2, 3, 5, 7, 10') for any unsupported scale.

```
throw new Error('The scale must be one of: 2, 3, 5, 7, 10')
```

### <a id="sv-web-f-0373"></a>`SV-WEB-F-0373`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-branching.ts#getNextSurveyStep (L51-134)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For Rating questions, throws Error('The response type must be an integer') if response is not an integer number.

```
throw new Error('The response type must be an integer')
```

### <a id="sv-web-f-0378"></a>`SV-WEB-F-0378`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-html-sanitizer.ts#sanitizeSurveyHtml (L122-142)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Disallows all ARIA attributes and data-* attributes by default when sanitizing survey HTML.

```
ALLOW_ARIA_ATTR: false,
```

### <a id="sv-web-f-0379"></a>`SV-WEB-F-0379`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-html-sanitizer.ts#sanitizeSurveyHtml (L122-142)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Restricts sanitized survey HTML elements to the XHTML namespace (http://www.w3.org/1999/xhtml).

```
ALLOWED_NAMESPACES: ['http://www.w3.org/1999/xhtml'],
```

### <a id="sv-web-f-0380"></a>`SV-WEB-F-0380`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-html-sanitizer.ts#sanitizeSurveyHtml (L122-142)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Strips all attributes except those in globalAttributes or in the element's entry in allowedAttributes.

```
ADD_ATTR: (attributeName, tagName) =>
```

### <a id="sv-web-f-0385"></a>`SV-WEB-F-0385`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-url-prefill.ts#extractPrefillParamsFromUrl (L20-69)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Skips query pairs that throw during decodeURIComponent

```
decodedKey = decodeURIComponent(pair.slice(0, separator))
```

### <a id="sv-web-f-0401"></a>`SV-WEB-F-0401`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-url-prefill.ts#convertPrefillToResponses (L74-149)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Catches conversion errors per question and logs via logger.error without aborting remaining questions

```
logger.error(`[Survey Prefill] Error converting q${index}:`, error)
```

### <a id="sv-web-f-0402"></a>`SV-WEB-F-0402`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/browser-common/src/surveys/survey-url-prefill.ts#calculatePrefillStartIndex (L162-212)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Caps question traversal at survey.questions.length + 1 iterations to prevent infinite branching loops

```
const MAX_ITERATIONS = survey.questions.length + 1
```

### <a id="sv-web-f-0435"></a>`SV-WEB-F-0435`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#sendSurveyEvent (L444-494)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs an error and returns early without capturing 'survey sent' when the PostHog instance is missing.

```
if (!posthog) {
```

### <a id="sv-web-f-0436"></a>`SV-WEB-F-0436`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#sendSurveyEvent (L444-494)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns early as a silent no-op if posthog.client?.canCapture is falsy.

```
if (!posthog.client?.canCapture) {
```

### <a id="sv-web-f-0438"></a>`SV-WEB-F-0438`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#sendSurveyEvent (L444-494)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Captures the 'survey sent' event with survey metadata, response properties, replay URL, and sets the $survey_responded/{} person property to true.

```
posthog.client?.capture(SurveyEventName.SENT, {
```

### <a id="sv-web-f-0439"></a>`SV-WEB-F-0439`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#sendSurveyEvent (L444-494)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Dispatches a 'PHSurveySent' CustomEvent on window with surveyId when isSurveyCompleted is true.

```
window.dispatchEvent(new CustomEvent('PHSurveySent', { detail: { surveyId: survey.id } }))
```

### <a id="sv-web-f-0451"></a>`SV-WEB-F-0451`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#getSurveySeen (L675-689)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Reads the survey seen flag from localStorage under the survey seen key.

```
surveySeen = localStorage.getItem(getSurveySeenKey(survey))
```

### <a id="sv-web-f-0452"></a>`SV-WEB-F-0452`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#getSurveySeen (L675-689)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs a localStorage error and falls back to treating the survey as unseen.

```
logger.error('localStorage error: ' + error)
```

### <a id="sv-web-f-0454"></a>`SV-WEB-F-0454`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#hasWaitPeriodPassed (L693-701)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Reads lastSeenSurveyDate from localStorage and checks whether waitPeriodInDays has elapsed.

```
lastSeenSurveyDate = localStorage.getItem(LAST_SEEN_SURVEY_DATE_KEY)
```

### <a id="sv-web-f-0455"></a>`SV-WEB-F-0455`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#hasWaitPeriodPassed (L693-701)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs a localStorage error and evaluates the wait period with a null lastSeenSurveyDate.

```
logger.error('localStorage error: ' + error)
```

### <a id="sv-web-f-0462"></a>`SV-WEB-F-0462`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#setInProgressSurveyState (L817-830)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Persists serialized InProgressSurveyState to localStorage under the survey's in-progress key.

```
localStorage.setItem(key, JSON.stringify(state))
```

### <a id="sv-web-f-0464"></a>`SV-WEB-F-0464`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#setInProgressSurveyState (L817-830)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs an error and falls back to storing the state in inMemoryInProgressSurveyState if localStorage.setItem throws.

```
logger.error('Error setting in-progress survey state in localStorage', e)
```

### <a id="sv-web-f-0466"></a>`SV-WEB-F-0466`

- **Surface:** `WEB` | **Type:** `storage` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#getInProgressSurveyState (L832-851)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Reads serialized in-progress survey state from localStorage under the survey's in-progress key.

```
const stateString = localStorage.getItem(key)
```

### <a id="sv-web-f-0468"></a>`SV-WEB-F-0468`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-common/src/surveys/surveys-extension-utils.tsx#getInProgressSurveyState (L832-851)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Logs an error and returns null if reading or parsing in-progress survey state from localStorage throws.

```
logger.error('Error getting in-progress survey state from localStorage', e)
```

### <a id="sv-web-f-0491"></a>`SV-WEB-F-0491`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Catches exceptions thrown by user survey callbacks and logs 'Survey callback failed'.

```
client?.logger.error('Survey callback failed', error)
```

### <a id="sv-web-f-0495"></a>`SV-WEB-F-0495`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Resolves the analytics extension during setup before initializing survey storage.

```
await value.resolveExtension('analytics')
```

### <a id="sv-web-f-0496"></a>`SV-WEB-F-0496`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to client reset events to invoke shared.reset().

```
resetSubscription = value.onReset(reset)
```

### <a id="sv-web-f-0497"></a>`SV-WEB-F-0497`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to client remote config updates to control remote survey enablement.

```
remoteSubscription = value.onRemoteConfig((result) => {
```

### <a id="sv-web-f-0501"></a>`SV-WEB-F-0501`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Registers a before-teardown listener on the analytics extension to flush page unload via shared.handlePageUnload().

```
teardownSubscription = client.analytics?.onBeforeTeardown?.(() => shared.handlePageUnload())
```

### <a id="sv-web-f-0502"></a>`SV-WEB-F-0502`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Registers a window pagehide event listener during setup when window is defined.

```
if (typeof window !== 'undefined') window.addEventListener('pagehide', pagehide)
```

### <a id="sv-web-f-0522"></a>`SV-WEB-F-0522`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Resolves the analytics extension during setup before initializing survey storage and aborts setup early if disposed while awaiting it.

```
await value.resolveExtension('analytics')
```

### <a id="sv-web-f-0523"></a>`SV-WEB-F-0523`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser-next/src/products/surveys/surveys-extension.ts#createSurveys (L22-264)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Initializes the client key-value store during survey extension setup and aborts setup early if disposed while awaiting it.

```
await value.kv.initialize()
```

### <a id="sv-web-f-0529"></a>`SV-WEB-F-0529`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.get (L14-29)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Surveys are disabled when config.disable_surveys is truthy.

```
disableSurveys: !!config?.disable_surveys,
```

### <a id="sv-web-f-0530"></a>`SV-WEB-F-0530`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.get (L14-29)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Cookieless mode for surveys is enabled when config.cookieless_mode is truthy.

```
cookielessMode: !!config?.cookieless_mode,
```

### <a id="sv-web-f-0531"></a>`SV-WEB-F-0531`

- **Surface:** `WEB` | **Type:** `guard` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.get (L14-29)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Advanced survey enablement is controlled by config.advanced_enable_surveys.

```
advancedEnableSurveys: !!config?.advanced_enable_surveys,
```

### <a id="sv-web-f-0540"></a>`SV-WEB-F-0540`

- **Surface:** `WEB` | **Type:** `port` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.getExtensions (L31-45)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Loads the external 'surveys' dependency via getExtensionLoader when available.

```
? (callback) => loadExternalDependency(instance, 'surveys', callback)
```

### <a id="sv-web-f-0541"></a>`SV-WEB-F-0541`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to PostHog capture events via _addCaptureHook to detect survey matching condition changes.

```
const unsubscribeCapture = this._instance?._addCaptureHook((event) => {
```

### <a id="sv-web-f-0542"></a>`SV-WEB-F-0542`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Invokes the matching-conditions callback when a $pageview event is captured.

```
event === '$pageview' ||
```

### <a id="sv-web-f-0543"></a>`SV-WEB-F-0543`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Invokes the matching-conditions callback when a 'survey shown' event is captured.

```
event === SurveyEventName.SHOWN ||
```

### <a id="sv-web-f-0544"></a>`SV-WEB-F-0544`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Invokes the matching-conditions callback when a 'survey dismissed' event is captured.

```
event === SurveyEventName.DISMISSED ||
```

### <a id="sv-web-f-0545"></a>`SV-WEB-F-0545`

- **Surface:** `WEB` | **Type:** `branch` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Invokes the matching-conditions callback when a 'survey sent' event is captured.

```
event === SurveyEventName.SENT
```

### <a id="sv-web-f-0546"></a>`SV-WEB-F-0546`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Registers a window listener for the 'PHSurveyShown' DOM event to trigger the matching-conditions callback.

```
addEventListener(window, 'PHSurveyShown', onShown)
```

### <a id="sv-web-f-0547"></a>`SV-WEB-F-0547`

- **Surface:** `WEB` | **Type:** `wiring` | **Source:** `posthog-js:packages/browser/src/browser-surveys.ts#BrowserSurveysConfigSource.onMatchingConditionsChanged (L46-78)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Subscribes to onFeatureFlags and invokes the callback only after synchronous registration completes (listening = true).

```
const unsubscribeFlags = this._instance?.onFeatureFlags(() => {
```

### <a id="sv-web-f-0583"></a>`SV-WEB-F-0583`

- **Surface:** `WEB` | **Type:** `error` | **Source:** `posthog-js:packages/core/src/surveys/validation.ts#getValidationError (L7-44)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** Returns 'This field is required' when optional is falsy and the trimmed value is empty.

```
if (!optional && trimmed === '') {
```

### <a id="sv-web-f-0585"></a>`SV-WEB-F-0585`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/core/src/surveys/validation.ts#getValidationError (L7-44)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For a MinLength rule with a defined value, returns rule.errorMessage or 'Please enter at least <value> characters' if trimmed.length < rule.value.

```
if (rule.value !== undefined && trimmed.length < rule.value) {
```

### <a id="sv-web-f-0586"></a>`SV-WEB-F-0586`

- **Surface:** `WEB` | **Type:** `limit` | **Source:** `posthog-js:packages/core/src/surveys/validation.ts#getValidationError (L7-44)` | **Pin:** `6ef60e8d3caf5ac58d0e`
- **Claim:** For a MaxLength rule with a defined value, returns rule.errorMessage or 'Please enter no more than <value> characters' if trimmed.length > rule.value.

```
if (rule.value !== undefined && trimmed.length > rule.value) {
```
