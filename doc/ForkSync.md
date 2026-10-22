# Synchronizing the VWC PrivateBin fork

This repository uses two remotes:

- `origin`: the VWC fork (`VWC-ICT-Telecom/PrivateBin`)
- `upstream`: the original PrivateBin repository (`PrivateBin/PrivateBin`)

The VWC branding is kept on `master` while upstream updates are merged into it. A
merge is preferred over rebasing because it does not rewrite the fork's published
history and therefore does not require a force-push.

## Before starting

Use a clean working tree and update the local view of both repositories:

```sh
git switch master
git status --short
git fetch --prune origin
git fetch --prune upstream
```

If `git status --short` prints modified or untracked files, commit or stash those
changes before continuing.

Review what will be imported:

```sh
git log --oneline --decorate master..upstream/master
git rev-list --left-right --count master...upstream/master
```

The first number printed by `rev-list` is the number of commits unique to the VWC
fork; the second is the number of upstream commits not yet merged.

## Merge upstream

Create the merge locally:

```sh
git merge upstream/master
```

If there are conflicts, `git status` lists them. Resolve each file so that current
PrivateBin behavior is retained together with the VWC-specific branding, then
stage the resolution:

```sh
git add path/to/resolved-file
```

If upstream deleted obsolete code, normally keep the deletion instead of bringing
the old code back:

```sh
git rm path/to/obsolete-file
```

After all conflicts are staged, complete the merge with `git commit`. To abandon
an unresolved merge and return to the pre-merge state, use `git merge --abort`.

## Verify the result

Confirm that no conflict markers remain and inspect the fork-only changes:

```sh
git diff --check
git diff upstream/master...master
```

In particular, confirm that these branding elements are still present:

- the branded favicon and application icons in `img/`
- `img/vwc/vwc-groep-logo.png`
- the VWC logo and footer customization in `tpl/bootstrap5.php`

Install dependencies when needed, then run both test suites:

```sh
composer install
(cd js && npm ci)
make test
```

Finally, review the history and push the merge to the fork:

```sh
git log --graph --oneline --decorate -20
git push origin master
```

Do not force-push this merge workflow. A normal push should succeed unless someone
else updated `origin/master`; in that case, fetch and reconcile those changes
before pushing.

## Adding the upstream remote on a fresh clone

The `upstream` remote only needs to be added once:

```sh
git remote add upstream https://github.com/PrivateBin/PrivateBin.git
git fetch upstream
```
