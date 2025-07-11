# Kirk - Slack JIRA Integration Bot

## Project Overview

Kirk is a Slack bot designed to integrate Slack and JIRA. Its primary function is to automatically post links to JIRA issues when JIRA issue IDs are detected in Slack channels that Kirk is monitoring.

### Key Features
- **Automatic JIRA linking**: Detects JIRA issue IDs (e.g., PROJ-123) in Slack messages and posts links
- **Multi-team support**: Supports multiple Slack teams with individual configurations
- **OAuth integration**: Complete Slack OAuth flow for team authorization
- **Direct message commands**: Handles bot commands via direct messages
- **Team configuration**: Per-team settings for JIRA base URLs and subscribed projects

## Architecture

Kirk follows a modular Go architecture with clear separation of concerns:

### Core Components

1. **main.go**: Application entry point
   - Initializes MongoDB connection
   - Recreates bot instances for all configured teams
   - Starts HTTP server on port 8080

2. **router.go**: HTTP routing setup
   - Configures Gin web framework
   - Sets up HTML template loading
   - Registers authentication and info routes

3. **Database Layer** (`db/`)
   - MongoDB connection management
   - Single database instance ("kirk") shared across the application

4. **Configuration Management** (`config/`)
   - Team configuration storage and retrieval
   - JIRA settings per team
   - OAuth token management

5. **Slack Integration** (`slackTeam/`)
   - Real-time messaging (RTM) connection handling
   - Message processing and JIRA issue ID detection
   - Multi-team bot instance management

6. **Command Processing** (`commands/`)
   - Bot command handling for direct messages
   - Configuration commands
   - Help system

7. **HTTP Routes** (`routes/`)
   - OAuth authorization flow
   - Health check endpoints
   - Team onboarding

8. **Templates** (`html/`)
   - OAuth success/error pages

### Data Flow

1. **Team Onboarding**: Teams visit `/authorize` → Slack OAuth → Kirk receives token → Stores in MongoDB
2. **Message Processing**: Slack RTM → Kirk receives message → Regex matching → JIRA link posting
3. **Direct Commands**: User DMs Kirk → Command processing → Response back to user

## Directory Structure

```
/
├── main.go              # Application entry point
├── router.go            # HTTP routing configuration
├── Dockerfile           # Docker container setup
├── docker-compose.yml   # Multi-container orchestration
├── db/
│   └── db.go           # MongoDB connection management
├── config/
│   └── config.go       # Team configuration CRUD operations
├── slackTeam/
│   └── watchTeam.go    # Slack RTM event handling
├── commands/
│   ├── commands.go     # Command dispatcher
│   ├── config.go       # Configuration commands
│   └── help.go         # Help command
├── routes/
│   ├── auth.go         # OAuth authentication flow
│   └── info.go         # Health check endpoint
└── html/
    ├── oauthSuccess.html # OAuth success page
    └── oauthError.html   # OAuth error page
```

## Key Files

### main.go
- Application bootstrap
- MongoDB initialization
- Bot instance creation for all teams
- HTTP server startup

### slackTeam/watchTeam.go
- Core Slack integration logic
- RTM connection management
- Message processing and JIRA issue detection
- Direct message command handling

### config/config.go
- Team configuration data model
- Database operations for team settings
- OAuth token storage

### routes/auth.go
- Complete Slack OAuth flow implementation
- Team authorization and token exchange
- Bot instance initialization for new teams

## Data Models

### TeamConfig
```go
type TeamConfig struct {
    Id                  bson.ObjectId `bson:"_id,omitempty"`
    Team_id             string        `bson:"team_id,omitempty"`
    Jira_base_url       string        `bson:"jira_base_url,omitempty"`
    Subscribed_projects []string      `bson:"subscribed_projects,omitempty"`
    Access_token        string        `bson:"access_token,omitempty"`
}
```

## Dependencies

### Core Libraries
- **gin-gonic/gin**: Web framework for HTTP routing
- **nlopes/slack**: Slack API client and RTM handling
- **mgo.v2**: MongoDB driver for Go

### Development Dependencies
- **pilu/fresh**: Hot reloading for development (Docker only)

## Environment Variables

Required environment variables:
- `SLACK_BOT_ACCESS_TOKEN`: Bot user OAuth token
- `SLACK_CLIENT_ID`: Slack app client ID
- `SLACK_CLIENT_SECRET`: Slack app client secret
- `PUBLIC_HOST`: Publicly accessible hostname:port (e.g., `localhost:8080`)
- `APP_ENV`: Set to `development` for hot reloading

## Development Workflow

### Local Development
1. **Setup**: Create `.env` file with required environment variables
2. **Run**: Use `docker-compose up` to start Kirk and MongoDB
3. **Development Mode**: Set `APP_ENV=development` for hot reloading
4. **MongoDB**: Accessible on port 27017
5. **Kirk**: Accessible on port 8080

### Docker Deployment
- **Build**: Uses Go 1.9.2 base image
- **Commands**: `go get ./` followed by `go build`
- **Hot Reload**: Uses `fresh` in development mode
- **Dependencies**: Links to MongoDB container

### Build Process
```bash
# Dependencies
go get ./

# Build
go build

# Run
./kirk
```

## Coding Conventions

### Go Standards
- Standard Go project structure (pre-modules era)
- Package-based organization
- Exported functions use PascalCase
- Private functions use camelCase

### Error Handling
- Explicit error returns
- Panic for critical initialization failures
- Logging for debugging and monitoring

### Database Patterns
- MongoDB collections accessed via helper functions
- Upsert operations for team configurations
- Connection pooling through single session

## Common Tasks

### Adding New Bot Commands
1. Add command case to `commands/commands.go`
2. Implement command handler in appropriate file
3. Update help text in `commands/help.go`

### Adding New JIRA Projects
Use the `config` command in a direct message to Kirk:
```
config jira_base_url https://your-jira.atlassian.net
config subscribed_projects PROJ1,PROJ2,PROJ3
```

### Extending Message Processing
1. Modify regex patterns in `slackTeam/watchTeam.go`
2. Add new event handlers in the RTM message loop
3. Update response formatting logic

### Adding New HTTP Endpoints
1. Create route handler in `routes/` directory
2. Register route in `router.go`
3. Add corresponding HTML templates if needed

## Important Context

### Legacy Go Project
- This project predates Go modules (uses GOPATH)
- Import paths use full GitHub URLs
- Dependencies managed via `go get`

### Slack Integration Quirks
- Uses RTM (Real-Time Messaging) API, not modern Events API
- Bot must be added to channels to monitor messages
- Direct message detection uses channel ID prefix checking

### MongoDB Considerations
- Uses legacy `mgo.v2` driver (not official MongoDB driver)
- Single database instance shared across all teams
- No connection pooling configuration

### Docker Networking
- MongoDB accessible as `mongo:27017` within Docker network
- Kirk exposes port 8080 for external access
- Environment variables passed via `.env` file

### OAuth Flow
- Requires public hostname for redirect URLs
- Bot tokens stored per team in MongoDB
- New bot instances created automatically on team authorization

### Security Notes
- No authentication on HTTP endpoints (except OAuth)
- Environment variables contain sensitive tokens
- MongoDB runs without authentication in default setup