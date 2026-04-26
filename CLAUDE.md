# unraid-templates - Project Context for Claude Code

A repository of Unraid Community Applications template XML files. One file per containerized project. Each template is hand-maintained alongside its upstream source repo (icons, descriptions, support links) but the XML lives here so CA's appfeed scans only template changes.

## Current state

| Template | Upstream | GHCR image | CA status |
| -------- | -------- | ---------- | --------- |
| `reddit_chat_bridge.xml` | [mmenanno/reddit_chat_bridge](https://github.com/mmenanno/reddit_chat_bridge) | `ghcr.io/mmenanno/reddit_chat_bridge:latest` | Pending submission (forum support thread + Asana form) |

## How CA picks up changes

CA's appfeed (Squidly271/AppFeed) re-scans every approved template repo on a ~2 hour cadence. The first submission flows through Squid's Asana form; after that, every change pushed to `main` here propagates to all CA users automatically. No separate notification or PR per change is needed once the repo is approved.

This means the lint workflow is the only gate between a `git push` and every CA user seeing the change. Treat lint failures as production breaks.

## Template field rules

These are enforced by `.github/workflows/lint.yml` and described in [Squid's Docker FAQ](https://forums.unraid.net/topic/57181-docker-faq/#comment-566084).

- **`<TemplateURL>`** must equal `https://raw.githubusercontent.com/mmenanno/unraid-templates/main/<filename>.xml`. CA dereferences this on every poll; a stale URL silently breaks updates.
- **`<Category>`** values must come from the official list in the [Docker Template Schema](https://wiki.unraid.net/DockerTemplateSchema). Made-up tags are rejected by the lint job and (more importantly) by the CA review.
- **`<Support>`** points to a thread on the Unraid forums (any subforum is fine; a moderator will move it to Docker Containers). Until the support thread exists, GitHub Issues is a placeholder; before submission it must be updated.
- **`<Icon>`** should be self-hosted. Reddit_chat_bridge's icon lives on its own repo's raw URL. External CDNs are discouraged because rotation breaks the card.
- **`<Screenshot>`** (one tag per URL, multiple lines for multiple screenshots) renders as a carousel on the CA app card. Pull from the upstream repo's existing curated screenshots — no need to maintain a parallel set here. `<Video>` and `<Photo>` are also supported per [Squid's CA assets post](https://forums.unraid.net/topic/57181-docker-faq/?do=findComment&comment=1083096) but rarely useful.
- **`<Changes>`** is wrapped in CDATA so markdown placeholders like `/unarchive <query>` don't get parsed as XML elements. Keep the block to the most recent ~5 user-visible versions; full history links out to the upstream `CHANGELOG.md`.
- **Host paths** (`<HostDir>`, `<Config>` `Default=`) are pre-filled with the conventional `/mnt/user/appdata/<container>` even though Squid's FAQ recommends against it. This matches the linuxserver / binhex convention and is what Unraid users expect — the FAQ recommendation predates that consensus.

## Workflow

```bash
# Add a new template
cp some-other.xml new-app.xml
# Edit. Make sure <TemplateURL> ends in /new-app.xml so the lint passes.
git add new-app.xml
git commit -m "Add new-app template"
git push
```

```bash
# Bump <Changes> for an upstream release
# Open new-app.xml, prepend the latest version into the CDATA block,
# trim the bottom so ~5 entries remain.
git commit -am "new-app: bump Changes for vX.Y.Z"
git push
```

Once CA is polling, no further action — every user sees the update within 2 hours.

## Lint workflow

`.github/workflows/lint.yml` has three jobs that run on every push and PR:

- **xml**: `xmllint --noout` on every `*.xml` at the repo root. Catches missing CDATA wrapping, malformed tags, the usual XML hygiene.
- **template-url**: each XML's `<TemplateURL>` must equal `https://raw.githubusercontent.com/${REPO}/main/${file}`. Catches the easy mistake of copy-pasting a template and forgetting to update the URL.
- **categories**: every space-separated token inside `<Category>` must appear in the official whitelist baked into the workflow. The list comes from the [Docker Template Schema](https://wiki.unraid.net/DockerTemplateSchema); update both the schema link and the workflow's allowlist if Unraid adds categories.

## Submission flow (one-time per template)

Per [Squid's Docker FAQ](https://forums.unraid.net/topic/57181-docker-faq/#comment-566084):

1. Push the template to this repo.
2. Create a support thread on the Unraid forums (any subforum is fine; a moderator will relocate to Docker Containers).
3. Update the template's `<Support>` URL to the forum thread.
4. Fill out the [CA submission Asana form](https://form.asana.com/?k=qtIUrf5ydiXvXzPI57BiJw&d=714739274360802).
5. Squid does a quick check, then within ~2h every CA user has access.

After step 5, this repo's `main` is the canonical source; no further CA action is needed for new templates **in this repo** — Squid only needs to add the repo once. New templates added later just appear after the next 2h poll. (New templates do still need their own forum support thread.)

## GitHub repo settings

- **2FA must stay on for the owning account.** CA policy requires it; if it lapses, the appfeed flags the repo and templates stop updating.
- **Branch is `main`.** CA reads only `main`/`master`, not arbitrary branches.

## Style

Templates are XML, not Ruby, so no rubocop here. Keep XML well-formed (CDATA for any markdown), keep the lint workflow as the gate, and keep this CLAUDE.md updated when adding a new convention.
