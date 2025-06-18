# RESTful API & GraphQL Fundamentals

## 🎯 Tujuan Pembelajaran
- Memahami prinsip-prinsip RESTful API design yang konsisten dan scalable
- Menguasai best practices dalam API versioning, authentication, dan error handling
- Mampu merancang API documentation yang comprehensive
- Memahami kapan menggunakan REST vs GraphQL
- Menerapkan API security dan rate limiting yang efektif

## 📚 Konsep Dasar

### REST (Representational State Transfer)
REST adalah architectural style untuk web services dengan prinsip-prinsip:

1. **Stateless**: Setiap request harus contain semua informasi yang dibutuhkan
2. **Client-Server**: Separation of concerns antara client dan server
3. **Cacheable**: Response harus dapat di-cache untuk improve performance
4. **Uniform Interface**: Consistent interface untuk semua resources
5. **Layered System**: Architecture dapat terdiri dari multiple layers
6. **Code on Demand** (optional): Server dapat mengirim executable code

### HTTP Methods dan Penggunaannya

| Method | Purpose | Idempotent | Safe | Body |
|--------|---------|------------|------|------|
| GET | Retrieve data | ✅ | ✅ | ❌ |
| POST | Create new resource | ❌ | ❌ | ✅ |
| PUT | Update/Replace entire resource | ✅ | ❌ | ✅ |
| PATCH | Partial update | ❌ | ❌ | ✅ |
| DELETE | Delete resource | ✅ | ❌ | ❌ |
| HEAD | Get headers only | ✅ | ✅ | ❌ |
| OPTIONS | Get allowed methods | ✅ | ✅ | ❌ |

### HTTP Status Codes

```javascript
// Success (2xx)
200 OK              // Successful GET, PUT, PATCH
201 Created         // Successful POST
202 Accepted        // Accepted for processing (async)
204 No Content      // Successful DELETE

// Client Error (4xx)
400 Bad Request     // Invalid request syntax
401 Unauthorized    // Authentication required
403 Forbidden       // Access denied
404 Not Found       // Resource not found
409 Conflict        // Resource conflict
422 Unprocessable Entity // Validation errors
429 Too Many Requests    // Rate limit exceeded

// Server Error (5xx)
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

## 🔧 Implementasi Praktis

### RESTful API Design

#### 1. Resource Naming Convention

```javascript
// ✅ GOOD: Gunakan nouns, bukan verbs
GET    /api/users              // Get all users
GET    /api/users/123          // Get specific user
POST   /api/users              // Create new user
PUT    /api/users/123          // Update entire user
PATCH  /api/users/123          // Partial update user
DELETE /api/users/123          // Delete user

// Nested resources
GET    /api/users/123/orders   // Get user's orders
POST   /api/users/123/orders   // Create order for user
GET    /api/orders/456/items   // Get order items

// ❌ BAD: Gunakan verbs dalam URL
GET    /api/getUsers
POST   /api/createUser
PUT    /api/updateUser/123
```

#### 2. Query Parameters untuk Filtering dan Pagination

```javascript
// Filtering
GET /api/products?category=electronics&status=active&price_min=100

// Sorting
GET /api/products?sort=price&order=desc
GET /api/products?sort=name,price&order=asc,desc

// Pagination
GET /api/products?page=2&limit=20
GET /api/products?offset=40&limit=20

// Field selection
GET /api/users?fields=id,name,email

// Search
GET /api/products?search=laptop&category=electronics
```

#### 3. Request/Response Format

**Request Headers:**
```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
X-API-Version: v1
X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
```

**Successful Response:**
```json
{
  "success": true,
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

**Paginated Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Product 1"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": false,
    "next_url": "/api/products?page=2&limit=20",
    "prev_url": null
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The given data was invalid.",
    "details": [
      {
        "field": "email",
        "message": "The email field is required.",
        "code": "required"
      },
      {
        "field": "password",
        "message": "The password must be at least 8 characters.",
        "code": "min_length"
      }
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

#### 4. API Versioning

**URL Versioning (Recommended):**
```javascript
// Version in URL path
GET /api/v1/users
GET /api/v2/users

// Version in subdomain
GET https://v1.api.example.com/users
GET https://v2.api.example.com/users
```

**Header Versioning:**
```http
GET /api/users
Accept: application/vnd.api+json;version=1
```

**Query Parameter Versioning:**
```javascript
GET /api/users?version=1
```

#### 5. Authentication & Authorization

**JWT Token Authentication:**
```javascript
// Login endpoint
POST /api/auth/login
{
  "email": "user@example.com",
  "password": "password123"
}

// Response
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "id": 123,
      "name": "John Doe",
      "email": "user@example.com",
      "roles": ["user"]
    }
  }
}

// Protected endpoint usage
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**API Key Authentication:**
```http
GET /api/users
X-API-Key: your-api-key-here
```

#### 6. Rate Limiting

**Response Headers:**
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
Retry-After: 3600
```

**Rate Limited Response:**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retry_after": 3600
  }
}
```

### GraphQL Fundamentals

#### Schema Definition
```graphql
# Types
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  published: Boolean!
  createdAt: DateTime!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  post: Post!
  createdAt: DateTime!
}

# Queries
type Query {
  users(limit: Int, offset: Int): [User!]!
  user(id: ID!): User
  posts(published: Boolean, authorId: ID): [Post!]!
  post(id: ID!): Post
}

# Mutations
type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  
  createPost(input: CreatePostInput!): Post!
  publishPost(id: ID!): Post!
}

# Input Types
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

input UpdateUserInput {
  name: String
  email: String
}

input CreatePostInput {
  title: String!
  content: String!
  authorId: ID!
}
```

#### GraphQL Queries
```graphql
# Simple Query
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    createdAt
  }
}

# Nested Query with Field Selection
query GetUserWithPosts($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
      published
      comments {
        id
        content
        author {
          name
        }
      }
    }
  }
}

# Query with Fragments
fragment UserInfo on User {
  id
  name
  email
  createdAt
}

query GetUsers {
  users {
    ...UserInfo
    posts {
      id
      title
    }
  }
}

# Mutation
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    id
    title
    content
    author {
      name
    }
    createdAt
  }
}
```

### API Documentation dengan OpenAPI/Swagger

```yaml
openapi: 3.0.0
info:
  title: LEX DEV API
  description: API documentation for LEX DEV project
  version: 1.0.0
  contact:
    name: LEX DEV Team
    email: dev@lexdev.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.lexdev.com/v1
    description: Production server
  - url: https://staging-api.lexdev.com/v1
    description: Staging server

paths:
  /users:
    get:
      summary: Get all users
      description: Retrieve a paginated list of users
      tags:
        - Users
      parameters:
        - name: page
          in: query
          description: Page number
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          description: Number of items per page
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
        - name: search
          in: query
          description: Search term
          required: false
          schema:
            type: string
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
      security:
        - BearerAuth: []

    post:
      summary: Create a new user
      description: Create a new user account
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '422':
          $ref: '#/components/responses/ValidationError'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          format: int64
          example: 123
        name:
          type: string
          example: "John Doe"
        email:
          type: string
          format: email
          example: "john@example.com"
        created_at:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"
        updated_at:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"

    CreateUserRequest:
      type: object
      required:
        - name
        - email
        - password
      properties:
        name:
          type: string
          minLength: 2
          maxLength: 100
          example: "John Doe"
        email:
          type: string
          format: email
          example: "john@example.com"
        password:
          type: string
          minLength: 8
          example: "password123"

    UserResponse:
      type: object
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: '#/components/schemas/User'
        meta:
          $ref: '#/components/schemas/Meta'

    UserListResponse:
      type: object
      properties:
        success:
          type: boolean
          example: true
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'
        meta:
          $ref: '#/components/schemas/Meta'

    Pagination:
      type: object
      properties:
        current_page:
          type: integer
          example: 1
        per_page:
          type: integer
          example: 20
        total:
          type: integer
          example: 150
        total_pages:
          type: integer
          example: 8
        has_next:
          type: boolean
          example: true
        has_prev:
          type: boolean
          example: false
        next_url:
          type: string
          nullable: true
          example: "/api/v1/users?page=2&limit=20"
        prev_url:
          type: string
          nullable: true
          example: null

    Meta:
      type: object
      properties:
        timestamp:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"
        request_id:
          type: string
          format: uuid
          example: "550e8400-e29b-41d4-a716-446655440000"

    Error:
      type: object
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          properties:
            code:
              type: string
              example: "VALIDATION_ERROR"
            message:
              type: string
              example: "The given data was invalid."
            details:
              type: array
              items:
                type: object
                properties:
                  field:
                    type: string
                    example: "email"
                  message:
                    type: string
                    example: "The email field is required."
                  code:
                    type: string
                    example: "required"
        meta:
          $ref: '#/components/schemas/Meta'

  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    
    Unauthorized:
      description: Unauthorized
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    
    ValidationError:
      description: Validation error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
```

## ✅ Best Practices

### RESTful API Best Practices

#### Do's ✅

1. **Consistent Naming Convention**
   ```javascript
   // ✅ GOOD: Consistent resource naming
   GET /api/v1/users
   GET /api/v1/products
   GET /api/v1/orders
   
   // ✅ GOOD: Plural nouns untuk collections
   GET /api/v1/users        // Get all users
   GET /api/v1/users/123    // Get specific user
   ```

2. **Proper HTTP Status Codes**
   ```javascript
   // ✅ GOOD: Use appropriate status codes
   app.post('/api/users', (req, res) => {
     const user = createUser(req.body);
     res.status(201).json({
       success: true,
       data: user
     });
   });
   
   app.get('/api/users/:id', (req, res) => {
     const user = findUser(req.params.id);
     if (!user) {
       return res.status(404).json({
         success: false,
         error: {
           code: 'USER_NOT_FOUND',
           message: 'User not found'
         }
       });
     }
     res.status(200).json({
       success: true,
       data: user
     });
   });
   ```

3. **Input Validation**
   ```javascript
   // ✅ GOOD: Comprehensive validation
   const createUserSchema = {
     type: 'object',
     required: ['name', 'email', 'password'],
     properties: {
       name: {
         type: 'string',
         minLength: 2,
         maxLength: 100
       },
       email: {
         type: 'string',
         format: 'email'
       },
       password: {
         type: 'string',
         minLength: 8,
         pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)'
       }
     }
   };
   ```

4. **Security Headers**
   ```javascript
   // ✅ GOOD: Security headers
   app.use((req, res, next) => {
     res.setHeader('X-Content-Type-Options', 'nosniff');
     res.setHeader('X-Frame-Options', 'DENY');
     res.setHeader('X-XSS-Protection', '1; mode=block');
     res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
     next();
   });
   ```

5. **CORS Configuration**
   ```javascript
   // ✅ GOOD: Proper CORS setup
   const corsOptions = {
     origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
     methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
     allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Version'],
     credentials: true,
     maxAge: 86400 // 24 hours
   };
   
   app.use(cors(corsOptions));
   ```

#### Don'ts ❌

1. **Jangan gunakan verbs dalam URL**
   ```javascript
   // ❌ BAD
   GET /api/getUsers
   POST /api/createUser
   PUT /api/updateUser/123
   
   // ✅ GOOD
   GET /api/users
   POST /api/users
   PUT /api/users/123
   ```

2. **Jangan expose internal structure**
   ```javascript
   // ❌ BAD: Exposing database structure
   GET /api/user_profiles
   GET /api/product_categories
   
   // ✅ GOOD: Clean resource names
   GET /api/profiles
   GET /api/categories
   ```

3. **Jangan return passwords atau sensitive data**
   ```javascript
   // ❌ BAD
   {
     "id": 123,
     "name": "John Doe",
     "password": "hashed_password",
     "credit_card": "1234-5678-9012-3456"
   }
   
   // ✅ GOOD
   {
     "id": 123,
     "name": "John Doe",
     "email": "john@example.com"
   }
   ```

### GraphQL Best Practices

#### Do's ✅

1. **Schema Design**
   ```graphql
   # ✅ GOOD: Descriptive field names and types
   type Product {
     id: ID!
     name: String!
     description: String
     price: Money!
     category: Category!
     inStock: Boolean!
     tags: [String!]!
   }
   
   # Custom scalar types
   scalar Money
   scalar DateTime
   scalar Email
   ```

2. **Pagination**
   ```graphql
   # ✅ GOOD: Cursor-based pagination
   type Query {
     products(
       first: Int
       after: String
       last: Int
       before: String
       filter: ProductFilter
     ): ProductConnection!
   }
   
   type ProductConnection {
     edges: [ProductEdge!]!
     pageInfo: PageInfo!
     totalCount: Int!
   }
   
   type ProductEdge {
     node: Product!
     cursor: String!
   }
   
   type PageInfo {
     hasNextPage: Boolean!
     hasPreviousPage: Boolean!
     startCursor: String
     endCursor: String
   }
   ```

#### Don'ts ❌

1. **Jangan buat schema yang terlalu nested**
   ```graphql
   # ❌ BAD: Too deep nesting
   type User {
     posts {
       comments {
         replies {
           author {
             posts {
               comments # This can cause N+1 problem
             }
           }
         }
       }
     }
   }
   ```

2. **Jangan expose implementation details**
   ```graphql
   # ❌ BAD
   type User {
     id: ID!
     user_table_id: Int  # Database implementation detail
     created_timestamp: String  # Use proper DateTime scalar
   }
   ```

## 🧪 Latihan Praktis

### Exercise 1: RESTful API Design
**Scenario**: Merancang API untuk sistem perpustakaan digital dengan fitur:
- Manajemen buku (CRUD)
- Sistem peminjaman
- User management
- Search dan filter buku
- Review dan rating

**Tugas**:
1. Buat endpoint design lengkap dengan HTTP methods
2. Definisikan request/response format
3. Buat OpenAPI documentation
4. Implementasi authentication dan authorization

**Contoh Solution**:
```javascript
// Books Management
GET    /api/v1/books                    // List books with pagination
GET    /api/v1/books/:id                // Get specific book
POST   /api/v1/books                    // Create new book (admin only)
PUT    /api/v1/books/:id                // Update book (admin only)
DELETE /api/v1/books/:id                // Delete book (admin only)

// Search and Filter
GET    /api/v1/books/search?q=:query&category=:cat&author=:author

// Borrowing System
GET    /api/v1/users/:id/borrowed-books  // User's borrowed books
POST   /api/v1/books/:id/borrow          // Borrow a book
POST   /api/v1/books/:id/return          // Return a book

// Reviews
GET    /api/v1/books/:id/reviews         // Get book reviews
POST   /api/v1/books/:id/reviews         // Add review
PUT    /api/v1/reviews/:id               // Update review
DELETE /api/v1/reviews/:id               // Delete review
```

### Exercise 2: GraphQL Schema Design
**Scenario**: E-learning platform dengan:
- Courses dan lessons
- User enrollment
- Progress tracking
- Assignments dan submissions

**Tugas**: Buat GraphQL schema lengkap dengan queries, mutations, dan subscriptions.

**Contoh Solution**:
```graphql
type Course {
  id: ID!
  title: String!
  description: String!
  instructor: User!
  lessons: [Lesson!]!
  enrollments: [Enrollment!]!
  price: Money!
  duration: Int # in minutes
  level: CourseLevel!
  tags: [String!]!
  createdAt: DateTime!
}

type Lesson {
  id: ID!
  title: String!
  content: String!
  course: Course!
  videoUrl: String
  duration: Int
  order: Int!
  assignments: [Assignment!]!
}

type Enrollment {
  id: ID!
  user: User!
  course: Course!
  progress: Float! # percentage
  enrolledAt: DateTime!
  completedAt: DateTime
}

type Query {
  courses(filter: CourseFilter, sort: CourseSort): [Course!]!
  course(id: ID!): Course
  myEnrollments: [Enrollment!]!
  courseProgress(courseId: ID!): Enrollment
}

type Mutation {
  enrollCourse(courseId: ID!): Enrollment!
  updateProgress(lessonId: ID!): Enrollment!
  submitAssignment(assignmentId: ID!, content: String!): Submission!
}

type Subscription {
  courseProgressUpdated(courseId: ID!): Enrollment!
  newAnnouncement(courseId: ID!): Announcement!
}
```

### Exercise 3: API Performance Optimization
**Given**: API endpoint yang lambat
```javascript
GET /api/v1/dashboard/stats
// Response time: 3-5 seconds
// Returns: user stats, recent orders, popular products, etc.
```

**Tugas**: Optimize API dengan:
1. Caching strategy
2. Response compression
3. Database query optimization
4. Pagination untuk large datasets

**Solution Approach**:
```javascript
// 1. Implement caching
app.get('/api/v1/dashboard/stats', 
  cache('5 minutes'),
  async (req, res) => {
    // Cached response
  }
);

// 2. Parallel data fetching
const [userStats, recentOrders, popularProducts] = await Promise.all([
  getUserStats(userId),
  getRecentOrders(userId, { limit: 10 }),
  getPopularProducts({ limit: 5 })
]);

// 3. Response compression
app.use(compression());

// 4. Partial responses
GET /api/v1/dashboard/stats?fields=userStats,recentOrders
```

## 📖 Resources Tambahan

### Documentation Tools
- **OpenAPI/Swagger**: [Swagger Editor](https://editor.swagger.io/)
- **Postman**: [API Documentation](https://documenter.getpostman.com/)
- **Insomnia**: REST client dengan documentation
- **GraphQL Playground**: Interactive GraphQL IDE

### Testing Tools
- **Jest**: Unit testing framework
- **Supertest**: HTTP assertions untuk Node.js
- **GraphQL Testing Library**: Testing utilities untuk GraphQL
- **Artillery**: Load testing tool

### Security Tools
- **OWASP API Security Top 10**: Security guidelines
- **JWT.io**: JWT token debugger
- **Helmet.js**: Security headers middleware
- **Rate Limiter**: Express rate limiting middleware

### Online Resources
- [RESTful API Design Best Practices](https://restfulapi.net/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [API Design Guidelines by Microsoft](https://github.com/Microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)

### Books
- "RESTful Web APIs" - Leonard Richardson
- "GraphQL in Action" - Samer Buna
- "API Design Patterns" - JJ Geewax

## ❓ FAQ

### Q: REST vs GraphQL - Kapan menggunakan masing-masing?
**A**: 
**Gunakan REST ketika**:
- Simple CRUD operations
- Caching requirements tinggi
- Team familiar dengan REST
- Mobile apps dengan bandwidth terbatas
- Microservices architecture

**Gunakan GraphQL ketika**:
- Complex data relationships
- Multiple clients dengan kebutuhan data berbeda
- Real-time features (subscriptions)
- Rapid frontend development
- Over-fetching/under-fetching problems

### Q: Bagaimana handle API versioning yang baik?
**A**: Best practices untuk API versioning:

```javascript
// 1. URL Path Versioning (Recommended)
/api/v1/users
/api/v2/users

// 2. Maintain backward compatibility
// Support v1 selama 12-18 bulan setelah v2 release

// 3. Clear deprecation warnings
{
  "data": {...},
  "deprecation_warning": {
    "message": "This endpoint will be deprecated on 2024-12-31",
    "new_endpoint": "/api/v2/users",
    "migration_guide": "https://docs.api.com/migration/v1-to-v2"
  }
}

// 4. Semantic versioning
v1.0.0 - Major version (breaking changes)
v1.1.0 - Minor version (new features, backward compatible)
v1.1.1 - Patch version (bug fixes)
```

### Q: Bagaimana implement real-time features di REST API?
**A**: Beberapa approaches:

1. **Server-Sent Events (SSE)**:
```javascript
app.get('/api/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });

  const sendEvent = (data) => {
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };

  // Send periodic updates
  const interval = setInterval(() => {
    sendEvent({ timestamp: new Date(), message: 'Hello' });
  }, 1000);

  req.on('close', () => {
    clearInterval(interval);
  });
});
```

2. **WebSockets**:
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    // Broadcast to all clients
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({
          type: 'notification',
          data: message
        }));
      }
    });
  });
});
```

3. **Polling dengan Conditional Requests**:
```javascript
// Client polls dengan Last-Modified header
GET /api/notifications
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT

// Server response
304 Not Modified (jika tidak ada update)
// atau
200 OK (dengan data baru)
```

### Q: Bagaimana handle file uploads di API?
**A**: Best practices untuk file uploads:

```javascript
// 1. Multipart form data
POST /api/v1/files
Content-Type: multipart/form-data

// 2. Direct upload dengan presigned URLs (recommended untuk large files)
// Step 1: Get upload URL
POST /api/v1/files/upload-url
{
  "filename": "document.pdf",
  "content_type": "application/pdf",
  "size": 1024000
}

// Response:
{
  "upload_url": "https://s3.amazonaws.com/bucket/...",
  "file_id": "uuid-here",
  "expires_in": 3600
}

// Step 2: Upload directly to storage
PUT https://s3.amazonaws.com/bucket/...
[file content]

// Step 3: Confirm upload
POST /api/v1/files/uuid-here/confirm

// 3. Base64 encoding (untuk small files)
POST /api/v1/files
{
  "filename": "image.jpg",
  "content_type": "image/jpeg",
  "data": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ..."
}
```

### Q: Bagaimana implement proper error handling?
**A**: Comprehensive error handling strategy:

```javascript
// 1. Consistent error format
const errorResponse = (code, message, details = null, statusCode = 400) => ({
  success: false,
  error: {
    code,
    message,
    details,
    timestamp: new Date().toISOString(),
    request_id: req.headers['x-request-id']
  }
});

// 2. Global error handler
app.use((error, req, res, next) => {
  console.error('API Error:', error);

  // Operational errors
  if (error.isOperational) {
    return res.status(error.statusCode).json(
      errorResponse(error.code, error.message, error.details, error.statusCode)
    );
  }

  // Programming errors - don't expose details
  return res.status(500).json(
    errorResponse('INTERNAL_SERVER_ERROR', 'Something went wrong')
  );
});

// 3. Validation errors
const validateRequest = (schema) => (req, res, next) => {
  const { error } = schema.validate(req.body);
  if (error) {
    const details = error.details.map(detail => ({
      field: detail.path.join('.'),
      message: detail.message,
      code: detail.type
    }));
    
    return res.status(422).json(
      errorResponse('VALIDATION_ERROR', 'Validation failed', details, 422)
    );
  }
  next();
};
```

---

**Catatan**: Panduan ini adalah living document yang akan terus diupdate sesuai perkembangan teknologi dan kebutuhan tim LEX DEV. Pastikan untuk selalu mengikuti security best practices dan melakukan testing yang comprehensive sebelum deployment ke production.
