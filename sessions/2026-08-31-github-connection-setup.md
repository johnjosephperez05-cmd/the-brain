# Session: Connecting GitHub to Claude Code

**Date:** 2026-08-31
**Goal:** Get Claude Code able to work with GitHub (push, pull, open PRs) directly from the terminal, then document the process here.

---

## What was asked, step by step

1. **"I want you to connect with my Github"**
   Clarified this meant: authenticate the GitHub CLI (`gh`) so Claude Code can run git/GitHub commands on your behalf.

2. **"if I connect my github here, can the other local session push, pull requests?"**
   Answered: yes. A persistent `gh auth login` stores the credential at the Windows user-account level (Credential Manager / `%APPDATA%\GitHub CLI\hosts.yml`), so *any* process running as your Windows user, including other local Claude Code sessions, can use it. The alternative is a session-only `GH_TOKEN` environment variable, which disappears when that session ends.

3. **Chose persistent login** (over session-only), understanding that trade-off.

4. **Provided a Personal Access Token (PAT)** to log in with.

5. **"now I want you to add contents of the github, push every instructions I gave to you in this session. Make it as define[d] and easy to understand as possible"**
   → This document.

6. **"check the existing repo, and use that"**
   → Found and used the only repo on the account: `johnjosephperez05-cmd/the-brain` (this one).

---

## What was actually done

### 1. Installed the GitHub CLI
`gh` wasn't installed. Installed it with Windows' built-in package manager:
```powershell
winget install --id GitHub.cli -e --source winget --accept-package-agreements --accept-source-agreements
```

### 2. Tried browser-based login first
```powershell
gh auth login --hostname github.com --git-protocol https --web
```
This didn't work in a non-interactive terminal session, it waits for a physical "press Enter" keystroke to open the browser, which an automated session can't provide. Killed it and switched to a Personal Access Token (PAT) instead.

### 3. How to create a PAT (for future reference)
1. Go to [github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta)
2. Click **Generate new token**
3. Name it (e.g. "Claude Code CLI")
4. Set an expiration (90 days is a reasonable default)
5. Under **Repository access**, pick "All repositories" or specific ones
6. Under **Permissions**, grant at least:
   - **Contents**: Read and write
   - **Pull requests**: Read and write
   - **Metadata**: Read-only (auto-included)
7. Click **Generate token** and copy it immediately, GitHub only shows it once

### 4. Logging in with the token, and the gotcha we hit
The obvious approach failed:
```powershell
"github_pat_xxx" | gh auth login --hostname github.com --git-protocol https --with-token
# → error validating token: HTTP 401: Bad credentials
```
This looked like a bad token at first. Confirmed the token itself was actually valid by hitting GitHub's API directly:
```bash
curl -s -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer github_pat_xxx" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/user
# → 200
```
So the token was fine, `gh`'s handling of piped stdin in this Windows/PowerShell setup was the problem. **The fix:** write the token to a temp file (no trailing newline) and redirect from the file instead of piping:
```powershell
$tokenFile = "$env:TEMP\gh_pat.txt"
[System.IO.File]::WriteAllText($tokenFile, "github_pat_xxx")
cmd /c "gh auth login --hostname github.com --git-protocol https --with-token < `"$tokenFile`""
Remove-Item $tokenFile -Force
```
This worked and stored the credential in the Windows keyring (persistent, survives across shells/sessions).

### 5. Verified it worked
```powershell
gh auth status
# ✓ Logged in to github.com account johnjosephperez05-cmd (keyring)

gh api user --jq ".login, .name"
# johnjosephperez05-cmd
```

---

## Result

- `gh` (GitHub CLI) is installed and authenticated as `johnjosephperez05-cmd`.
- Login is **persistent** (stored in Windows Credential Manager), so any local session on this machine can push, pull, open PRs, etc. as this account.
- This repo (`the-brain`) was cloned and used to store this write-up, since its stated purpose is holding session summaries.

## Key takeaway for next time

If `gh auth login --with-token` ever fails with "Bad credentials" again on this machine even though the token is valid, skip piping the token in and use the temp-file + `cmd /c "... < file"` redirection method from step 4 above.
