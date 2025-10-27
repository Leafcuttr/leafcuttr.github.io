# HTTP

## API Endpoints

### `POST /topics/{topicName}`

> Produces records to the specified topic 

**Request Format:**
The request body must contain a `records` array, where each record has `key` and `value` fields (both as JSON).

**Response Format:**
Returns partition offsets for each produced record, along with optional error information.

#### Example Request and Response

The following will post two records to the `jsontest` topic.

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"records":[{"key":"someKeyText", "value":{"name": "testUser"}}, {"key":"someKeyText", "value":{"name": "testUser2"}}]}' \
  http://localhost:8080/topics/jsontest | jq
```

Sample response:
```json
{
  "offsets": [
    {
      "partition": 0,
      "offset": 23,
      "error_code": null,
      "error": null
    },
    {
      "partition": 0,
      "offset": 24,
      "error_code": null,
      "error": null
    }
  ],
  "value_schema_id": null,
  "key_schema_id": null
}
```