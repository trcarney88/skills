---
name: linear-graphql-ops
description: Operate Linear via GraphQL for project and issue lifecycle tasks including project docs, milestones, statuses, and comments.
license: MIT
compatibility: opencode
metadata:
  audience: engineers
  api: linear-graphql
---

# Linear GraphQL Ops

Use this skill to perform common Linear project and issue operations through the GraphQL API.

## Prerequisites

- `LINEAR_API_KEY` must be set in the environment.
- Use endpoint `https://api.linear.app/graphql`.
- Send headers:
  - `Content-Type: application/json`
  - `Authorization: $LINEAR_API_KEY`

## When to use me

- Create or update projects.
- Pull projects and list issues for a project.
- Create or edit issues.
- Create milestones and assign issues to milestones.
- Change issue state or project status.
- Add a document to a project.
- Add comments to issues.

## Core workflow

1. Resolve IDs from human names first.
2. Execute the target query or mutation.
3. Read back updated entities to verify.
4. Return IDs and human-friendly identifiers.

## API helper patterns

```bash
# Basic request pattern
curl -sS https://api.linear.app/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: $LINEAR_API_KEY" \
  --data '{"query":"query { viewer { id name email } }"}'
```

```bash
# Variable-based request pattern (recommended)
curl -sS https://api.linear.app/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: $LINEAR_API_KEY" \
  --data '{"query":"query($name:String!){ projects(first:10, filter:{name:{eq:$name}}){ nodes { id name state } } }","variables":{"name":"Home Depot Ordering Integration"}}'
```

## ID resolution recipes

### Resolve project by exact name

```graphql
query ResolveProjectByName($name: String!) {
  projects(first: 50, filter: { name: { eq: $name } }) {
    nodes {
      id
      name
      state
      teams {
        nodes {
          id
          key
          name
        }
      }
    }
  }
}
```

### Resolve issue by identifier (`TEAM-123`)

Parse into `teamKey=TEAM` and `number=123`.

```graphql
query ResolveIssueByIdentifier($teamKey: String!, $number: Float!) {
  issues(first: 1, filter: { team: { key: { eq: $teamKey } }, number: { eq: $number } }) {
    nodes {
      id
      identifier
      title
      team {
        id
        key
      }
      state {
        id
        name
        type
      }
      project {
        id
        name
      }
      projectMilestone {
        id
        name
      }
    }
  }
}
```

### Resolve milestone by name within a project

```graphql
query ResolveMilestone($projectId: ID!, $name: String!) {
  projectMilestones(first: 50, filter: { project: { id: { eq: $projectId } }, name: { eq: $name } }) {
    nodes {
      id
      name
      targetDate
      project {
        id
        name
      }
    }
  }
}
```

### Resolve issue states for a team

```graphql
query ResolveWorkflowStates($teamId: ID!) {
  workflowStates(first: 100, filter: { team: { id: { eq: $teamId } } }) {
    nodes {
      id
      name
      type
      team {
        id
        key
      }
    }
  }
}
```

### Resolve project statuses

```graphql
query ResolveProjectStatuses {
  projectStatuses(first: 50) {
    nodes {
      id
      name
      type
      position
    }
  }
}
```

## Operation recipes

### Create project

```graphql
mutation CreateProject($input: ProjectCreateInput!) {
  projectCreate(input: $input) {
    success
    project {
      id
      name
      state
      url
    }
  }
}
```

Minimal input:

```json
{
  "name": "My Project",
  "teamIds": ["<team-id>"]
}
```

### Pull project and its issues

Use two-step reads to avoid complexity errors.

```graphql
query PullProject($name: String!) {
  projects(first: 50, filter: { name: { eq: $name } }) {
    nodes {
      id
      name
      state
      description
      teams {
        nodes {
          id
          key
          name
        }
      }
    }
  }
}
```

```graphql
query PullIssuesForProject($projectId: ID!, $after: String) {
  issues(first: 100, after: $after, filter: { project: { id: { eq: $projectId } } }) {
    nodes {
      id
      identifier
      title
      priority
      state {
        id
        name
        type
      }
      assignee {
        id
        name
      }
      projectMilestone {
        id
        name
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Create issue on project

```graphql
mutation CreateIssue($input: IssueCreateInput!) {
  issueCreate(input: $input) {
    success
    issue {
      id
      identifier
      title
      url
      team {
        id
        key
      }
      project {
        id
        name
      }
    }
  }
}
```

Minimal input:

```json
{
  "teamId": "<team-id>",
  "projectId": "<project-id>",
  "title": "New issue title"
}
```

### Create milestone on project

```graphql
mutation CreateMilestone($input: ProjectMilestoneCreateInput!) {
  projectMilestoneCreate(input: $input) {
    success
    projectMilestone {
      id
      name
      targetDate
      project {
        id
        name
      }
    }
  }
}
```

Minimal input:

```json
{
  "projectId": "<project-id>",
  "name": "M1"
}
```

### Add issue to milestone

```graphql
mutation AddIssueToMilestone($id: String!, $milestoneId: String!) {
  issueUpdate(id: $id, input: { projectMilestoneId: $milestoneId }) {
    success
    issue {
      id
      identifier
      title
      projectMilestone {
        id
        name
      }
    }
  }
}
```

### Edit issue

```graphql
mutation EditIssue($id: String!, $input: IssueUpdateInput!) {
  issueUpdate(id: $id, input: $input) {
    success
    issue {
      id
      identifier
      title
      description
      priority
      dueDate
      assignee {
        id
        name
      }
    }
  }
}
```

### Update issue status

Resolve `stateId` from team workflow states, then:

```graphql
mutation UpdateIssueStatus($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) {
    success
    issue {
      id
      identifier
      state {
        id
        name
        type
      }
    }
  }
}
```

### Update project status

Resolve `statusId` from project statuses, then:

```graphql
mutation UpdateProjectStatus($id: String!, $statusId: String!) {
  projectUpdate(id: $id, input: { statusId: $statusId }) {
    success
    project {
      id
      name
      state
      status {
        id
        name
        type
      }
    }
  }
}
```

### Add document to project

```graphql
mutation AddProjectDocument($input: DocumentCreateInput!) {
  documentCreate(input: $input) {
    success
    document {
      id
      title
      url
      project {
        id
        name
      }
    }
  }
}
```

Minimal input:

```json
{
  "title": "Project Notes",
  "content": "# Notes\n\nInitial draft.",
  "projectId": "<project-id>"
}
```

### Add comment to issue

```graphql
mutation AddIssueComment($input: CommentCreateInput!) {
  commentCreate(input: $input) {
    success
    comment {
      id
      body
      url
      issue {
        id
        identifier
      }
    }
  }
}
```

Minimal input:

```json
{
  "issueId": "<issue-id>",
  "body": "Adding context from integration testing."
}
```

## Guardrails

- Prefer exact matches for project and milestone names.
- If multiple matches are found, stop and ask the user to choose.
- Always resolve IDs before mutations.
- Keep list queries shallow and paginate to avoid complexity errors.
- For status updates, match by exact state/status name (case-insensitive fallback if exact fails).
- Return mutation `success` and read back the entity to confirm final state.

## Troubleshooting

- `LINEAR_API_KEY is not set`: export it in your shell.
- `Query too complex`: split nested queries; fetch project and issues separately.
- `GRAPHQL_VALIDATION_FAILED`: inspect field shape and avoid selecting subfields from scalar fields.
- Empty results: check team key, project name, and archived filters.
