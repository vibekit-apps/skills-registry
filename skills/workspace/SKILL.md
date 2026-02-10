# VibeKit Workspace Skill

You are operating inside a VibeKit workspace.

## Environment Variables

Check these environment variables to understand your context:
- `GITHUB_REPO` — Current repository (owner/repo format). If empty, you're starting a new project.
- `GITHUB_TOKEN` — GitHub access token (always available in platform mode)
- `GITHUB_USERNAME` — GitHub username or org name
- `VIBEKIT_PLATFORM_MODE` — If "true", you can create repos in vibekit-apps org

## New Project Flow (IMPORTANT)

When `GITHUB_REPO` is not set, you MUST create a repo before finishing:

1. Create project files in `/home/agent/workspace/`
2. Generate a short repo name from the project (e.g., "drink-tracker", "landing-page")
3. Initialize and create repo:
   ```bash
   cd /home/agent/workspace
   git init
   # Create repo in vibekit-apps org
   curl -s -X POST https://api.github.com/orgs/vibekit-apps/repos \
     -H "Authorization: token $GITHUB_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"name": "REPO_NAME", "private": false}' 
   git remote add origin https://x-access-token:$GITHUB_TOKEN@github.com/vibekit-apps/REPO_NAME.git
   git add -A
   git commit -m "Initial commit"
   git branch -M main
   git push -u origin main
   ```
4. Tell user: "Created repo vibekit-apps/REPO_NAME and pushed. Use `/deploy` to deploy to Vercel."

DO NOT just create files and tell user to deploy. ALWAYS create the repo and push first.

## Existing Project Flow

When `GITHUB_REPO` is set:
1. Clone the repo: `git clone https://x-access-token:$GITHUB_TOKEN@github.com/$GITHUB_REPO.git /home/agent/workspace`
2. Make changes in `/home/agent/workspace/`
3. Commit and push:
   ```bash
   git add -A && git commit -m "Description of changes" && git push
   ```
4. Tell user: "Pushed changes. Use `/deploy` to deploy."

## Topics (Forum Groups)

## Topics

The workspace has these topics (channels):

- **General** — Main chat with the user. Use for conversations, questions, and responses.
- **Deployments** — Build and deploy notifications. Post here when: deployment starts, succeeds, or fails.
- **Logs** — Verbose output and debugging. Post here for: detailed build logs, error traces, progress updates.
- **Projects** — Project management. Post here for: project switching, status updates, repo info.
- **Schedule** — Cron and scheduled tasks. Post here for: task schedules, cron runs, scheduled job output.

## Routing Guidelines

When sending messages, route them to the appropriate topic:

1. **User conversation** → General (or no topic if DM)
2. **"Deployed successfully!" / "Build failed"** → Deployments
3. **Detailed error output / build progress** → Logs
4. **"Switched to project X" / project status** → Projects
5. **"Scheduled task ran" / cron output** → Schedule

## Handling Deleted Topics

If a user deletes a topic, fall back gracefully to General. Don't error or stop working.

## Not a Forum?

If the chat is a regular DM or non-forum group, send all messages normally (no topic routing).
The forum router handles this automatically.

## Commands

- `/setup` — Creates the workspace topics (Deployments, Logs, Projects, Schedule)
- `/project` — Switch between projects (posts to Projects topic)
- `/schedule` — Manage scheduled tasks (posts to Schedule topic)
