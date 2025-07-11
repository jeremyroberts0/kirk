# Kirk - Slack-JIRA Integration Bot

## Project Overview

Kirk is a Slack bot designed to integrate Slack and JIRA. The bot automatically posts links to JIRA when a JIRA issue ID is detected in channels where Kirk is a member. It provides OAuth-based authentication for Slack teams and maintains team configurations in a MongoDB database.

### Key Features
- Automatic JIRA issue link posting when issue IDs are mentioned in Slack
- OAuth 2.0 integration with Slack
- Multi-team support with per-team configuration
- Real-time message monitoring via Slack RTM API
- Web-based configuration interface
- MongoDB persistence for team configurations

## Architecture

Kirk follows a modular Go application architecture:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Slack RTM     │    │   HTTP Server   │    │   MongoDB       │
│   (Real-time    │    │   (OAuth &      │    │   (Team Config  │
│    Messages)    │    │    Config)      │    │    Storage)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Kirk Main     │
                    │   Application   │
                    └─────────────────┘
```

### Core Components

1. **Main Application** (`main.go`): Entry point that initializes MongoDB, recreates bot instances, and starts the HTTP server
2. **HTTP Router** (`router.go`): Gin-based HTTP server for OAuth flows and API endpoints
3. **Slack Team Watcher** (`slackTeam/`): Manages real-time Slack connections and message processing
4. **OAuth Routes** (`routes/`): Handles Slack OAuth authentication flow
5. **Commands** (`commands/`): Processes bot commands from Slack users
6. **Configuration** (`config/`): Manages team-specific configurations in MongoDB
7. **Database** (`db/`): MongoDB connection and session management

## Directory Structure

```
kirk/
├── main.go              # Application entry point
├── router.go            # HTTP router setup
├── Dockerfile           # Container configuration
├── docker-compose.yml   # Development environment setup
├── commands/            # Bot command handlers
│   ├── commands.go      # Command dispatcher
│   ├── config.go        # Config command implementation
│   └── help.go          # Help command implementation
├── config/              # Team configuration management
│   └── config.go        # MongoDB operations for team configs
├── db/                  # Database connection
│   └── db.go            # MongoDB session management
├── html/                # HTML templates
│   ├── oauthError.html  # OAuth error page
│   └── oauthSuccess.html # OAuth success page
├── routes/              # HTTP route handlers
│   ├── auth.go          # OAuth authentication flow
│   └── info.go          # Health check endpoint
└── slackTeam/           # Slack integration
    └── watchTeam.go     # Real-time message monitoring
```

## Key Files

### Core Application Files
- **`main.go`**: Application bootstrap, MongoDB initialization, bot instance creation
- **`router.go`**: HTTP server setup with Gin framework
- **`slackTeam/watchTeam.go`**: Real-time Slack message monitoring and JIRA issue detection

### Configuration & Data
- **`config/config.go`**: Team configuration CRUD operations, MongoDB interactions
- **`db/db.go`**: MongoDB connection management
- **`docker-compose.yml`**: Development environment with MongoDB

### HTTP Routes
- **`routes/auth.go`**: Slack OAuth 2.0 authentication flow
- **`routes/info.go`**: Health check endpoint (`/info`)

### Bot Commands
- **`commands/commands.go`**: Command routing and dispatch
- **`commands/config.go`**: Team configuration commands
- **`commands/help.go`**: Help command implementation

## Coding Conventions

### Go Standards
- Standard Go project structure with package-based organization
- Descriptive package names (`slackTeam`, `commands`, `config`)
- Pascal case for exported functions and types
- Camel case for internal functions and variables

### Error Handling
- Panic for critical initialization errors (MongoDB connection, OAuth setup)
- Error return values for recoverable operations
- Logging for debugging and monitoring

### Configuration Pattern
- Environment variables for sensitive configuration (tokens, secrets)
- Database storage for team-specific settings
- Struct-based configuration with BSON tags for MongoDB

### Slack Integration Patterns
- Real-time API (RTM) for message monitoring
- OAuth 2.0 flow for team authorization
- Bot user tokens for API access

## Dependencies

### Core Dependencies
- **Gin** (`gopkg.in/gin-gonic/gin.v1`): HTTP web framework
- **Slack Go SDK** (`github.com/nlopes/slack`): Slack API client
- **MongoDB Go Driver** (`gopkg.in/mgo.v2`): MongoDB database driver

### Runtime Dependencies
- **Go 1.9.2+**: Application runtime
- **MongoDB**: Team configuration storage
- **Docker**: Containerization and development environment

## Development Workflow

### Local Development Setup
1. **Environment Variables**: Create `.env` file with required tokens:
   ```
   SLACK_BOT_ACCESS_TOKEN=xoxb-...
   SLACK_CLIENT_ID=...
   SLACK_CLIENT_SECRET=...
   PUBLIC_HOST=localhost:8080
   APP_ENV=development
   ```

2. **Start Development Environment**:
   ```bash
   docker-compose up
   ```

3. **Development Mode**: Set `APP_ENV=development` for auto-reload with `fresh`

### Building & Deployment
- **Docker Build**: `docker build .`
- **Direct Build**: `go build` (produces `kirk` binary)
- **Production Deploy**: Use Docker container with production environment variables

### Testing
- No formal test suite currently implemented
- Manual testing via Slack integration
- Health check available at `/info` endpoint

## Common Tasks

### Adding New Bot Commands
1. Add command case to `commands/commands.go`
2. Implement command handler in appropriate file
3. Update help text in `commands/help.go`

### Modifying Team Configuration
1. Update `TeamConfig` struct in `config/config.go`
2. Add database migration logic if needed
3. Update config command handlers in `commands/config.go`

### Adding New HTTP Routes
1. Create route handler in `routes/` directory
2. Register route in `router.go`
3. Add HTML templates in `html/` if needed

### Slack Integration Changes
1. Modify `slackTeam/watchTeam.go` for message processing
2. Update OAuth scopes in `routes/auth.go` if needed
3. Test with development Slack app

## Important Context

### Legacy Go Version
- Uses Go 1.9.2 (older version)
- Uses older dependency management (no go.mod)
- Dependencies fetched via `go get`

### MongoDB Connection
- Hardcoded connection to `mongodb://mongo:27017`
- Database name: `kirk`
- Collection: `config` for team configurations

### Slack OAuth Scopes
Required scopes for bot functionality:
- `channels:read`, `chat:write:bot`, `team:read`
- `channels:history`, `groups:history`, `im:history`
- `im:read`, `im:write`, `mpim:history`, `bot`

### Development vs Production
- Development mode uses `fresh` for auto-reload
- Production mode runs compiled binary
- Environment detection via `APP_ENV` variable

### Security Considerations
- OAuth tokens stored in database
- Environment variables for sensitive data
- No encryption at rest mentioned in codebase

### Potential Gotchas
- MongoDB connection must be available before startup
- Missing `.env` file will cause runtime errors
- OAuth redirect URI must match `PUBLIC_HOST` setting
- Bot must be invited to channels to monitor messages
- JIRA integration logic appears to be in message processing but details not fully visible in current codebase