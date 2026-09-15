# In-App Announcement System

This guide explains how to use the in-app announcement system to notify users about important updates, issues, or information.

## Overview

The announcement system supports:
- **Multiple announcements** - Users see one at a time, highest priority first
- **Version targeting** - Only reach the app versions an announcement applies to
- **Start dates** - Publish ahead of time and let it switch itself on
- **Expiry dates** - Announcements can auto-expire after a set date
- **Priority** - Decide which announcement is shown first
- **Offline cache** - The last payload is reused when the fetch fails
- **Localization** - English and Arabic support
- **Action buttons** - Clear cache, open links, or dismiss

> Version targeting, start dates, priority and the offline cache were added in
> app version 3.0.0. Older installs ignore those fields, so an announcement
> using them still shows on 2.x — set `min_version` only when the message would
> be wrong for an older version rather than merely irrelevant.

## Step 1: Create the Announcement File

Create a file at `config/announcement.json` in your `Jordan_Prayer_Times_API_Data` repository.

## Step 2: JSON Structure

### Multiple Announcements Format (Recommended)

```json
{
  "announcements": [
    {
      "id": "cities_update_2026_01",
      "title_en": "Cities List Updated",
      "title_ar": "تم تحديث قائمة المدن",
      "message_en": "Please clear cache and reselect your city.",
      "message_ar": "يرجى مسح الذاكرة المؤقتة وإعادة اختيار مدينتك.",
      "type": "action",
      "is_active": true,
      "expires": 0,
      "action_text_en": "Clear Cache",
      "action_text_ar": "مسح الذاكرة المؤقتة",
      "action_type": "clear_cache"
    },
    {
      "id": "ramadan_2026",
      "title_en": "Ramadan Mubarak!",
      "title_ar": "رمضان مبارك!",
      "message_en": "Wishing you a blessed Ramadan.",
      "message_ar": "نتمنى لكم رمضان مبارك.",
      "type": "info",
      "is_active": true,
      "expires": 1,
      "expiry_date": "2026-04-01",
      "start_date": "2026-03-18",
      "priority": 10
    },
    {
      "id": "whats_new_3_0",
      "title_en": "What's new in 3.0",
      "title_ar": "ما الجديد في 3.0",
      "message_en": "Custom alarm sounds, widget colours, and reminders that open the right adhkar.",
      "message_ar": "أصوات تنبيه مخصصة، وألوان للأدوات، وتذكيرات تفتح الأذكار المقصودة.",
      "type": "info",
      "is_active": true,
      "expires": 1,
      "expiry_date": "2026-11-01",
      "min_version": "3.0.0"
    }
  ]
}
```

## Field Descriptions

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (e.g., `"cities_update_2026_01"`). Users who dismiss won't see the same ID again |
| `title_en` | Yes | English title shown in dialog |
| `title_ar` | Yes | Arabic title shown in dialog |
| `message_en` | Yes | English message body |
| `message_ar` | Yes | Arabic message body |
| `type` | Yes | `"info"` (blue), `"warning"` (amber), `"error"` (red), `"action"` (orange) |
| `is_active` | Yes | `true` to show, `false` to hide |
| `expires` | No | `0` = never expires (default), `1` = has expiry date |
| `expiry_date` | No | Date after which announcement is hidden. Format: `"yyyy-MM-dd"` (e.g., `"2026-02-01"`). Only used if `expires` is `1` |
| `start_date` | No | Date before which the announcement stays hidden. Format: `"yyyy-MM-dd"`. Lets you publish ahead of a release |
| `min_version` | No | Lowest app version that should see it, e.g. `"3.0.0"`. Compared number by number, so `2.10.0` is correctly newer than `2.9.0` |
| `max_version` | No | Highest app version that should see it. Useful for "please update" notices aimed only at old installs |
| `priority` | No | Higher shows first when several apply. Defaults to `0` |
| `action_text_en` | No | English button text (optional) |
| `action_text_ar` | No | Arabic button text (optional) |
| `action_type` | No | `"clear_cache"`, `"link"`, `"dismiss"` |
| `action_link` | No | URL if `action_type` is `"link"` |

## Version Targeting

`min_version` and `max_version` keep a message away from the versions it does
not describe. Both are inclusive, and either can be used on its own.

| Fields | Who sees it |
|--------|-------------|
| Neither | Every version |
| `min_version: "3.0.0"` | 3.0.0 and newer |
| `max_version: "2.9.9"` | Everything up to and including 2.9.9 |
| Both | Only versions inside the window |

Two things worth knowing:

- Versions are compared **numerically per segment**, so `2.10.0` is newer than
  `2.9.0`. Plain text comparison gets that backwards.
- Segment counts need not match: `min_version: "3.0"` accepts `3.0.0`.

**Typical use — "please update":** `max_version` set to the last broken build,
so the people already on the fix are never nagged.

## Scheduling with `start_date`

An announcement with a future `start_date` is fetched but not shown until that
date arrives. Publish the release note the day before the rollout and leave it
alone, rather than editing the file at the moment of release.

`start_date` and `expiry_date` combine into a window: shown from the start date,
hidden again after the expiry date (when `expires` is `1`).

## Priority

When more than one announcement applies, the highest `priority` shows first,
then the rest on subsequent launches as each is dismissed. Without it, ordering
falls back to the order of the array, which makes the file's layout matter more
than it should. A Ramadan greeting at `priority: 10` will come before a routine
notice at the default `0`.

## Expiry System

The expiry system helps manage announcement visibility over time:

| `expires` | `expiry_date` | Behavior |
|-----------|---------------|----------|
| `0` or not set | Not needed | Announcement shows until manually disabled or dismissed |
| `1` | `"2026-02-01"` | Announcement automatically hides after Feb 1, 2026 |

**Benefits:**
- New users won't see old, irrelevant announcements
- Important announcements (like city changes) can stay active with `expires: 0`
- Temporary announcements (like Ramadan greetings) auto-expire

## Action Types

- **`clear_cache`** - Clears all cached data and forces city reselection
- **`link`** - Opens a URL (requires `action_link` field)
- **`dismiss`** - Just closes the dialog

## Examples

### Example 1: Cities Changed (No Expiry - Important)

```json
{
  "id": "cities_update_2026_01",
  "title_en": "Cities List Updated",
  "title_ar": "تم تحديث قائمة المدن",
  "message_en": "We've updated the cities list. Please clear your cache and reselect your city for accurate prayer times.",
  "message_ar": "تم تحديث قائمة المدن. يرجى مسح الذاكرة المؤقتة وإعادة اختيار مدينتك للحصول على أوقات صلاة دقيقة.",
  "type": "action",
  "is_active": true,
  "expires": 0,
  "action_text_en": "Clear Cache",
  "action_text_ar": "مسح الذاكرة المؤقتة",
  "action_type": "clear_cache"
}
```

### Example 2: Seasonal Greeting (With Expiry)

```json
{
  "id": "ramadan_2026",
  "title_en": "Ramadan Mubarak!",
  "title_ar": "رمضان مبارك!",
  "message_en": "Wishing you a blessed Ramadan. Prayer times are adjusted for the holy month.",
  "message_ar": "نتمنى لكم رمضان مبارك. تم تعديل أوقات الصلاة للشهر الفضيل.",
  "type": "info",
  "is_active": true,
  "expires": 1,
  "expiry_date": "2026-04-01"
}
```

### Example 3: App Update Available

```json
{
  "id": "update_v2.1.0",
  "title_en": "Update Available",
  "title_ar": "تحديث متاح",
  "message_en": "A new version with important fixes is available. Please update the app.",
  "message_ar": "إصدار جديد مع إصلاحات مهمة متاح. يرجى تحديث التطبيق.",
  "type": "warning",
  "is_active": true,
  "expires": 1,
  "expiry_date": "2026-03-01",
  "action_text_en": "Update Now",
  "action_text_ar": "تحديث الآن",
  "action_type": "link",
  "action_link": "https://play.google.com/store/apps/details?id=com.mbf.jordan_prayer_times_app"
}
```

## How It Works

1. App launches → waits 2 seconds → fetches `config/announcement.json`
2. The payload is cached on the device. If the fetch fails — no connection, a
   rate limit — the last cached payload is used instead, so an announcement
   already downloaded still reaches the user offline
3. Filters announcements: `is_active` must be `true`, not expired, already
   started, and applicable to the installed app version
4. Drops any the user has dismissed, then sorts what remains by `priority`
5. Shows the top one → user can dismiss or tap the action button
6. Dismissed IDs are stored locally (the same ID is never shown again)

## Multiple Announcements Behavior

When you have multiple announcements:
- Users see **one at a time** — the applicable one with the highest `priority`
- After dismissing, the next one shows on the next app launch
- Expired, not-yet-started and out-of-version announcements are skipped

## To Disable an Announcement

Three options:
1. Set `"is_active": false` - Hides immediately
2. Set `"expires": 1` with a past `expiry_date` - Auto-hidden
3. Remove from the array - Gone permanently

Note that a device holding a cached payload keeps using it until it can fetch
again, so switching an announcement off reaches an offline user only once they
are back online.

## Release Checklist

Publishing a "what's new" alongside an app release:

1. Add the entry with `min_version` set to the version being released, so
   nobody on an older build is told about features they do not have.
2. Set `start_date` to the rollout date and `expiry_date` a month or two later —
   a "what's new" is stale long before it is wrong.
3. Give it a `priority` above any standing notice if it should come first.
4. Push it with `is_active: true` ahead of the rollout; the start date holds it
   back until the update is actually out there.

## To Re-show to All Users

Change the `id` to a new unique value (e.g., `"cities_update_2026_01"` → `"cities_update_2026_02"`).

## Dialog Appearance by Type

| Type | Icon | Color |
|------|------|-------|
| `info` | Info icon | Blue (primary) |
| `warning` | Warning icon | Amber |
| `error` | Error icon | Red |
| `action` | Touch/tap icon | Orange |

## Legacy Format Support

The system also supports the old single-announcement format for backward compatibility:

```json
{
  "id": "single-announcement",
  "title_en": "Title",
  ...
}
```
