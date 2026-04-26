# unraid-templates - Project Context for Claude Code

A repository of Unraid Community Applications template XML files. One file per containerized project. Each template is hand-maintained alongside its upstream source repo (icons, descriptions, support links) but the XML lives here so CA's appfeed scans only template changes.

## Current state

| Template | Upstream | GHCR image | CA status |
| -------- | -------- | ---------- | --------- |
| `reddit_chat_bridge.xml` | [mmenanno/reddit_chat_bridge](https://github.com/mmenanno/reddit_chat_bridge) | `ghcr.io/mmenanno/reddit_chat_bridge:latest` | Pending submission (forum support thread + Asana form) |

## How CA picks up changes

CA's appfeed (Squidly271/AppFeed) re-scans every approved template repo on a ~2 hour cadence. The first submission flows through Squid's Asana form; after that, every change pushed to `main` here propagates to all CA users automatically. No separate notification or PR per change is needed once the repo is approved.

This means the lint workflow is the only gate between a `git push` and every CA user seeing the change. Treat lint failures as production breaks.

## Format constraints

Per [the schema reference thread](https://forums.unraid.net/topic/38619-docker-template-xml-schema/), CA's parser is third-party and only tolerates the exact shape that Unraid's `dockerMan` produces when you Save a template in **Settings → Docker Authoring Mode → Add Container → fill → Save**. Manual edits can break the parser silently.

Concrete rules from that thread:

- **No extra whitespace inside `<Config>` element attributes.** Squid calls this out specifically: extra spacing between attributes ("breaks" in the third-party parser sense) will result in errors or wrong values when CA installs the container. dockerMan generates them single-spaced; we match that.
- **`&` must be `&amp;`.** Unescaped ampersands cause the template to silently disappear from CA. Apply to all text content (descriptions, URLs with query strings, etc).
- **`<Overview>` takes precedence over `<Description>`.** Per the schema thread, when both are present, "Description is completely ignored." We currently keep both because most modern CA templates do, but if the card ever shows the wrong text, consolidate into Overview and drop Description.
- **At least one of `<Overview>` or `<Description>` is required and non-empty.** The card won't render without it.
- **When in doubt, round-trip through dockerMan.** Test box: enable Authoring Mode, paste this template's URL into Add Container, hit Save with no edits. Diff the resulting XML against this repo's file. Any difference dockerMan introduces is the canonical shape; this repo should match.

## Tag inventory

Sourced from [the schema reference thread](https://forums.unraid.net/topic/38619-docker-template-xml-schema/) and Squid's [Docker FAQ](https://forums.unraid.net/topic/57181-docker-faq/#comment-566084).

### Required (one of)

| Tag | Purpose |
| --- | ------- |
| `<Overview>` or `<Description>` | App blurb on the card. `<Overview>` wins when both are present. |

### Lint-enforced (we check these in CI)

| Tag | Purpose |
| --- | ------- |
| `<TemplateURL>` | Must equal `https://raw.githubusercontent.com/mmenanno/unraid-templates/main/<filename>.xml`. CA dereferences this on every poll; a stale URL silently breaks updates. |
| `<Category>` | Values must come from the [official whitelist](https://wiki.unraid.net/DockerTemplateSchema). Made-up tags are rejected by both the lint job and Squid's review. |

### Strongly recommended

| Tag | Purpose |
| --- | ------- |
| `<Support>` | Unraid forum thread URL. Any subforum at submission time — a moderator relocates it. Until the forum thread exists, a placeholder (e.g. GitHub Issues) is fine; must be updated before CA submission. |
| `<Project>` | Upstream homepage / repo. |
| `<Icon>` | Self-hosted PNG. External CDNs discouraged because URL rotation breaks the card identity. We use the upstream repo's raw URL. |
| `<Screenshot>` | Carousel on the app card. One tag per URL; multiple lines for multiple screenshots. We reuse the upstream README's curated set. |
| `<Changes>` | Hand-maintained changelog block, CDATA-wrapped so markdown like `/unarchive <query>` doesn't get parsed as XML. Keep ~5 most recent user-visible versions; link out to the upstream `CHANGELOG.md` for the full history. |
| `<License>` | Short license name (e.g. `MIT`). Renders on the card. |
| `<ReadMe>` | URL to a markdown readme. Surfaces a "Read Me" button in the CA UI. We point at the upstream repo's README. Unraid 6.10-rc3+ generates this automatically when saving via dockerMan. |
| `<ExtraSearchTerms>` | Space-separated terms to widen search hits beyond `<Name>` and `<Description>`. |
| `<Requires>` | Free-text note about additional system requirements. Unraid 6.10+ generates this in authoring mode. |

### Optional / rarely useful

| Tag | Purpose |
| --- | ------- |
| `<Video>`, `<Photo>` | Same idea as `<Screenshot>` for video / single-photo media. |
| `<Branch>` | When the image has multiple tags users should pick from (e.g. `:latest`, `:beta`); CA prompts for a choice. We ship one image, one tag — N/A. |
| `<Date>` / `<DateInstalled>` | Container created/updated timestamp. CA tracks ingestion date independently; only adds info when the upstream cadence diverges from the appfeed cadence. |
| `<Beta>` | Set by the [Application Categorizer plugin](https://forums.unraid.net/topic/38431-plug-in-application-template-categorizer/) when a template is flagged beta. We just use `Status:Beta` in `<Category>` instead. |

### Deprecated / handled by CA profile

| Tag | Status |
| --- | ------ |
| `<DonateText>`, `<DonateLink>`, `<DonateImg>` | As of 2021-02-21, donation links live on the CA profile, not the template. Squid's note: "all you have to do is fill out the appropriate links within the CA profile and those links will get carried over to the templates automatically." We currently leave `<DonateText>`/`<DonateLink>` for the pre-CA-profile case (they harmlessly render nothing without `<DonateImg>`); after CA submission, the profile takes over and these tags can be removed. |

### Internal-only (do not use)

CA emits other tags into the XML it passes to dockerMan at install time. Those are CA-internal state, not template inputs. Don't add them.

## Host paths

`<HostDir>` and `<Config>` `Default=` are pre-filled with the conventional `/mnt/user/appdata/<container>` even though Squid's FAQ recommends against pre-filling host paths. This matches the linuxserver / binhex convention and is what Unraid users expect — the FAQ recommendation predates that consensus.

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
