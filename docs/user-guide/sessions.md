# Sessions

EpidBot supports multiple chat sessions with persistent history, sharing, and export.

## Sessions Page

Click **Sessions** in the sidebar to see all your conversations. Each session card displays:

- Session name
- Message count
- Last activity time (e.g., "2 hours ago")
- **Shared** badge if the session has been shared with or by others

### Session Actions

| Button | Action |
|--------|--------|
| New Session | Create a new conversation |
| Rename | Rename the current session (pencil icon) |
| Delete | Delete the current session (trash icon, with confirmation) |
| Resume | Click the session card to open the conversation |

### Collapsible Sidebar

In the chat view, the session sidebar can be **collapsed** by clicking the collapse button in its header. This gives you more space for the chat. Click the expand button to bring it back.

## Sharing Sessions

You can share sessions with other EpidBot users:

1. Click the **share icon** on a session card
2. Type a **username** in the dialog (autocomplete will suggest matches)
3. Choose a permission level:
   - **Can view** — read-only access to the conversation
   - **Can edit** — full access to continue the conversation
4. Click **Share**

To revoke access, open the share dialog and click **Revoke access** next to the user's name.

### Shared Session Restrictions

- Sessions shared with you as "read-only" cannot be modified — the input area is replaced with a lock icon and message
- Sessions received from others cannot be renamed, deleted, or re-shared
- When someone shares a session with you, a blue notification banner appears at the top of your chat

## Export Sessions

### Export Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| Markdown | `.md` | Human-readable, includes images |
| JSON | `.json` | Machine-readable, full data |
| HTML | `.html` | Standalone web page |
| TXT | `.txt` | Plain text |
| PDF | `.pdf` | Formatted document with embedded images |

### Export Steps

1. Open the session you want to export
2. Click the **Export** button at the top of the chat
3. Select the format (PDF, Markdown, etc.)
4. Download the file

!!! note
    Export lives in the chat view. The Sessions page itself offers create, rename, delete, share and **add-to-project** actions on session cards, but no export controls.

## Organizing Sessions into Projects

Use **Add to project** on a session card to link it to a [project](../tools/collaboration-tools.md) alongside related reports, plots and datasets.

## Search Sessions

Search across all sessions:

1. Navigate to **Search** in the sidebar
2. Enter search query
3. Optionally filter by session or date range
4. Click on a result to navigate to that conversation

## Session Management

### Automatic Behavior

- A default session is created automatically
- Sessions are saved to PostgreSQL
- Messages include timestamps
- Images are stored as BLOBs with file references

### Best Practices

1. **Create topic-specific sessions**: Separate sessions for different analyses
2. **Use descriptive names**: Name sessions after the analysis topic
3. **Export important sessions**: Save valuable conversations
4. **Clean up periodically**: Delete old, unneeded sessions
5. **Share for collaboration**: Use sharing to work with colleagues on analyses

## Next Steps

- [Chat Interface](chat.md)
- [My Data](my-data.md)
- [My Plots](my-plots.md)
