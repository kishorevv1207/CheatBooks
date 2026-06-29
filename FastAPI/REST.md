
# REST Architecture – Interview Notes (Quick Revision)

## 1. REST Full Form

* **REST** = Representational State Transfer
* Architectural style for designing web services
* Introduced by **Roy Fielding** (2000)

***

# 2. REST Principles / Constraints

### 1. Client-Server

* Separation of UI and backend
* Independent development

### 2. Stateless

* Server does not store client session
* Every request contains all required information

### 3. Cacheable

* Responses can be cached
* Improves performance

### 4. Uniform Interface

* Standard URI structure
* Consistent communication

### 5. Layered System

* Client unaware of intermediate layers
* Supports proxies, gateways

### 6. Code on Demand (Optional)

* Server can send executable code (JavaScript)

***

# 3. REST Components

### Resource

* Any object/data
* Example:

```
/users
/products
/orders
```

### Representation

Data format exchanged:

* JSON (Most used)
* XML
* HTML
* Plain Text

Example:

```json
{
  "id": 101,
  "name": "Kishore"
}
```

### URI (Uniform Resource Identifier)

Resource Identifier

Example:

```
GET /users/101
```

***

# 4. HTTP Methods

| Method  | Purpose                |
| ------- | ---------------------- |
| GET     | Fetch data             |
| POST    | Create resource        |
| PUT     | Update entire resource |
| PATCH   | Partial update         |
| DELETE  | Remove resource        |
| HEAD    | Header only            |
| OPTIONS | Supported operations   |

### Examples

```http
GET /users
POST /users
PUT /users/101
PATCH /users/101
DELETE /users/101
```

***

# 5. CRUD Mapping

| CRUD   | HTTP Method |
| ------ | ----------- |
| Create | POST        |
| Read   | GET         |
| Update | PUT/PATCH   |
| Delete | DELETE      |

***

# 6. HTTP Status Codes

## 2xx Success

| Code | Meaning    |
| ---- | ---------- |
| 200  | OK         |
| 201  | Created    |
| 202  | Accepted   |
| 204  | No Content |

***

## 3xx Redirection

| Code | Meaning           |
| ---- | ----------------- |
| 301  | Moved Permanently |
| 302  | Found             |
| 304  | Not Modified      |

***

## 4xx Client Errors

| Code | Meaning                |
| ---- | ---------------------- |
| 400  | Bad Request            |
| 401  | Unauthorized           |
| 403  | Forbidden              |
| 404  | Not Found              |
| 405  | Method Not Allowed     |
| 409  | Conflict               |
| 415  | Unsupported Media Type |
| 429  | Too Many Requests      |

***

## 5xx Server Errors

| Code | Meaning               |
| ---- | --------------------- |
| 500  | Internal Server Error |
| 502  | Bad Gateway           |
| 503  | Service Unavailable   |
| 504  | Gateway Timeout       |

***

# 7. Idempotency

### Idempotent Methods

Multiple calls produce same result.

✅ GET  
✅ PUT  
✅ DELETE  
✅ HEAD  
✅ OPTIONS

### Non-Idempotent

❌ POST

Example:

```http
DELETE /users/101
DELETE /users/101
```

Result remains same.

***

# 8. Safe Methods

Methods that do not modify data.

✅ GET  
✅ HEAD  
✅ OPTIONS

***

# 9. Request Structure

```http
GET /users/101 HTTP/1.1
Host: example.com
Authorization: Bearer token
Content-Type: application/json
```

### Components

* Method
* URI
* Headers
* Body (Optional)

***

# 10. Response Structure

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
   "id":101
}
```

### Components

* Status Code
* Headers
* Response Body

***

# 11. HTTP Headers

### Request Headers

```http
Authorization
Content-Type
Accept
User-Agent
Cookie
```

### Response Headers

```http
Cache-Control
Location
Set-Cookie
ETag
```

***

# 12. Content Types (MIME Types)

```http
application/json
application/xml
text/html
text/plain
multipart/form-data
```

***

# 13. Authentication

### Basic Authentication

```http
Authorization: Basic Base64(username:password)
```

### Bearer Token

```http
Authorization: Bearer <token>
```

### JWT Authentication

* Header
* Payload
* Signature

***

# 14. JWT Quick Notes

Structure:

```
xxxxx.yyyyy.zzzzz
```

Parts:

* Header
* Payload
* Signature

Benefits:

* Stateless
* Secure token exchange

***

# 15. Query Parameters

Used for filtering/searching.

Example:

```http
GET /users?city=Mysore
```

Common Uses:

* Filtering
* Searching
* Sorting
* Pagination

***

# 16. Path Parameters

Example:

```http
GET /users/101
```

Here:

```
101
```

is path parameter.

***

# 17. Pagination

Example:

```http
GET /products?page=1&size=10
```

Parameters:

```http
page
size
limit
offset
```

***

# 18. Versioning

### URI Versioning

```http
/api/v1/users
/api/v2/users
```

### Header Versioning

```http
Accept-Version: v1
```

***

# 19. REST Best Practices

### Use Nouns

✅

```http
/users
/orders
```

❌

```http
/getUsers
/createUser
```

### Use Plural Resources

```http
/users
/products
```

### Proper Status Codes

```http
200
201
400
404
500
```

### Stateless APIs

### Use HTTPS

### Consistent Naming

***

# 20. REST vs SOAP

| REST        | SOAP               |
| ----------- | ------------------ |
| Lightweight | Heavy              |
| JSON/XML    | XML Only           |
| Fast        | Slower             |
| Easy        | Complex            |
| Uses HTTP   | Multiple Protocols |

***

# 21. REST API Maturity (Richardson Model)

### Level 0

* Single endpoint

### Level 1

* Resources introduced

### Level 2

* HTTP methods used

### Level 3

* HATEOAS

***

# 22. HATEOAS

Hypermedia As The Engine Of Application State

Example:

```json
{
  "id":101,
  "name":"Kishore",
  "links":[
    {"rel":"self","href":"/users/101"},
    {"rel":"orders","href":"/users/101/orders"}
  ]
}
```

***

# 23. CORS

Cross-Origin Resource Sharing

Headers:

```http
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
```

Purpose:

* Cross-domain access control

***

# 24. Caching

Headers:

```http
Cache-Control
Expires
ETag
Last-Modified
```

Benefits:

* Reduced latency
* Reduced server load

***

# 25. Common Interview Questions

### What is REST?

Architectural style for web services.

### Why Stateless?

Scalable, simple requests.

### PUT vs PATCH?

* PUT → Full update
* PATCH → Partial update

### POST vs PUT?

* POST → Create
* PUT → Create/Replace

### Idempotent Methods?

GET, PUT, DELETE, HEAD, OPTIONS

### Safe Methods?

GET, HEAD, OPTIONS

### Resource vs Representation?

* Resource = Actual data
* Representation = JSON/XML form

### 200 vs 201?

* 200 → Success
* 201 → Resource Created

### 401 vs 403?

* 401 → Authentication required
* 403 → Access denied

### 204?

Success without response body.

***

# One-Minute Revision

```text
REST = Representational State Transfer

Constraints:
Client-Server
Stateless
Cacheable
Uniform Interface
Layered System
Code on Demand

Methods:
GET
POST
PUT
PATCH
DELETE

CRUD:
Create -> POST
Read -> GET
Update -> PUT/PATCH
Delete -> DELETE

Status:
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
500 Internal Server Error

Auth:
Basic Auth
Bearer Token
JWT

URI:
Path Param -> /users/1
Query Param -> ?page=1

PUT = Full Update
PATCH = Partial Update

POST = Non-Idempotent
GET/PUT/DELETE = Idempotent

Best Practice:
Use nouns
Use plural resources
Use proper status codes
Use HTTPS
Keep APIs stateless
```
