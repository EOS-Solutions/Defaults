# SCM Sales Forecasting API - Quick Reference

## Base URL
**SaaS**: `https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/eosSolutions/scmDataAPI/v1.0`  
**On-Premise**: `http://localhost:7048/{instance}/api/eosSolutions/scmDataAPI/v1.0`

## Authentication
- **OAuth 2.0**: Bearer token (recommended for SaaS)
- **Basic Auth**: Username + Web Service Access Key (on-premise)

---

## API Endpoints Summary

| Endpoint | Methods | Access | Description |
|----------|---------|--------|-------------|
| `/scmItems` | GET | Read-Only | Item master data |
| `/scmDataEntries` | GET, POST, PATCH, DELETE | Full CRUD | Staging data entries for forecasting |
| `/scmItemLedgerEntries` | GET | Read-Only | Historical item transactions |
| `/scmBaseCalendars` | GET | Read-Only | Calendar working/non-working days |
| `/scmManufSetup` | GET | Read-Only | Manufacturing & inventory setup |
| `/scmROPJnlLines` | GET, POST, PATCH, DELETE | Full CRUD | Reorder Point calculation journal |

---

## Common OData Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `$filter` | Filter results | `$filter=itemNo eq '1000'` |
| `$top` | Limit results | `$top=20` |
| `$skip` | Skip results (pagination) | `$skip=40` |
| `$select` | Select specific fields | `$select=itemNo,quantity` |
| `$expand` | Expand related entities | `$expand=item` |

---

## Quick Examples

### Get Items
```http
GET /scmItems?$filter=itemCategoryCode eq 'BIKE'&$top=10
Authorization: Bearer {token}
```

### Create Data Entry
```http
POST /scmDataEntries
Content-Type: application/json
Authorization: Bearer {token}

{
  "itemNo": "1000",
  "postingDate": "2026-02-15",
  "quantity": 100,
  "locationCode": "MAIN"
}
```

### Update Data Entry
```http
PATCH /scmDataEntries({id})
If-Match: {etag}
Content-Type: application/json
Authorization: Bearer {token}

{
  "quantity": 150,
  "excluded": true
}
```

### Get Item Ledger Entries
```http
GET /scmItemLedgerEntries?$filter=itemNo eq '1000' and postingDate ge 2026-01-01
Authorization: Bearer {token}
```

### Create ROP Journal Line
```http
POST /scmROPJnlLines
Content-Type: application/json
Authorization: Bearer {token}

{
  "journalTemplateName": "ROP",
  "journalBatchName": "DEFAULT",
  "lineNo": 10000,
  "itemNo": "1000",
  "avgDemand": 50,
  "newSafetyStockValue": 100
}
```

---

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No Content (Delete successful) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 404 | Not Found |
| 412 | Precondition Failed (ETag mismatch) |

---

## Key Entity Fields

### scmItem
- `no` - Item number
- `description` - Item description
- `itemCategoryCode` - Category code
- `baseUnitOfMeasure` - Base UOM

### scmDataEntry
- `itemNo` - Item number
- `postingDate` - Posting date
- `quantity` - Quantity
- `entryType` - Entry type
- `excluded` - Excluded from calculations
- `fixed` - Fixed entry
- `unusual` - Unusual demand

### scmROPJnlLine
- `itemNo` - Item number
- `avgDemand` - Average demand
- `avgLeadTime` - Average lead time
- `newSafetyStockValue` - New safety stock
- `newReorderPointValue` - New reorder point
- `updateSafetyStock` - Update flag

---

## Important Notes

1. **Concurrency**: Use ETags (If-Match header) for updates/deletes
2. **Pagination**: Use `$top` and `$skip` for large datasets
3. **Filtering**: Always filter to reduce payload size
4. **Rate Limits**: Be aware of Business Central API rate limits
5. **Security**: Protect access tokens and credentials

---

## Resources

- **Full Documentation**: [API-Documentation.md](./API-Documentation.md)
- **OpenAPI Spec**: [openapi-specification.yaml](./openapi-specification.yaml)
- **Publisher**: EOS Solutions - https://www.eos-solutions.it/
- **Support**: support@eos-solutions.it

---

**API Version**: v1.0  
**Last Updated**: February 16, 2026
