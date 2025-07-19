# API Documentation

This document provides comprehensive documentation for all public APIs in RepJC1.

## Table of Contents

- [Authentication](#authentication)
- [Core APIs](#core-apis)
- [Data APIs](#data-apis)
- [Utility APIs](#utility-apis)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Response Formats](#response-formats)

## Authentication

All API requests require authentication using API keys or tokens.

### API Key Authentication

```javascript
const apiKey = 'your-api-key';
const headers = {
  'Authorization': `Bearer ${apiKey}`,
  'Content-Type': 'application/json'
};
```

### Example Request

```javascript
fetch('/api/v1/data', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer your-api-key',
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

## Core APIs

### User Management API

#### GET /api/v1/users

Retrieves a list of users.

**Parameters:**
- `page` (optional): Page number for pagination (default: 1)
- `limit` (optional): Number of items per page (default: 10)
- `sort` (optional): Sort field (default: 'created_at')

**Response:**
```json
{
  "data": [
    {
      "id": "user_123",
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2023-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "pages": 10
  }
}
```

**Example Usage:**
```javascript
// Get first page of users
const users = await fetch('/api/v1/users?page=1&limit=10');

// Get users sorted by name
const sortedUsers = await fetch('/api/v1/users?sort=name');
```

#### POST /api/v1/users

Creates a new user.

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "data": {
    "id": "user_123",
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2023-01-01T00:00:00Z"
  }
}
```

**Example Usage:**
```javascript
const newUser = await fetch('/api/v1/users', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer your-api-key',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'John Doe',
    email: 'john@example.com',
    password: 'secure_password'
  })
});
```

#### PUT /api/v1/users/:id

Updates an existing user.

**Parameters:**
- `id`: User ID

**Request Body:**
```json
{
  "name": "Updated Name",
  "email": "updated@example.com"
}
```

**Example Usage:**
```javascript
const updatedUser = await fetch('/api/v1/users/user_123', {
  method: 'PUT',
  headers: {
    'Authorization': 'Bearer your-api-key',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Updated Name',
    email: 'updated@example.com'
  })
});
```

#### DELETE /api/v1/users/:id

Deletes a user.

**Parameters:**
- `id`: User ID

**Response:**
```json
{
  "message": "User deleted successfully"
}
```

**Example Usage:**
```javascript
await fetch('/api/v1/users/user_123', {
  method: 'DELETE',
  headers: {
    'Authorization': 'Bearer your-api-key'
  }
});
```

## Data APIs

### GET /api/v1/data

Retrieves application data with filtering and search capabilities.

**Parameters:**
- `search` (optional): Search query string
- `filter` (optional): Filter criteria in JSON format
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)

**Example Usage:**
```javascript
// Search for data
const searchResults = await fetch('/api/v1/data?search=example');

// Apply filters
const filteredData = await fetch('/api/v1/data?filter={"status":"active"}');

// Combine search and filters
const combinedResults = await fetch('/api/v1/data?search=example&filter={"category":"important"}');
```

### POST /api/v1/data

Creates new data entries.

**Request Body:**
```json
{
  "title": "Data Entry Title",
  "content": "Data entry content",
  "category": "example",
  "tags": ["tag1", "tag2"],
  "metadata": {
    "priority": "high",
    "source": "api"
  }
}
```

**Example Usage:**
```javascript
const newData = await fetch('/api/v1/data', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer your-api-key',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    title: 'Important Data',
    content: 'This is important information',
    category: 'business',
    tags: ['important', 'business']
  })
});
```

## Utility APIs

### GET /api/v1/health

Health check endpoint.

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2023-01-01T00:00:00Z",
  "version": "1.0.0"
}
```

### GET /api/v1/version

Returns API version information.

**Response:**
```json
{
  "version": "1.0.0",
  "build": "abc123",
  "environment": "production"
}
```

## Error Handling

All API endpoints return consistent error responses:

### Error Response Format

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": "Additional error details",
    "timestamp": "2023-01-01T00:00:00Z"
  }
}
```

### Common Error Codes

- `400` - Bad Request: Invalid request parameters
- `401` - Unauthorized: Authentication required
- `403` - Forbidden: Insufficient permissions
- `404` - Not Found: Resource not found
- `422` - Unprocessable Entity: Validation errors
- `429` - Too Many Requests: Rate limit exceeded
- `500` - Internal Server Error: Server error

### Example Error Handling

```javascript
try {
  const response = await fetch('/api/v1/users', {
    headers: { 'Authorization': 'Bearer invalid-token' }
  });
  
  if (!response.ok) {
    const error = await response.json();
    console.error('API Error:', error.error.message);
    throw new Error(error.error.message);
  }
  
  const data = await response.json();
  return data;
} catch (error) {
  console.error('Request failed:', error.message);
  throw error;
}
```

## Rate Limiting

API requests are subject to rate limiting:

- **Standard users**: 1000 requests per hour
- **Premium users**: 5000 requests per hour
- **Enterprise users**: 10000 requests per hour

### Rate Limit Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### Handling Rate Limits

```javascript
async function makeAPIRequest(url, options) {
  const response = await fetch(url, options);
  
  if (response.status === 429) {
    const resetTime = response.headers.get('X-RateLimit-Reset');
    const waitTime = (resetTime * 1000) - Date.now();
    
    console.log(`Rate limit exceeded. Waiting ${waitTime}ms`);
    await new Promise(resolve => setTimeout(resolve, waitTime));
    
    // Retry the request
    return makeAPIRequest(url, options);
  }
  
  return response;
}
```

## Response Formats

### Success Response

```json
{
  "data": {
    // Response data
  },
  "meta": {
    "timestamp": "2023-01-01T00:00:00Z",
    "request_id": "req_123"
  }
}
```

### Paginated Response

```json
{
  "data": [
    // Array of items
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "pages": 10,
    "has_next": true,
    "has_prev": false
  },
  "meta": {
    "timestamp": "2023-01-01T00:00:00Z",
    "request_id": "req_123"
  }
}
```