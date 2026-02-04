---
name: documentation-api
description: API documentation with OpenAPI/Swagger, including schema design, code generation, and documentation best practices
tags: [openapi, swagger, api, documentation]
author: Antigravity Team
version: 1.0.0
---

# API Documentation Skill

Create excellent API documentation.

## OpenAPI 3.0 Specification

```yaml
openapi: 3.0.3
info:
  title: User Management API
  description: API for managing users in the system
  version: 1.0.0
  contact:
    email: api@example.com

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging

tags:
  - name: users
    description: User operations
  - name: auth
    description: Authentication

paths:
  /users:
    get:
      summary: List all users
      tags: [users]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
    
    post:
      summary: Create a new user
      tags: [users]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /users/{id}:
    get:
      summary: Get user by ID
      tags: [users]
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        role:
          type: string
          enum: [user, admin]
        createdAt:
          type: string
          format: date-time
      required: [id, email, name]

    CreateUser:
      type: object
      properties:
        email:
          type: string
          format: email
        name:
          type: string
          minLength: 2
        password:
          type: string
          minLength: 8
      required: [email, name, password]

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string

  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    Unauthorized:
      description: Unauthorized
    NotFound:
      description: Resource not found
```

## Auto-generation from Code

```typescript
// NestJS with Swagger
@ApiTags('users')
@Controller('users')
export class UsersController {
  @Get()
  @ApiOperation({ summary: 'Get all users' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiResponse({ status: 200, type: [UserDto] })
  findAll(@Query() query: PaginationDto) {
    return this.usersService.findAll(query);
  }

  @Post()
  @ApiOperation({ summary: 'Create user' })
  @ApiBody({ type: CreateUserDto })
  @ApiResponse({ status: 201, type: UserDto })
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}

@ApiProperty()
export class UserDto {
  @ApiProperty({ example: 'uuid-here' })
  id: string;

  @ApiProperty({ example: 'user@example.com' })
  email: string;
}
```

## Best Practices

1. **Use examples** in schemas
2. **Document all responses** (including errors)
3. **Version your API** in the URL
4. **Use consistent naming**
5. **Keep docs in sync** with code
