# SCM Sales Forecasting and Prediction API Documentation

## Overview

This document provides comprehensive documentation for the **SCM Sales Forecasting and Prediction API**, a Microsoft Dynamics 365 Business Central extension developed by EOS Solutions. The API enables integration with external systems for managing items, data entries, item ledger entries, calendar configurations, manufacturing setup, and Reorder Point (ROP) calculations.

## API Specification

The complete OpenAPI 3.2.0 specification is available in:
- **YAML Format**: [openapi-specification.yaml](./openapi-specification.yaml)

The specification conforms to the [OpenAPI Specification v3.2.0](https://spec.openapis.org/oas/v3.2.0.html).

## Base URLs

### SaaS (Cloud)
```
https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/eosSolutions/scmDataAPI/v1.0
```
Replace `{tenant}` with your Business Central tenant identifier.

### On-Premise
```
http://localhost:7048/{instance}/api/eosSolutions/scmDataAPI/v1.0
```
Replace `{instance}` with your Business Central server instance name (e.g., BC260).

## Authentication

The API supports two authentication methods:

### 1. OAuth 2.0 (Recommended for SaaS)
- **Authorization URL**: `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`
- **Token URL**: `https://login.microsoftonline.com/common/oauth2/v2.0/token`
- **Scope**: `user_impersonation`

### 2. Basic Authentication (For On-Premise)
- **Username**: Your Business Central username
- **Password**: Web service access key (generated in Business Central)

## API Endpoints

### 1. Items API
**Purpose**: Retrieve item master data for forecasting

#### Endpoints:
- `GET /scmItems` - List all items
- `GET /scmItems({id})` - Get a specific item by system ID

#### Access Level: Read-Only

#### Example Request:
```http
GET /scmItems?$filter=itemCategoryCode eq 'BIKE'&$top=10
Authorization: Bearer {token}
```

#### Example Response:
```json
{
  "value": [
    {
      "systemId": "5d115c9c-44e3-ea11-bb43-000d3a2feca1",
      "no": "1000",
      "description": "Bicycle",
      "description2": "Mountain Bike",
      "itemCategoryCode": "BIKE",
      "baseUnitOfMeasure": "PCS"
    }
  ]
}
```

---

### 2. Data Entry API
**Purpose**: Manage staging data entries for sales forecasting calculations

#### Endpoints:
- `GET /scmDataEntries` - List all data entries
- `GET /scmDataEntries({id})` - Get a specific data entry
- `POST /scmDataEntries` - Create a new data entry
- `PATCH /scmDataEntries({id})` - Update a data entry
- `DELETE /scmDataEntries({id})` - Delete a data entry

#### Access Level: Full CRUD

#### Example Create Request:
```http
POST /scmDataEntries
Content-Type: application/json
Authorization: Bearer {token}

{
  "itemNo": "1000",
  "postingDate": "2026-02-15",
  "quantity": 100,
  "locationCode": "MAIN",
  "entryType": 1,
  "excluded": false,
  "fixed": false
}
```

#### Example Update Request:
```http
PATCH /scmDataEntries(5d115c9c-44e3-ea11-bb43-000d3a2feca1)
Content-Type: application/json
If-Match: W/"JzQ0O0VnQUFBQUo3QlRnQU1BQXdBREFBTUFBd0FEeHJRVUFBO1NBQUFBQUFBQUFBd0FEQUFNQUF3QURBQScn"
Authorization: Bearer {token}

{
  "quantity": 150,
  "excluded": true
}
```

---

### 3. Item Ledger Entries API
**Purpose**: Access historical item transactions for analysis

#### Endpoints:
- `GET /scmItemLedgerEntries` - List item ledger entries
- `GET /scmItemLedgerEntries({id})` - Get a specific entry

#### Access Level: Read-Only

#### Example Request:
```http
GET /scmItemLedgerEntries?$filter=itemNo eq '1000' and postingDate ge 2026-01-01
Authorization: Bearer {token}
```

---

### 4. Base Calendars API
**Purpose**: Retrieve calendar working/non-working day information

#### Endpoints:
- `GET /scmBaseCalendars` - List calendar changes
- `GET /scmBaseCalendars({id})` - Get a specific calendar change

#### Access Level: Read-Only

#### Note:
The API automatically filters by the "Default Calendar Code (API)" setting in the SCM Setup if configured.

#### Example Request:
```http
GET /scmBaseCalendars?$filter=nonworking eq true and date ge 2026-02-01
Authorization: Bearer {token}
```

---

### 5. Manufacturing Setup API
**Purpose**: Retrieve manufacturing and inventory configuration

#### Endpoints:
- `GET /scmManufSetup` - List setup configurations
- `GET /scmManufSetup({id})` - Get specific setup entry

#### Access Level: Read-Only

#### Example Request:
```http
GET /scmManufSetup
Authorization: Bearer {token}
```

---

### 6. ROP Journal API
**Purpose**: Manage Reorder Point calculation journal lines

#### Endpoints:
- `GET /scmROPJnlLines` - List ROP journal lines
- `GET /scmROPJnlLines({id})` - Get a specific line
- `POST /scmROPJnlLines` - Create a new line
- `PATCH /scmROPJnlLines({id})` - Update a line
- `DELETE /scmROPJnlLines({id})` - Delete a line

#### Access Level: Full CRUD

#### Example Create Request:
```http
POST /scmROPJnlLines
Content-Type: application/json
Authorization: Bearer {token}

{
  "journalTemplateName": "ROP",
  "journalBatchName": "DEFAULT",
  "lineNo": 10000,
  "itemNo": "1000",
  "locationCode": "MAIN",
  "postingDate": "2026-02-15",
  "avgDemand": 50,
  "avgLeadTime": 7,
  "newSafetyStockValue": 100,
  "newReorderPointValue": 200,
  "updateSafetyStock": true
}
```

---

## OData Query Options

The API supports standard OData query parameters:

### $filter
Filter results based on property values.

**Examples:**
```
$filter=itemNo eq '1000'
$filter=postingDate ge 2026-01-01 and postingDate le 2026-12-31
$filter=quantity gt 100
$filter=excluded eq false
```

### $top
Limit the number of results returned (max: 10000).

**Example:**
```
$top=20
```

### $skip
Skip the first n results (for pagination).

**Example:**
```
$skip=40
```

### $select
Specify which properties to return.

**Example:**
```
$select=itemNo,description,quantity
```

### $expand
Expand related entities (if navigation properties exist).

**Example:**
```
$expand=item
```

### Combining Parameters
```
GET /scmDataEntries?$filter=itemNo eq '1000'&$top=50&$skip=0&$select=itemNo,quantity,postingDate
```

---

## Optimistic Concurrency Control

For UPDATE and DELETE operations, the API uses ETags for optimistic concurrency control to prevent update conflicts.

### Workflow:
1. **GET** the resource and extract the ETag from the response header
2. Include the ETag in the **If-Match** header when updating:
   ```http
   If-Match: W/"JzQ0O0VnQUFBQUo3QlRnQU1BQXdBREFBTUFBd0FEeHJRVUFBO1NBQUFBQUFBQUFBd0FEQUFNQUF3QURBQScn"
   ```
3. If the resource has been modified by another user, you'll receive a **412 Precondition Failed** error

---

## Error Handling

The API returns standard HTTP status codes:

| Code | Description |
|------|-------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 204 | No Content - Delete successful |
| 400 | Bad Request - Invalid request parameters |
| 401 | Unauthorized - Authentication required |
| 404 | Not Found - Resource doesn't exist |
| 412 | Precondition Failed - ETag mismatch |
| 500 | Internal Server Error |

### Error Response Format:
```json
{
  "error": {
    "code": "BadRequest",
    "message": "The request is invalid",
    "target": "itemNo",
    "details": [
      {
        "code": "ValidationError",
        "message": "Item No. is required",
        "target": "itemNo"
      }
    ]
  }
}
```

---

## Code Examples

### C# Example
```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

public class ScmApiClient
{
    private readonly HttpClient _httpClient;
    private readonly string _baseUrl;

    public ScmApiClient(string baseUrl, string accessToken)
    {
        _httpClient = new HttpClient();
        _baseUrl = baseUrl;
        _httpClient.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", accessToken);
    }

    public async Task<string> GetItemsAsync()
    {
        var response = await _httpClient.GetAsync($"{_baseUrl}/scmItems");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsStringAsync();
    }

    public async Task<string> CreateDataEntryAsync(string itemNo, DateTime postingDate, decimal quantity)
    {
        var json = $@"{{
            ""itemNo"": ""{itemNo}"",
            ""postingDate"": ""{postingDate:yyyy-MM-dd}"",
            ""quantity"": {quantity}
        }}";

        var content = new StringContent(json, Encoding.UTF8, "application/json");
        var response = await _httpClient.PostAsync($"{_baseUrl}/scmDataEntries", content);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsStringAsync();
    }
}
```

### Python Example
```python
import requests
from datetime import date

class ScmApiClient:
    def __init__(self, base_url, access_token):
        self.base_url = base_url
        self.headers = {
            'Authorization': f'Bearer {access_token}',
            'Content-Type': 'application/json'
        }
    
    def get_items(self, filter_query=None):
        url = f"{self.base_url}/scmItems"
        if filter_query:
            url += f"?$filter={filter_query}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def create_data_entry(self, item_no, posting_date, quantity):
        url = f"{self.base_url}/scmDataEntries"
        data = {
            "itemNo": item_no,
            "postingDate": posting_date.isoformat(),
            "quantity": quantity
        }
        response = requests.post(url, json=data, headers=self.headers)
        response.raise_for_status()
        return response.json()

# Usage
client = ScmApiClient(
    "https://api.businesscentral.dynamics.com/v2.0/production/api/eosSolutions/scmDataAPI/v1.0",
    "your_access_token_here"
)

# Get items
items = client.get_items(filter_query="itemCategoryCode eq 'BIKE'")
print(items)

# Create data entry
entry = client.create_data_entry("1000", date(2026, 2, 15), 100)
print(entry)
```

### JavaScript/Node.js Example
```javascript
const axios = require('axios');

class ScmApiClient {
    constructor(baseUrl, accessToken) {
        this.baseUrl = baseUrl;
        this.headers = {
            'Authorization': `Bearer ${accessToken}`,
            'Content-Type': 'application/json'
        };
    }

    async getItems(filterQuery) {
        const url = filterQuery 
            ? `${this.baseUrl}/scmItems?$filter=${filterQuery}`
            : `${this.baseUrl}/scmItems`;
        
        const response = await axios.get(url, { headers: this.headers });
        return response.data;
    }

    async createDataEntry(itemNo, postingDate, quantity) {
        const url = `${this.baseUrl}/scmDataEntries`;
        const data = {
            itemNo,
            postingDate,
            quantity
        };
        
        const response = await axios.post(url, data, { headers: this.headers });
        return response.data;
    }

    async updateDataEntry(id, etag, updates) {
        const url = `${this.baseUrl}/scmDataEntries(${id})`;
        const headers = {
            ...this.headers,
            'If-Match': etag
        };
        
        const response = await axios.patch(url, updates, { headers });
        return response.data;
    }
}

// Usage
const client = new ScmApiClient(
    'https://api.businesscentral.dynamics.com/v2.0/production/api/eosSolutions/scmDataAPI/v1.0',
    'your_access_token_here'
);

// Example: Get items and create data entry
(async () => {
    try {
        const items = await client.getItems("itemCategoryCode eq 'BIKE'");
        console.log('Items:', items);

        const entry = await client.createDataEntry('1000', '2026-02-15', 100);
        console.log('Created entry:', entry);
    } catch (error) {
        console.error('Error:', error.response?.data || error.message);
    }
})();
```

---

## Testing the API

### Using Postman
1. Import the [openapi-specification.yaml](./openapi-specification.yaml) file into Postman
2. Configure authentication (OAuth 2.0 or Basic Auth)
3. Set the appropriate base URL
4. Execute requests from the generated collection

### Using Swagger UI
1. Open [Swagger Editor](https://editor.swagger.io/)
2. Load the `openapi-specification.yaml` file
3. Use the interactive UI to test endpoints

### Using cURL
```bash
# Get items
curl -X GET "https://api.businesscentral.dynamics.com/v2.0/production/api/eosSolutions/scmDataAPI/v1.0/scmItems" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Create data entry
curl -X POST "https://api.businesscentral.dynamics.com/v2.0/production/api/eosSolutions/scmDataAPI/v1.0/scmDataEntries" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "itemNo": "1000",
    "postingDate": "2026-02-15",
    "quantity": 100
  }'
```

---

## Best Practices

1. **Use Filtering**: Always use `$filter` to retrieve only the data you need
2. **Implement Pagination**: Use `$top` and `$skip` for large datasets
3. **Select Specific Fields**: Use `$select` to reduce payload size
4. **Handle ETags**: Always include ETags when updating or deleting resources
5. **Error Handling**: Implement robust error handling for all API calls
6. **Rate Limiting**: Be mindful of API rate limits (consult Microsoft documentation)
7. **Security**: Never expose access tokens in client-side code
8. **Batch Operations**: For multiple operations, consider using Business Central's batch endpoints if available

---

## API Versioning

- **API Group**: `scmDataAPI`
- **API Publisher**: `eosSolutions`
- **API Version**: `v1.0`

Future versions will be published under new version paths (e.g., `v2.0`) while maintaining backward compatibility for `v1.0`.

---

## Support and Resources

- **Publisher**: EOS Solutions
- **Website**: [https://www.eos-solutions.it/](https://www.eos-solutions.it/)
- **Product Page**: [Advanced Sales Forecasting and SCM](https://www.eos-solutions.it/en/advanced-sales-forecasting-and-scm.html)
- **Privacy Policy**: [https://www.eos-solutions.it/en/privacy.html](https://www.eos-solutions.it/en/privacy.html)
- **EULA**: [https://www.eos-solutions.it/en/eula.html](https://www.eos-solutions.it/en/eula.html)

For technical support, please contact: support@eos-solutions.it

---

## License

This API is part of the SCM Sales Forecasting and Prediction extension for Microsoft Dynamics 365 Business Central, published by EOS Solutions. Usage is subject to the terms and conditions outlined in the [EULA](https://www.eos-solutions.it/en/eula.html).

---

**Document Version**: 1.0.0  
**Last Updated**: February 16, 2026  
**API Specification**: OpenAPI 3.2.0
