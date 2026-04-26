# unraid-templates - Project Context for Claude Code

A repository of Unraid Community Applications template XML files. One file per containerized project, at the repo root. Each template is hand-maintained alongside its upstream source repo (icons, descriptions, support links) but the XML lives here so CA's appfeed scans only template changes.

## How CA picks up changes

CA's appfeed (Squidly271/AppFeed) re-scans every approved template repo on a ~2 hour cadence. The first time a repo is added, it goes through Squid's [Asana submission form](https://form.asana.com/?k=qtIUrf5ydiXvXzPI57BiJw&d=714739274360802); after that, every change pushed to `main` here propagates to all CA users automatically.

The lint workflow is therefore the only gate between a `git push` and every CA user seeing the change. Treat lint failures as production breaks.

CA policy requires 2FA on the owning GitHub account, the repo public, and the templates on `main` (or `master`).

## Format constraints

CA's parser is third-party and only tolerates the exact shape that Unraid's `dockerMan` produces when you Save a template via **Settings → Docker Authoring Mode → Add Container → fill → Save**. Manual edits can break the parser silently. Round-trip through dockerMan when in doubt: paste this template's URL into Add Container, hit Save with no edits, diff the output against the source. Anything dockerMan strips is a CA-only tag (those still belong in source — see the inventory below); anything dockerMan reorders or rewrites is the canonical shape and the source should match.

Concrete rules (from [the schema reference thread](https://forums.unraid.net/topic/38619-docker-template-xml-schema/)):

- **`<Config>` attributes are single-spaced.** Extra whitespace between attributes results in errors or wrong values when CA installs the container. Match dockerMan's output exactly.
- **`&` must be `&amp;`** in any text content. Unescaped ampersands cause the template to silently disappear from CA.
- **`<Overview>` takes precedence over `<Description>`.** When both are present, `<Description>` is ignored. Modern templates often keep both; consolidate into Overview if the wrong text ever surfaces in the card.
- **At least one of `<Overview>` or `<Description>` is required and non-empty.**
- **Element ordering** matches the dockerMan-7.2.4 output:

  ```
  Name → Repository → Registry → Network → MyIP → Shell → Privileged →
  Support → Project → ReadMe → [License → ExtraSearchTerms] → Overview →
  Category → WebUI → TemplateURL → Icon → [Screenshot×N] → ExtraParams →
  PostArgs → CPUset → DateInstalled → DonateText → DonateLink → Requires →
  [Changes] → Config×N
  ```

  Tags in `[brackets]` are CA-only — dockerMan strips them on save (it doesn't know about them). They live in this repo's source XML, CA reads them from `<TemplateURL>` on its 2-hour poll, and the user's locally-saved copy is stripped of them. The locally-saved XML is dockerMan's runtime config; the URL-fetched source is the CA card source.

- **v1 legacy blocks (`<Networking>`, `<Data>`, `<Environment>`, `<Labels>`)** were superseded by `<Config>` blocks in Unraid 6.10. dockerMan no longer emits them on save and the source should not include them.
- **`<TailscaleStateDir/>`** is Unraid 7.2+. dockerMan adds it on save based on the host's version; older Unraid versions don't recognize it. Source should not include it.

## Tag inventory

Sourced from [the schema reference thread](https://forums.unraid.net/topic/38619-docker-template-xml-schema/) and Squid's [Docker FAQ](https://forums.unraid.net/topic/57181-docker-faq/#comment-566084).

| Tag | Status | Purpose |
| --- | ------ | ------- |
| `<Overview>` | Required (or `<Description>`) | App blurb on the card. Wins over `<Description>` when both are present. |
| `<TemplateURL>` | Lint-enforced | Must equal `https://raw.githubusercontent.com/mmenanno/unraid-templates/main/<filename>.xml`. CA dereferences this on every poll; a stale URL silently breaks updates. |
| `<Category>` | Lint-enforced | Values must come from the [official whitelist](https://wiki.unraid.net/DockerTemplateSchema). |
| `<Support>` | Recommended | URL where users report issues. GitHub Issues on the upstream project repo is acceptable for docker app templates. (The FAQ's "insist on a forum thread" applies to plugin templates.) |
| `<Project>` | Recommended | Upstream homepage / repo. |
| `<Icon>` | Recommended | Self-hosted PNG. URL rotation on external CDNs breaks the card identity, so prefer the upstream repo's raw URL. |
| `<Screenshot>` | Recommended (CA-only) | Carousel on the app card. One tag per URL; multiple lines for multiple screenshots. Reuse the upstream README's curated set. |
| `<Changes>` | Recommended (CA-only) | Hand-maintained changelog block, CDATA-wrapped so any markdown angle-brackets (e.g. `/cmd <arg>`) don't get parsed as XML. Keep ~5 most recent user-visible versions and link out to the upstream `CHANGELOG.md` for full history. |
| `<License>` | Recommended (CA-only) | Short license name (e.g. `MIT`). Renders on the card. |
| `<ReadMe>` | Recommended | URL to a markdown readme. Surfaces a "Read Me" button in the CA UI. Unraid 6.10-rc3+ generates this when saving via dockerMan. |
| `<ExtraSearchTerms>` | Recommended (CA-only) | Space-separated terms to widen search hits beyond `<Name>` and `<Description>`. |
| `<Requires>` | Recommended | Free-text note about additional system requirements. Unraid 6.10+ generates this in authoring mode. |
| `<Video>`, `<Photo>` | Optional | Same idea as `<Screenshot>` for video / single-photo media. |
| `<Branch>` | Optional | For images with multiple tags users should pick from (e.g. `:latest`, `:beta`); CA prompts for a choice. |
| `<Date>` / `<DateInstalled>` | Optional | Container created/updated timestamp. CA tracks ingestion date independently. |
| `<Beta>` | Optional | Flagged-beta indicator from the [Application Categorizer plugin](https://forums.unraid.net/topic/38431-plug-in-application-template-categorizer/). Equivalent to setting `Status:Beta` in `<Category>`. |
| `<DonateText>`, `<DonateLink>`, `<DonateImg>` | Deprecated | As of 2021-02-21, donation links live on the CA profile, not the template; CA carries them over automatically once the profile is filled out. |

CA-only tags survive in the source XML but get stripped from the user's locally-saved copy when dockerMan saves. CA reads them from the URL on each poll, so the card always reflects the source.

## Host paths

`<HostDir>` and `<Config>` `Default=` are pre-filled with `/mnt/user/appdata/<container>`. Squid's FAQ recommends against pre-filling host paths, but the linuxserver / binhex convention (and Unraid user expectation) is to pre-fill, so the source matches that convention.

## Lint workflow

`.github/workflows/lint.yml` has three jobs that run on every push and PR:

- **xml** — `xmllint --noout` on every `*.xml` at the repo root. Catches malformed tags, missing CDATA wrapping, the usual XML hygiene.
- **template-url** — each XML's `<TemplateURL>` must equal `https://raw.githubusercontent.com/${REPO}/main/${file}`. Catches the easy mistake of copy-pasting a template and forgetting to update the URL.
- **categories** — every space-separated token inside `<Category>` must appear in the official whitelist baked into the workflow. The list comes from the [Docker Template Schema](https://wiki.unraid.net/DockerTemplateSchema); update both the schema link and the workflow's allowlist if Unraid adds categories.

## Workflow

```bash
# Add a new template
cp existing.xml new-app.xml
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

Once CA is polling this repo, every user sees the update within 2 hours of the push.
