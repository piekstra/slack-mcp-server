# Slack MCP Server

A Model Context Protocol (MCP) server that provides tools for interacting with Slack workspaces through the Slack Web API.

<a href="https://glama.ai/mcp/servers/@piekstra/slack-mcp-server">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@piekstra/slack-mcp-server/badge" alt="Slack Server MCP server" />
</a>

## Features

### Channel Management
- **list_channels** - List all channels in the workspace
- **get_channel_info** - Get detailed information about a specific channel
- **create_channel** - Create a new channel
- **archive_channel** - Archive a channel
- **unarchive_channel** - Unarchive a channel
- **set_channel_topic** - Set channel topic
- **set_channel_purpose** - Set channel purpose

### User Management
- **list_users** - List all users in the workspace
- **get_user_info** - Get detailed information about a specific user
- **invite_to_channel** - Invite users to a channel

### Messaging
- **send_message** - Send a message to a channel (supports Block Kit)
- **update_message** - Update an existing message (supports Block Kit)
- **delete_message** - Delete a message
- **get_channel_history** - Get message history for a channel
- **search_messages** - Search for messages across the workspace

### Rich Message Formatting (Block Kit)
- **send_formatted_message** - Send formatted messages with headers, sections, fields
- **send_notification_message** - Send status notifications with emoji indicators
- **send_list_message** - Send formatted lists with titles and descriptions

### File Management
- **upload_file** - Upload a file to one or more channels

### Reactions
- **add_reaction** - Add a reaction to a message
- **remove_reaction** - Remove a reaction from a message

### Workspace
- **get_team_info** - Get information about the Slack workspace/team

## Installation

### Using pip (Recommended - Global Install)

For use with Claude Desktop or Claude CLI, install globally:

```bash
pip install slack-mcp-server
```

### From source

```bash
git clone <repository-url>
cd slack-mcp-server
pip install -e .
```

## Configuration

### Secure Credential Storage (Recommended)

The Slack MCP Server uses macOS Keychain for secure credential storage. This is much safer than storing tokens in files.

#### Quick Setup

```bash
# Run the interactive setup
slack-mcp-setup
# or
python -m slack_mcp.setup
```

The setup wizard will:
1. Guide you through obtaining a Slack API token
2. Securely store credentials in your macOS Keychain
3. Validate your configuration

#### Manual Keychain Setup

```python
from slack_mcp.credentials import CredentialManager

manager = CredentialManager()
manager.store_credential("api_token", "xoxb-your-slack-bot-token")
manager.store_credential("workspace_id", "T1234567890")  # Optional
```

### Environment Variables (Fallback)

If you prefer environment variables or are not on macOS, create a `.env` file:

```bash
# Required
SLACK_API_TOKEN=xoxb-your-slack-bot-token

# Optional
SLACK_WORKSPACE_ID=your-workspace-id
```

**⚠️ Security Warning**: Environment variables and `.env` files are less secure than keychain storage.

### Getting a Slack API Token

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Create a new app or select an existing one
3. Navigate to "OAuth & Permissions"
4. Add the required OAuth scopes (see below)
5. Install the app to your workspace
6. Copy the "Bot User OAuth Token" (starts with `xoxb-`)

### Required OAuth Scopes

The following OAuth scopes are required for full functionality:

**Core Scopes:**
- `channels:read` - View basic channel information
- `channels:manage` - Create and manage public channels
- `groups:read` - View private channel information
- `groups:write` - Create and manage private channels
- `chat:write` - Send messages
- `files:write` - Upload files
- `im:read` - View direct message channels
- `mpim:read` - View group direct message channels
- `reactions:read` - View reactions
- `reactions:write` - Add and remove reactions
- `team:read` - View team information
- `users:read` - View user information

**⚠️ Search Limitation:**
The `search_messages` tool uses Slack's search API which requires user tokens with the `search:read` scope. Since this server is designed for bot tokens (`xoxb-`), search functionality will not work. Consider removing this feature or switching to user token authentication if search is needed.

## Usage with Claude CLI (Recommended)

After installing globally, add the server using Claude CLI. First, find your slack-mcp-server installation path:

```bash
# Find your slack-mcp-server executable path
which slack-mcp-server
```

Then add it with Claude CLI:

```bash
claude mcp add slack /path/to/slack-mcp-server
```

For example:
```bash
claude mcp add slack /usr/local/bin/slack-mcp-server
```

This automatically configures the server in your Claude Desktop configuration.

## Usage with Claude Desktop (Manual Configuration)

Alternatively, you can manually add this to your `claude_desktop_config.json`:

#### With Keychain Storage (Recommended)

```json
{
  "mcpServers": {
    "slack": {
      "command": "python",
      "args": ["-m", "slack_mcp"]
    }
  }
}
```

#### With Environment Variables

```json
{
  "mcpServers": {
    "slack": {
      "command": "python",
      "args": ["-m", "slack_mcp"],
      "env": {
        "SLACK_API_TOKEN": "xoxb-your-slack-bot-token"
      }
    }
  }
}
```

## Usage with MCP Client

```python
from mcp import Client

# Initialize client
client = Client()

# Connect to the Slack MCP server
await client.connect("python", ["-m", "slack_mcp"])

# List available tools
tools = await client.list_tools()

# Use a tool
result = await client.call_tool("list_channels", {
    "types": "public_channel",
    "exclude_archived": True,
    "limit": 10
})
```

## Development

### Setting up the development environment

```bash
# Clone the repository
git clone <repository-url>
cd slack-mcp-server

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install in development mode with dev dependencies
pip install -e ".[dev]"

# Set up credentials securely
slack-mcp-setup
```

### Running tests

```bash
pytest
```

### Code formatting

```bash
# Format with black
black slack_mcp/

# Lint with ruff
ruff check slack_mcp/
```

## Examples

### List public channels

```python
result = await client.call_tool("list_channels", {
    "types": "public_channel",
    "exclude_archived": True,
    "limit": 20
})
```

### Send a message

```python
result = await client.call_tool("send_message", {
    "channel": "C1234567890",  # Channel ID
    "text": "Hello from MCP!"
})
```

### Search for messages

```python
result = await client.call_tool("search_messages", {
    "query": "project deadline",
    "sort": "timestamp",
    "sort_dir": "desc",
    "count": 10
})
```

### Upload a file

```python
result = await client.call_tool("upload_file", {
    "channels": "C1234567890,C0987654321",  # Comma-separated channel IDs
    "content": "This is the file content",
    "filename": "report.txt",
    "title": "Weekly Report",
    "initial_comment": "Here's this week's report"
})
```

### Send a formatted message with Block Kit

```python
result = await client.call_tool("send_formatted_message", {
    "channel": "C1234567890",
    "title": "Project Update",
    "text": "Here's the latest update on our project progress",
    "fields": "Status: In Progress, Due Date: Next Friday, Assignee: @john",
    "context": "Last updated 2 hours ago"
})
```

### Send a notification message

```python
result = await client.call_tool("send_notification_message", {
    "channel": "C1234567890",
    "status": "success",
    "title": "Deployment Complete",
    "description": "The application has been successfully deployed to production",
    "details": "Build #123 deployed at 14:30 UTC"
})
```

### Send a list message

```python
result = await client.call_tool("send_list_message", {
    "channel": "C1234567890",
    "title": "Meeting Agenda",
    "description": "Items to discuss in today's standup",
    "items": "Sprint review\nBlocker discussion\nNext week planning\nDemo preparation"
})
```

### Send message with custom Block Kit

```python
result = await client.call_tool("send_message", {
    "channel": "C1234567890",
    "text": "Custom formatted message",
    "blocks": json.dumps([
        {
            "type": "section",
            "text": {
                "type": "mrkdwn",
                "text": "*Important Notice*\nThis is a custom Block Kit message"
            }
        },
        {"type": "divider"},
        {
            "type": "context",
            "elements": [
                {
                    "type": "mrkdwn",
                    "text": "Created with custom blocks"
                }
            ]
        }
    ])
})
```

## Error Handling

The server returns JSON-formatted error messages when operations fail:

```json
{
  "error": "Slack API error: channel_not_found"
}
```

Common error codes:
- `not_authed` - Invalid or missing API token
- `channel_not_found` - Channel doesn't exist
- `user_not_found` - User doesn't exist
- `message_not_found` - Message doesn't exist
- `no_permission` - Bot lacks required permissions
- `rate_limited` - API rate limit exceeded

## Security Considerations

- **🔐 Keychain Storage**: Use macOS Keychain for secure credential storage (recommended)
- **API Token**: Never commit your Slack API token to version control
- **Permissions**: Only grant the minimum required OAuth scopes
- **Rate Limits**: The server respects Slack's rate limits (see [Slack Rate Limits](https://api.slack.com/docs/rate-limits))
- **Message Content**: Be mindful of sensitive information in messages and files

### Credential Management Commands

```bash
# Interactive setup wizard
slack-mcp-setup

# View stored credentials
python -c "from slack_mcp.credentials import CredentialManager; m = CredentialManager(); print(m.list_stored_credentials())"

# Delete specific credential
python -c "from slack_mcp.credentials import CredentialManager; CredentialManager().delete_credential('api_token')"
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see LICENSE file for details

## Support

For issues and questions, please create an issue in the repository.