# AGENTS.md - WireGuard UI Development Guide

This document provides essential guidelines for agentic coding agents working on the WireGuard UI project.

## Project Overview

WireGuard UI is a web interface for managing WireGuard VPN configurations, built with Go (backend) and vanilla JavaScript/HTML (frontend). The project uses Echo framework for HTTP routing and JSON file storage for data persistence.

## Build Commands

### Development & Testing
```bash
# Build the application
go build -o wireguard-ui .

# Run the application directly
./wireguard-ui

# Run with custom configuration
./wireguard-ui --bind-address=":8080" --disable-login=false

# Install dependencies (frontend assets)
yarn install

# Prepare frontend assets
./prepare_assets.sh
```

### Linting & Code Quality
```bash
# Run golangci-lint (primary linter)
golangci-lint run

# Format Go code
gofmt -s -w .

# Run go imports
goimports -w .

# Check Go vet
go vet ./...

# Check for unused code
go run golang.org/x/tools/cmd/deadcode@latest ./...

# Fix misspellings
misspell -w .
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests for specific package
go test ./handler
go test ./model
go test ./store

# Run single test
go test -run TestSpecificFunction ./handler

# Run tests with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

## Code Style Guidelines

### Import Organization
```go
// Standard library imports first
import (
    "crypto/sha512"
    "embed"
    "flag"
    "fmt"
    "net/http"
    "time"

    // Third-party imports second
    "github.com/labstack/echo/v4"
    "github.com/labstack/gommon/log"

    // Local imports last
    "github.com/ngoduykhanh/wireguard-ui/handler"
    "github.com/ngoduykhanh/wireguard-ui/model"
    "github.com/ngoduykhanh/wireguard-ui/store"
)
```

### Naming Conventions
- **Variables**: camelCase (`clientData`, `serverConfig`)
- **Constants**: SCREAMING_SNAKE_CASE (`DEFAULT_EMAIL_SUBJECT`)
- **Functions**: PascalCase for exported (`NewClient`), camelCase for private (`initServerConfig`)
- **Structs**: PascalCase (`Client`, `ServerConfig`)
- **Interfaces**: PascalCase with 'I' prefix (`IStore`)
- **Packages**: lowercase, single word (`handler`, `model`, `store`)

### Function Structure
```go
// Handler functions follow this pattern
func HandlerName(dependencies) echo.HandlerFunc {
    return func(c echo.Context) error {
        // Implementation
        return c.JSON(http.StatusOK, response)
    }
}

// Regular functions
func functionName(param1 type1, param2 type2) (returnType, error) {
    // Implementation
    return result, nil
}
```

### Error Handling
```go
// Always check errors immediately
result, err := someFunction()
if err != nil {
    log.Errorf("Failed to perform action: %v", err)
    return echo.NewHTTPError(http.StatusInternalServerError, "Internal server error")
}

// Use log.Fatalf for initialization errors
if err := db.Init(); err != nil {
    log.Fatalf("Cannot initialize database: %v", err)
}

// Return appropriate HTTP status codes
return echo.NewHTTPError(http.StatusBadRequest, "Invalid input")
return echo.NewHTTPError(http.StatusNotFound, "Resource not found")
return echo.NewHTTPError(http.StatusInternalServerError, "Internal server error")
```

### Struct Tags
```go
type Client struct {
    ID           string    `json:"id"`
    PrivateKey   string    `json:"private_key"`
    Name         string    `json:"name"`
    Enabled      bool      `json:"enabled"`
    CreatedAt    time.Time `json:"created_at"`
}
```

### Middleware Usage
```go
// Apply middleware in correct order
app.POST("/endpoint", handler.SomeHandler(db), handler.ValidSession, handler.ContentTypeJson)

// Security middleware for non-GET routes
app.POST("/api/endpoint", handler.APIHandler(db), handler.ValidSession, handler.ContentTypeJson)
```

## File Organization

```
├── main.go              # Application entry point
├── handler/             # HTTP handlers and middleware
├── model/              # Data models and structs
├── store/              # Data storage interfaces and implementations
├── router/             # Echo router configuration
├── util/               # Utility functions
├── emailer/            # Email functionality
├── telegram/           # Telegram bot integration
├── templates/          # HTML templates
├── assets/             # Static assets (CSS, JS, images)
├── examples/           # Docker compose examples
└── db/                 # JSON database files
```

## Database Patterns

### Store Interface Usage
```go
// Always use the interface, not concrete implementation
func HandlerName(db store.IStore) echo.HandlerFunc {
    return func(c echo.Context) error {
        clients, err := db.GetClients(false)
        if err != nil {
            return echo.NewHTTPError(http.StatusInternalServerError, err.Error())
        }
        return c.JSON(http.StatusOK, clients)
    }
}
```

### Model Validation
```go
// Validate input before database operations
if client.Name == "" {
    return echo.NewHTTPError(http.StatusBadRequest, "Client name is required")
}

// Use regexp for validation where appropriate
var usernameRegexp = regexp.MustCompile("^\\w[\\w\\-.]*$")
if !usernameRegexp.MatchString(username) {
    return echo.NewHTTPError(http.StatusBadRequest, "Invalid username format")
}
```

## Security Guidelines

- Always use `handler.ContentTypeJson` middleware for non-GET routes to prevent CSRF
- Validate session with `handler.ValidSession` middleware
- Use `handler.NeedsAdmin` for admin-only endpoints
- Sanitize user input and validate against expected formats
- Hash passwords using appropriate algorithms
- Use secure session secrets (environment variables)

## Environment Variables

Key environment variables for development:
- `BIND_ADDRESS`: Server bind address (default: "0.0.0.0:5000")
- `DISABLE_LOGIN`: Disable authentication (default: false)
- `SESSION_SECRET`: Session encryption key
- `WGUI_LOG_LEVEL`: Log level (DEBUG, INFO, WARN, ERROR, OFF)

## Testing Guidelines

- Write tests for all handler functions
- Mock the database using interfaces
- Test both success and error cases
- Use table-driven tests for multiple scenarios
- Test middleware behavior separately

## Common Patterns

### Handler Registration
```go
app.GET(util.BasePath+"/api/clients", handler.GetClients(db), handler.ValidSession)
app.POST(util.BasePath+"/api/client", handler.CreateClient(db), handler.ValidSession, handler.ContentTypeJson)
```

### Response Patterns
```go
// Success response
return c.JSON(http.StatusOK, map[string]interface{}{
    "success": true,
    "data":    result,
})

// Error response
return echo.NewHTTPError(http.StatusBadRequest, "Validation error")
```

### Time Handling
```go
// Always use UTC for stored timestamps
client.CreatedAt = time.Now().UTC()
client.UpdatedAt = time.Now().UTC()
```

## Docker Integration

- Use multi-stage builds for production images
- Embed templates and assets using Go embed
- Support both TCP sockets and Unix domain sockets
- Handle graceful shutdown for containers