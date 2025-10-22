# HTTP

## API Endpoints

### `POST /topics/{topicName}`

> Produces records to the specified topic 

**Request Format:**
The request body must contain a `records` array, where each record has `key` and `value` fields (both as JSON).

**Response Format:**
Returns partition offsets for each produced record, along with optional error information.
