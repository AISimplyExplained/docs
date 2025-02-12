# Bank Statement & Invoice Parser API

A high-performance API service for parsing and analyzing bank statements and invoices using AI. Built with Go, this service provides secure, scalable, and rate-limited endpoints for document processing.

## 🚀 Features

- **Document Processing**
  - Bank statement parsing (PDF)
  - Invoice analysis (PDF, JPG, JPEG, PNG)
  - Multi-page document support
  - Batch processing capabilities

- **Security & Performance**
  - API key authentication via Keyfolio
  - Rate limiting and usage tracking
  - Request validation and sanitization
  - Parallel processing for batch requests

- **Storage & Queue**
  - Supabase storage integration
  - Async job processing
  - Job status tracking
  - Batch operation support

## 🔑 Authentication

All API endpoints (except `/`) require authentication using Bearer tokens:

```bash
curl -H "Authorization: Bearer your_api_key" ...
```

To get an API key:

1. Contact the administrator for a Keyfolio API key
2. Use the key in the Authorization header
3. Monitor your usage in the Keyfolio dashboard

## 📚 API Documentation

### File Upload

**POST** `/api/v1/parse/upload`
- Uploads files for processing
- Supports PDF, JPG, JPEG, PNG
- Max 50 files per request
- Max 10MB per file
- **Required**: `type` parameter must be either "statement" or "invoice"

Request:
```bash
# For bank statements
curl -X POST "https://api.invaro.ai/api/v1/parse/upload" \
  -H "Authorization: Bearer your_api_key" \
  -F "files=@statement.pdf" \
  -F "type=statement"

# For invoices
curl -X POST "https://api.invaro.ai/api/v1/parse/upload" \
  -H "Authorization: Bearer your_api_key" \
  -F "files=@invoice.pdf" \
  -F "type=invoice"
```

Response:
```json
{
  "success": true,
  "data": {
    "file_id": "1234567890_statement.pdf",
    "file_url": "https://storage.googleapis.com/bank-statements/1234567890_statement.pdf",
    "doc_id": "uuid-of-document",
    "status": "done"
  }
}
```

### Bank Statement Processing

**POST** `/api/v1/parse/statements`
- Analyzes bank statements
- Supports single or multiple files

Request (Option 1 - Simplified):
```json
{
  "document_id": "uuid-of-document"
}
```

Request (Option 2 - Legacy):
```json
{
  "file_id": "1234567890_statement.pdf",
  "document_id": "uuid-of-document"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "job_id": "job_123",
    "status": "pending",
    "created_at": "2024-01-07T10:00:00Z",
    "updated_at": "2024-01-07T10:00:00Z"
  }
}
```

### Invoice Processing

**POST** `/api/v1/parse/invoices`
- Analyzes invoices using AI
- Extracts key information

Request (Option 1 - Simplified):
```json
{
  "document_id": "uuid-of-document"
}
```

Request (Option 2 - Legacy):
```json
{
  "file_id": "1234567890_invoice.pdf",
  "document_id": "uuid-of-document"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "job_id": "job_456",
    "status": "pending",
    "created_at": "2024-01-07T10:00:00Z",
    "updated_at": "2024-01-07T10:00:00Z"
  }
}
```

### Job Status

**GET** `/api/v1/parse/statements/{job_id}` or `/api/v1/parse/invoices/{job_id}`
- Checks processing status
- Returns results when complete

Example:
```bash
curl -X GET "https://api.invaro.ai/api/v1/parse/statements/job_1234567890" \
  -H "Authorization: Bearer your_api_key"
```

Response:
```json
{
  "success": true,
  "data": {
    "job_id": "job_1234567890",
    "status": "completed",
    "file_id": "1234567890_statement.pdf",
    "created_at": "2024-01-07T10:00:00Z",
    "updated_at": "2024-01-07T10:05:00Z",
    "result": {
      // Parsed document data
    }
  }
}
```

### Batch Processing

**POST** `/api/v1/parse/statements/batch` or `/api/v1/parse/invoices/batch`
- Process multiple documents at once
- Maximum 10 documents per batch

Request:
```json
{
  "files": [
    {
      "document_id": "uuid-of-document-1"
    },
    {
      "document_id": "uuid-of-document-2"
    }
  ]
}
```

Response:
```json
{
  "success": true,
  "data": {
    "batch_id": "batch_123",
    "status": "pending",
    "job_ids": ["job_1", "job_2"],
    "created_at": "2024-01-07T10:00:00Z",
    "updated_at": "2024-01-07T10:00:00Z"
  }
}
```

### List Jobs

**GET** `/api/v1/parse/statements` or `/api/v1/parse/invoices`
- Lists processing jobs
- Supports pagination and filtering

Parameters:
- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)
- `status`: Filter by status (pending/processing/completed/failed)

Response:
```json
{
  "success": true,
  "data": {
    "jobs": [
      {
        "job_id": "job_123",
        "status": "completed",
        "created_at": "2024-01-07T10:00:00Z"
      }
    ],
    "total": 50,
    "page": 1,
    "page_size": 20
  }
}
```

## 🔒 Error Handling

All errors follow a standard format:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "details": "Additional error details"
  }
}
```

Common error codes:
- `UNAUTHORIZED`: Authentication issues
- `INVALID_REQUEST`: Malformed requests
- `RATE_LIMIT_EXCEEDED`: Rate limiting
- `VALIDATION_ERROR`: Request validation failures
- `INTERNAL_SERVER_ERROR`: Server errors

## 🔄 Rate Limiting

- Default: 60 requests per minute per API key
- Batch operations count as multiple requests
- Headers indicate remaining quota:
  - `X-RateLimit-Limit`
  - `X-RateLimit-Remaining`
  - `X-RateLimit-Reset`

## 📚 Complete API Flow Examples

### 1. Bank Statement Processing Flow

```bash
# 1. Upload bank statement
curl -X POST "https://api.invaro.ai/api/v1/parse/upload" \
  -H "Authorization: Bearer your_api_key" \
  -F "files=@statement.pdf" \
  -F "type=statement"

# Response:
{
  "success": true,
  "data": {
    "files": [
      {
        "file_id": "1737201629689465000_statement.pdf",
        "file_url": "https://storage.googleapis.com/bank-statements/1737201629689465000_statement.pdf",
        "doc_id": "879ba2d7-5ac7-4eea-aab7-cf4a5be01d1d"
      }
    ],
    "status": "done"
  }
}

# 2. Start processing
curl -X POST "https://api.invaro.ai/api/v1/parse/statements" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "file_id": "1737201629689465000_statement.pdf",
    "document_id": "879ba2d7-5ac7-4eea-aab7-cf4a5be01d1d"
  }'

# Response:
{
  "success": true,
  "data": {
    "job_id": "job_1737201642026659000",
    "status": "pending"
  }
}

# 3. Check status
curl -X GET "https://api.invaro.ai/api/v1/parse/statements/job_1737201642026659000" \
  -H "Authorization: Bearer your_api_key"

# Response:
{
  "success": true,
  "data": {
    "job_id": "job_1737201642026659000",
    "status": "completed",
    "file_id": "1737201629689465000_statement.pdf",
    "created_at": "2024-01-18T12:34:56Z",
    "updated_at": "2024-01-18T12:35:26Z",
    "result": {
      "account_no": "10103631549",
      "opening_balance": "1,700.31",
      "closing_balance": "11,468.10",
      "total_credit": "11,78,713.00",
      "total_debit": "11,68,945.21",
      "statement_period": "2023-11-30 TO 2024-11-29",
      "transactions": [
        {
          "date": "2024-01-01",
          "description": "SALARY CREDIT",
          "amount": 5000.00,
          "type": "credit",
          "balance": 5000.00
        }
      ]
    }
  }
}
```

### 2. Invoice Processing Flow

```bash
# 1. Upload multiple invoices
curl -X POST "https://api.invaro.ai/api/v1/parse/upload" \
  -H "Authorization: Bearer your_api_key" \
  -F "files=@invoice1.pdf" \
  -F "files=@invoice2.jpg" \
  -F "callback_url=https://your-domain.com/webhook"

# 2. Start batch processing
curl -X POST "https://api.invaro.ai/api/v1/parse/invoices/batch" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "file_ids": [
      "1234567890_invoice1.pdf",
      "1234567890_invoice2.jpg"
    ],
    "callback_url": "https://your-domain.com/webhook"
  }'

# 3. List all jobs
curl "https://api.invaro.ai/api/v1/parse/invoices?page=1&page_size=10&status=completed" \
  -H "Authorization: Bearer your_api_key"
```

### 3. Webhook Notifications

Your callback URL will receive POST requests with job updates:

```json
{
  "event": "job.completed",
  "job_id": "job_123",
  "job_type": "statement_parsing",
  "status": "completed",
  "created_at": "2024-01-07T10:00:00Z",
  "completed_at": "2024-01-07T10:05:00Z",
  "results": {
    // Parsed document data
  }
}
```

## 📡 Available Endpoints

### File Management
- `POST /api/v1/parse/upload` - Upload files
- `GET /api/v1/parse/files/{file_id}` - Get file info
- `DELETE /api/v1/parse/files/{file_id}` - Delete file

### Bank Statement Processing
- `POST /api/v1/parse/statements` - Process single statement
- `POST /api/v1/parse/statements/batch` - Process multiple statements
- `GET /api/v1/parse/statements/{job_id}` - Get job status
- `GET /api/v1/parse/statements` - List all jobs

### Invoice Processing
- `POST /api/v1/parse/invoices` - Process single invoice
- `POST /api/v1/parse/invoices/batch` - Process multiple invoices
- `GET /api/v1/parse/invoices/{job_id}` - Get job status
- `GET /api/v1/parse/invoices` - List all jobs

### Job Management
- `GET /api/v1/jobs` - List all jobs (statements and invoices)
- `DELETE /api/v1/jobs/{job_id}` - Cancel job
- `POST /api/v1/jobs/{job_id}/retry` - Retry failed job

## 🎯 Common Use Cases

### 1. Process Bank Statements in Bulk

```bash
# Upload multiple statements
for file in *.pdf; do
  curl -X POST "https://api.invaro.ai/api/v1/parse/upload" \
    -H "Authorization: Bearer your_api_key" \
    -F "files=@$file"
done

# Start batch processing (using document_ids from upload response)
curl -X POST "https://api.invaro.ai/api/v1/parse/statements/batch" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "files": [
      {"document_id": "doc-id-1"},
      {"document_id": "doc-id-2"}
    ],
    "callback_url": "https://your-domain.com/webhook",
    "options": {
      "extract_transactions": true,
      "categorize": true,
      "detect_anomalies": true
    }
  }'
```

### 2. Real-time Invoice Processing

```bash
# Upload and process immediately
curl -X POST "https://api.invaro.ai/api/v1/parse/invoices" \
  -H "Authorization: Bearer your_api_key" \
  -F "files=@invoice.pdf" \
  -F "process=true" \
  -F "callback_url=https://your-domain.com/webhook" \
  -F "options={\"extract_items\":true,\"detect_tax\":true}"

# Process using document_id
curl -X POST "https://api.invaro.ai/api/v1/parse/invoices" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "document_id": "uuid-from-upload",
    "callback_url": "https://your-domain.com/webhook"
  }'
```

### 3. Retry Failed Jobs

```bash
# List failed jobs
curl "https://api.invaro.ai/api/v1/jobs?status=failed" \
  -H "Authorization: Bearer your_api_key"

# Retry specific job
curl -X POST "https://api.invaro.ai/api/v1/jobs/job_123/retry" \
  -H "Authorization: Bearer your_api_key"
```

## 🔄 Callback URL Feature

The API supports webhook notifications for asynchronous job updates:

1. **Registration**: Add callback_url when starting a job:
   ```bash
   curl -X POST "https://api.invaro.ai/api/v1/parse/statements" \
     -H "Authorization: Bearer your_api_key" \
     -H "Content-Type: application/json" \
     -d '{
       "file_ids": ["file.pdf"],
       "callback_url": "https://your-domain.com/webhook"
     }'
   ```

2. **Events**: Your webhook will receive these events:
   - `job.created` - Job created
   - `job.started` - Processing started
   - `job.progress` - Processing progress update
   - `job.completed` - Processing completed
   - `job.failed` - Processing failed

3. **Security**: Webhooks include signature header:
   ```bash
   X-Signature: sha256=HMAC_SHA256(secret_key, body)
   ```

4. **Retry Policy**: Failed webhook deliveries are retried:
   - Initial retry after 5 seconds
   - Exponential backoff (max 1 hour)
   - Maximum 5 retry attempts

[Rest of the README remains the same] 