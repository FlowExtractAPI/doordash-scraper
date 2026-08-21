# 🍔 DoorDash Scraper

**Scrape store listings, full menus, featured items, and customer reviews from DoorDash with location-aware search, multi-URL support, and optional review extraction.**

## 🎬 Video Tutorial

[![Watch the Tutorial](https://img.shields.io/badge/YouTube-Watch%20Tutorial-red?style=for-the-badge&logo=youtube)](https://youtube.com/@FlowExtractAPI)

https://www.youtube.com/watch?v=q5PGi7bmdXw

---

## Key Features

### Comprehensive Store Data Extraction
- **Store Identity**  name, description, cuisine type, business entity, and tag taxonomy with priority scores
- **Ratings & Reviews**  average rating, total count, star distribution (1★–5★ breakdown), review count
- **Pricing & Fees**  price range, delivery fee, pickup fee, surge indicators, DashPass eligibility
- **Availability**  delivery ETA, pickup ETA, open/closed status, and full unavailability reasons
- **Capabilities**  delivery, pickup, group order, scheduling, catering, drone delivery flags
- **Location**  full address, GPS coordinates, distance from your delivery address, timezone
- **Media**  header image, cover image, and all UGC photo assets with UUIDs

### Menu Extraction
- **Featured Items**  DoorDash's algorithmic "Featured Items" carousel with grid positions, recommendation signals, and badges (e.g. `#1 Most liked`)
- **Menu Categories**  all menu categories including the "Most Ordered" section, with per-item ratings, badges, and card position signals
- **Menu Hours**  daily open/close times per menu

### Customer Reviews (Optional)
- Full review text, star rating, verification status, and timestamps
- Complete reviewer metadata  display name, contribution count, reviewer tier
- `marked_up_review_text`  review text with `<<taggedItemId=>>` markup linking mentions to specific menu items by ID
- Extracted `tagged_item_ids` for direct joins back to menu data

### Two Scraping Modes
- **Search mode**  paste any DoorDash search URL and scrape multiple stores at once
- **Store mode**  paste a direct store URL and get full detail for that one store
---

## Quick Start

### Search for stores by keyword

```json
{
  "address": "350 5th Ave, New York, NY",
  "startUrls": [
    { "url": "https://www.doordash.com/search/store/pizza?event_type=search" }
  ],
  "maxResults": 10
}
```

### Scrape a specific store

```json
{
  "address": "350 5th Ave, New York, NY",
  "startUrls": [
    { "url": "https://www.doordash.com/store/joes-pizza-145565" }
  ]
}
```

### Scrape stores with reviews

```json
{
  "address": "350 5th Ave, New York, NY",
  "startUrls": [
    { "url": "https://www.doordash.com/search/store/sushi?event_type=search" }
  ],
  "maxResults": 5,
  "fetchReviews": true,
  "maxReviews": 20
}
```

### Multiple URLs in one run

```json
{
  "address": "Chicago, IL",
  "startUrls": [
    { "url": "https://www.doordash.com/search/store/burgers?event_type=search" },
    { "url": "https://www.doordash.com/store/shake-shack-12345" }
  ],
  "maxResults": 10,
  "fetchReviews": true,
  "maxReviews": 0
}
```

---

## 📋 Input Configuration

### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `address` | string | ✅ | Chicago, IL | Delivery address for location-aware results. **Important:** DoorDash results depend on location  always set this. |
| `startUrls` | array | ✅ |  | One or more DoorDash search or store URLs |
| `maxResults` | integer | ❌ | 10 | Max stores to scrape per search URL. `0` = no limit. Ignored in store mode. |
| `fetchReviews` | boolean | ❌ | false | Fetch customer reviews for each store |
| `maxReviews` | integer | ❌ | 10 | Max reviews per store. `0` = all reviews. Only used when `fetchReviews` is enabled. |

### Address Field  Why It Matters

DoorDash results are entirely location-dependent. A search for "pizza" in New York returns completely different stores from the same search in Los Angeles. If you leave this field blank, the Actor defaults to **Chicago, IL** and you may get results from the wrong area.

Always set `address` to match the city or neighborhood you want to scrape.

### Supported URL Formats

| Mode | Example URL |
|------|-------------|
| Search | `https://www.doordash.com/search/store/pizza?event_type=search` |
| Search with filters | `https://www.doordash.com/search/store/sushi?filterQuery-star_rating=4.6` |
| Store (with slug) | `https://www.doordash.com/store/joes-pizza-new-york-145565` |
| Store (numeric ID) | `https://www.doordash.com/store/145565` |
| Store (full URL) | `https://www.doordash.com/store/145565?cursor=...` |

DoorDash filter parameters in the URL (star rating, price range, DashPass, ETA, etc.) are honored automatically.

Full DoorDash store URLs containing `cursor`, ranking context, or other query parameters are fully supported. Extra parameters are ignored unless relevant.

---

## 📤 Output Structure

The Actor produces two record types in the same dataset, identified by the `record_type` field.

### Store Record (`record_type: "store"`)

One record per store. Contains store identity, ratings, fees, availability, location, media, featured items, and full menu categories  all embedded in a single document.

```json
{
  "record_type": "store",
  "store_id": "27544161",
  "name": "Lena's Italian Kitchen",
  "description": "Italian",
  "business": {
    "id": "29040069",
    "name": "Lena's Italian Kitchen",
    "vertical_id": 1
  },
  "tags": [
    { "id": "96", "name": "Italian", "priority": 99 },
    { "id": "9", "name": "Salads", "priority": 90 },
    { "id": "37", "name": "Casual Dining", "priority": 89 }
  ],
  "rating": 4.6,
  "num_ratings": 1167,
  "num_ratings_display": "1k+",
  "stars_distribution": { "one": 20, "two": 27, "three": 27, "four": 200, "five": 893 },
  "num_reviews": 31,
  "num_reviews_display": "30+ public reviews",
  "price_range": 2,
  "price_range_display": "$$",
  "currency": "USD",
  "is_dashpass": true,
  "delivery_fee_display": "$0 delivery fee",
  "delivery_fee_is_surging": false,
  "pickup_fee_display": "No fees",
  "is_open": false,
  "asap_minutes": null,
  "asap_status": {
    "is_available": false,
    "unavailable_status": "UNAVAILABLE_CLOSED",
    "unavailable_reason": "CLOSED",
    "display_reason": "Your address is not in this store's delivery area"
  },
  "asap_pickup_minutes": 15,
  "asap_pickup_status": {
    "is_available": false,
    "display_status": "Ready by 11:50 AM",
    "operating_summary": { ... }
  },
  "offers": {
    "delivery": true,
    "pickup": true,
    "group_order": true,
    "scheduling": true,
    "catering": false,
    "cannabis": false,
    "drone_delivery": false
  },
  "flags": {
    "is_newly_added": false,
    "is_convenience": false,
    "fulfills_own_deliveries": false
  },
  "address": "168 Mulberry St, New York, NY 10013, USA",
  "city": "New York",
  "state": "NY",
  "country": "US",
  "lat": 40.71956,
  "lng": -73.99734,
  "distance_from_consumer": "1.2 mi",
  "timezone": "America/New_York",
  "header_image": "https://img.cdn4dd.com/...",
  "cover_image": "https://img.cdn4dd.com/...",
  "cover_square_image": "https://img.cdn4dd.com/...",
  "additional_images": [
    { "url": "https://img.cdn4dd.com/u/media/...", "photo_uuid": "abc123" }
  ],
  "photo_count": "18 photos",
  "current_menu_id": "64599346",
  "menu_hours": [
    {
      "id": "64599346",
      "name": "Menu",
      "open_hours": [{ "open_time": "11:00:00", "close_time": "20:40:00" }],
      "display_hours": "11:00 AM - 8:40 PM"
    }
  ],
  "featured_items": {
    "collection_id": "recommended_items_for_you",
    "collection_name": "Featured Items",
    "collection_type": "mixed_grid_carousel",
    "num_rows": 2,
    "num_columns": 13,
    "total_items": 25,
    "items": [
      {
        "item_id": "11839454237",
        "name": "Pasta Fiorentina",
        "price_display": "$18.95",
        "price_cents": 1895,
        "image_url": "https://img.cdn4dd.com/...",
        "rating_pct": 94,
        "rating_count": 87,
        "badges": [
          { "text": "#1 Most liked", "type": "most_liked_1", "placement": "ITEM" }
        ],
        "recommendation": {
          "container": "recommended_items_for_you",
          "card_position": 0,
          "row": 0,
          "col": 0,
          "menu_id": "64599346",
          "category_id": "228676533"
        }
      }
    ]
  },
  "menu_categories": [
    {
      "category_id": "popular-items",
      "category_name": "Most Ordered",
      "is_popular": true,
      "collection_type": "item_list",
      "collection_position": 2,
      "visible_item_count": 5,
      "total_items": 12,
      "items": [
        {
          "item_id": "11839454237",
          "name": "Pasta Fiorentina",
          "description": "Spinach, mushroom, roasted garlic in a light cream sauce",
          "price_display": "$18.95",
          "price_cents": 1895,
          "image_url": "https://img.cdn4dd.com/...",
          "quick_add_eligible": true,
          "has_customization": true,
          "rating_pct": 94,
          "rating_count": 87,
          "badges": [
            { "text": "#1 Most liked", "type": "most_liked_1", "placement": "ITEM" }
          ],
          "card_position": 0,
          "cell_type": "SQUARE",
          "menu_id": "64599346",
          "category_id": "228676533"
        }
      ]
    }
  ],
  "url": "https://www.doordash.com/store/27544161",
  "_source": "doordash-scraper",
  "_scrapedAt": "2026-05-12T10:49:34.000Z"
}
```

### Review Record (`record_type: "review"`)

One record per customer review. The original DoorDash review payload is preserved intact, with dataset metadata and convenience fields added on top.

```json
{
  "record_type": "review",
  "store_id": "27544161",
  "review_id": "a0d43b28-a5f4-4914-8ed3-8528e5f709e4",
  "reviewer_name": "Max G",
  "rating": 5,
  "tagged_item_ids": ["11839454237", "17241668102"],
  "consumer_review_uuid": "a0d43b28-a5f4-4914-8ed3-8528e5f709e4",
  "reviewer_display_name": "Max G",
  "star_rating": 5,
  "reviewed_at": "2026-04-27T02:23:34.680827Z",
  "review_text": "Lena's has great, big portions and the food is overall very tasty...",
  "is_verified": true,
  "experience": "DOORDASH",
  "review_source": "DOORDASH",
  "reviewer_data": {
    "display_name": "Max G",
    "description": "Emerging Expert • 15 contributions",
    "profile_image": { "url": "" },
    "is_verified": false,
    "creator_profile_uri": "/consumer/profile/1553580317",
    "consumer_id": "1553580317"
  },
  "consumer_review_source": "CONSUMER_REVIEW_SOURCE_DOORDASH",
  "marked_up_review_text": "Lena's has great, big portions... <<taggedItemId=11839454237>>Chicken Marsala<</taggedItemId>> was hearty...",
  "_source": "doordash-scraper",
  "_scrapedAt": "2026-05-12T10:49:34.000Z"
}
```

### Store Record  Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `record_type` | string | Always `"store"` |
| `store_id` | string | DoorDash internal store identifier |
| `name` | string | Store display name |
| `description` | string | Cuisine type or short description |
| `business` | object | Business entity  id, name, vertical_id |
| `tags` | array | Tag taxonomy with id, name, priority score |
| `rating` | number | Average star rating (0–5) |
| `num_ratings` | integer | Total number of ratings |
| `num_ratings_display` | string | Display string e.g. "1k+" |
| `stars_distribution` | object | Rating breakdown by star level (one–five) |
| `num_reviews` | integer | Total number of written reviews |
| `price_range` | integer | 1=$, 2=$$, 3=$$$, 4=$$$$ |
| `price_range_display` | string | Display string e.g. "$$" |
| `is_dashpass` | boolean | DashPass partner status |
| `delivery_fee_display` | string | Delivery fee display string |
| `delivery_fee_is_surging` | boolean | Whether delivery fee is surging |
| `pickup_fee_display` | string | Pickup fee display string |
| `is_open` | boolean | Whether store is currently open for delivery |
| `asap_minutes` | integer | Estimated delivery time in minutes |
| `asap_status` | object | Full delivery availability object with reason |
| `asap_pickup_minutes` | integer | Estimated pickup time in minutes |
| `asap_pickup_status` | object | Full pickup availability object |
| `offers` | object | Capability flags  delivery, pickup, group order, scheduling, catering, cannabis, drone |
| `flags` | object | Store flags  newly_added, convenience, fulfills_own_deliveries |
| `address` | string | Full street address |
| `city` / `state` / `country` | string | Location components |
| `lat` / `lng` | number | GPS coordinates |
| `distance_from_consumer` | string | Distance from the input delivery address |
| `timezone` | string | Store timezone |
| `header_image` | string | Header image URL |
| `cover_image` | string | Cover image URL |
| `cover_square_image` | string | Square cover image URL |
| `additional_images` | array | UGC photos with url and photo_uuid |
| `photo_count` | string | Display string e.g. "18 photos" |
| `menu_hours` | array | Daily open/close times per menu |
| `featured_items` | object | Featured items carousel with grid positions and recommendation signals |
| `menu_categories` | array | All menu categories with full item detail, badges, and position signals |
| `url` | string | DoorDash store URL |
| `_source` | string | Always `"doordash-scraper"` |
| `_scrapedAt` | string | ISO 8601 scrape timestamp |

### Review Record  Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `record_type` | string | Always `"review"` |
| `store_id` | string | Join key back to the store record |
| `review_id` | string | Alias for `consumer_review_uuid` |
| `reviewer_name` | string | Alias for `reviewer_display_name` |
| `rating` | integer | Alias for `star_rating` (1–5) |
| `tagged_item_ids` | array | Item IDs extracted from `marked_up_review_text` |
| `consumer_review_uuid` | string | Original DoorDash review UUID |
| `star_rating` | integer | Original star rating field (1–5) |
| `review_text` | string | Plain review text |
| `marked_up_review_text` | string | Review text with `<<taggedItemId=N>>` item markup |
| `is_verified` | boolean | Whether the review is from a verified order |
| `reviewed_at` | string | ISO 8601 review timestamp |
| `reviewer_data` | object | Full reviewer object  display name, description, profile URI, consumer ID |
| `review_source` | string | Source platform e.g. `"DOORDASH"` |
| `experience` | string | Experience type e.g. `"DOORDASH"` |
| `_source` | string | Always `"doordash-scraper"` |
| `_scrapedAt` | string | ISO 8601 scrape timestamp |

---

## 📊 Pre-Configured Dataset Views

### 1. Stores
All store records in a flat table  identity, ratings, fees, availability, and embedded menu/featured data.

**Fields:** photo, store ID, name, cuisine, rating, ratings count, reviews count, price range, DashPass, open status, delivery ETA, delivery fee, pickup ETA, pickup fee, city, state, address, distance, featured items, menu categories, link

### 2. Reviews
All review records in a flat table  one row per customer review.

**Fields:** record type, store ID, review ID, star rating, reviewer name, review text, reviewed at, verified status, tagged item IDs, marked-up text, reviewer info, experience, source

---

## ⚠️ Important Notes

### Data Accuracy
- Store availability, fees, and menus change frequently  re-run the Actor for up-to-date data
- Reviews are a combination of preview reviews embedded in the store detail response and results from the paginated reviews endpoint, deduplicated by review UUID

### Review Extraction Behavior
- When `fetchReviews` is disabled, no review records are pushed  not even preview reviews
- When `fetchReviews` is enabled, up to `maxReviews` reviews are pushed per store (set to `0` for all)
- Review fetching is billed separately from store detail extraction

### Legal Considerations
This Actor extracts publicly available data from DoorDash. Users must:
- Comply with DoorDash's Terms of Service
- Follow applicable data protection laws (GDPR, CCPA, etc.)
- Use extracted data responsibly and ethically

**The Actor creator is not responsible for how users utilize the extracted data.**

---

## 💬 Support & Contact

### Get Help

- 🌐 **Website**: [flowextractapi.com](https://flowextractapi.com)
- 📧 **Email**: [flowextractapi@outlook.com](mailto:flowextractapi@outlook.com)
- 🙋 **Apify Profile**: [FlowExtract API](https://apify.com/dz_omar?fpr=smcx63)
- 💬 **GitHub Issues**: [FlowExtractAPI](https://github.com/FlowExtractAPI)

### Social Media

- 💼 **LinkedIn**: [flowextract-api](https://www.linkedin.com/in/flowextract-api/)
- 🐦 **Twitter**: [@FlowExtractAPI](https://x.com/@FlowExtractAPI)
- 📱 **Facebook**: [flowextractapi](https://www.facebook.com/flowextractapi)

---

## 🌟 Related Actors by FlowExtract API

### 🍽️ Food & Delivery

**[DoorDash Scraper](https://apify.com/dz_omar/doordash-scraper?fpr=smcx63)**
Scrape store listings, menus, featured items, and customer reviews from DoorDash.

### 🏠 Real Estate Data

**[Realtor.com Scraper](https://apify.com/dz_omar/realtor-scraper?fpr=smcx63)**
Extract US property listings from Realtor.com. Pricing, agent contacts, tax history, AVM estimates, nearby schools, flood and noise data, and full listing history.

**[PropertyFinder Scraper](https://apify.com/dz_omar/propertyfinder-scraper?fpr=smcx63)**
Extract property listings from PropertyFinder across UAE, Saudi Arabia, Bahrain, Egypt, and Qatar.

**[Idealista Scraper API](https://apify.com/dz_omar/idealista-scraper-api?fpr=smcx63)**
Advanced Idealista property data extraction with API access.

**[Idealista Scraper](https://apify.com/dz_omar/idealista-scraper?fpr=smcx63)**
Extract Spanish real estate listings from Idealista.

**[Leboncoin.fr Scraper](https://apify.com/dz_omar/leboncoin-scraper?fpr=smcx63)**
Extract French classified listings from Leboncoin  real estate, vehicles, jobs, and more.

**[AI Contact Intelligence Extractor](https://apify.com/dz_omar/ai-contact-intelligence?fpr=smcx63)**
Extract emails, phones, contacts & custom data using AI.

### 🎬 Video & Media Tools

**[YouTube Transcript & Metadata Extractor](https://apify.com/dz_omar/youtube-transcript-metadata?fpr=smcx63)**
Extract complete video transcripts with timestamps and comprehensive metadata.

**[YouTube Full Channel, Playlists, Shorts, Live](https://apify.com/dz_omar/Youtube-Scraper-Pro?fpr=smcx63)**
Extract complete playlist information with all video details from any YouTube playlist.

**[Zoom Scraper | Downloader & Transcript](https://apify.com/dz_omar/zoom-scraper?fpr=smcx63)**
Extract Zoom meeting recordings, transcripts, and metadata.

**[Loom Scraper | Downloader & Transcript](https://apify.com/dz_omar/loom-video-scraper?fpr=smcx63)**
Download Loom videos and extract transcripts.

### 🛠️ Developer & Security Tools

**[Screenshot](https://apify.com/dz_omar/screenshot?fpr=smcx63)**
Fast, reliable webpage screenshots with customizable options.

**[Ultimate Screenshot](https://apify.com/dz_omar/ultimate-screenshot?fpr=smcx63)**
Advanced screenshot tool with full-page capture, custom viewports, and quality controls.

**[Network Security Scanner](https://apify.com/dz_omar/network-security-scanner?fpr=smcx63)**
Scan websites for security vulnerabilities and get comprehensive security reports.

### 📱 Social Media Tools

**[Facebook Ads Scraper Pro](https://apify.com/dz_omar/facebook-ads-scraper-pro?fpr=smcx63)**
Extract Facebook ads data for competitor analysis and market research.

**[LinkedIn Ad Library Scraper](https://apify.com/dz_omar/linkedin-ads-scraper?fpr=smcx63)**
Extract comprehensive advertising data from LinkedIn's Ad Library.

---

**Ready to extract DoorDash data?** [Start using DoorDash Scraper now!](https://apify.com/dz_omar/doordash-scraper?fpr=smcx63)