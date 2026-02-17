# AI Audit: amplify-android Fork

**Date:** 2026-02-17  
**Auditor:** Android Developer agent (opus)  
**Repo:** bfansports/amplify-android  
**Upstream:** aws-amplify/amplify-android  

---

## Executive Summary

Unlike the `amplify-swift` fork (which was found to be **unused**), the `amplify-android` fork is **actively used in production** -- but only for **2 out of 189 app flavors** (`fcnantes` and `olympiquedemarseille`). The remaining 187 flavors use the official upstream Amplify SDK from Maven Central.

The fork contains **4 legitimate customizations**, all concentrated in the `aws-auth-cognito` module. These modifications replace Chrome Custom Tabs with an in-app WebView for Hosted UI authentication -- a change that was needed to support SSO cookie injection for Olympique de Marseille's custom auth flow.

The fork is based on upstream **v2.29.2** (August 2025), while the Android app's non-fork flavors use upstream **v2.33.0**. The fork has **not been synced with upstream since August 2025** (~6 months of drift), missing security patches, bug fixes, and new features.

---

## Critical

### C1: Fork is 4+ versions behind upstream (v2.29.2 vs v2.33.0)

**Impact:** Security vulnerabilities, missed bug fixes, Kotlin/AGP compatibility gaps.  
**Details:** The fork's `VERSION_NAME` is `2.29.2`. The Android app's `libs.versions.toml` pins `amplifyVersion = "2.33.0"` for non-fork flavors. Upstream has had at least 4 releases since the fork point, including auth bug fixes (`SIGNED_IN` event after autoSignIn), storage improvements, and SDK dependency updates. The upstream is also adding Kotlin Multiplatform support.
**Files:** `gradle.properties` (VERSION_NAME=2.29.2), upstream CHANGELOG.md  
**Recommendation:** Sync fork with upstream `main`. The bFAN changes are isolated to `aws-auth-cognito/src/main/java/.../activities/WebViewActivity.kt` and a few surrounding files -- merge conflicts should be manageable.

### C2: Hardcoded OM-specific cookies in SDK library code

**Impact:** Security concern, maintenance burden, wrong abstraction layer.  
**Details:** The fork hardcodes `connect.om.fr` and `connect.athena.om.fr` cookie domains directly in `WebViewActivity.kt`. This means every flavor that uses the fork (currently just 2) gets OM-specific cookies injected. If other flavors are added to the fork in the future, they will also receive these cookies. Cookie injection should be configurable, not hardcoded.
**Files:** `aws-auth-cognito/src/main/java/.../activities/WebViewActivity.kt`  
**Recommendation:** Extract cookie configuration to a parameter passed via `Intent` extras or an Amplify plugin configuration, rather than hardcoding in the library.

---

## High

### H1: Fork serves only 2 of 189 flavors -- disproportionate maintenance cost

**Impact:** High maintenance overhead for narrow use case.  
**Details:** The `build.gradle.kts` in the Android app conditionally selects the fork only for `fcnantes` and `olympiquedemarseille` flavors. All other 187 flavors use the upstream SDK from Maven Central. Maintaining a 2,372-file fork of a fast-moving AWS SDK for 2 flavors is an unfavorable cost-benefit ratio.
**Files:** `SA-User-WhiteLabelApps-Android/app/build.gradle.kts` (line 642)  
**Recommendation:** Investigate whether the WebView auth changes can be contributed upstream (AWS has accepted similar PRs), or implemented as a wrapper/interceptor layer without forking the entire SDK. If a fork remains necessary, consider a minimal fork of only the `aws-auth-cognito` module.

### H2: No upstream remote configured -- no sync workflow

**Impact:** Fork drift will continue to grow.  
**Details:** The fork's `origin` remote points to `bfansports/amplify-android`. There is no `upstream` remote configured for `aws-amplify/amplify-android`. The last upstream merge was a manual `git merge` by Chase Lawrence on 2025-08-25. There is no documented sync process, no CI to detect drift, and no schedule.
**Recommendation:** Add upstream remote. Document sync process in CLAUDE.md. Consider a quarterly sync schedule at minimum.

### H3: JitPack SNAPSHOT distribution -- no version pinning

**Impact:** Non-reproducible builds, potential regression introduction.  
**Details:** The fork is consumed via JitPack as `main-SNAPSHOT` (`com.github.bfansports.amplify-android`). Every build of the affected flavors fetches the latest `main` branch state. If someone pushes a broken commit to `main`, all subsequent builds of `fcnantes` and `olympiquedemarseille` will break with no rollback path.
**Files:** `SA-User-WhiteLabelApps-Android/gradle/libs.versions.toml` (line 57)  
**Recommendation:** Use tagged releases on JitPack instead of `main-SNAPSHOT`. Pin to specific commit SHAs or release tags.

---

## Medium

### M1: No tests for bFAN-specific WebView changes

**Impact:** Regressions go undetected.  
**Details:** The 4 PRs that introduced bFAN customizations (#1, #4, #5, #7, #8) all have empty test checklists. The `WebViewActivity`, cookie injection, and session continuity changes have no unit or instrumentation tests.
**Recommendation:** Add at minimum: (1) unit test for cookie injection logic, (2) instrumentation test for WebView auth flow, (3) test for redirect URI handling.

### M2: Single maintainer risk

**Impact:** Bus factor of 1.  
**Details:** All 4 bFAN-specific PRs were authored and self-merged by a single developer (Chase Lawrence / @Kirvlawc). No code reviews were performed on any PR.
**Recommendation:** Require at least 1 reviewer for fork changes. Document the customization rationale so others can maintain it.

### M3: 2 stale dependabot PRs unmerged

**Impact:** Known vulnerabilities remain unpatched.  
**Details:** PRs #9 (`glob` bump) and #10 (`lodash-es` bump) have been open since Nov 2025 and Jan 2026 respectively. These are security dependency updates for the AppSync test backend.
**Recommendation:** Merge or close these PRs. Consider disabling dependabot if the fork is not actively maintained at the dependency level.

### M4: PR #7 cookie fix changed header name from `X-Requested-From` to `X-Requested-With`

**Impact:** Potential SSO regression for earlier testing.  
**Details:** PR #5 set `X-Requested-From=WebView` cookies. PR #7 ("Fix cookie typo") changed this to `X-Requested-With=WebView`, and PR #8 kept `X-Requested-With`. This suggests the original header name was wrong, but there is no documentation about which header the OM SSO server expects or why.
**Recommendation:** Document which header name the OM SSO backend requires and why.

---

## Low

### L1: `BFanSSO-Android` does NOT reference the fork

**Details:** Grep of `BFanSSO-Android/` Gradle files found no references to `amplify` or `bfansports`. The SSO module either uses the upstream Amplify SDK or does not use Amplify at all. The CLAUDE.md claim that BFanSSO-Android consumes the fork is inaccurate.
**Recommendation:** Correct CLAUDE.md to remove BFanSSO-Android from the consumers list.

### L2: Fork CHANGELOG.md is from upstream -- does not document bFAN changes

**Details:** The CHANGELOG.md still references `aws-amplify/amplify-android` issues and PRs. bFAN's 4 customization PRs are not documented anywhere.
**Recommendation:** Add a `BFAN-CHANGELOG.md` or a section at the top of CHANGELOG.md documenting fork-specific changes.

### L3: No `.gitignore` or build caching optimizations for fork development

**Details:** Minor -- the fork inherits upstream's `.gitignore` which is adequate.

---

## Agent Skill Improvements

### S1: Android Developer SKILL.md should list amplify-android fork details

The agent should know that the fork exists, serves only 2 flavors, and where the customizations live (`aws-auth-cognito` module). This prevents incorrect assumptions about "all flavors use the fork."

### S2: Hub CLAUDE.md should cross-reference fork status

The hub's documentation about `amplify-android` should note the fork's narrow scope (2 flavors only) and link to this FINDINGS.md for full context. This parallels the amplify-swift finding.

---

## Positive Observations

### P1: Fork customizations are well-scoped

All bFAN changes are confined to a single module (`aws-auth-cognito`) and a single feature area (Hosted UI WebView auth). The changes are small, focused, and don't touch core SDK logic. This makes upstream syncing and eventual deprecation much easier.

### P2: The fork justification is legitimate

Replacing Chrome Custom Tabs with WebView for auth is a real requirement for OM's SSO flow (cookie injection for `connect.om.fr`). Chrome Custom Tabs do not support cookie injection, so the fork is not gratuitous -- it solves a problem that cannot be solved with the upstream SDK alone.

### P3: PR #8 significantly improved WebView implementation quality

The session continuity fix (PR #8) rewrote `WebViewActivity` with better cookie management, proper lifecycle handling (`onPause`/`onResume`/`onTrimMemory` cookie flushing), third-party cookie support, and configurable redirect schemes. This is production-quality code.

### P4: Build variant architecture isolates fork impact

The Android app's Gradle setup cleanly isolates fork usage to specific flavors. Adding or removing flavors from the fork is a single-line change in `build.gradle.kts`. This is good defensive architecture.
