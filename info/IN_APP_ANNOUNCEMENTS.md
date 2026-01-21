# In-App Announcement System

This guide explains how to use the in-app announcement system to notify users about important updates, issues, or information.

## Overview

The announcement system supports:
- **Multiple announcements** - Users see announcements one at a time (first unseen)
- **Expiry dates** - Announcements can auto-expire after a set date
- **Localization** - English and Arabic support
- **Action buttons** - Clear cache, open links, or dismiss

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
      "expiry_date": "2026-04-01"
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
| `action_text_en` | No | English button text (optional) |
| `action_text_ar` | No | Arabic button text (optional) |
| `action_type` | No | `"clear_cache"`, `"link"`, `"dismiss"` |
| `action_link` | No | URL if `action_type` is `"link"` |

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
2. Filters announcements: `is_active` must be `true` AND not expired
3. Checks user's dismissed list → finds first unseen announcement
4. Shows dialog → user can dismiss or tap action button
5. Dismissed IDs are stored locally (user won't see same ID again)

## Multiple Announcements Behavior

When you have multiple announcements:
- Users see **one announcement at a time** (the first unseen one)
- After dismissing, the next unseen announcement shows on next app launch
- Expired announcements are automatically skipped

## To Disable an Announcement

Three options:
1. Set `"is_active": false` - Hides immediately
2. Set `"expires": 1` with a past `expiry_date` - Auto-hidden
3. Remove from the array - Gone permanently

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
