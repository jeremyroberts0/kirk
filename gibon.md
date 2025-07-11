# Kirk - Slack JIRA Integration Bot

## Project Overview

Kirk is a Slack bot that integrates Slack with JIRA, providing automated JIRA ticket linking and management capabilities. The bot monitors Slack channels for JIRA ticket IDs and automatically posts links to those tickets. It also provides configuration commands through direct messages.

### Key Features
- Automatically detects JIRA issue IDs in Slack channels and posts links
- OAuth-based Slack integration with multi-team support
- Direct message commands for configuration management
- MongoDB storage for team configurations
- Docker-based deployment with development mode support

## Architecture

Kirk follows a modular Go architecture with the following key components:

### Core Components
- **HTTP Server**: Gin-based web server handling OAuth flows and info endpoints
- **Real-time Messaging**: Slack RTM (Real-Time Messaging) client for monitoring channels
- **Database Layer**: MongoDB integration for storing team configurations
- **Command System**: Pluggable command handlers for bot interactions

### Data Flow
1. Teams authenticate via OAuth flow (`/authorize` → Slack → `/authorized`)
2. Bot instances are created for each authenticated team
3. RTM clients monitor channels for JIRA ticket patterns
4. Configuration is managed through direct message commands

## Directory Structure

```
.
├── commands/           # Bot command handlers
│   ├── commands.go     # Command dispatcher
│   ├── config.go       # Configuration commands
│   └── help.go         # Help command
├── config/             # Team configuration management
│   └── config.go       # Database models and operations
├── db/                 # Database connection and setup
│   └── db.go           # MongoDB connection initialization
├── html/               # HTML templates for OAuth flow
│   ├── oauthError.html
│   └── oauthSuccess.html
├── routes/             # HTTP route handlers
│   ├── auth.go         # OAuth authentication routes
│   └── info.go         # Health check endpoint
├── slackTeam/          # Slack team management
│   └── watchTeam.go    # RTM client and message processing
├── main.go             # Application entry point
├── router.go           # HTTP router configuration
├── Dockerfile          # Container configuration
└── docker-compose.yml  # Multi-container setup
```

## Key Files

### Core Application Files
- **`main.go`**: Application entry point, initializes database, creates bot instances, starts HTTP server
- **`router.go`**: HTTP router setup with template loading and route registration
- **`slackTeam/watchTeam.go`**: Core bot logic for monitoring Slack channels and processing messages

### Configuration & Database
- **`config/config.go`**: Team configuration model and database operations
- **`db/db.go`**: MongoDB connection and database initialization
- **`routes/auth.go`**: OAuth flow implementation with Slack integration

### Commands
- **`commands/commands.go`**: Command dispatcher for bot interactions
- **`commands/config.go`**: Configuration management commands
- **`commands/help.go`**: Help text and command documentation

## Coding Conventions

### Go Style
- Package names use lowercase, single words where possible
- Exported functions use PascalCase
- Private functions use camelCase
- Error handling follows Go idioms with explicit error returns

### Code Organization
- Each major feature area has its own package
- Database operations are centralized in the `config` package
- HTTP handlers are separated by functionality in the `routes` package
- Bot commands are modular and handled in the `commands` package

### Naming Conventions
- Database collections use lowercase names (e.g., "config")
- Environment variables use UPPERCASE_WITH_UNDERSCORES
- Route paths use lowercase with hyphens (e.g., "/authorized")

## Dependencies

### Core Dependencies
- **Gin Web Framework**: `gopkg.in/gin-gonic/gin.v1` - HTTP server and routing
- **Slack Go SDK**: `github.com/nlopes/slack` - Slack API integration
- **MongoDB Driver**: `gopkg.in/mgo.v2` - Database operations
- **Docker**: Container runtime and orchestration

### Development Dependencies
- **Fresh**: `github.com/pilu/fresh` - Auto-reload during development
- **MongoDB**: Document database for storing team configurations

## Development Workflow

### Environment Setup
1. Set required environment variables in `.env` file:
   ```
   SLACK_BOT_ACCESS_TOKEN=<slack-bot-token>
   SLACK_CLIENT_ID=<slack-client-id>
   SLACK_CLIENT_SECRET=<slack-client-secret>
   PUBLIC_HOST=<public-hostname:port>
   APP_ENV=development  # Optional for development mode
   ```

### Local Development
```bash
# Start with Docker Compose (recommended)
docker-compose up

# Or build and run manually
go get ./...
go build
./kirk
```

### Production Deployment
```bash
# Build container
docker build -t kirk .

# Run with production environment
docker run -p 8080:8080 --env-file .env kirk
```

## Common Tasks

### Adding New Bot Commands
1. Add command handler in `commands/` package
2. Register command in `commands/commands.go` switch statement
3. Update help text in `commands/help.go`

### Extending Team Configuration
1. Add new fields to `TeamConfig` struct in `config/config.go`
2. Update database operations as needed
3. Add configuration commands in `commands/config.go`

### Adding New Routes
1. Create route handler in `routes/` package
2. Register route in `router.go`
3. Add any necessary HTML templates in `html/`

### Modifying Message Processing
1. Update `slackTeam/watchTeam.go` message event handling
2. Add new regex patterns or processing logic
3. Test with various message formats

## Important Context

### Database Schema
The application uses MongoDB with a single "config" collection storing team configurations:
```go
type TeamConfig struct {
    Id                  bson.ObjectId `bson:"_id,omitempty"`
    Team_id             string        `bson:"team_id,omitempty"`
    Jira_base_url       string        `bson:"jira_base_url,omitempty"`
    Subscribed_projects []string      `bson:"subscribed_projects,omitempty"`
    Access_token        string        `bson:"access_token,omitempty"`
}
```

### OAuth Flow
1. User visits `/authorize` and is redirected to Slack
2. Slack redirects to `/authorized` with authorization code
3. App exchanges code for access token
4. Token is stored in database and bot instance is created

### Message Processing
- Bot ignores its own messages to prevent loops
- Direct messages trigger command processing
- Channel messages are scanned for JIRA ticket patterns
- Only configured project IDs are linked

### Development Mode
- Set `APP_ENV=development` to enable auto-reload with Fresh
- MongoDB runs in a separate container via docker-compose
- Hot reloading watches for Go file changes

### Legacy Dependencies
- Uses older versions of dependencies (Go 1.9.2, gin.v1, mgo.v2)
- Consider updating to modern versions for new features
- No Go modules - uses older GOPATH-based dependency management

### Security Notes
- Access tokens are stored in database
- OAuth secrets should be kept secure
- No rate limiting or request validation implemented
- Consider adding authentication for configuration endpoints