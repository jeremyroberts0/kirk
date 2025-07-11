# Kirk - Slack & JIRA Integration Bot

## Project Overview

Kirk is a Slack bot designed to integrate Slack workspaces with JIRA by automatically posting links to JIRA issues when issue IDs are detected in Slack channels. The bot provides seamless integration between team communication and project management.

### Key Features
- **Automatic JIRA Link Detection**: Detects JIRA issue IDs (e.g., `PROJECT-123`) in Slack messages and posts clickable links
- **OAuth Integration**: Handles Slack app authorization flow with proper token management
- **Multi-team Support**: Supports multiple Slack teams with individual configurations
- **Direct Message Commands**: Responds to configuration commands via direct messages
- **Configurable Project Filtering**: Only links to issues from subscribed projects

## Architecture

Kirk follows a modular architecture built in Go with the following key components:

### Core Components
1. **Main Application** (`main.go`, `router.go`): Entry point and HTTP server setup
2. **Slack Team Watcher** (`slackTeam/`): Real-time message monitoring via Slack RTM API
3. **Database Layer** (`db/`): MongoDB integration for configuration storage
4. **Configuration Management** (`config/`): Team-specific settings and JIRA configuration
5. **Command Processor** (`commands/`): Direct message command handling
6. **OAuth Routes** (`routes/`): Slack app authorization flow
7. **HTML Templates** (`html/`): OAuth success/error pages

### Data Flow
1. **Initialization**: App starts, connects to MongoDB, loads team configurations
2. **OAuth Flow**: New teams authorize via `/authorize` endpoint
3. **Message Processing**: RTM connections monitor team messages for JIRA IDs
4. **Link Generation**: Matching issue IDs generate JIRA links posted to channels
5. **Command Handling**: Direct messages processed for configuration commands

## Directory Structure

```
├── main.go              # Application entry point
├── router.go            # HTTP route setup
├── commands/            # Direct message command handling
│   ├── commands.go      # Command dispatcher
│   ├── config.go        # Configuration commands
│   └── help.go          # Help command
├── config/              # Team configuration management
│   └── config.go        # Database models and operations
├── db/                  # Database connection layer
│   └── db.go            # MongoDB connection setup
├── routes/              # HTTP route handlers
│   ├── auth.go          # OAuth authorization flow
│   └── info.go          # Basic info endpoint
├── slackTeam/           # Slack integration
│   └── watchTeam.go     # RTM message monitoring
├── html/                # HTML templates
│   ├── oauthSuccess.html
│   └── oauthError.html
├── Dockerfile           # Container configuration
├── docker-compose.yml   # Multi-container setup
└── README.md           # Basic documentation
```

## Key Files

### `main.go`
Application entry point that:
- Initializes MongoDB connection
- Loads existing team configurations
- Starts Slack RTM watchers for each team
- Launches HTTP server on port 8080

### `slackTeam/watchTeam.go`
Core message processing logic that:
- Establishes RTM connections to Slack
- Monitors messages for JIRA issue IDs using regex
- Posts JIRA links when matches are found
- Handles direct message commands

### `config/config.go`
Team configuration management including:
- `TeamConfig` struct with team settings
- Database operations for team data
- OAuth token storage and retrieval

### `routes/auth.go`
OAuth flow implementation:
- Initial authorization redirect to Slack
- Token exchange and validation
- Team registration in database

## Dependencies

### Key External Libraries
- **Gin Web Framework**: `gopkg.in/gin-gonic/gin.v1` - HTTP routing and middleware
- **Slack API Client**: `github.com/nlopes/slack` - Slack RTM and Web API integration
- **MongoDB Driver**: `gopkg.in/mgo.v2` - Database operations and models
- **Standard Library**: `regexp`, `strings`, `fmt`, `log`, `os`, `net/url`, `sync`

### Development Dependencies
- **Fresh**: `github.com/pilu/fresh` - Auto-restart during development (Docker only)

## Coding Conventions

### Go Standards
- **Package Structure**: Logical separation by functionality
- **Error Handling**: Explicit error returns with descriptive messages
- **Naming**: PascalCase for exported functions, camelCase for private
- **Comments**: Public functions documented with purpose and behavior

### Database Conventions
- **MongoDB Collections**: Snake_case field names in structs
- **BSON Tags**: Proper tagging for MongoDB document mapping
- **ObjectID**: Uses `bson.ObjectId` for primary keys

### Slack Integration Patterns
- **RTM Connections**: Goroutines for concurrent team monitoring
- **Message Filtering**: Ignore self-messages and check channel types
- **Token Management**: Secure storage and validation of OAuth tokens

## Configuration

### Environment Variables
The application requires these environment variables:

```bash
# Slack OAuth Configuration
SLACK_BOT_ACCESS_TOKEN=xoxb-...     # Bot token (obtained via OAuth)
SLACK_CLIENT_ID=1234567890.123      # Slack app client ID
SLACK_CLIENT_SECRET=abcdef123456    # Slack app client secret

# Application Settings
PUBLIC_HOST=localhost:8080          # Public hostname for OAuth redirects
APP_ENV=development                 # Environment (development/production)
```

### Database Configuration
- **MongoDB**: Connects to `mongodb://mongo:27017` (Docker) or localhost
- **Database Name**: `kirk`
- **Collection**: `config` (stores team configurations)

## Development Workflow

### Local Development Setup
1. **Environment Setup**: Create `.env` file with required variables
2. **Database**: Use Docker Compose for MongoDB
3. **Development Mode**: Set `APP_ENV=development` for auto-restart
4. **Run Application**: 
   ```bash
   # With Docker Compose
   docker-compose up
   
   # Direct Go execution
   go run main.go router.go
   ```

### Container Development
The project includes Docker configuration for consistent development:
- **Dockerfile**: Multi-stage build with Go 1.9.2
- **docker-compose.yml**: Application + MongoDB setup
- **Volume Mounting**: Live code reloading in development mode

### Build Process
```bash
# Install dependencies
go get ./

# Build binary
go build

# Run tests (if any exist)
go test ./...
```

## Common Tasks

### Adding New Slack Commands
1. Add command case to `commands/commands.go:HandleCommand()`
2. Implement command function in appropriate file
3. Update help text in `commands/help.go`

### Modifying JIRA Link Detection
1. Update regex patterns in `slackTeam/watchTeam.go`
2. Modify `TeamConfig` struct in `config/config.go` for new settings
3. Add configuration commands for new settings

### Adding New OAuth Scopes
1. Update `oauthScopes` array in `routes/auth.go`
2. Handle new permissions in message processing logic
3. Update documentation for required permissions

### Database Schema Changes
1. Modify `TeamConfig` struct in `config/config.go`
2. Add migration logic if needed
3. Update related database operations

## Important Context

### Legacy Dependencies
- **Go Version**: Built for Go 1.9.2 (consider updating for modern Go features)
- **mgo Driver**: Uses legacy MongoDB driver (consider migrating to official driver)
- **Gin Version**: Uses older v1 version of Gin framework

### Slack API Considerations
- **RTM API**: Uses Real-Time Messaging API (consider Events API for scale)
- **Rate Limiting**: No explicit rate limiting implemented
- **Token Revocation**: TODO item to handle token revocation events

### Security Notes
- **Environment Variables**: Sensitive tokens stored in environment
- **OAuth State**: No CSRF protection in OAuth flow
- **Input Validation**: Limited input sanitization on commands

### Performance Considerations
- **Database Queries**: Config fetched on every message (caching TODO)
- **Goroutines**: One goroutine per team for RTM connections
- **Regex Performance**: Compiled regex patterns for JIRA ID matching

### MongoDB Connection
- **Connection String**: Hardcoded to `mongodb://mongo:27017`
- **Session Management**: Single session shared across operations
- **Error Handling**: Panics on connection failure

### Docker Considerations
- **Development Mode**: Uses `fresh` for auto-restart
- **Volume Mounting**: Source code mounted for live reloading
- **Port Exposure**: Application runs on port 8080
- **Database Persistence**: MongoDB data persisted to host filesystem

This architecture provides a solid foundation for a Slack-JIRA integration bot with room for improvements in error handling, testing, and modern Go practices.