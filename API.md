# API Documentation

## Overview

The Web Value Task Flow API provides programmatic access to task management functionality. This RESTful API uses JSON for requests and responses, and requires authentication via API tokens.

## Base URL

```
https://api.webvalue.com/v1
```

## Authentication

All API requests require an API token in the Authorization header:

```http
Authorization: Bearer your-api-token
```

To obtain an API token:
1. Log in to your account
2. Go to Settings > API Access
3. Generate a new token

## Rate Limiting

- 100 requests per minute per IP
- 1000 requests per hour per token
- Status 429 when exceeded

## Endpoints

### Authentication

#### Login

```http
POST /auth/login
```

Request:
```json
{
    "email": "user@example.com",
    "password": "secure_password"
}
```

Response:
```json
{
    "token": "your-api-token",
    "expires_at": "2024-12-31T23:59:59Z",
    "user": {
        "id": 1,
        "name": "John Doe",
        "role": "admin"
    }
}
```

### Tasks

#### List Tasks

```http
GET /tasks
```

Parameters:
- `status` (string): Filter by status
- `priority` (string): Filter by priority
- `page` (integer): Page number
- `per_page` (integer): Items per page

Response:
```json
{
    "data": [
        {
            "id": 1,
            "title": "Task Title",
            "description": "Task Description",
            "status": "pending",
            "priority": "high",
            "due_date": "2024-01-31T23:59:59Z",
            "created_at": "2024-01-01T12:00:00Z",
            "creator": {
                "id": 1,
                "name": "John Doe"
            }
        }
    ],
    "meta": {
        "current_page": 1,
        "per_page": 10,
        "total": 100
    }
}
```

#### Create Task

```http
POST /tasks
```

Request:
```json
{
    "title": "New Task",
    "description": "Task Description",
    "priority": "high",
    "due_date": "2024-01-31T23:59:59Z",
    "assignees": [1, 2, 3]
}
```

Response:
```json
{
    "id": 2,
    "title": "New Task",
    "status": "pending",
    "created_at": "2024-01-01T12:00:00Z"
}
```

#### Update Task

```http
PUT /tasks/{id}
```

Request:
```json
{
    "status": "completed",
    "comment": "Task completed successfully"
}
```

#### Delete Task

```http
DELETE /tasks/{id}
```

### Comments

#### List Comments

```http
GET /tasks/{id}/comments
```

Response:
```json
{
    "data": [
        {
            "id": 1,
            "content": "Comment text",
            "user": {
                "id": 1,
                "name": "John Doe"
            },
            "created_at": "2024-01-01T12:00:00Z"
        }
    ]
}
```

#### Add Comment

```http
POST /tasks/{id}/comments
```

Request:
```json
{
    "content": "New comment"
}
```

### Files

#### Upload File

```http
POST /tasks/{id}/files
```

Request:
```http
Content-Type: multipart/form-data

file: (binary)
```

Response:
```json
{
    "id": 1,
    "filename": "document.pdf",
    "size": 1048576,
    "url": "https://files.webvalue.com/..."
}
```

#### Download File

```http
GET /files/{id}
```

### Users

#### List Users

```http
GET /users
```

Response:
```json
{
    "data": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com",
            "role": "admin"
        }
    ]
}
```

## Error Handling

### Error Codes

- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Validation Error
- 429: Rate Limit Exceeded
- 500: Server Error

### Error Response

```json
{
    "error": {
        "code": "validation_error",
        "message": "The given data was invalid",
        "details": {
            "title": ["The title field is required"]
        }
    }
}
```

## Webhooks

### Available Events

- task.created
- task.updated
- task.completed
- comment.created
- file.uploaded

### Webhook Format

```json
{
    "event": "task.created",
    "timestamp": "2024-01-01T12:00:00Z",
    "data": {
        "task": {
            "id": 1,
            "title": "Task Title"
        }
    }
}
```

## SDK Examples

### PHP

```php
$client = new WebValueAPI('your-api-token');
$tasks = $client->tasks->list(['status' => 'pending']);
```

### JavaScript

```javascript
const api = new WebValueAPI('your-api-token');
const tasks = await api.tasks.list({ status: 'pending' });
```

### Python

```python
client = WebValueAPI('your-api-token')
tasks = client.tasks.list(status='pending')
```

## Best Practices

1. Use pagination for large datasets
2. Cache responses when appropriate
3. Handle rate limiting gracefully
4. Implement retry logic
5. Validate webhook signatures
6. Use HTTPS for all requests

## Support

- Email: api-support@webvalue.com
- Documentation: https://docs.webvalue.com/api
- Status: https://status.webvalue.com

## Changelog

### v1.3.0
- Added webhook support
- Enhanced error responses
- Added file management endpoints

### v1.2.0
- Added comment management
- Improved pagination
- Added rate limiting

### v1.1.0
- Added user management
- Enhanced authentication
- Added task filtering

## Future Updates

Planned features for upcoming releases:
- GraphQL support
- Real-time updates via WebSocket
- Bulk operations
- Advanced search capabilities
