# Kirk - Slack/JIRA Integration Bot

## Project Overview

Kirk is a Go-based Slack bot that integrates Slack with JIRA. The bot automatically detects JIRA issue IDs in Slack messages and posts links to the corresponding JIRA issues. It provides a seamless way for teams to reference JIRA issues in Slack conversations.

### Key Features
- **Automatic JIRA Link Detection**: Detects JIRA issue IDs (e.g., PROJECT-123) in Slack messages
- **Multi-team Support**: Supports multiple Slack teams with individual configurations
- **OAuth Integration**: Uses Slack OAuth for secure team authorization
- **Direct Message Commands**: Supports configuration commands via direct messages
- **MongoDB Integration**: Stores team configurations and access tokens

## Architecture

Kirk follows a modular Go architecture with clear separation of concerns:

### Core Components
- **HTTP Server**: Gin-based web server handling OAuth and info endpoints
- **Real-time Messaging**: Slack RTM (Real-Time Messaging) for live message processing
- **Database Layer**: MongoDB integration for persistent storage
- **Configuration Management**: Team-specific settings and OAuth tokens
- **Command Processing**: Direct message command handling

### Data Flow
1. Teams authenticate via OAuth (`/authorize` → Slack → `/authorized`)
2. Bot instances are created for each authorized team
3. RTM connections monitor Slack channels for messages
4. JIRA issue IDs are detected using regex patterns
5. Links are posted back to relevant channels

## Directory Structure

```
/
├── main.go              # Application entry point
├── router.go            # HTTP router configuration
├── commands/            # Bot command handlers
│   ├── commands.go      # Command dispatcher
│   ├── config.go        # Configuration commands
│   └── help.go          # Help command
├── config/              # Team configuration management
│   └── config.go        # MongoDB config operations
├── db/                  # Database connectivity
│   └── db.go            # MongoDB initialization
├── routes/              # HTTP route handlers
│   ├── auth.go          # OAuth flow handlers
│   └── info.go          # Health check endpoint
├── slackTeam/           # Slack team management
│   └── watchTeam.go     # RTM message processing
├── html/                # HTML templates
│   ├── oauthError.html  # OAuth error page
│   └── oauthSuccess.html # OAuth success page
├── Dockerfile           # Container configuration
├── docker-compose.yml   # Development environment
└── README.md            # Project documentation
```

## Key Files

### Core Application Files
- **`main.go`**: Application entry point, initializes database, creates bot instances, starts HTTP server
- **`router.go`**: Configures Gin router with HTML templates and route handlers
- **`config/config.go`**: Manages team configurations, OAuth tokens, and JIRA settings
- **`slackTeam/watchTeam.go`**: Core bot logic for message processing and JIRA link detection

### Configuration Structure
```go
type TeamConfig struct {
    Id                  bson.ObjectId `bson:"_id,omitempty"`
    Team_id             string        `bson:"team_id,omitempty"`
    Jira_base_url       string        `bson:"jira_base_url,omitempty"`
    Subscribed_projects []string      `bson:"subscribed_projects,omitempty"`
    Access_token        string        `bson:"access_token,omitempty"`
}
```

### HTTP Endpoints
- **`GET /info`**: Health check endpoint
- **`GET /authorize`**: Initiates Slack OAuth flow
- **`GET /authorized`**: Handles OAuth callback from Slack

## Coding Conventions

### Go Standards
- Package names use lowercase
- Function names use PascalCase for exported functions
- Variable names use snake_case for struct fields (following MongoDB conventions)
- Error handling follows Go idioms (returning error as last value)

### Project-Specific Patterns
- Database operations centralized in respective package files
- HTTP handlers organized by functional area (auth, info)
- Bot commands use a dispatcher pattern in `commands/commands.go`
- Configuration management abstracted into dedicated package

### Code Style
- Uses `fmt.Println()` for logging (simple approach for this project)
- Panic-on-critical-error pattern for database connectivity
- Gin framework for HTTP routing and middleware
- MongoDB queries use `bson.M` for query construction

## Dependencies

### Core Dependencies
- **Gin Web Framework**: `gopkg.in/gin-gonic/gin.v1` - HTTP routing and middleware
- **Slack Go SDK**: `github.com/nlopes/slack` - Slack API integration
- **MongoDB Driver**: `gopkg.in/mgo.v2` - MongoDB connectivity and operations

### Development Dependencies
- **Fresh**: Hot-reloading for development (installed via Dockerfile)
- **Docker**: Containerization for consistent development environment
- **MongoDB**: Database for storing team configurations

### Runtime Requirements
- Go 1.9.2+ (based on Dockerfile)
- MongoDB instance
- Slack app with appropriate OAuth scopes

## Development Workflow

### Environment Setup
1. **Environment Variables**: Create `.env` file with required variables:
   ```
   SLACK_BOT_ACCESS_TOKEN=<your-token>
   SLACK_CLIENT_ID=<your-client-id>
   SLACK_CLIENT_SECRET=<your-client-secret>
   PUBLIC_HOST=localhost:8080
   APP_ENV=development
   ```

2. **Docker Development**: Use docker-compose for local development:
   ```bash
   docker-compose up
   ```

### Build Process
- **Local Build**: `go build` produces `kirk` executable
- **Docker Build**: Multi-stage build with development hot-reloading
- **Dependencies**: `go get ./` installs all dependencies

### OAuth Flow
1. Navigate to `http://localhost:8080/authorize`
2. Complete Slack OAuth authorization
3. Bot automatically starts monitoring the authorized team

## Common Tasks

### Adding New Commands
1. Add command handler in `commands/` directory
2. Register command in `commands/commands.go` switch statement
3. Update help text in `commands/help.go`

### Modifying JIRA Detection
- Edit regex patterns in `slackTeam/watchTeam.go`
- Modify `issueIdRegex` construction for different project patterns
- Update link generation logic in the message processing loop

### Adding New OAuth Scopes
- Update `oauthScopes` array in `routes/auth.go`
- Ensure new scopes are documented in README.md

### Database Schema Changes
- Modify `TeamConfig` struct in `config/config.go`
- Add migration logic if needed for existing teams
- Update related query and upsert operations

## Important Context

### Quirks and Gotchas
- **MongoDB Connection**: Uses `mongo` hostname (Docker service name) for database connection
- **Token Management**: Bot access tokens are stored in MongoDB, not user tokens
- **Regex Limitations**: Go regex doesn't support negative lookahead, so link detection uses workaround logic
- **Development Mode**: `APP_ENV=development` enables hot-reloading with Fresh
- **Port Exposure**: Application runs on port 8080 internally and externally

### Architecture Decisions
- **Single Binary**: All functionality packaged in one executable
- **In-Memory Bot Instances**: Each team gets a separate goroutine for RTM connection
- **Synchronous Processing**: Messages processed sequentially per team
- **Simple Logging**: Uses `fmt.Println()` rather than structured logging

### Performance Considerations
- **TODO**: Team config caching mentioned in code but not implemented
- **Database Queries**: Each message triggers a database query for team config
- **Concurrent Teams**: Multiple teams handled with separate goroutines
- **Memory Usage**: Bot instances maintained in memory for real-time processing

### Security Notes
- **OAuth Tokens**: Stored in MongoDB, ensure database security
- **Environment Variables**: Sensitive data should be in `.env` file (gitignored)
- **Public Host**: Must be accessible to Slack for OAuth callback
- **Bot Permissions**: Requires specific Slack scopes for channel access

### Legacy Considerations
- **Go Version**: Built for Go 1.9.2 (older version)
- **Dependencies**: Uses older versions of Gin and mgo libraries
- **Docker**: Uses older Docker compose version 2 syntax