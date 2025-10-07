# OpenRTB Tracker Disclosure Extension Specification

**Version:** 1.0  
**Release Date:** October 2025  
**Document Type:** Protocol Extension  
**Extends:** OpenRTB 2.6 Specification

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Scope](#2-scope)
3. [Extension Overview](#3-extension-overview)
4. [Object Specifications](#4-object-specifications)
5. [Enumerated Lists](#5-enumerated-lists)
6. [Implementation Examples](#6-implementation-examples)
7. [Best Practices](#7-best-practices)
8. [Revision History](#8-revision-history)

---

## 1. Introduction

This document specifies an extension to the OpenRTB 2.6 protocol for transparent disclosure of tracking technologies embedded within advertising creatives. The extension enables advertisers, agencies, and demand-side platforms to communicate tracker information to supply-side platforms and publishers, facilitating privacy compliance and informed decision-making.

### 1.1 Background

With increasing privacy regulations including GDPR, ePrivacy Directive, and regional data protection laws, transparency regarding data collection mechanisms in advertising has become essential. This extension provides a standardized method for disclosing tracking technologies within the programmatic advertising ecosystem.

### 1.2 Objectives

- Provide transparency on tracking technologies included in ad creatives
- Enable compliance verification with IAB Transparency & Consent Framework (TCF)
- Distinguish between directly loaded and redirect-based trackers
- Facilitate publisher-side filtering and compliance enforcement

---

## 2. Scope

This extension adds an optional `trackers` object to the `Ad` object (Section 3.2.6 of OpenRTB 2.6). The extension is backward compatible and does not affect existing OpenRTB implementations.

**Applicability:**
- Display advertising
- Video advertising
- Native advertising
- Audio advertising

---

## 3. Extension Overview

The tracker disclosure extension introduces a new `trackers` array within the `ext` field of the `Ad` object. Each entry in the array represents a distinct tracking endpoint or technology embedded in or invoked by the ad creative.

### 3.1 Integration Point

**Object:** Ad Object (OpenRTB 2.6, Section 3.2.6)  
**Field:** `ext.trackers`  
**Type:** Array of objects

---

## 4. Object Specifications

### 4.1 Object: Ad.ext.trackers

Array of tracker disclosure objects providing transparency about tracking technologies.

| Attribute | Type | Description |
|-----------|------|-------------|
| domain | string; required | The fully qualified domain name of the tracking endpoint (e.g., "analytics.example.com"). |
| is_tcf_vendor | integer; required | Indicates whether the tracker is registered in the IAB TCF Global Vendor List. See List 5.1. |
| tcf_vendor_id | integer; recommended | IAB Transparency & Consent Framework Global Vendor List ID. REQUIRED if `is_tcf_vendor=1`. |
| load_type | integer; required | Indicates how the tracker is loaded by the ad creative. See List 5.2. |
| name | string | Human-readable identifier or name of the tracking service. |
| purpose | array of strings | General purpose categories for the tracker (e.g., "measurement", "attribution", "personalization"). |
| tcf_purposes | array of integers | Array of IAB TCF Purpose IDs (1-11) for which the tracker processes data. References IAB TCF Policy specification. |
| tcf_special_features | array of integers | Array of IAB TCF Special Feature IDs (1-2) utilized by the tracker. |
| tcf_legitimate_interest | integer | Indicates if tracker relies on legitimate interest. See List 5.1. |

### 4.2 Notes on Implementation

**Required Fields:**
- `domain`: Essential for identification
- `is_tcf_vendor`: Critical for compliance assessment
- `load_type`: Necessary for technical evaluation

**Conditional Fields:**
- `tcf_vendor_id`: MUST be provided when `is_tcf_vendor=1`
- `tcf_purposes`: RECOMMENDED when `is_tcf_vendor=1`

**Optional Fields:**
- `name`: Improves human readability
- `purpose`: Provides additional context
- `tcf_special_features`: For enhanced TCF compliance detail
- `tcf_legitimate_interest`: For legal basis transparency

---

## 5. Enumerated Lists

### 5.1 TCF Vendor Status

Indicates IAB TCF Global Vendor List registration status.

| Value | Description |
|-------|-------------|
| 0 | Not a registered IAB TCF vendor |
| 1 | Registered IAB TCF vendor |

### 5.2 Tracker Load Type

Specifies the mechanism by which the tracker is invoked.

| Value | Description |
|-------|-------------|
| 1 | **Direct** - Tracker is explicitly embedded in the ad markup delivered in the bid response. This includes image pixels, script tags, iframes, and other directly referenced resources. |
| 2 | **Redirect** - Tracker is loaded through redirect chains, server-side calls, or dynamically injected after initial ad load. This includes trackers invoked by JavaScript, redirect URLs, or secondary resource loading. |

### 5.3 IAB TCF Purpose IDs

Reference the IAB Transparency & Consent Framework Policies for complete definitions.

| Purpose ID | Purpose Name |
|------------|--------------|
| 1 | Store and/or access information on a device |
| 2 | Select basic ads |
| 3 | Create a personalized ads profile |
| 4 | Select personalized ads |
| 5 | Create a personalized content profile |
| 6 | Select personalized content |
| 7 | Measure ad performance |
| 8 | Measure content performance |
| 9 | Apply market research to generate audience insights |
| 10 | Develop and improve products |
| 11 | Use limited data to select advertising |

---

## 6. Implementation Examples

### 6.1 Example 1: Basic Tracker Disclosure

```json
{
  "id": "ad_12345",
  "impid": "imp_001",
  "adomain": ["advertiser-example.com"],
  "iurl": "https://cdn.adserver.com/creative/12345.jpg",
  "adm": "<script src='https://ad.adserver.com/serve.js'></script>",
  "ext": {
    "trackers": [
      {
        "domain": "analytics.adtech.com",
        "is_tcf_vendor": 1,
        "tcf_vendor_id": 755,
        "load_type": 1,
        "name": "AdTech Analytics",
        "purpose": ["measurement"],
        "tcf_purposes": [7]
      }
    ]
  }
}
```

### 6.2 Example 2: Multiple Trackers with Mixed TCF Status

```json
{
  "id": "ad_67890",
  "impid": "imp_002",
  "adomain": ["brand-example.com"],
  "iurl": "https://cdn.adserver.com/creative/67890.jpg",
  "adm": "<div>...</div>",
  "ext": {
    "trackers": [
      {
        "domain": "tracking.major-adtech.com",
        "is_tcf_vendor": 1,
        "tcf_vendor_id": 123,
        "load_type": 1,
        "name": "Major AdTech Platform",
        "purpose": ["measurement", "attribution"],
        "tcf_purposes": [1, 7, 8],
        "tcf_legitimate_interest": 0
      },
      {
        "domain": "pixels.attribution-service.com",
        "is_tcf_vendor": 1,
        "tcf_vendor_id": 456,
        "load_type": 2,
        "name": "Attribution Service",
        "purpose": ["attribution"],
        "tcf_purposes": [2, 7],
        "tcf_legitimate_interest": 1
      },
      {
        "domain": "custom-tracker.advertiser.com",
        "is_tcf_vendor": 0,
        "load_type": 1,
        "name": "Advertiser First-Party Tracker",
        "purpose": ["measurement"]
      }
    ]
  }
}
```

### 6.3 Example 3: Video Ad with Tracking Disclosure

```json
{
  "id": "video_ad_999",
  "impid": "imp_003",
  "adomain": ["video-advertiser.com"],
  "iurl": "https://cdn.video-adserver.com/thumb/999.jpg",
  "adm": "<?xml version=\"1.0\"?><VAST>...</VAST>",
  "ext": {
    "trackers": [
      {
        "domain": "video-analytics.adtech.com",
        "is_tcf_vendor": 1,
        "tcf_vendor_id": 789,
        "load_type": 1,
        "name": "Video Analytics Platform",
        "purpose": ["measurement"],
        "tcf_purposes": [7, 8],
        "tcf_special_features": [1]
      },
      {
        "domain": "viewability.measurement.com",
        "is_tcf_vendor": 1,
        "tcf_vendor_id": 234,
        "load_type": 2,
        "name": "Viewability Measurement",
        "purpose": ["measurement"],
        "tcf_purposes": [7]
      }
    ]
  }
}
```

---

## 7. Best Practices

### 7.1 For Demand-Side Platforms (DSPs) and Advertisers

**Complete Disclosure:**
- Disclose ALL trackers that will fire as a result of ad serving, including those loaded through redirects
- Regularly audit and update tracker lists as creative implementations change
- Include trackers from all parties in the supply chain (advertiser, agency, third-party vendors)

**Accurate TCF Information:**
- Verify TCF vendor IDs against the current Global Vendor List
- Accurately represent purpose and legal basis information
- Update implementations when vendors modify their TCF registrations

**Load Type Classification:**
- Classify trackers based on actual loading mechanism
- Use `load_type=2` for any tracker not directly in the initial ad markup

### 7.2 For Supply-Side Platforms (SSPs) and Publishers

**Compliance Verification:**
- Validate tracker disclosures against publisher privacy policies
- Implement filtering based on consent availability for TCF vendors
- Log undisclosed trackers detected during ad serving

**Consent String Validation:**
- Compare disclosed `tcf_vendor_id` values against available consent
- Reject or filter ads when required consent is not available
- Consider `tcf_legitimate_interest` flags in legal basis evaluation

**Monitoring and Enforcement:**
- Implement technical scanning to detect undisclosed trackers
- Establish penalties for systematic disclosure violations
- Provide feedback mechanisms to improve disclosure accuracy

### 7.3 Technical Considerations

**Performance:**
- Extension adds minimal overhead to bid response size
- Consider implementing compression for responses with extensive tracker lists

**Backward Compatibility:**
- Extension is fully optional and backward compatible
- Systems not implementing the extension can safely ignore the `ext.trackers` field

**Version Management:**
- Monitor IAB TCF Global Vendor List updates
- Implement processes to refresh vendor ID mappings

---

## 8. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | October 2025 | Initial release of Tracker Disclosure Extension specification |

---

## Appendix A: References

- **OpenRTB 2.6 Specification**: IAB Tech Lab, November 2020
- **IAB Transparency & Consent Framework**: IAB Europe, Current Version
- **IAB TCF Global Vendor List**: https://vendor-list.consensu.org/
- **GDPR**: Regulation (EU) 2016/679
- **ePrivacy Directive**: Directive 2002/58/EC

---

