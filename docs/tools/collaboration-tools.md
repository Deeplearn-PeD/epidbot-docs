# Collaboration & Workspace Tools

Tools for organizing work into projects, working with Git repositories, reusing code snippets and skills, and delegating to external agents.

## Projects

Projects group related artifacts — sessions, reports, figures and datasets — under one roof with per-member access. Managed in the UI under **Projects**; these tools let the agent organize artifacts for you.

| Tool | Description |
|------|-------------|
| `list_projects` | List the user's projects |
| `create_project` | Create a new project |
| `add_to_project` | Link a session, report, figure or dataset to a project |
| `remove_from_project` | Unlink an artifact from a project |
| `get_project_details` | Show a project and its linked artifacts |

Projects support roles (`project_roles` is a Pro feature), sharing and subscriptions — see the [Projects section of the manual](../user-guide/analysis.md).

## Git Integrations

Read and write access to the user's GitHub/GitLab repositories from the chat. Read tools are gated to Pro (`git_tools`); write tools additionally require `git_write`.

**Read (Pro):**

| Tool | Description |
|------|-------------|
| `git_list_repositories` | List the user's GitHub/GitLab repositories |
| `git_get_repository` | Get metadata for one repository |
| `git_list_repository_files` | List files in a repository |
| `git_get_file_content` | Read a file's content |
| `git_list_branches` | List branches |
| `git_list_commits` | List recent commits |
| `git_list_issues` | List issues |
| `git_get_issue` | Get one issue |
| `git_list_pull_requests` | List PRs (GitHub) / merge requests (GitLab) |
| `git_clone_repository` | Clone a repository into the session workspace |

**Write (Pro + `git_write`):**

| Tool | Description |
|------|-------------|
| `git_create_issue` | Create an issue |
| `git_comment_on_issue` | Comment on an issue |
| `git_create_pull_request` | Open a PR (GitHub) / merge request (GitLab) |

Repositories are connected per-user in **Settings → Git Integrations**.

## Code Snippets

| Tool | Description |
|------|-------------|
| `save_code_snippet` | Save a reusable code snippet |
| `search_code_snippets` | Search saved snippets |
| `list_code_snippets` | List all snippets |

Snippets are also produced automatically by plotting/analysis tools so results can be re-run in the [in-browser IDE](sandbox-tools.md).

## Skills

| Tool | Description |
|------|-------------|
| `list_skills` | List all skills |
| `search_skills` | Search skills by keyword |
| `get_skill_content` | Get the full content of a skill |

Skills are curated prompt playbooks (methodology guides) the agent can consult; admins manage them in **Admin → Skills**.

## External Agents

| Tool | Description |
|------|-------------|
| `query_external_agent` | Send a message to a user-registered external agent and return its reply |

External agents (e.g. a local [nanobot](https://github.com/nanobot-ai) or any OpenAI-compatible endpoint) are registered per-user in **Settings → External Agents** with a base URL and Bearer token (encrypted at rest). Messages are sent via `/v1/chat/completions`. Gated to the `external_agents` feature (Pro).
