# API Documentation

## Overview

The Bank Statement & Invoice Parser API provides endpoints for processing financial documents. All endpoints use JSON for request/response bodies except for file uploads which use multipart/form-data.

## Base URL

```
https://your-api-domain.com/api/v1
```

## Authentication

All endpoints require Bearer token authentication:

```http
Authorization: Bearer your_api_key_here
```

## Endpoints

### 1. File Upload

Upload files for processing.

**Endpoint:** `POST /parse/upload`

**Content-Type:** `multipart/form-data`

**Request Parameters:**
- `files`: Array of files (PDF, JPG, JPEG, PNG)
  - Maximum 50 files
  - Maximum 10MB per file

**Example Request:**
```bash
curl -X POST "https://your-api-domain.com/api/v1/parse/upload" \
  -H "Authorization: Bearer your_api_key" \
  -F "files=@statement.pdf" \
  -F "files=@invoice.jpg"
```

**Success Response:**
```json
{
  "success": true,
  "data": {
    "files": [
      {
        "file_id": "1234567890_statement.pdf",
        "file_url": "https://storage.com/files/1234567890_statement.pdf",
        "error": null
      },
      {
        "file_id": "1234567890_invoice.jpg",
        "file_url": "https://storage.com/files/1234567890_invoice.jpg",
        "error": null
      }
    ],
    "status": "done"
  }
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid file type",
    "details": "Only PDF, JPG, JPEG, and PNG files are allowed"
  }
}
```

### 2. Bank Statement Processing

Process bank statements for data extraction.

**Endpoint:** `POST /parse/statements`

**Content-Type:** `application/json`

**Request Body:**
```json
{
  "file_ids": [
    "1234567890_statement.pdf"
  ]
}
```

**Success Response:**
```json
{
  "success": true,
  "data": {
    "job_id": "job_123abc",
    "status": "processing",
    "estimated_completion": "2024-01-07T10:05:00Z"
  }
}
```

### 3. Invoice Processing

Process invoices for data extraction.

**Endpoint:** `POST /parse/invoices`

**Content-Type:** `application/json`

**Request Body:**
```json
{
  "file_ids": [
    "1234567890_invoice.jpg"
  ]
}
```

**Success Response:**
```json
{
  "success": true,
  "data": {
    "job_id": "job_456def",
    "status": "processing",
    "estimated_completion": "2024-01-07T10:05:00Z"
  }
}
```

### 4. Check Job Status

Check the status of a processing job.

**Endpoint:** `GET /parse/statements/{job_id}` or `/parse/invoices/{job_id}`

**Success Response (Processing):**
```json
{
  "success": true,
  "data": {
    "job_id": "job_123abc",
    "status": "processing",
    "progress": 45,
    "estimated_completion": "2024-01-07T10:05:00Z"
  }
}
```

**Success Response (Completed):**
```json
{
  "success": true,
  "data": {
    "job_id": "job_123abc",
    "status": "completed",
    "results": {
      "account_number": "1234567890",
      "bank_name": "Example Bank",
      "transactions": [
        {
          "date": "2024-01-01",
          "description": "Payment",
          "amount": 100.50,
          "type": "credit"
        }
      ],
      "confidence_score": 0.95
    }
  }
}
```

### 5. List Jobs

List all processing jobs with pagination.

**Endpoint:** `GET /parse/statements` or `/parse/invoices`

**Query Parameters:**
- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)
- `status`: Filter by status (pending/processing/completed/failed)

**Example Request:**
```bash
curl "https://your-api-domain.com/api/v1/parse/statements?page=1&page_size=20&status=completed" \
  -H "Authorization: Bearer your_api_key"
```

**Success Response:**
```json
{
  "success": true,
  "data": {
    "jobs": [
      {
        "job_id": "job_123abc",
        "status": "completed",
        "created_at": "2024-01-07T10:00:00Z",
        "completed_at": "2024-01-07T10:05:00Z",
        "file_count": 1
      }
    ],
    "pagination": {
      "total": 50,
      "page": 1,
      "page_size": 20,
      "total_pages": 3
    }
  }
}
```

## Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| UNAUTHORIZED | Authentication failed | 401 |
| INVALID_REQUEST | Malformed request | 400 |
| RATE_LIMIT_EXCEEDED | Too many requests | 429 |
| VALIDATION_ERROR | Request validation failed | 400 |
| INTERNAL_SERVER_ERROR | Server error | 500 |

## Rate Limiting

Rate limits are enforced per API key:

- Default limit: 60 requests per minute
- Batch operations count as multiple requests
- Rate limit headers:
  ```
  X-RateLimit-Limit: 60
  X-RateLimit-Remaining: 45
  X-RateLimit-Reset: 1704624000
  ```

## Webhooks

You can configure webhooks to receive job completion notifications:

1. Register webhook URL in Keyfolio dashboard
2. Receive POST requests with job results:
```json
{
  "event": "job.completed",
  "job_id": "job_123abc",
  "status": "completed",
  "results": {
    // Job results
  }
}
```

## Best Practices

1. **Handling Rate Limits:**
   - Implement exponential backoff
   - Cache responses when possible
   - Use batch endpoints for multiple files

2. **Error Handling:**
   - Always check the `success` field
   - Handle rate limits gracefully
   - Implement retry logic for 5xx errors

3. **File Uploads:**
   - Compress images before upload
   - Use batch uploads for multiple files
   - Verify file checksums

4. **Job Status:**
   - Poll status with increasing intervals
   - Use webhooks for real-time updates
   - Handle timeout scenarios 