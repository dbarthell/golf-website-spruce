---
modes:
  claude:
    binary: claude
    args:
      - --mcp-config
      - '{"mcpServers":{"spruce":{"command":"{{sidecar_path_jsonsafe}}","args":["mcp","serve"]}}}'
      - --system-prompt
      - "{{action_content}}"
      - --allowedTools
      - "Read,Glob,Grep,Edit,Write,Bash,mcp__spruce__*"
  codex:
    binary: codex
    args:
      - -c
      - 'mcp_servers.spruce.command="{{sidecar_path_jsonsafe}}"'
      - -c
      - 'mcp_servers.spruce.args=["mcp","serve"]'
      - -c
      - 'mcp_servers.spruce.default_tools_approval_mode="approve"'
      - --sandbox
      - workspace-write
      - -a
      - never
      - "{{action_content}}"
  kiro:
    binary: kiro-cli
    args:
      - chat
      - --trust-tools
      - "read,glob,grep,write,shell,@spruce"
      - "{{action_content}}"
  opencode:
    binary: opencode
    args:
      - --prompt
      - "{{action_content}}"
    env:
      OPENCODE_CONFIG_CONTENT: '{"mcp":{"spruce":{"type":"local","command":["{{sidecar_path_jsonsafe}}","mcp","serve"]}}}'
  gemini:
    binary: gemini
    args:
      - --allowed-mcp-server-names
      - spruce
      - --prompt-interactive
      - "{{action_content}}"
    session_files:
      gemini_settings:
        content: '{"mcpServers":{"spruce":{"command":"{{sidecar_path_jsonsafe}}","args":["mcp","serve"]}}}'
    env:
      GEMINI_CLI_SYSTEM_SETTINGS_PATH: "{{session_file:gemini_settings}}"
forward:
  - comment-added
display:
  color: '#3b82f6'
  icon: terminal-2
---
You are a software developer working on a Spruce project.

The current date and time is: {{current_time}}

Your session ID is {{session_id}}.

## Current Context

You are viewing artifact: {{artifact_id}}

Get it using `mcp__spruce__artifact_get` to understand what you're working with.

{{context:artifact-system}}

{{context:comment-system}}

The user will tell you what they need help with.
