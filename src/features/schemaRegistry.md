# Schema Registry

## Overview

The schema-service module provides a Schema Registry implementation compatible with the Confluent Schema Registry API. It runs as an embedded HTTP server (default port 8081) and provides REST endpoints for managing schemas, subjects, versions, configurations, and compatibility checking.

The service uses Kafka as its storage backend (via a `_schemas` topic) and supports schema validation and compatibility checking, primarily for Apache Avro schemas.

## Configuration

**`lc.schema.registry.enable`**
- **Type:** Boolean
- **Default:** `false`
- **Description:** Controls whether the embedded Schema Registry is enabled

**`lc.schema.registry.port`**
- **Type:** Int
- **Default:** 8081
- **Description:** Port that the Schema Registry's HTTP server listens to

## Implicit Configurations

Currently, the following are hard-coded:

- **Default Kafka Bootstrap**: Advertised listeners in Kafka config
- **Storage Topic**: `_schemas`
- **Content-Type**: `application/vnd.schemaregistry.v1+json` or `application/json`

## API Endpoints

### Root Endpoint

#### GET /
Returns an empty JSON object to verify the service is running.

**Response**: `200 OK`
```json
{}
```

---

### Schema Endpoints

#### GET /schemas/ids/{id}
Retrieves a schema by its global ID.

**Path Parameters**:
- `id` (integer): The schema ID

**Response**: `200 OK`
```json
{
  "schema": "<schema_string>"
}
```

**Error Responses**:
- `404 Not Found` (40403): Schema not found

---

#### GET /schemas/ids/{id}/schema
Retrieves the raw schema string by its global ID.

**Path Parameters**:
- `id` (integer): The schema ID

**Response**: `200 OK`
- Returns the raw schema string (not wrapped in JSON)

**Error Responses**:
- `404 Not Found` (40403): Schema not found

---

#### GET /schemas/types
Lists all supported schema types.

**Response**: `200 OK`
```json
["AVRO", "JSON", "PROTOBUF"]
```

---

### Subject Endpoints

#### GET /subjects
Lists all registered subjects.

**Query Parameters**:
- `deleted` (boolean, optional): Include soft-deleted subjects (default: false)

**Response**: `200 OK`
```json
["subject1", "subject2", "subject3"]
```

---

#### GET /subjects/{subject}/versions
Lists all version numbers for a subject.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Response**: `200 OK`
```json
[1, 2, 3]
```

**Error Responses**:
- `404 Not Found` (40401): Subject not found

---

#### GET /subjects/{subject}/versions/{version}
Retrieves a specific schema version for a subject.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)
- `version` (string): Version number, "latest", or "-1" for latest

**Response**: `200 OK`
```json
{
  "subject": "example-subject",
  "id": 123,
  "version": 1,
  "schema": "<schema_string>",
  "schemaType": "AVRO"
}
```

Note: `schemaType` field is only included for non-AVRO schemas.

**Error Responses**:
- `404 Not Found` (40401): Subject not found
- `404 Not Found` (40402): Version not found
- `422 Unprocessable Entity` (42202): Invalid version

---

#### GET /subjects/{subject}/versions/{version}/schema
Retrieves the raw schema string for a specific version.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)
- `version` (string): Version number, "latest", or "-1" for latest

**Response**: `200 OK`
- Returns the raw schema string (not wrapped in JSON)

**Error Responses**:
- `404 Not Found` (40401): Subject not found
- `404 Not Found` (40402): Version not found

---

#### POST /subjects/{subject}/versions
Registers a new schema version for a subject.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Request Body**:
```json
{
  "schema": "<schema_string>",
  "schemaType": "AVRO",
  "references": []
}
```

Fields:
- `schema` (string, required): The schema definition
- `schemaType` (string, optional): Schema type (default: "AVRO")
- `references` (array, optional): Schema references

**Response**: `200 OK`
```json
{
  "id": 123
}
```

Returns the existing ID if the schema is identical to the latest version (idempotent operation).

**Error Responses**:
- `409 Conflict`: Schema is incompatible with the latest schema
- `422 Unprocessable Entity` (42201): Invalid or empty schema

---

#### POST /subjects/{subject}
Checks if a schema exists under the subject and returns its version information.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Request Body**:
```json
{
  "schema": "<schema_string>",
  "schemaType": "AVRO"
}
```

**Response**: `200 OK`
```json
{
  "subject": "example-subject",
  "id": 123,
  "version": 1,
  "schema": "<schema_string>",
  "schemaType": null
}
```

**Error Responses**:
- `404 Not Found` (40401): Subject not found
- `404 Not Found` (40403): Schema not found
- `422 Unprocessable Entity` (42201): Invalid schema

---

#### DELETE /subjects/{subject}
Deletes a subject and all its versions.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Query Parameters**:
- `permanent` (boolean, optional): Perform hard delete with tombstones (default: false for soft delete)

**Response**: `200 OK`
```json
[1, 2, 3]
```

Returns the list of deleted version numbers.

**Error Responses**:
- `404 Not Found` (40401): Subject not found

**Notes**:
- Soft delete marks versions as deleted but they remain accessible by ID
- Hard delete writes tombstones to permanently remove the subject

---

#### DELETE /subjects/{subject}/versions/{version}
Deletes a specific version of a subject.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)
- `version` (string): Version number, "latest", or "-1" for latest

**Query Parameters**:
- `permanent` (boolean, optional): Perform hard delete with tombstone (default: false for soft delete)

**Response**: `200 OK`
```json
1
```

Returns the deleted version number.

**Error Responses**:
- `404 Not Found` (40401): Subject not found
- `404 Not Found` (40402): Version not found
- `422 Unprocessable Entity` (42202): Invalid version

---

### Configuration Endpoints

#### GET /config
Retrieves the global compatibility level configuration.

**Response**: `200 OK`
```json
{
  "compatibilityLevel": "BACKWARD"
}
```

---

#### PUT /config
Updates the global compatibility level configuration.

**Request Body**:
```json
{
  "compatibility": "BACKWARD"
}
```

Valid compatibility levels:
- `BACKWARD`
- `BACKWARD_TRANSITIVE`
- `FORWARD`
- `FORWARD_TRANSITIVE`
- `FULL`
- `FULL_TRANSITIVE`
- `NONE`

**Response**: `200 OK`
```json
{
  "compatibility": "BACKWARD"
}
```

**Error Responses**:
- `422 Unprocessable Entity` (42203): Invalid compatibility level

---

#### GET /config/{subject}
Retrieves the compatibility level for a specific subject.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Query Parameters**:
- `defaultToGlobal` (boolean, optional): Return global config if subject has no specific config (default: false)

**Response**: `200 OK`
```json
{
  "compatibilityLevel": "BACKWARD"
}
```

**Error Responses**:
- `404 Not Found` (40401): Subject not found or no specific config exists (when defaultToGlobal=false)

---

#### PUT /config/{subject}
Updates the compatibility level for a specific subject.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Request Body**:
```json
{
  "compatibility": "FORWARD"
}
```

**Response**: `200 OK`
```json
{
  "compatibility": "FORWARD"
}
```

**Error Responses**:
- `422 Unprocessable Entity` (42203): Invalid compatibility level

---

#### DELETE /config/{subject}
Deletes the subject-specific compatibility configuration, reverting to global default.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Response**: `200 OK`
```json
{
  "compatibilityLevel": "BACKWARD"
}
```

Returns the compatibility level that was deleted (before falling back to global).

**Error Responses**:
- `404 Not Found` (40401): Subject config not found

---

### Compatibility Check Endpoints

#### POST /compatibility/subjects/{subject}/versions/{version}
Tests if a schema is compatible with a specific version.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)
- `version` (string): Version number, "latest", or "-1" for latest

**Request Body**:
```json
{
  "schema": "<schema_string>",
  "schemaType": "AVRO"
}
```

**Response**: `200 OK`
```json
{
  "is_compatible": true
}
```

**Error Responses**:
- `404 Not Found` (40401): Subject not found
- `404 Not Found` (40402): Version not found
- `422 Unprocessable Entity` (42201): Invalid schema

---

#### POST /compatibility/subjects/{subject}/versions
Tests if a schema is compatible with the subject based on its compatibility level.

**Path Parameters**:
- `subject` (string): The subject name (URL-encoded)

**Request Body**:
```json
{
  "schema": "<schema_string>",
  "schemaType": "AVRO"
}
```

**Response**: `200 OK`
```json
{
  "is_compatible": true
}
```

Returns `true` if no existing schemas exist (compatible by default).

**Error Responses**:
- `422 Unprocessable Entity` (42201): Invalid schema

**Notes**:
- This endpoint checks compatibility against relevant versions based on the configured compatibility level
- For BACKWARD, checks against the latest version
- For TRANSITIVE modes, checks against all relevant historical versions

---

### Internal/Unstable Endpoints

#### POST /restart
Restarts the schema registry service (clears state and reinitializes).

**Response**: `200 OK`
```json
{
  "success": true
}
```

**Error Responses**:
- `404 Not Found` (40401): Endpoint disabled (requires `unstable.api.versions.enable` in config)
- `500 Internal Server Error` (50001): Error during restart

**Notes**:
- This is an internal API endpoint
- Must be explicitly enabled via configuration (`allowRestarts` flag)
- Shuts down the service, clears all in-memory state, and reinitializes

---

## Error Response Format

All error responses follow this format:

```json
{
  "error_code": 40401,
  "message": "Subject not found"
}
```

### Common Error Codes

- `400`: Bad Request - Invalid JSON or request format
- `404`: Not Found - Resource not found
- `40401`: Subject not found
- `40402`: Version not found
- `40403`: Schema not found
- `409`: Conflict - Schema incompatible
- `415`: Unsupported Media Type - Invalid Content-Type
- `422`: Unprocessable Entity - Invalid schema or parameters
- `42201`: Invalid schema
- `42202`: Invalid version
- `42203`: Invalid compatibility level
- `500`: Internal Server Error
- `50001`: Internal server error with details

---

## Data Models

### SchemaRequest
```json
{
  "schema": "string (required)",
  "schemaType": "string (optional, default: AVRO)",
  "references": "array (optional)"
}
```

### SchemaResponse
```json
{
  "schema": "string"
}
```

### SchemaVersionResponse
```json
{
  "subject": "string",
  "id": "integer",
  "version": "integer",
  "schema": "string",
  "schemaType": "string (optional, omitted for AVRO)"
}
```

### IdResponse
```json
{
  "id": "integer"
}
```

### CompatibilityResponse
```json
{
  "is_compatible": "boolean"
}
```

### ErrorResponse
```json
{
  "error_code": "integer",
  "message": "string"
}
```

---

## Implementation Details

### Storage Backend
- Uses Kafka topic `_schemas` for persistent storage
- All state changes are written to Kafka and replayed on startup
- In-memory cache is maintained for fast reads
- Consumer group processes updates asynchronously

### Schema Validation
- Primary support for Apache Avro schemas
- Extensible architecture for JSON and Protobuf (declared but may not be fully implemented)
- Uses `AvroSchemaValidator` for parsing and validation

### Compatibility Checking
- Uses `AvroCompatibilityChecker` for compatibility validation
- Supports all standard compatibility modes (BACKWARD, FORWARD, FULL, NONE, and TRANSITIVE variants)
- Compatibility checks are performed before schema registration

### URL Encoding
- Subject names in URLs must be URL-encoded
- The service automatically decodes URL-encoded path parameters
