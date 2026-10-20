---
name: jira
description: Create and log Jira stories in a Jira Cloud project with team conventions (required custom fields, epic, acceptance criteria, sprint, assignee). Use when the user wants to create a Jira/ticket/story, log work or progress on a Jira, add a comment to an issue, or find their Jiras. Also use when the user says "time to go home" (or similar end-of-day wording) to collect today's progress and post it as comments on the matching Jira tickets.
---

# Jira

## Configuration

Before using this skill, replace these placeholders with your own values:

| Placeholder | Meaning | Example |
|---|---|---|
| `<SITE>` | Your Jira Cloud site URL | `https://your-team.atlassian.net` |
| `<PROJECT>` | Project key | `ABC` |
| `<BOARD_ID>` | Scrum board ID (from the board URL) | `2` |
| `<STAGE_FIELD>` | Display name of a required dropdown custom field, if your project has one | `Capability Stage` |

Board: `<SITE>/jira/software/projects/<PROJECT>/boards/<BOARD_ID>`

All calls go through the Atlassian MCP server's tools (`createJiraIssue`, `editJiraIssue`, `getJiraIssue`, `searchJiraIssuesUsingJql`, `addCommentToJiraIssue`, `addWorklogToJiraIssue`, `getJiraProjectIssueTypesMetadata`, `getJiraIssueTypeMetaWithFields`, `getTransitionsForJiraIssue`, `transitionJiraIssue`, `atlassianUserInfo`). Pass the site URL as `cloudId`. If no Atlassian tools are available, tell the user to run
`claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp` and authenticate via `/mcp`, then stop.

## Creating a story

Every story follows these rules:

1. **Issue type** is always `Story`.
2. **Field metadata first.** Before the first create in a session, call `getJiraProjectIssueTypesMetadata` for `<PROJECT>` to get the Story's numeric issue type ID. `getJiraIssueTypeMetaWithFields` needs the ID, not the name. Then call `getJiraIssueTypeMetaWithFields` with `requiredFieldsOnly: false`. From the result, note the `customfield_XXXXX` IDs and allowed values for every required field. These usually include the stage dropdown, story points, acceptance criteria and sprint.
3. **Stage dropdown** (`<STAGE_FIELD>`, if your project has one) is required. Set it as `{"customfield_XXXXX": {"value": "<option>"}}`, using an option spelled exactly as in the metadata. If the user didn't say which option, list the options and ask. Never guess.
4. **Epic** is required. In team-managed Jira Cloud projects, a story is linked to its epic through the `parent` field (`"parent": "<PROJECT>-NNN"`). Ask the user "which epic?" rather than "which parent?". If they didn't name one, search with `project = <PROJECT> AND issuetype = Epic AND statusCategory != Done ORDER BY updated DESC` and suggest the best matches. Never create a story without an epic.
5. **Story points**: if the metadata marks the story point field as required, ask for an estimate.
6. **Acceptance criteria** are required. If the metadata has an "Acceptance Criteria" custom field, put them there. Otherwise add an `## Acceptance Criteria` section at the end of the description. Write them as a checklist of testable pass/fail statements. If the user gave none, draft 3–5 and show them for confirmation.
7. **Assignee**: assign the story to the current user (get their account ID from `atlassianUserInfo`) unless they name someone else. Otherwise Jira may auto-assign it to someone else.
8. **Sprint**: put the story in the board's active sprint. Find it with `project = <PROJECT> AND sprint in openSprints()`, requesting the sprint field (often `customfield_10020`). Pick the entry with `"state": "active"` and `boardId` = `<BOARD_ID>`, and set the sprint field to its numeric `id`.
9. **Summary** is a short imperative title, under about 80 characters.
10. **Description** covers the context or goal (why) and the scope (what).

Before calling `createJiraIssue`, show the user the summary, epic, stage, story points, description and acceptance criteria in one compact block and get a yes. After creating it, reply with the issue key and its link: `<SITE>/browse/<KEY>`.

To create several stories at once, confirm them all in one table, then create them one by one.

## Logging on a Jira

"Log" can mean either of these. Infer which from wording, and ask if it's unclear:
- **Progress or notes**: use `addCommentToJiraIssue` with a concise update covering what was done, what's next, and any blockers. For completed technical work, include the exact commands or steps and their results, and say plainly if any acceptance criterion was not met.
- **Time**: use `addWorklogToJiraIssue` with `timeSpent` in Jira format (`1h 30m`) and an optional comment.

If the user names no key, search with JQL such as `project = <PROJECT> AND assignee = currentUser() AND statusCategory != Done ORDER BY updated DESC` and let them pick. To change status (for example to Done), get the transitions first, then call `transitionJiraIssue` with the matching transition ID.

## End of day: "time to go home"

When the user says "time to go home" (or "wrapping up", "end of day", "EOD"), collect today's progress and log it on the right tickets.

1. **Collect today's work.** Use local time and today's date only. Sources, in order:
   - Claude Code session transcripts changed today: `find ~/.claude/projects -name '*.jsonl' -newermt "$(date +%F)"`. Read the user's requests, what was done, and its results. Skim them; don't dump them.
   - Git commits made today by the user in repos they worked in: `git log --since=midnight --author="$(git config user.email)" --oneline`.
   - Anything the user says in the same message.
   Ignore failed attempts that were later redone, routine reads and chit-chat. Keep outcomes: what changed, what was verified, numbers, blockers, and what's next.
2. **Find candidate tickets.** Search `project = <PROJECT> AND assignee = currentUser() AND sprint in openSprints() ORDER BY updated DESC`, plus any issue keys mentioned in today's sessions or commit messages.
3. **Map the work to tickets.** Match each piece of work to a ticket by its key, summary, description or epic. Don't force a match. Work with no clear ticket goes in an "Unmapped" list, with a suggestion to create a story for it (following the rules above).
4. **Draft one comment per ticket**, headed `EOD update – <YYYY-MM-DD>`, with:
   - **Done today:** concrete outcomes, with commands, versions and results where they matter
   - **Next:** the next steps
   - **Blockers:** only if there are any
   Keep each comment short, a few bullets, and factual. Never claim something was finished or verified unless the session shows it was.
5. **Confirm before posting.** Show the user one table of ticket, summary and draft comment, plus the Unmapped list. Post only after they say yes, applying any edits they ask for. Never change status, log time or create stories in this flow unless the user asks.
6. **Post** with `addCommentToJiraIssue`, then reply with the list of tickets updated and their links.

## Finding Jiras

Use `searchJiraIssuesUsingJql`. Show the results as a short table of key, summary, status and stage.
