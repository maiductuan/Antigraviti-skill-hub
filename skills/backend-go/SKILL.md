---
name: backend-go
description: Go backend development with Gin/Echo frameworks, concurrency patterns, and idiomatic Go practices
tags: [go, golang, gin, echo, backend, concurrency]
author: Antigravity Team
version: 1.0.0
---

# Go Backend Development Skill

Idiomatic Go backend patterns.

## Project Structure

```
project/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── handlers/
│   ├── models/
│   ├── repository/
│   ├── services/
│   └── middleware/
├── pkg/
│   └── utils/
├── config/
└── go.mod
```

## Gin Framework

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    
    // Middleware
    r.Use(gin.Logger())
    r.Use(gin.Recovery())
    r.Use(CORSMiddleware())
    
    // Routes
    api := r.Group("/api/v1")
    {
        api.GET("/users", getUsers)
        api.GET("/users/:id", getUser)
        api.POST("/users", createUser)
        api.PUT("/users/:id", updateUser)
        api.DELETE("/users/:id", deleteUser)
    }
    
    r.Run(":8080")
}

func getUser(c *gin.Context) {
    id := c.Param("id")
    user, err := userService.GetByID(id)
    if err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }
    c.JSON(http.StatusOK, user)
}

func createUser(c *gin.Context) {
    var input CreateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    
    user, err := userService.Create(input)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusCreated, user)
}
```

## Error Handling

```go
// Custom error types
type AppError struct {
    Code    int    `json:"-"`
    Message string `json:"message"`
    Err     error  `json:"-"`
}

func (e *AppError) Error() string {
    return e.Message
}

func NewNotFoundError(msg string) *AppError {
    return &AppError{Code: 404, Message: msg}
}

// Error middleware
func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()
        
        if len(c.Errors) > 0 {
            err := c.Errors.Last().Err
            if appErr, ok := err.(*AppError); ok {
                c.JSON(appErr.Code, appErr)
                return
            }
            c.JSON(500, gin.H{"error": "Internal server error"})
        }
    }
}
```

## Concurrency Patterns

```go
// Worker pool
func processItems(items []Item) []Result {
    numWorkers := runtime.NumCPU()
    jobs := make(chan Item, len(items))
    results := make(chan Result, len(items))
    
    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                results <- processItem(item)
            }
        }()
    }
    
    // Send jobs
    for _, item := range items {
        jobs <- item
    }
    close(jobs)
    
    // Collect results
    go func() {
        wg.Wait()
        close(results)
    }()
    
    var output []Result
    for result := range results {
        output = append(output, result)
    }
    return output
}

// Context with timeout
func fetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return io.ReadAll(resp.Body)
}
```

## Database with GORM

```go
import "gorm.io/gorm"

type User struct {
    gorm.Model
    Name  string `gorm:"size:100;not null"`
    Email string `gorm:"uniqueIndex;not null"`
    Posts []Post
}

type Post struct {
    gorm.Model
    Title   string
    Content string
    UserID  uint
    User    User
}

// Repository pattern
type UserRepository struct {
    db *gorm.DB
}

func (r *UserRepository) FindByID(id uint) (*User, error) {
    var user User
    result := r.db.Preload("Posts").First(&user, id)
    return &user, result.Error
}

func (r *UserRepository) Create(user *User) error {
    return r.db.Create(user).Error
}
```

## Best Practices

1. **Accept interfaces, return structs**
2. **Handle errors explicitly**
3. **Use context for cancellation**
4. **Prefer composition over inheritance**
5. **Keep packages focused and small**
