# stellar-docs-bot — one-time setup

The PR agent and issue agent in this repo currently post as **github-actions[bot]**. This
one-time setup gives them a single branded identity — **stellar-docs-bot[bot]** — by
registering a GitHub App that both workflows act as. Nothing about *what* the bots do
changes, only *who* the comments, labels, and merges come from. If you never run this
setup, everything keeps working exactly as it does today (the workflows fall back to
`GITHUB_TOKEN` automatically).

Three terms, one sentence each:

- **GitHub App** — a first-class bot account with its own name, avatar, and tightly
  scoped permissions that workflows can act as.
- **App ID** — the app's public identification number; workflows use it to say which app
  they are.
- **Private key** — the app's credential, used by workflows to mint short-lived access
  tokens; it is secret (never printed, never committed).

## What you do: run one script, click twice

```bash
./setup-github-app.sh
```

1. **Click 1 — Create.** A browser tab opens (make sure you're logged in to GitHub as the
   account that owns this repo) and forwards a prepared app manifest to GitHub. Click the
   green **Create GitHub App** button. If GitHub complains the name is taken, edit the
   name right on that page and create — the script picks up whatever GitHub returns.
2. *(no click — automatic)* GitHub redirects back to a temporary local server; the script
   exchanges the one-time code for the App ID and private key, saves them as the repo
   secrets `APP_ID` and `APP_PRIVATE_KEY`, and keeps a private local backup in
   `~/.config/stellar-docs-bot/`.
3. **Click 2 — Install.** The app's install page opens. Choose **Only select
   repositories**, pick this repo, and click **Install**.

Done. The next PR or issue the agents touch is answered by **stellar-docs-bot[bot]**.

## Notes

- **Safe to re-run.** The script reuses the app it already created and just re-sets the
  secrets. Pass `--force-new` to register a fresh app instead.
- **One known exception: the Copilot hand-off.** Assigning the Copilot coding agent to an
  issue requires a *user* token — GitHub only offers `copilot-swe-agent` as assignable to
  user identities, not to app/installation tokens (including `GITHUB_TOKEN`). So that
  single call keeps using the `ISSUE_AGENT_PAT` secret (a classic PAT); if that secret is
  missing, the hand-off is skipped gracefully and everything else still works. A
  fully-app-based alternative exists (a user-to-server OAuth flow) but is much heavier —
  a PAT is the pragmatic choice for this one call.
- **Different repo?** `REPO=owner/name ./setup-github-app.sh` sets that repo's secrets;
  the app must also be installed on it (the script prints the install link).
