# Digital Nutrition Label API Standard — Draft Proposal

**Status:** Draft / Open for Discussion
**Version:** 0.1.0
**Last Updated:** April 2026
**Editors:** Digital Nutrition Label Initiative
**Repository:** [github.com/digital-nutrition-label](https://github.com/digital-nutrition-label)

---

## Abstract

This document proposes a standardized REST API specification for algorithmic behavioral transparency. The API enables users of algorithmic feed platforms (social media, content recommendation services) to access machine-readable reports about their own behavioral data, including attention metrics, monetization data, and behavioral signals collected by the platform.

## Status of This Document

This is a **draft proposal** intended for public discussion. It has no official standing and is not endorsed by any standards body. We welcome feedback via [GitHub Issues](https://github.com/digital-nutrition-label/digital-nutrition-label.github.io/issues) or email at hello@digitalnutritionlabel.eu.

The goal is to refine this proposal through community input before formal submission to standards bodies (W3C, ETSI, CEN-CENELEC) and EU policymakers.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [API Specification](#3-api-specification)
4. [Data Schema](#4-data-schema)
5. [Authentication & Privacy](#5-authentication--privacy)
6. [Compliance Requirements](#6-compliance-requirements)
7. [Implementation Notes](#7-implementation-notes)
8. [Open Questions](#8-open-questions)
9. [References](#9-references)
10. [Changelog](#10-changelog)

---

## 1. Introduction

### 1.1 Problem Statement

Algorithmic feed platforms (Facebook, Instagram, TikTok, YouTube, LinkedIn, X, etc.) collect extensive behavioral data and use it to personalize content delivery. Users currently have no standardized way to understand:

- How much of their attention is captured by algorithmic feeds vs. self-directed browsing
- How much advertising revenue their activity generates
- What behavioral signals are collected and used for targeting
- How concentrated their content consumption has become (filter bubble metrics)

### 1.2 Proposed Solution

A mandatory, standardized API that platforms must implement, allowing each user to retrieve their own behavioral transparency report in a machine-readable format (JSON). This enables:

- **Personal insight:** Users understand their own platform usage
- **Third-party tools:** Trusted apps can help users analyze and compare their data
- **Parental oversight:** Parents can monitor their children's algorithmic exposure
- **Research:** Aggregated, consented data enables academic study

### 1.3 Scope

This specification applies to:

- Services operating in the EU
- Services using algorithmic content recommendation
- Services with more than 1 million monthly active users in the EU

---

## 2. Design Principles

### 2.1 User-Centric Access

**Only the user can access their own data.** No government official, regulator, or third party receives access to individual user reports. Regulatory compliance is verified through technical audits (checking that the API exists and functions correctly), not by inspecting user data.

### 2.2 Outcome Transparency, Not Algorithm Exposure

The API exposes **outcomes** (what happened to the user), not **methods** (how the algorithm works). This protects legitimate trade secrets while ensuring users understand the effects of algorithmic decisions.

### 2.3 Machine-Readable First

Data is provided in JSON format following a standardized schema, enabling automated processing, comparison tools, and third-party applications.

### 2.4 Privacy by Design

- Authentication required for all requests
- User must explicitly authorize any third-party access
- No cross-user data aggregation at the API level
- Data retention policies apply (platforms may limit historical data availability)

---

## 3. API Specification

### 3.1 Base URL

Platforms must expose the API at a standardized endpoint:

```
https://{platform-domain}/api/dnl/v1/
```

### 3.2 Endpoints

#### GET /transparency/weekly

Returns the user's behavioral transparency report for a specific week.

**Request:**
```http
GET /api/dnl/v1/transparency/weekly?period=2026-W11
Authorization: Bearer {user_access_token}
Accept: application/json
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `period` | string | Yes | ISO 8601 week format (YYYY-Www) |

**Response:** See [Data Schema](#4-data-schema)

#### GET /transparency/monthly

Returns aggregated data for a calendar month.

**Request:**
```http
GET /api/dnl/v1/transparency/monthly?period=2026-03
Authorization: Bearer {user_access_token}
Accept: application/json
```

#### GET /transparency/available-periods

Returns a list of periods for which data is available.

**Request:**
```http
GET /api/dnl/v1/transparency/available-periods
Authorization: Bearer {user_access_token}
Accept: application/json
```

**Response:**
```json
{
  "available_periods": {
    "weekly": ["2026-W10", "2026-W11", "2026-W12"],
    "monthly": ["2026-02", "2026-03"]
  },
  "retention_policy": {
    "weekly": "52 weeks",
    "monthly": "24 months"
  }
}
```

### 3.3 HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Invalid request (e.g., malformed period) |
| 401 | Authentication required |
| 403 | Forbidden (user not authorized for this data) |
| 404 | Data not available for requested period |
| 429 | Rate limit exceeded |
| 500 | Server error |

### 3.4 Rate Limiting

Platforms may implement rate limiting but must allow at least:
- 100 requests per user per day
- 10 requests per user per hour

Rate limit headers should be included in responses:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1714521600
```

---

## 4. Data Schema

### 4.1 Weekly Report Schema

```json
{
  "$schema": "https://digitalnutritionlabel.eu/schema/v1.0/weekly-report.json",
  "schema_version": "1.0.0",
  "platform": "example.com",
  "user_id_hash": "sha256:a1b2c3...",
  "period": "2026-W11",
  "generated_at": "2026-03-17T00:00:00Z",

  "attention": {
    "total_minutes": 862,
    "algorithmic_feed_minutes": 588,
    "self_directed_minutes": 274,
    "sessions_count": 47,
    "average_session_minutes": 18.3
  },

  "monetization": {
    "ad_revenue_eur": 1.84,
    "sponsored_items_served": 247,
    "purchases_influenced": 4,
    "purchases_influenced_value_eur": 89.50,
    "cheaper_alternatives_available": 3
  },

  "behavioral_signals": {
    "total_signals_collected": 1842,
    "signal_categories": {
      "engagement": 523,
      "content_preference": 412,
      "temporal_patterns": 298,
      "social_graph": 189,
      "location": 120,
      "device": 300
    },
    "engagement_predictions_used": 389,
    "preference_drift_events": 12
  },

  "content_metrics": {
    "items_served": 2847,
    "items_engaged": 342,
    "concentration_index": 0.73,
    "topic_diversity_score": 0.41,
    "source_diversity_score": 0.38
  },

  "metadata": {
    "data_completeness": 0.98,
    "processing_notes": []
  }
}
```

### 4.2 Field Definitions

#### attention
| Field | Type | Description |
|-------|------|-------------|
| `total_minutes` | integer | Total time spent on platform |
| `algorithmic_feed_minutes` | integer | Time spent in algorithmically-curated feeds |
| `self_directed_minutes` | integer | Time spent in search, direct navigation, etc. |
| `sessions_count` | integer | Number of distinct sessions |
| `average_session_minutes` | float | Average session duration |

#### monetization
| Field | Type | Description |
|-------|------|-------------|
| `ad_revenue_eur` | float | Estimated advertising revenue generated by user activity |
| `sponsored_items_served` | integer | Number of sponsored/promoted content items shown |
| `purchases_influenced` | integer | Purchases made via platform links/ads |
| `purchases_influenced_value_eur` | float | Total value of influenced purchases |
| `cheaper_alternatives_available` | integer | Cases where cheaper alternatives existed for influenced purchases |

#### behavioral_signals
| Field | Type | Description |
|-------|------|-------------|
| `total_signals_collected` | integer | Total behavioral data points collected |
| `signal_categories` | object | Breakdown by category |
| `engagement_predictions_used` | integer | Times engagement prediction influenced content ranking |
| `preference_drift_events` | integer | Detected shifts in user preferences not initiated by explicit user action |

#### content_metrics
| Field | Type | Description |
|-------|------|-------------|
| `items_served` | integer | Total content items shown to user |
| `items_engaged` | integer | Items user interacted with |
| `concentration_index` | float | 0.0–1.0 scale; 1.0 = maximum filter bubble |
| `topic_diversity_score` | float | 0.0–1.0 scale; higher = more diverse topics |
| `source_diversity_score` | float | 0.0–1.0 scale; higher = more diverse content sources |

---

## 5. Authentication & Privacy

### 5.1 Authentication

Platforms must support OAuth 2.0 with the following scope:

```
dnl:transparency:read
```

Users must explicitly grant this scope. Third-party applications requesting this scope must:
- Be registered with the platform
- Display clear purpose statement to users
- Comply with platform's developer terms

### 5.2 User Consent

Before a third-party app can access a user's transparency data:
1. User must authenticate with the platform
2. User must explicitly authorize the specific app
3. Authorization must be revocable at any time

### 5.3 Data Minimization

- API returns only the user's own data
- No endpoint exists for bulk user data retrieval
- User ID is hashed in responses (platforms may use internal IDs)

---

## 6. Compliance Requirements

### 6.1 For Platforms

Platforms in scope must:

1. **Implement the API** according to this specification
2. **Provide documentation** for developers at a public URL
3. **Support OAuth 2.0** authentication with the specified scope
4. **Retain data** for at least 52 weeks (weekly) / 24 months (monthly)
5. **Respond within SLA** (99.5% uptime, <500ms p95 latency)
6. **Not discriminate** against third-party tools using the API

### 6.2 Verification

Regulatory bodies verify compliance by:
- Checking API availability and correct implementation
- Testing with synthetic/test accounts
- Reviewing developer documentation

**Regulators never access real user data.**

---

## 7. Implementation Notes

### 7.1 Calculating Metrics

#### Concentration Index
The concentration index measures filter bubble intensity using content topic distribution:

```
concentration_index = 1 - (topic_entropy / max_entropy)
```

Where `topic_entropy` is calculated using Shannon entropy over the distribution of content topics served.

#### Preference Drift
A preference drift event is detected when:
- User engagement patterns shift significantly (>2 standard deviations)
- The shift was not preceded by explicit user action (search, follow, etc.)
- The shift correlates with algorithmic content delivery changes

### 7.2 Revenue Attribution

Ad revenue attribution should use the platform's existing internal attribution models. The goal is transparency about the user's economic value, not perfect accuracy.

---

## 8. Open Questions

We invite community input on the following:

1. **Granularity:** Should daily reports be required, or is weekly sufficient?
2. **Historical depth:** How far back should platforms be required to retain data?
3. **Child accounts:** Should there be special provisions for parental access to children's data?
4. **Real-time access:** Should users be able to access current-day data, or only completed periods?
5. **Cross-platform aggregation:** Should there be a standard for aggregating data across platforms?
6. **Verification methodology:** How should regulators verify revenue figures without accessing user data?

Please open issues on GitHub or email hello@digitalnutritionlabel.eu with your input.

---

## 9. References

### Standards & Specifications
- [OpenAPI Specification 3.1](https://spec.openapis.org/oas/v3.1.0.html)
- [JSON Schema 2020-12](https://json-schema.org/specification.html)
- [OAuth 2.0 (RFC 6749)](https://tools.ietf.org/html/rfc6749)
- [ISO 8601 Date/Time Format](https://www.iso.org/iso-8601-date-and-time-format.html)

### EU Legislation Context
- [GDPR (Regulation 2016/679)](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
- [Digital Services Act (Regulation 2022/2065)](https://eur-lex.europa.eu/eli/reg/2022/2065/oj)
- [AI Act (Regulation 2024/1689)](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
- [PSD2 (Directive 2015/2366)](https://eur-lex.europa.eu/eli/dir/2015/2366/oj) — as analogy for financial data access

### Academic Research
- Lam, M.S. et al. (2022). "End-User Audits: A System Empowering Communities to Lead Large-Scale Investigations of Harmful Algorithmic Behavior." ACM CHI.
- Cole, M.D. (2023). "Algorithmic Transparency and Accountability of Digital Services." IRIS Special Report, Council of Europe.

---

## 10. Changelog

### v0.1.0 (April 2026)
- Initial draft release
- Core schema definition
- Basic endpoint specification
- Authentication requirements
- Open questions for community input

---

## Contributing

This specification is developed openly. To contribute:

1. **Open an issue** for discussion of proposed changes
2. **Submit a pull request** for concrete text changes
3. **Join the mailing list** (coming soon) for broader discussions

All contributions are welcome under the [CC BY 4.0 license](https://creativecommons.org/licenses/by/4.0/).

---

*This document is maintained by the Digital Nutrition Label Initiative. It is not affiliated with or endorsed by any standards body or EU institution.*