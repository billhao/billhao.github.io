# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is
Personal notes/writing site at https://billhao.github.io — Jekyll + minima on GitHub Pages, repo billhao/billhao.github.io. Simplicity is the explicit top requirement: no features beyond publishing text.

## There is no local build — do not try to create one
Jekyll is NOT installed; only system Ruby 2.6, which is too old for it. There is no Gemfile and no CI workflow. GitHub Pages compiles the site server-side (build_type: legacy, "deploy from a branch") on every push to main, in roughly 40 seconds.
So "build and test" means: commit, push, wait, check the live URL. Never run jekyll, bundle, or gem; never add a Gemfile or a .github/workflows file to make local builds work — the absence of a toolchain is the design.
Check a build: gh api /repos/billhao/billhao.github.io/pages/builds/latest — status must be "built", not "errored".
Check a page: curl -s -o /dev/null -w '%{http_code}' https://billhao.github.io/PATH/
Critical: the build reports success even when permalinks are wrong. Always verify an actual post URL, not just build status.

## Content layout — the gotcha that silently breaks every tag URL
Posts live at TAG/_posts/YYYY-MM-DD-slug.md, NOT _posts/TAG/YYYY-MM-DD-slug.md.
Jekyll derives a post's category from the directory ABOVE _posts. With permalink: /:categories/:title/ in _config.yml that yields /TAG/slug/. Putting posts under _posts/TAG/ gives them no category at all, the permalink collapses to /slug/, and every tag URL 404s while the build still reports success. This was hit and fixed in commit 9197f78; do not "tidy" the tag directories back into one _posts/ tree.
Each tag also needs TAG/index.md (layout: tag, permalink: /TAG/) as its listing page. The tags: front matter list is the source of truth for tag chips and /tags/; the first entry is the storage directory. Tags must be lowercase ASCII because they become URLs — Chinese-titled posts still get an English slug.

## Publishing is done by a skill, not by hand
The /publish skill lives outside this repo at ~/projects/common/claude/skills/publish/ (SKILL.md + prompt.md). Two steps by design:
- /publish TEXT-OR-FILE — parses line 1 as title and line 3+ as body, rewrites per prompt.md, writes drafts/TAG/DATE-slug.md, prints the post, commits nothing.
- /publish with no arguments — re-reads the draft from disk (user edits win, no second rewrite), moves it to TAG/_posts/, creates missing tag stubs, commits, pushes, polls until live.
prompt.md is the user's rewrite prompt and they edit it freely; do not rewrite it unprompted. Its rules: strip dictation fillers, regroup sentences by topic, merge redundant sentences, and never add, infer, translate, or formalize.
drafts/ is gitignored on purpose so drafts stay local and never reach the public repo. Never git add it.

## Deliberate choices — do not "fix" these
- No CI file. Classic branch build was chosen over GitHub Actions for zero maintenance. Consequence: only GitHub's whitelisted plugins work. jekyll-feed and jekyll-seo-tag are in use; jekyll-archives and similar will silently do nothing.
- _includes/footer.html is intentionally empty. The footer was removed at the user's request along with their email address. Do not restore minima's footer.
- There is no email: key in _config.yml. Do not add one.
- RSS stays discoverable through the <link rel="alternate"> tag in <head>, which is why removing the footer link was safe.

## Theme overrides
minima is the theme; these files override parts of it. _layouts/post.html adds tag chips to minima's post layout. _layouts/tag.html renders a per-tag listing from site.tags[page.tag]. assets/main.scss imports minima then adds .post-tag styling. index.md uses minima's home layout, which lists site.posts automatically.

## Shell
The user's shell is zsh, which does NOT word-split an unquoted variable: for t in $TAGS creates one directory literally named "tag1 tag2". Iterate newline-separated with `| while IFS= read -r t` instead.
