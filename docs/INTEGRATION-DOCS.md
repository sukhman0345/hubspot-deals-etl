# 📋 HubSpot Deals ETL Service - Integration with HubSpot CRM API

This document explains the HubSpot CRM REST API v3 endpoints required by the HubSpot Deals ETL Service to extract deals data from HubSpot instances.

---

## 📋 Overview

The HubSpot Deals ETL Service integrates with HubSpot CRM API v3 endpoints to extract deal information. Below are the required and optional endpoints:

### ✅ **Required Endpoint (Essential)**
| **API Endpoint**                    | **Purpose**                          | **Version** | **Required Permissions**       | **Usage**    |
|-------------------------------------|--------------------------------------|-------------|-------------------------------|--------------|
| `/crm/v3/objects/deals`             | Search and list all deals            | v3          | `crm.objects.deals.read`      | **Required** |

### 🔧 **Optional Endpoints (Advanced Features)**
| **API Endpoint**                              | **Purpose**                              | **Version** | **Required Permissions**             | **Usage**  |
|-----------------------------------------------|------------------------------------------|-------------|--------------------------------------|------------|
| `/crm/v3/objects/deals/{dealId}`              | Get detailed deal information            | v3          | `crm.objects.deals.read`             | Optional   |
| `/crm/v3/objects/deals/{dealId}/associations` | Get deal associations (contacts, companies) | v3       | `crm.objects.contacts.read`          | Optional   |
| `/crm/v3/properties/deals`                    | Get all available deal properties        | v3          | `crm.objects.deals.read`             | Optional   |
| `/crm/v3/pipelines/deals`                     | Get deal pipeline and stage configuration | v3         | `crm.objects.deals.read`             | Optional   |

### 🎯 **Recommendation**
**Start with only the required endpoint.** The `/crm/v3/objects/deals` endpoint provides all essential deal data needed for basic deals analytics and extraction.

---

## 🔐 Authentication Requirements

### **Private App Token Authentication**
```http
Authorization: Bearer YOUR_PRIVATE_APP_ACCESS_TOKEN
Content-Type: application/json
```

### **How to Get Your Access Token**
1. Go to your HubSpot account
2. Navigate to **Settings** → **Integrations** → **Private Apps**
3. Click **Create a private app**
4. Name it: `DLT Deals Extractor`
5. Under **Scopes**, select `crm.objects.deals.read`
6. Click **Create app** and copy the **Access Token**

### **Required Permissions**
- **`crm.objects.deals.read`**: Read access to deals objects (required)
- **`crm.objects.contacts.read`**: Read access to contacts (optional, for associations)
- **`crm.objects.companies.read`**: Read access to companies (optional, for associations)

---

## 🌐 HubSpot CRM API Endpoints

### 🎯 **PRIMARY ENDPOINT (Required for Basic Deal Extraction)**

### 1. **List All Deals** - `/crm/v3/objects/deals` ✅ **REQUIRED**

**Purpose**: Get paginated list of all deals — **THIS IS ALL YOU NEED FOR BASIC DEAL EXTRACTION**

**Method**: `GET`

**Base URL**: `https://api.hubapi.com`

**Full URL**: `https://api.hubapi.com/crm/v3/objects/deals`

**Query Parameters**:
| Parameter    | Type    | Description                                              | Example                          |
|--------------|---------|----------------------------------------------------------|----------------------------------|
| `limit`      | integer | Number of deals per page (max 100)                       | `limit=100`                      |
| `after`      | string  | Cursor for next page pagination                          | `after=eyJsaW1pdCI6MTAwf...`     |
| `properties` | string  | Comma-separated list of deal properties to return        | `properties=dealname,amount,dealstage` |
| `archived`   | boolean | Include archived/deleted deals                           | `archived=false`                 |

**Request Example**:
```http
GET https://api.hubapi.com/crm/v3/objects/deals?limit=100&properties=dealname,amount,dealstage,closedate,pipeline,hubspot_owner_id
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Response Structure**:
```json
{
  "results": [
    {
      "id": "123456789",
      "properties": {
        "dealname": "Enterprise Software Deal - Google",
        "amount": "50000",
        "dealstage": "qualifiedtobuy",
        "closedate": "2026-07-31T00:00:00.000Z",
        "pipeline": "default",
        "hubspot_owner_id": "55551234",
        "createdate": "2026-01-15T10:30:00.000Z",
        "hs_lastmodifieddate": "2026-06-01T08:00:00.000Z",
        "hs_deal_stage_probability": "0.6"
      },
      "createdAt": "2026-01-15T10:30:00.000Z",
      "updatedAt": "2026-06-01T08:00:00.000Z",
      "archived": false
    },
    {
      "id": "987654321",
      "properties": {
        "dealname": "Startup Package - Acme Corp",
        "amount": "5000",
        "dealstage": "closedwon",
        "closedate": "2026-05-15T00:00:00.000Z",
        "pipeline": "default",
        "hubspot_owner_id": "55551234",
        "createdate": "2026-03-10T09:00:00.000Z",
        "hs_lastmodifieddate": "2026-05-15T16:00:00.000Z",
        "hs_deal_stage_probability": "1.0"
      },
      "createdAt": "2026-03-10T09:00:00.000Z",
      "updatedAt": "2026-05-15T16:00:00.000Z",
      "archived": false
    }
  ],
  "paging": {
    "next": {
      "after": "eyJsaW1pdCI6MTAwLCJhZnRlciI6Ijk4NzY1NDMyMSJ9",
      "link": "https://api.hubapi.com/crm/v3/objects/deals?after=eyJsaW1pdCI6MTAwLCJhZnRlciI6Ijk4NzY1NDMyMSJ9"
    }
  }
}
```

**✅ This endpoint provides ALL the default deal fields:**
- `id` — Unique HubSpot deal ID
- `dealname` — Name of the deal
- `amount` — Deal value in currency
- `dealstage` — Current pipeline stage (e.g. `qualifiedtobuy`, `closedwon`, `closedlost`)
- `closedate` — Expected or actual close date
- `pipeline` — Pipeline the deal belongs to
- `hubspot_owner_id` — ID of the assigned sales rep
- `createdate` / `hs_lastmodifieddate` — Timestamps
- `hs_deal_stage_probability` — Win probability (0.0 to 1.0)

**Rate Limit**: 150 requests per 10 seconds per token

---

## 🔧 **OPTIONAL ENDPOINTS (Advanced Features Only)**

> **⚠️ Note**: These endpoints are NOT required for basic deal extraction. Only implement if you need advanced deal analytics.

### 2. **Get Single Deal** - `/crm/v3/objects/deals/{dealId}` 🔧 **OPTIONAL**

**Purpose**: Get detailed information for a specific deal

**Method**: `GET`

**URL**: `https://api.hubapi.com/crm/v3/objects/deals/{dealId}`

**Request Example**:
```http
GET https://api.hubapi.com/crm/v3/objects/deals/123456789
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Response Structure**:
```json
{
  "id": "123456789",
  "properties": {
    "dealname": "Enterprise Software Deal - Google",
    "amount": "50000",
    "dealstage": "qualifiedtobuy",
    "closedate": "2026-07-31T00:00:00.000Z",
    "description": "Annual enterprise software license",
    "pipeline": "default",
    "hubspot_owner_id": "55551234",
    "createdate": "2026-01-15T10:30:00.000Z",
    "hs_lastmodifieddate": "2026-06-01T08:00:00.000Z"
  },
  "createdAt": "2026-01-15T10:30:00.000Z",
  "updatedAt": "2026-06-01T08:00:00.000Z",
  "archived": false
}
```

---

### 3. **Get Deal Associations** - `/crm/v3/objects/deals/{dealId}/associations/{toObjectType}` 🔧 **OPTIONAL**

**Purpose**: Get contacts or companies associated with a deal

**Method**: `GET`

**URL**: `https://api.hubapi.com/crm/v3/objects/deals/{dealId}/associations/contacts`

**Request Example**:
```http
GET https://api.hubapi.com/crm/v3/objects/deals/123456789/associations/contacts
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Response Structure**:
```json
{
  "results": [
    {
      "id": "contact_001",
      "type": "deal_to_contact"
    }
  ]
}
```

---

### 4. **Get Deal Properties** - `/crm/v3/properties/deals` 🔧 **OPTIONAL**

**Purpose**: Get all available deal property definitions

**Method**: `GET`

**URL**: `https://api.hubapi.com/crm/v3/properties/deals`

**Request Example**:
```http
GET https://api.hubapi.com/crm/v3/properties/deals
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Response Structure**:
```json
{
  "results": [
    {
      "name": "dealname",
      "label": "Deal Name",
      "type": "string",
      "fieldType": "text",
      "description": "The name given to this deal.",
      "groupName": "dealinformation"
    },
    {
      "name": "amount",
      "label": "Amount",
      "type": "number",
      "fieldType": "number",
      "description": "The total value of the deal.",
      "groupName": "dealinformation"
    },
    {
      "name": "dealstage",
      "label": "Deal Stage",
      "type": "enumeration",
      "fieldType": "select",
      "description": "The stage of the deal.",
      "groupName": "dealinformation",
      "options": [
        {"label": "Qualified To Buy", "value": "qualifiedtobuy"},
        {"label": "Presentation Scheduled", "value": "presentationscheduled"},
        {"label": "Decision Maker Bought-In", "value": "decisionmakerboughtin"},
        {"label": "Contract Sent", "value": "contractsent"},
        {"label": "Closed Won", "value": "closedwon"},
        {"label": "Closed Lost", "value": "closedlost"}
      ]
    }
  ]
}
```

---

### 5. **Get Pipeline Configuration** - `/crm/v3/pipelines/deals` 🔧 **OPTIONAL**

**Purpose**: Get deal pipeline stages and configuration

**Method**: `GET`

**URL**: `https://api.hubapi.com/crm/v3/pipelines/deals`

**Request Example**:
```http
GET https://api.hubapi.com/crm/v3/pipelines/deals
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Response Structure**:
```json
{
  "results": [
    {
      "id": "default",
      "label": "Sales Pipeline",
      "displayOrder": 0,
      "stages": [
        {"id": "qualifiedtobuy", "label": "Qualified To Buy", "probability": 0.2},
        {"id": "presentationscheduled", "label": "Presentation Scheduled", "probability": 0.4},
        {"id": "closedwon", "label": "Closed Won", "probability": 1.0},
        {"id": "closedlost", "label": "Closed Lost", "probability": 0.0}
      ]
    }
  ]
}
```

---

## 📊 Data Extraction Flow

### 🎯 **SIMPLE FLOW (Recommended)**

```python
def extract_all_deals(access_token):
    """Extract all deals using cursor-based pagination"""
    base_url = "https://api.hubapi.com"
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json"
    }
    properties = "dealname,amount,dealstage,closedate,pipeline,hubspot_owner_id"
    
    all_deals = []
    after = None

    while True:
        params = {
            "limit": 100,
            "properties": properties,
            "archived": False
        }
        if after:
            params["after"] = after

        response = requests.get(
            f"{base_url}/crm/v3/objects/deals",
            params=params,
            headers=headers
        )
        response.raise_for_status()
        data = response.json()

        deals = data.get("results", [])
        all_deals.extend(deals)

        # Check for next page
        paging = data.get("paging", {})
        next_page = paging.get("next", {})
        after = next_page.get("after")

        if not after:
            break  # No more pages

    return all_deals
```

---

## ⚡ Performance Considerations

### **Rate Limiting**
- **Default Limit**: 150 requests per 10 seconds per API token
- **Best Practice**: Implement exponential backoff on `429` responses
- **Retry-After**: Check `Retry-After` header on rate limit responses

### **Pagination**
- Use cursor-based pagination via `after` parameter
- Max `limit` per request: 100 deals
- Save `after` cursor as checkpoint every 5 pages for resumability

### **Error Handling**
```http
# Rate limit exceeded
HTTP/429 Too Many Requests
Retry-After: 10

# Authentication failed
HTTP/401 Unauthorized

# Insufficient permissions
HTTP/403 Forbidden

# Deal not found
HTTP/404 Not Found
```

---

## 🔒 Security Requirements

### **Required Scopes (Minimum)**
```
crm.objects.deals.read
```

### **Optional Scopes (Advanced Features)**
```
crm.objects.contacts.read   (for contact associations)
crm.objects.companies.read  (for company associations)
```

### **Security Best Practices**
- Never commit access tokens to version control
- Store tokens in `.env` file (listed in `.gitignore`)
- Use dedicated test HubSpot account for development
- Rotate tokens regularly

---

## 🧪 Testing API Integration

### **Test Authentication**
```bash
curl -X GET \
  "https://api.hubapi.com/crm/v3/objects/deals?limit=1" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

### **Test Deal List with Properties**
```bash
curl -X GET \
  "https://api.hubapi.com/crm/v3/objects/deals?limit=5&properties=dealname,amount,dealstage" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

---

## 🚨 Common Issues & Solutions

### **Issue**: 401 Unauthorized
**Solution**: Verify your Private App access token is valid and not expired

### **Issue**: 403 Forbidden
**Solution**: Check your Private App has `crm.objects.deals.read` scope enabled

### **Issue**: 429 Rate Limited
**Solution**: Implement retry with exponential backoff:
```python
import time
import random

def retry_with_backoff(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError:
            wait_time = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait_time)
    raise Exception("Max retries exceeded")
```

### **Issue**: Empty results list
**Solution**: Check that deals exist in your HubSpot account and the token has correct permissions

---

## 📞 Support Resources

- **HubSpot CRM API Docs**: https://developers.hubspot.com/docs/api/crm/deals
- **Rate Limiting Guide**: https://developers.hubspot.com/docs/api/usage-details
- **Authentication Guide**: https://developers.hubspot.com/docs/api/private-apps
- **Deal Properties Reference**: https://developers.hubspot.com/docs/api/crm/properties
