# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. The previous CLAUDE.md files are saved as `.claude/TASK_1_20260904.md` through `.claude/TASK_11_LUCAS_2015.md`. Please look at these to understand where we are and for reference to the code repo for which this is the documentation.

## Internal links (resolved 2026-10-06)

The site is served from `https://xspatula.github.io/xspatula_lucas_docs/`. Plain Markdown links are never rewritten by Jekyll, so a root-relative link like `/lucas_2009/prepare_data/` points to `xspatula.github.io/lucas_2009/...` online (404) while working locally (local baseurl is `""`).

Rule: every internal link must carry the baseurl, e.g.

    [prepare_data]: {{ site.baseurl }}/lucas_2009/prepare_data/

`baseurl` in `_config.yml` must keep its leading slash: `"/xspatula_lucas_docs"`. Links in `_data/navigation.yml` stay without baseurl — the theme applies `relative_url` to them.
