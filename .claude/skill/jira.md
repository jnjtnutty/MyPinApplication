# Jira MCP Skill

How to use the Jira MCP server (`mcp__jira__jira_get`, `mcp__jira__jira_post`, `mcp__jira__jira_put`) in this project.

## Setup

The Jira MCP server is pre-configured. No authentication step is needed at runtime.

- Project key: **SCRUM**
- Base API: `/rest/api/3`

## Reading a ticket

When the user gives a ticket key (e.g. `SCRUM-38`), fetch it immediately:

```
mcp__jira__jira_get
  path: /rest/api/3/issue/SCRUM-38
```

### Key fields to extract

| Field | Path | Purpose |
|---|---|---|
| Summary | `fields.summary` | Short title of the task |
| Description | `fields.description` | Full user story, acceptance criteria, design links |
| Status | `fields.status.name` | Current workflow state |
| Parent (Epic) | `fields.parent.fields.summary` | Which epic this belongs to |
| Priority | `fields.priority.name` | Task priority |
| Labels | `fields.labels` | e.g. `frontend`, `backend` |

### Optimized query with jq

For listing multiple tickets, always use `jq` to reduce token cost:

```
mcp__jira__jira_get
  path: /rest/api/3/search/jql
  queryParams: { "jql": "project=SCRUM AND status='Ready for AI'", "maxResults": "10" }
  jq: "issues[*].{key: key, summary: fields.summary, status: fields.status.name}"
```

## Extracting requirements from description

Jira descriptions use Atlassian Document Format (ADF). Key patterns:

- **User Story** — paragraph after the bold "User Story" heading
- **Figma link** — inside a `blockCard` node with `attrs.url` containing `figma.com/design/...`
- **Acceptance Criteria** — bullet list items prefixed with "AC 1:", "AC 2:", etc.
- **Technical Notes** — bullet list with implementation hints

### Figma URL extraction

Look for `blockCard` nodes in `fields.description.content[]`. The URL format is:

```
https://www.figma.com/design/<fileKey>/<fileName>?node-id=<nodeId>
```

Extract `fileKey` and `nodeId` (replace `-` with `:` in nodeId) to pass to the Figma MCP tools.

## Updating ticket status

After completing work, update the ticket status via transitions:

```
# 1. Get available transitions
mcp__jira__jira_get
  path: /rest/api/3/issue/SCRUM-38/transitions

# 2. Transition the ticket
mcp__jira__jira_post
  path: /rest/api/3/issue/SCRUM-38/transitions
  body: { "transition": { "id": "<transition-id>" } }
```

## Adding comments

```
mcp__jira__jira_post
  path: /rest/api/3/issue/SCRUM-38/comment
  body: {
    "body": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "paragraph",
          "content": [{ "type": "text", "text": "Implementation complete. Visual match score: 88%." }]
        }
      ]
    }
  }
```

## Workflow

1. **Read ticket** — extract user story, acceptance criteria, Figma link, technical notes
2. **Implement** — follow the design-to-code workflow in CLAUDE.md
3. **Verify** — check each acceptance criterion is met
4. **Update status** — transition ticket when done
5. **Comment** — leave a summary with match score and any deviations
