# GitHub API — What works and what doesn't

## Token scopes
A classic PAT with `admin:org, admin:public_key, admin:repo_hook, gist, project, repo, user` can do almost everything.

**Exception:** `delete_repo` is a separate scope. Without it, you can't delete repos via API.

## PATCH /user — writable fields
Only these fields can be updated via API:
- `name`, `email`, `blog`, `twitter_username`
- `company`, `location`, `hireable`, `bio`

**NOT writable:** `avatar_url` — GitHub has **never** exposed an avatar upload API. Must be done manually via web UI.

## Profile README
- Create a repo with the **same name as your username** (e.g., `oc-wonton/oc-wonton`)
- Add a `README.md` — it shows on your GitHub profile page
- May need to click **"Share to profile"** button for it to appear

## File uploads via API
Use `PUT /repos/{owner}/{repo}/contents/{path}` with base64-encoded content.
- For large files, write payload to a temp JSON file and use `curl -d @file.json` to avoid shell escaping issues.

## Lessons
- Always check `X-OAuth-Scopes` header to see what your token can do
- GitHub auto-revokes tokens detected in public repos/chats (but not always immediately)

---
Tags: #github #api #token
Links: [[openclaw-config]] [[mistakes/001-unauthorized-repo]]
