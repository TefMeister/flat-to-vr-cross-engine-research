# Research method: GitLab's REST API returns raw files and trees where the web UI returns an empty shell

Filed by: `/gr`, 2026-09-02
Suggested home: `docs/watch-list.md` (how sources are read) or the techniques page's method notes — engine-agnostic, applies to any GitLab-hosted mod or tool.
Origin: found by the modding lane on `psychonauts-vr` (2026-09-01, notes/70 §2) and used by `/gr` on 2026-09-02 to read Astralathe's source; recorded in `psychonauts-vr/external-research/README.md` → "Research toolbox".

**The trap.** GitLab project pages and wikis render client-side, so an automated fetch of a repo, file or wiki URL returns only the page skeleton (`Loading`). That looks exactly like an empty page, and on 2026-09-01 it produced a "licence and install method unread — needs a browser" finding that was really a tooling failure. `[verified-live 2026-09-01, n=2 sessions]`

**The route.** The REST API is plain JSON and raw bytes, no token needed for public projects:

- tree: `https://gitlab.com/api/v4/projects/<id>/repository/tree?path=<dir>&recursive=true&per_page=100`
- file: `https://gitlab.com/api/v4/projects/<id>/repository/files/<url-encoded path>/raw?ref=<branch>`
- wiki page: `https://gitlab.com/api/v4/projects/<id>/wikis/<slug>`

The numeric project id is on the project's front page (or `…/api/v4/projects/<namespace>%2F<name>`). Paths must be URL-encoded (`/` → `%2F`).

**Why it belongs in the library.** The library's own rules say a negative from an automated fetch is not a negative until the fetch is shown able to return a positive; this is the concrete fix for the single most common way that rule bites on modding sources, since a large share of RE-engine, Unreal and retro-game tools live on GitLab. Same discipline as the `strings -n 4` trap: the tool, not the source, was the negative.
