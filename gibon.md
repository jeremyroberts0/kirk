# Kirk - Slack-JIRA Integration Bot

## Project Overview

Kirk is a Slack bot designed to integrate Slack and JIRA by automatically posting links to JIRA issues when issue IDs are detected in Slack channels. The bot monitors Slack channels and provides convenient access to JIRA issues through automatic link generation.

### Key Features
- **Auto-linking**: Automatically detects JIRA issue IDs in Slack messages and posts clickable links
- **OAuth Integration**: Secure Slack OAuth flow for team authorization
- **Multi-team Support**: Can handle multiple Slack teams with individual configurations
- **Direct Message Commands**: Supports configuration commands via direct messages
- **MongoDB Storage**: Persistent storage of team configurations and access tokens

## Architecture

Kirk follows a modular Go architecture with clear separation of concerns:

### Core Components
- **HTTP Server**: Gin-based web server handling OAuth flows and info endpoints
- **Slack RTM Client**: Real-time messaging client for monitoring Slack channels
- **MongoDB Integration**: Database layer for persistent configuration storage
- **Command System**: Modular command handling for bot interactions

### Request Flow
1. **OAuth Flow**: Users authorize Kirk via `/authorize` endpoint
2. **Token Storage**: Access tokens stored in MongoDB with team-specific configurations
3. **Real-time Monitoring**: Bot instances created for each authorized team
4. **Message Processing**: Incoming messages scanned for JIRA issue patterns
5. **Link Generation**: Matching patterns converted to clickable JIRA links

## Directory Structure

```
├── commands/          # Bot command handlers
│   ├── commands.go    # Command router and dispatcher
│   ├── config.go      # Configuration management commands
│   └── help.go        # Help text and usage information
├── config/            # Configuration management
│   └── config.go      # Team config CRUD operations
├── db/                # Database layer
│   └── db.go          # MongoDB connection and initialization
├── html/              # HTML templates
│   ├── oauthError.html    # OAuth error page
│   └── oauthSuccess.html  # OAuth success page
├── routes/            # HTTP route handlers
│   ├── auth.go        # OAuth authentication routes
│   └── info.go        # Health check and info endpoints
├── slackTeam/         # Slack integration
│   └── watchTeam.go   # Real-time message monitoring
├── main.go            # Application entry point
├── router.go          # HTTP router configuration
├── Dockerfile         # Container configuration
└── docker-compose.yml # Development environment setup
```

## Key Files

### Application Entry Points
- **`main.go`**: Application bootstrap, database initialization, and HTTP server startup
- **`router.go`**: HTTP route configuration using Gin framework

### Core Logic
- **`slackTeam/watchTeam.go`**: Core Slack integration with RTM client, message processing, and JIRA issue detection
- **`routes/auth.go`**: OAuth flow implementation with Slack authorization
- **`config/config.go`**: Team configuration management with MongoDB persistence

### Commands
- **`commands/commands.go`**: Command dispatcher for bot direct message interactions
- **`commands/config.go`**: Configuration commands for JIRA URL and project setup

### Database
- **`db/db.go`**: MongoDB connection management and database initialization

## Architecture Patterns

### Data Models
```go
type TeamConfig struct {
    Id                  bson.ObjectId `bson:"_id,omitempty"`
    Team_id             string        `bson:"team_id,omitempty"`
    Jira_base_url       string        `bson:"jira_base_url,omitempty"`
    Subscribed_projects []string      `bson:"subscribed_projects,omitempty"`
    Access_token        string        `bson:"access_token,omitempty"`
}
```

### Message Processing Flow
1. **RTM Connection**: Establishes WebSocket connection to Slack
2. **Event Filtering**: Processes only relevant message events
3. **Pattern Matching**: Uses regex to detect JIRA issue IDs (e.g., `PROJECT-123`)
4. **Link Prevention**: Prevents duplicate links by checking existing URLs
5. **Response Generation**: Creates formatted messages with JIRA links

## Coding Conventions

### Go Standards
- **Package Structure**: Functional organization with clear package boundaries
- **Error Handling**: Explicit error checking with panic for critical failures
- **Import Grouping**: Standard library, external packages, then local packages
- **Naming**: Go-standard camelCase for unexported, PascalCase for exported

### Slack Integration Patterns
- **Token Management**: Secure storage and retrieval of OAuth tokens
- **Channel Detection**: Different handling for direct messages vs. channels
- **Message Formatting**: Consistent message formatting with newlines for readability

### Database Patterns
- **Collection Access**: Centralized collection getter functions
- **BSON Marshaling**: Proper struct tags for MongoDB document mapping
- **Error Propagation**: Consistent error return patterns

## Dependencies

### Core Dependencies
- **`gopkg.in/gin-gonic/gin.v1`**: HTTP web framework
- **`github.com/nlopes/slack`**: Slack API client and RTM support
- **`gopkg.in/mgo.v2`**: MongoDB driver and BSON handling

### Development Dependencies
- **`github.com/pilu/fresh`**: Hot reloading for development (Docker only)

### System Dependencies
- **Go 1.9.2**: Base runtime (as specified in Dockerfile)
- **MongoDB**: Database for persistent storage

## Development Workflow

### Local Development Setup
1. **Environment Variables**: Create `.env` file with required Slack credentials
2. **Database**: Start MongoDB (handled by docker-compose)
3. **Hot Reloading**: Set `APP_ENV=development` for automatic restarts
4. **Container Build**: Use `docker-compose up` for complete environment

### Required Environment Variables
```bash
SLACK_BOT_ACCESS_TOKEN=xoxb-your-token-here
SLACK_CLIENT_ID=your-client-id
SLACK_CLIENT_SECRET=your-client-secret
PUBLIC_HOST=localhost:8080  # Or your public hostname
```

### Build Process
```bash
# Dependencies
go get ./

# Build
go build

# Run
./kirk
```

### Docker Development
```bash
# Start all services
docker-compose up

# Development mode with hot reloading
APP_ENV=development docker-compose up
```

## Common Tasks

### Adding New Commands
1. **Command Handler**: Add new case to `commands/commands.go` switch statement
2. **Implementation**: Create handler function in appropriate commands file
3. **Help Text**: Update `commands/help.go` with command documentation

### Extending JIRA Integration
1. **Pattern Matching**: Modify regex in `slackTeam/watchTeam.go`
2. **Link Generation**: Update link formatting logic
3. **Configuration**: Add new fields to `TeamConfig` struct

### Adding New Routes
1. **Route Handler**: Create handler function in `routes/` package
2. **Registration**: Add route to `router.go`
3. **Templates**: Add HTML templates to `html/` directory if needed

### Database Schema Changes
1. **Struct Updates**: Modify `TeamConfig` in `config/config.go`
2. **Migration**: Handle backward compatibility in accessor functions
3. **Indexing**: Consider MongoDB index requirements for new fields

## Important Context

### Legacy Go Version
- Project uses Go 1.9.2 (older version)
- Uses `gopkg.in` imports instead of Go modules
- No `go.mod` file present - uses GOPATH-style dependency management

### Slack API Considerations
- Uses Real-Time Messaging (RTM) API for live message monitoring
- Requires specific OAuth scopes for channel access and messaging
- Bot tokens stored securely in MongoDB

### Message Processing Quirks
- **Duplicate Prevention**: Complex logic to avoid re-linking already linked issues
- **Channel Type Detection**: Different handling for direct messages vs. channels
- **Regex Limitations**: Works around Go's lack of negative lookahead in regex

### Development Environment
- **Docker-first**: Designed to run in containerized environment
- **Hot Reloading**: Uses `fresh` package for development file watching
- **MongoDB Dependency**: Requires MongoDB connection for all operations

### Security Notes
- **Token Storage**: OAuth tokens stored in database, not in configuration files
- **Environment Variables**: Sensitive credentials managed via `.env` file
- **No Authentication**: Web routes have no authentication beyond OAuth flow

### Potential Issues
- **No Error Recovery**: Bot instances may not recover from connection failures
- **No Rate Limiting**: No protection against Slack API rate limits
- **Legacy Dependencies**: Older package versions may have security vulnerabilities