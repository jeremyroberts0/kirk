# Kirk - Slack JIRA Integration Bot

## Project Overview

Kirk is a Go-based Slack bot that provides seamless integration between Slack and JIRA. The bot automatically detects JIRA issue IDs mentioned in Slack channels and posts direct links to those issues, making it easier for teams to reference and access JIRA tickets during conversations.

### Key Features
- **Automatic JIRA Link Detection**: Scans messages for JIRA issue IDs (e.g., PROJ-123) and posts clickable links
- **OAuth Integration**: Secure Slack app installation and authorization flow
- **Multi-team Support**: Handles multiple Slack teams with individual configurations
- **Direct Message Commands**: Supports configuration commands via DMs to the bot
- **Docker Deployment**: Containerized application with MongoDB persistence

## Architecture

Kirk follows a modular architecture with clear separation of concerns:

```
main.go → HTTP Server (Gin) + Bot Manager
├── routes/          → HTTP endpoints (OAuth, info)
├── slackTeam/       → Slack RTM event handling
├── commands/        → Bot command processors
├── config/          → Team configuration management
├── db/              → MongoDB connection and operations
└── html/            → OAuth flow templates
```

### Key Components

1. **HTTP Server**: Gin-based web server handling OAuth flows and webhooks
2. **Bot Manager**: Manages multiple Slack bot instances (one per team)
3. **Event Processor**: Real-time message processing and JIRA link detection
4. **Configuration System**: Per-team settings for JIRA base URLs and projects
5. **Database Layer**: MongoDB for storing team configurations and tokens

## Directory Structure

```
kirk/
├── main.go              # Application entry point
├── router.go            # HTTP route configuration
├── commands/            # Bot command handlers
│   ├── commands.go      # Command dispatcher
│   ├── config.go        # Configuration commands
│   └── help.go          # Help command
├── config/              # Team configuration management
│   └── config.go        # Config CRUD operations
├── db/                  # Database layer
│   └── db.go            # MongoDB connection
├── routes/              # HTTP route handlers
│   ├── auth.go          # OAuth endpoints
│   └── info.go          # Info endpoints
├── slackTeam/           # Slack integration
│   └── watchTeam.go     # RTM event handling
├── html/                # HTML templates
│   ├── oauthSuccess.html
│   └── oauthError.html
├── Dockerfile           # Container configuration
├── docker-compose.yml   # Development environment
└── README.md           # Project documentation
```

## Key Files

### Core Application Files
- **`main.go`**: Entry point that initializes MongoDB, creates bot instances, and starts the HTTP server
- **`router.go`**: Configures HTTP routes using Gin framework
- **`slackTeam/watchTeam.go`**: Core bot logic for handling Slack events and message processing

### Configuration & Database
- **`config/config.go`**: Team configuration management with MongoDB persistence
- **`db/db.go`**: MongoDB connection and database initialization

### HTTP Routes
- **`routes/auth.go`**: OAuth 2.0 flow implementation for Slack app installation
- **`routes/info.go`**: Basic info endpoints

### Bot Commands
- **`commands/commands.go`**: Command dispatcher for direct message commands
- **`commands/config.go`**: Configuration commands (set JIRA URL, manage projects)
- **`commands/help.go`**: Help command implementation

## Dependencies

Kirk uses the following key external dependencies:

- **`gopkg.in/gin-gonic/gin.v1`**: HTTP web framework
- **`github.com/nlopes/slack`**: Slack API client library
- **`gopkg.in/mgo.v2`**: MongoDB driver for Go

**Note**: This project uses pre-Go modules dependency management (`go get`). It does not have a `go.mod` file.

## Development Workflow

### Environment Setup
1. Create a `.env` file with required environment variables:
   ```
   SLACK_BOT_ACCESS_TOKEN=xoxb-your-token
   SLACK_CLIENT_ID=your-client-id
   SLACK_CLIENT_SECRET=your-client-secret
   PUBLIC_HOST=localhost:8080
   APP_ENV=development
   ```

### Local Development
```bash
# Using Docker Compose (recommended)
docker-compose up

# Manual setup
go get ./...
go build
./kirk
```

### Building
```bash
# Standard Go build
go build

# Docker build
docker build -t kirk .
```

### Testing
The project does not currently have automated tests. Testing is done manually through:
- OAuth flow verification
- Message processing in Slack channels
- Direct message command testing

## Coding Conventions

### Style Guidelines
- **Package Structure**: Each major feature has its own package
- **Error Handling**: Explicit error checking with panic for critical failures
- **Logging**: Uses Go's standard log package with custom prefixes
- **Configuration**: Environment variables for deployment settings

### Naming Conventions
- **Variables**: snake_case for struct fields (MongoDB compatibility)
- **Functions**: camelCase following Go conventions
- **Constants**: camelCase with descriptive names
- **Packages**: lowercase, single words

### Important Patterns
- **Goroutines**: Used for concurrent bot instances (one per team)
- **Channels**: Slack RTM events processed through channels
- **Struct Tags**: MongoDB BSON tags for data persistence
- **Error Propagation**: Explicit error returns from functions

## Common Tasks

### Adding New Bot Commands
1. Add command case to `commands/commands.go` HandleCommand function
2. Create command handler function in appropriate file
3. Update help text in `commands/help.go`

### Modifying JIRA Link Detection
Edit the regex pattern in `slackTeam/watchTeam.go` around line 92:
```go
issueIdRegex := "(" + strings.Join(teamConfig.Subscribed_projects, "|") + `)-\d+`
```

### Adding New HTTP Endpoints
1. Create handler function in appropriate `routes/` file
2. Register route in `router.go` getRouter function
3. Add any necessary HTML templates to `html/` directory

### Database Schema Changes
Modify struct definitions in `config/config.go` and update the corresponding MongoDB operations.

## Important Context

### Deployment Notes
- **Environment**: Uses `APP_ENV=development` for auto-reload with Fresh
- **Port**: Always runs on port 8080 (hardcoded)
- **Database**: Expects MongoDB at `mongodb://mongo:27017`
- **OAuth**: Requires publicly accessible host for Slack callbacks

### Known Limitations
- No automated testing suite
- Hard-coded MongoDB connection string
- Uses deprecated mgo.v2 driver
- No graceful shutdown handling
- Limited error recovery mechanisms

### Security Considerations
- OAuth tokens stored in MongoDB (ensure database security)
- Environment variables contain sensitive data
- No rate limiting on HTTP endpoints
- Bot has broad Slack permissions (see `routes/auth.go`)

### Bot Behavior Quirks
- Responds with periods in sentences (e.g., "I. don't know. how to handle. that command.")
- Only processes messages in channels where the bot is a member
- Ignores messages from itself to prevent loops
- Requires manual project configuration per team

This bot represents a focused, single-purpose integration that effectively bridges Slack and JIRA workflows for development teams.