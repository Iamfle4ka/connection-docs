# Apps section restructure — conservation report

**Branch:** `PRDCT-358-apps-restructure` · **Stage:** structure + form (accuracy and
Kai-weaving are out of scope) · **Base:** `main`

This report accounts for every paragraph of every old `data-apps` page: where it
went (moved / merged / split), or — if removed — why (logged below as a
deliberate-deletion candidate for human review). It also lists the redirects that
are now needed.

> **Note on the draft skeleton.** The task referenced an authoritative
> `apps-section/` draft (STRUCTURE.md + seed files). It does not exist on disk, in
> any git ref, or in scratch. Per decision, the new structure was built **from the
> task's old→new map**, moving all factual content verbatim, writing only minimal
> connective prose + mandatory `description` frontmatter, and adding fresh
> `<!-- VERIFY(owner) -->` flags only where old text is demonstrably wrong. No
> draft `VERIFY`/`TODO` handoff comments existed to preserve (the old pages
> contained none).

## New structure

```
src/content/docs/data-apps/
  index.md              hub (nav + why + 3 build paths)
  what-are-apps.md      explanation (concept)
  build-with-kai.md     how-to — AI-assisted Python/JS build
  build-locally.md      how-to — local build + data access (Input Mapping, Storage API, Storage Access)
  reference.md          env vars, backend versions/runtime, sleep/resume, app management, terminal log, troubleshooting, key terms, Storage Access reference, limits
  authentication.md     auth hub + 4 OIDC providers merged as sections
  streamlit/
    index.md            Streamlit subtree hub (kept; framed supported-but-specialized)
    lock-version.md      merge of the 3 lock-streamlit-version pages
    design-guide.md      moved from general-design-guide
```

## Old → new content map

### `index.md` (reshaped into hub)
| Old chunk | New home |
|---|---|
| "What is an app" definitional prose (intro) | `what-are-apps.md` (intro) |
| "Why Build Apps?" + 4 subsections | `what-are-apps.md` → "Why build apps?" (full text); a short teaser + link kept in `index.md` hub |
| "Choose Your Technology" (Streamlit / Python-JS) | `what-are-apps.md` → "Choose your technology"; condensed 3-path nav in `index.md` |
| "Creating Your First App" (+ `app-modal.png`) | `what-are-apps.md` → "Creating your first app" |
| "Common Features" (auth, data, secrets, resources, workflow) | `what-are-apps.md` → "Common features" |
| "Sleep and Resume" (+ `proxy-wakeup.png`, `proxy-error-wakeing-up.png`, `deploy-timeout-backedsize.png`) | `reference.md` → "Sleep and resume" |
| "Deployment and App Management" (+ `manage-redeploy.png`) | `reference.md` → "Deployment and app management" |
| "Debugging App Deployment" (+ `job-error-log.png`) | `reference.md` → "Debugging app deployment" |
| "Example Apps" (6 examples) | `what-are-apps.md` → "Example apps" |
| frontmatter `slug: data-apps`, `redirect_from /components/data-apps/` | preserved on `index.md` |

### `python-js/index.md` (split)
| Old chunk | New home |
|---|---|
| Intro / "full control" overview prose | `what-are-apps.md` → "Python/JS (custom frameworks)" |
| "How It Works" (Nginx/port mechanics) | `what-are-apps.md` → "How Python/JS apps work" |
| "What You Need Before Starting" | `build-with-kai.md` |
| "Repository Structure - The Golden Rule" | `build-with-kai.md` |
| Steps 1–5 (repo, code, keboola-config, dependencies, deploy) | `build-with-kai.md` |
| "Example: Hello World App" | `build-with-kai.md` |
| "Working with Keboola Data" (Input Mapping, Storage API) | `build-locally.md` → "Working with Keboola data" |
| "Secrets and Environment Variables" | `reference.md` → "Secrets and environment variables" |
| "Troubleshooting" | `reference.md` → "Troubleshooting" |
| "Key Terms Explained" | `reference.md` → "Key terms" |

### `storage-access/index.md` (split)
| Old chunk | New home |
|---|---|
| Intro + Snowflake-only caution | `build-locally.md` → "Storage Access" (intro/caution) |
| "When to Use Storage Access" | `build-locally.md` |
| "Setting Up Storage Access" (Steps 1–3 + programmatic JSON) | `build-locally.md` |
| "Reading Data from Storage" | `build-locally.md` |
| "Writing Data Back to Storage" | `build-locally.md` |
| "Example: Read-Write Data App" | `build-locally.md` → "Example: read-write app" |
| "Best Practices" (5 patterns) | `build-locally.md` → "Best practices" |
| "How It Works" (architecture + workspace lifecycle) | `reference.md` → "Storage Access reference → How it works" |
| "Environment Variables" table | `reference.md` → "Storage Access environment variables" |
| "Comparison: Input Mapping vs Direct Storage Access" | `reference.md` → "Input Mapping vs direct Storage Access" |
| "Limitations" | `reference.md` → "Limits" (placed at the bottom per form rules) |

### `authentication/index.md` + 4 OIDC pages (merged → `authentication.md`)
| Old page / chunk | New home |
|---|---|
| `authentication/index.md` (None, Basic, OIDC intro, GitHub, GitLab, JumpCloud, Callback URL) | `authentication.md` (same sections, sentence-case) |
| `authentication/index.md` images | `authentication.png` → `auth-options.png`; `select-oidc-provider.png` → `auth-select-oidc-provider.png` (moved to `data-apps/` root) |
| `oidc/auth0/index.md` (6 steps) | `authentication.md` → "### Auth0" (steps demoted to `####`) |
| `oidc/google-cloud-platform/index.md` (6 steps) | `authentication.md` → "### Google Cloud Platform" |
| `oidc/microsoft-entra-id/index.md` (6 steps incl. group claim) | `authentication.md` → "### Microsoft Entra ID" |
| `oidc/okta/index.md` (6 steps) | `authentication.md` → "### Okta" |
| OIDC provider links in old hub (`/data-apps/authentication/<provider>/`) | rewritten to in-page anchors (`#auth0`, etc.) |

### `backend-versions/index.md` (folded into `reference.md`)
| Old chunk | New home |
|---|---|
| Intro, "Version Format", "Choosing a Version", "Pre-installed Packages" | `reference.md` → "Backend versions and runtime" |
| **"Release Changelog" (1.12.0–1.15.2, 11 entries)** | **DELETED — see deliberate deletions** |

### `lock-streamlit-version/*` (3 → 1 at `streamlit/lock-version.md`)
| Old chunk | New home |
|---|---|
| `lock-streamlit-version/index.md` intro + rationale + deployment-methods summary | `streamlit/lock-version.md` intro + "Why lock package versions?" + method chooser |
| `code-deployment/index.md` (Dev best practices, steps 1–3, best practices) + 5 screenshots | `streamlit/lock-version.md` → "Code deployment" (images renamed `lock-*.png`) |
| `git-deployment/index.md` (tutorial links, venv steps, example requirements.txt, best practices) | `streamlit/lock-version.md` → "Git repository deployment" |
| Rationale text duplicated across all 3 pages | **deduplicated to one** "Why lock package versions?" block (the fuller wording kept; identical copies dropped — not a content loss) |

### `general-design-guide/index.md` (moved → `streamlit/design-guide.md`)
| Old chunk | New home |
|---|---|
| All sections (Theming, Header, Body, Footer, Storage communication) | `streamlit/design-guide.md` verbatim, sentence-case headings |
| Images `pic1/pic3/pic4.png` | `streamlit/design-guide-folder.png` / `-save-button.png` / `-footer.png` |
| Link `/data-apps/#theming` | rewritten to `/data-apps/streamlit/#theming` |

### `streamlit/index.md` (kept)
| Old chunk | New home |
|---|---|
| All content | kept in place; form pass only (sentence-case headings, removed "Just", added `description`, framed supported-but-specialized) |
| Link `/data-apps/backend-versions/` | rewritten to `/data-apps/reference/#backend-versions-and-runtime` |

### `terminal-log-tab/terminal-log-tab.md` (folded into `reference.md`)
| Old chunk | New home |
|---|---|
| Intro, "Key Features", "Key Benefits" (+ image) | `reference.md` → "Terminal log tab"; image `hello-world.png` → `terminal-log.png` |

## Deliberate deletions (MISSING list — for your delete/restore decision)

1. **`backend-versions/index.md` → "Release Changelog" section** — the full
   per-version changelog (1.15.2, 1.15.1, 1.15.0, 1.14.1, 1.14.0, 1.13.3, 1.13.2,
   1.13.1, 1.13.0, 1.12.1, 1.12.0, with dates and notes). **Removed**, not moved.
   Rationale: per the task, release-changelog content belongs in the changelog,
   not the docs. The old page itself noted it lived here only "because the source
   repository is private." **Restore candidate** if there is no changelog home for
   it yet — the raw text is recoverable from
   `git show main:src/content/docs/data-apps/backend-versions/index.md`.

No other content was dropped. All non-changelog prose, code blocks, tables, and
UI-locator screenshots were relocated.

## Screenshots

All surveyed screenshots are UI locators and were kept (relocated where their host
page moved). No decorative-only screenshots were found to drop. One follow-up flag:

- `streamlit/hello-world-code.png` shows sample code inside the Code text area.
  Per the screenshot policy, that embedded code should be transcribed into a fenced
  block. The exact code is not legible from outside the image, so a
  `<!-- VERIFY(owner) -->` flag was left in `streamlit/index.md` instead of
  inventing it.

## Redirects

### Already in place (preserved `redirect_from`, verified generating redirect pages)
Carried forward from the old pages onto their new homes:
- `index.md`: `/components/data-apps/`
- `authentication.md`: `/components/data-apps/authentication/`, `/data-apps/oidc/`,
  `/components/data-apps/oidc/`, and all eight `/{data-apps,components/data-apps}/oidc/<provider>/`
- `reference.md`: `/components/data-apps/backend-versions/`, `/components/data-apps/terminal-log-tab/`
- `streamlit/lock-version.md`: `/components/data-apps/lock-streamlit-version/`(+ `/code-deployment/`, `/git-deployment/`)
- `streamlit/design-guide.md`: `/components/data-apps/general-design-guide/`
- `streamlit/index.md`: `/components/data-apps/streamlit/`

### Redirects still NEEDED (NOT auto-added — your call, per task rule 7)
These were live URLs (page slugs) that have moved. They currently 404 and need a
redirect decision. The mechanism in this repo is the `redirect_from` frontmatter
array on the destination page.

| Old live URL | Suggested destination |
|---|---|
| `/data-apps/python-js/` | `/data-apps/build-with-kai/` (primary) — content split with `/data-apps/build-locally/` |
| `/data-apps/storage-access/` | `/data-apps/build-locally/#storage-access` |
| `/data-apps/backend-versions/` | `/data-apps/reference/#backend-versions-and-runtime` |
| `/data-apps/terminal-log-tab/` | `/data-apps/reference/#terminal-log-tab` |
| `/data-apps/general-design-guide/` | `/data-apps/streamlit/design-guide/` |
| `/data-apps/lock-streamlit-version/` | `/data-apps/streamlit/lock-version/` |
| `/data-apps/lock-streamlit-version/code-deployment/` | `/data-apps/streamlit/lock-version/#code-deployment` |
| `/data-apps/lock-streamlit-version/git-deployment/` | `/data-apps/streamlit/lock-version/#git-repository-deployment` |
| `/data-apps/authentication/auth0/` | `/data-apps/authentication/#auth0` |
| `/data-apps/authentication/google-cloud-platform/` | `/data-apps/authentication/#google-cloud-platform` |
| `/data-apps/authentication/microsoft-entra-id/` | `/data-apps/authentication/#microsoft-entra-id` |
| `/data-apps/authentication/okta/` | `/data-apps/authentication/#okta` |

### Internal links fixed
- `kai/python-client.md`: `/data-apps/python-js/` → `/data-apps/build-with-kai/`,
  and `/data-apps/python-js/#secrets` → `/data-apps/reference/#secrets-and-environment-variables`.
- Within-section links updated to the new homes/anchors (Streamlit "Base image" →
  reference; design guide theming link → Streamlit theming; OIDC provider links →
  anchors).

## Open flags handed to section owners
- `authentication.md` → Auth0 Step 3: `<!-- VERIFY(owner) -->` — "the correct
  issuer URL for Google OAuth 2.0" appears to be a copy-paste error on the Auth0
  page (left verbatim; this is not the accuracy pass).
- `streamlit/index.md` → Code deployment: `<!-- VERIFY(owner) -->` — transcribe the
  code embedded in `hello-world-code.png`.

## Verification
- `npm run gen:sidebar` regenerated `src/sidebar.mjs` from the updated nav.
- `npm run build` completes (248 pages; 151 redirect pages). The only warning
  (`/404` route conflict) is pre-existing on `main`.
- Repo-wide grep confirms zero remaining internal links to old `data-apps` paths
  (outside intentional `redirect_from` entries).
