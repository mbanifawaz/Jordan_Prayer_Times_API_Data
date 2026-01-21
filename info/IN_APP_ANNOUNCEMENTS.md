# In-App Announcement System

This guide explains how to use the in-app announcement system to notify users about important updates, issues, or information.

## Step 1: Create the Announcement File

Create a file at `config/announcement.json` in your `Jordan_Prayer_Times_API_Data` repository.

## Step 2: JSON Structure

```json
{
  "id": "unique-id-here",
  "title_en": "English Title",
  "title_ar": "العنوان بالعربي",
  "message_en": "English message explaining the issue or update.",
  "message_ar": "الرسالة بالعربي توضح المشكلة أو التحديث.",
  "type": "info",
  "is_active": true,
  "action_text_en": "Button Text",
  "action_text_ar": "نص الزر",
  "action_type": "clear_cache"
}
```

## Field Descriptions

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (e.g., `"2026-01-cities-fix"`). Users who dismiss won't see the same ID again |
| `title_en` | Yes | English title shown in dialog |
| `title_ar` | Yes | Arabic title shown in dialog |
| `message_en` | Yes | English message body |
| `message_ar` | Yes | Arabic message body |
| `type` | Yes | `"info"` (blue), `"warning"` (amber), `"error"` (red), `"action"` (orange) |
| `is_active` | Yes | `true` to show, `false` to hide |
| `action_text_en` | No | English button text (optional) |
| `action_text_ar` | No | Arabic button text (optional) |
| `action_type` | No | `"clear_cache"`, `"link"`, `"dismiss"` |
| `action_link` | No | URL if `action_type` is `"link"` |

## Action Types

- **`clear_cache`** - Clears all cached data and forces city reselection
- **`link`** - Opens a URL (requires `action_link` field)
- **`dismiss`** - Just closes the dialog

## Examples

### Example 1: Cities Changed Notification

```json
{
  "id": "2026-01-cities-update",
  "title_en": "Cities List Updated",
  "title_ar": "تم تحديث قائمة المدن",
  "message_en": "We've updated the cities list. Please clear your cache and reselect your city for accurate prayer times.",
  "message_ar": "تم تحديث قائمة المدن. يرجى مسح الذاكرة المؤقتة وإعادة اختيار مدينتك للحصول على أوقات صلاة دقيقة.",
  "type": "action",
  "is_active": true,
  "action_text_en": "Clear Cache",
  "action_text_ar": "مسح الذاكرة المؤقتة",
  "action_type": "clear_cache"
}
```

### Example 2: Simple Info Announcement

```json
{
  "id": "2026-02-ramadan",
  "title_en": "Ramadan Mubarak!",
  "title_ar": "رمضان مبارك!",
  "message_en": "Wishing you a blessed Ramadan. Prayer times are adjusted for the holy month.",
  "message_ar": "نتمنى لكم رمضان مبارك. تم تعديل أوقات الصلاة للشهر الفضيل.",
  "type": "info",
  "is_active": true
}
```

### Example 3: Link to Update

```json
{
  "id": "2026-03-update-available",
  "title_en": "Update Available",
  "title_ar": "تحديث متاح",
  "message_en": "A new version with important fixes is available. Please update the app.",
  "message_ar": "إصدار جديد مع إصلاحات مهمة متاح. يرجى تحديث التطبيق.",
  "type": "warning",
  "is_active": true,
  "action_text_en": "Update Now",
  "action_text_ar": "تحديث الآن",
  "action_type": "link",
  "action_link": "https://play.google.com/store/apps/details?id=com.mbf.jordan_prayer_times_app"
}
```

## How It Works

1. App launches → waits 2 seconds → fetches `config/announcement.json`
2. If `is_active` is `true` and user hasn't dismissed this `id` → shows dialog
3. User can tap "Dismiss" (won't see this `id` again) or action button
4. To show announcement again to everyone → change the `id`

## To Disable an Announcement

Set `"is_active": false` or delete the file.

## Dialog Appearance by Type

| Type | Icon | Color |
|------|------|-------|
| `info` | Info icon | Blue (primary) |
| `warning` | Warning icon | Amber |
| `error` | Error icon | Red |
| `action` | Touch/tap icon | Orange |
