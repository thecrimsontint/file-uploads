<!-- gitea-cutover -->
# Gitea cutover runbook — `jstewart612/file-uploads`

This repository is mirrored **read-only** from GitHub into the homelab Gitea at
`https://gitea.thecrimsontint.com/jstewart/file-uploads`. **GitHub is the source of
truth**; Gitea pulls every 8h, and Gitea Actions do **not** run on a mirror.

CI parity is staged: `.github/workflows/` runs on GitHub today; an identical
`.gitea/workflows/` (runner-label aligned) is committed so it is already
present in Gitea and runs the moment Gitea becomes this repo's live host.

## Make Gitea the live host (cutover)

1. **Stop the pull-mirror.** In the `terraform/gitea` workspace, drop this repo
   from the mirror set (or set its `mirror = false`) and `atlantis apply`.
   (Equivalently, in Gitea: repo -> Settings -> uncheck "This repository is a
   mirror" to convert it to a regular repo.) The Gitea copy becomes writable.
2. **Repoint your working clone:**
   ```sh
   git remote set-url origin git@gitea.thecrimsontint.com:jstewart/file-uploads.git
   git remote add github  git@github.com:jstewart612/file-uploads.git   # keep GitHub as a backup
   ```
3. **Push-mirror Gitea -> GitHub** (so GitHub stays a live backup): Gitea repo ->
   Settings -> Mirror Settings -> add a **push** mirror to
   `https://github.com/jstewart612/file-uploads.git` with a GitHub PAT.
4. **Enable Actions + recreate secrets:** Gitea repo -> Settings -> Actions ->
   enable; recreate any `secrets`/`vars` the workflows use under
   Settings -> Actions -> Secrets. The shared runner (`gitea-act-runner`) is
   already registered with labels `ubuntu-latest` / `ubuntu-22.04` /
   `ubuntu-24.04`.
5. **Verify:** push a commit and confirm `.gitea/workflows/` runs under the
   repo's Actions tab. Adjust for any GitHub-only features (`github-script`,
   Pages, OIDC, `GITHUB_TOKEN` scopes).
6. **Optional:** archive the GitHub repo once Gitea is authoritative and
   push-mirroring back, so nobody pushes to the old primary.

## Roll back
Re-enable the pull-mirror in `terraform/gitea` and remove the Gitea push
mirror; Gitea returns to a read-only copy of GitHub.
