# claude-jira-skill

A [Claude Code](https://claude.com/claude-code) skill for creating and logging Jira stories in a Jira Cloud project, following team conventions: required custom fields, epic, acceptance criteria, sprint and assignee. It can also post an end-of-day progress update on your tickets when you say "time to go home".

## What it does

- **Create stories** that always have an epic, acceptance criteria, the required stage field, story points (when required), the current user as assignee, and the board's active sprint. Claude shows you the full story and waits for a yes before creating it.
- **Log on a Jira**: add a progress comment, or log time (`1h 30m`) as a worklog. Change an issue's status when asked.
- **End-of-day update**: say "time to go home", "wrapping up", "end of day" or "EOD". Claude gathers today's work from your Claude Code sessions and today's git commits, matches it to your open sprint tickets, and drafts one `EOD update – <date>` comment per ticket. Nothing is posted until you approve. Work that matches no ticket is listed separately, with a suggestion to create a story.
- **Find Jiras** with JQL, shown as a table of key, summary, status and stage.

## Requirements

- Claude Code
- A Jira Cloud project with a Scrum board
- The Atlassian MCP server, added and authenticated:

  ```bash
  claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp
  ```

  Then run `/mcp` in Claude Code to sign in.

## Installation

Copy the `jira` folder into your Claude skills directory:

- Personal: `~/.claude/skills/jira/`
- Per project: `<project>/.claude/skills/jira/`

## Configuration

Edit `jira/SKILL.md` and replace these placeholders with your own values:

| Placeholder | Meaning | Example |
|---|---|---|
| `<SITE>` | Your Jira Cloud site URL | `https://your-team.atlassian.net` |
| `<PROJECT>` | Project key | `ABC` |
| `<BOARD_ID>` | Scrum board ID (from the board URL) | `2` |
| `<STAGE_FIELD>` | Display name of a required dropdown custom field, if your project has one | `Capability Stage` |

If your project has no required stage dropdown, remove that rule from the skill.

## Example prompts

- "Create a story for adding retry logic to the upload service"
- "Log 2h on ABC-42"
- "Add a comment to ABC-42: deployed to staging, smoke tests pass"
- "Move ABC-42 to Done"
- "Show my open Jiras"
- "Time to go home"

## Repository layout

```
README.md
jira/
  SKILL.md   # the skill Claude loads
```
