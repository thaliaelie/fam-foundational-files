---
description: Apply when the user's request involves sending data to, querying, or integrating with the CloudZero API (e.g., telemetry, allocation, views, billing, AnyCost, CostFormation endpoints).
---

# CloudZero API Reference

## Local API Spec Files

Full OpenAPI/Swagger specifications are available locally:
- **V2 API**: `api.cloudzero.com/documentation/v2/openapi.yaml`
- **V1 Telemetry API**: `feature-unit-cost/swagger.yaml`
- **Telemetry Examples**: `cloudzero-telemetry-library/snowflake_queries/`

---

## CloudZero API V2

Base URL: `https://api.cloudzero.com`

### Authentication
All requests require an API key in the `Authorization` header (not Bearer token):
```
Authorization: <cloudzero-api-key>
```

### Endpoints Overview

| Resource | Endpoints | Description |
|----------|-----------|-------------|
| **Budgets** | `/v2/budgets`, `/v2/budgets/{budget_id}` | Create, read, update, delete budgets |
| **CostFormation** | `/v2/costformation/definition/versions`, `/v2/costformation/definition/versions/{version}` | Manage cost definition versions |
| **Insights** | `/v2/insights`, `/v2/insights/{insight_id}`, `/v2/insights/{insight_id}/comments` | Manage insights and comments |
| **Views** | `/v2/views`, `/v2/views/{view_id}` | Custom view management |
| **Billing** | `/v2/billing/costs`, `/v2/billing/dimensions` | Cost and dimension retrieval |
| **Billing Connections** | `/v2/connections/billing`, `/v2/connections/billing/{connection_id}` | Connection CRUD |
| **AnyCost** | `/v2/connections/billing/anycost/{connection_id}/billing_drops` | AnyCost data ingestion |
| **Recommendations** | `/v2/optimize/recommendations`, `/v2/optimize/recommendations/{recommendation_id}` | Optimization recommendations |
| **Comments** | `/v2/optimize/comments`, `/v2/optimize/comments/{comment_id}` | Comments on recommendations |

### Common Response Codes
- `200`: Success
- `400`: Invalid request
- `403`: Not authorized
- `422`: Validation error

### Views API

Full CRUD support for managing Views (anomaly detection, cost visibility).

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v2/views` | List all views |
| POST | `/v2/views` | Create a new view |
| GET | `/v2/views/{view_id}` | Get a single view |
| PATCH | `/v2/views/{view_id}` | Update a view |
| DELETE | `/v2/views/{view_id}` | Delete a view |

**Create View Request Body:**
```json
{
  "name": "My Account View",
  "filter": {
    "User:Defined:AccountName": ["Account Value 1", "Account Value 2"]
  },
  "principal_dimension": "User:Defined:AccountName",
  "connections": {
    "email": {
      "addresses": ["alerts@example.com"],
      "include_all_organizers": false
    },
    "slack": [{"id": "C01234567", "name": "#alerts-channel"}]
  },
  "anomalies": {
    "enabled": true,
    "threshold_type": "Automatic",
    "threshold_value": 50
  }
}
```

**Required Fields:** `name`, `filter`, `principal_dimension`, `connections`

**Anomaly Threshold Types:**
- `Automatic` - CloudZero calculates threshold based on view's cost (threshold_value ignored)
- `Percent` - Uses threshold_value as percentage for day-over-day spike detection

**Note:** Views do not support filtering by `Resource` or `Resource Summary`.

### Billing Dimensions API

Use to discover valid dimension IDs for filters and principal_dimension:

```bash
GET /v2/billing/dimensions
```

Returns all available dimensions with `id` and `name`. Example response:
```json
{
  "dimensions": [
    {"id": "Account", "name": "Account"},
    {"id": "Service", "name": "Service"},
    {"id": "User:Defined:AccountName", "name": "Account Name"},
    {"id": "Tag:Environment", "name": "Environment"}
  ]
}
```

---

## CloudZero Telemetry API V1

Base URL: `https://api.cloudzero.com`

### Rate Limits
- **Send Rate**: Up to 100 records/second
- **Burst Capacity**: 8,640,000 records max (one day's worth)
- **Request Size**: 5MB max (~10,000 records at 512 bytes each)

### Allocation Telemetry Endpoints

Used to split cloud costs via custom allocation dimensions.

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/unit-cost/v1/telemetry/allocation/{stream_name}` | Post records (legacy, same as /sum) |
| POST | `/unit-cost/v1/telemetry/allocation/{stream_name}/sum` | Sum records to existing data |
| POST | `/unit-cost/v1/telemetry/allocation/{stream_name}/replace` | Replace matching records |
| POST | `/unit-cost/v1/telemetry/allocation/{stream_name}/delete` | Delete matching records |

**Allocation Record Schema:**
```json
{
  "records": [
    {
      "value": 16.4,
      "timestamp": "2020-03-03T15:32:44+00:00",
      "granularity": "HOURLY",
      "element_name": "CustomerName",
      "filter": {
        "custom:My Dimension Display Name": ["GroupValue"]
      }
    }
  ]
}
```

- `value`: Number > 0 (usage amount, used as proportional weight)
- `timestamp`: ISO 8601 format
- `granularity`: `HOURLY`, `DAILY`, or `MONTHLY`
- `element_name`: Attribution target (customer, tenant, etc.)
- `filter`: Dimension mapping (`"*"` or `{}` applies to all spend)

#### CRITICAL: Filter Key Format

**The `filter` key for custom dimensions MUST use the dimension's display `Name` from CostFormation, NOT the YAML key.**

```yaml
# CostFormation definition:
MyDimensionYamlKey:                    # ← YAML key (DO NOT USE in filter)
    Name: My Dimension Display Name    # ← Display Name (USE THIS in filter)
```

```json
// CORRECT:
"filter": {"custom:My Dimension Display Name": ["GroupValue"]}

// WRONG (silently fails — records accepted by API but no costs allocated):
"filter": {"custom:MyDimensionYamlKey": ["GroupValue"]}
```

**How to find the correct name:** Look at the CostFormation YAML — every dimension has a `Name:` field. Use that exact string after `custom:`. If you have the CostFormation file, search for the dimension's YAML key and read its `Name:` property.

**Filter key prefixes:**
- `custom:<Display Name>` — CostFormation-defined dimensions
- `tag:<tag_name>` — AWS resource tags
- `accounts` — AWS account IDs
- `services` — AWS service names

#### Endpoint Selection

| Endpoint | Use When |
|----------|----------|
| `/sum` | Adding new records (additive — re-uploads double-count) |
| `/replace` | Automated/repeatable uploads (idempotent — safe to re-run) |
| `/delete` | Removing records for a specific period |

**For automated Lambda/scheduled processes, prefer `/replace`** to ensure idempotency.

### Building an Allocation Telemetry Automation

When building an automated process to send allocation telemetry:

#### 1. Understand the CostFormation Chain

Allocation streams connect through a dimension chain:
```
Source Dimension (groups resources by tag/attribute)
    → Allocation Dimension (Type: Allocation, AllocateByStreams)
        → Final Dimension (GroupBy with CoalesceSources: true)
```

Read the customer's CostFormation to trace this chain before writing any code. Identify:
- Which dimension groups the shared resources (and what `Source` tag it uses)
- Which telemetry stream feeds the allocation dimension
- Which final dimension surfaces the allocated values to users

#### 2. Verify Resource Identity

**Always confirm what the CostFormation's `Source` tag actually maps to in the customer's infrastructure.** Tag values, physical hostnames, logical instance names, and display names can all be different identifiers.

**Ask the customer**: "Which column in your data corresponds to the value in `Tag:<tag_name>` used by the CostFormation dimension?"

Common pitfalls:
- Physical hostnames vs logical service names (e.g., cluster node vs SQL instance)
- The tag on the resource may not match what the customer calls the resource
- Multiple resources may share similar naming patterns but represent different things

#### 3. Cross-Reference Dimension Values

Before building the telemetry, validate that:
- The filter values you'll send actually exist as dimension values in CloudZero
- The dimension values match the column you're using from the source data
- There aren't split groups (same logical resource appearing under multiple dimension values with separate costs)

Use the CloudZero MCP tools or API to query existing dimension values and compare against the source data.

#### 4. Validate Before and After

**Before sending:** Query the telemetry stream for the target month — should be empty
```
get_telemetry_data(stream, date_range, group_by=[usage_date, element_name, dimension])
```

**After sending:** Same query — should show your records with correct values

**1-2 hours later:** Query cost data grouped by the final customer dimension to verify allocation flowed through the CostFormation chain

### Unit Metric Telemetry Endpoints

Used for unit cost analytics (cost per request, per customer, etc.)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/unit-cost/v1/telemetry/metric/{metric_name}` | Get recent records |
| POST | `/unit-cost/v1/telemetry/metric/{metric_name}` | Post records (legacy) |
| POST | `/unit-cost/v1/telemetry/metric/{metric_name}/sum` | Sum records |
| POST | `/unit-cost/v1/telemetry/metric/{metric_name}/replace` | Replace records |
| POST | `/unit-cost/v1/telemetry/metric/{metric_name}/delete` | Delete records |

**Metric Record Schema:**
```json
{
  "records": [
    {
      "value": 1500,
      "timestamp": "2020-03-03T15:32:44+00:00",
      "granularity": "DAILY",
      "associated_cost": {
        "custom:Product": "WidgetAPI"
      }
    }
  ]
}
```

- `granularity`: `DAILY` or `MONTHLY` only (no HOURLY)
- `associated_cost`: Optional dimension filter for cost correlation

### Stream Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/unit-cost/v1/telemetry/{stream_name}/records` | Get processed records (hours delay) |
| DELETE | `/unit-cost/v1/telemetry/{stream_name}` | Delete entire stream |

## Documentation Links

- [API Reference](https://docs.cloudzero.com/reference/introduction)
- [Telemetry API](https://docs.cloudzero.com/reference/telemetry)
