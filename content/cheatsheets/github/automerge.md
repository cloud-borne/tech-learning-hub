---
title: AutoMerge
linktitle: ⛙ AutoMerge
toc: true
type: book
date: "2019-05-05T00:00:00+01:00"
tags:
  - GitHub

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 5
---

Automated merging of release branches into your main branch with ```GitHub Actions```

<!--more-->

## Overview

A typical release process for Git workflows involves creating a release branch, performing various tests on that branch, and applying any necessary fixes or changes to that branch. Once stable and ready to release, you create a build from the release branch, create a git tag, and finally merge the release branch changes back into your main branch.

We can create a workflow using [GitHub Actions](https://github.com/features/actions) to do this for us.

## How it works

We are going to use this [Cascading Auto Merge Action](https://actionsdesk.github.io/cascading-downstream-merge/#/). You should read the documentation before using this action. If the action runs into a problem (like a merge conflict), it will automatically open an issue assigned to the user who merged the originating pull request. The user can then go and fix any conflicts and try to address the problem directly.

Here is an example git graph to show the automatic branch auto-merge flow. When a feature branch is merged into a release branch, it then gets forward auto-merged into subsequent releases based on their semantic version order. The final merge in the automatic cascade is into the refBranch (in this case the main branch).
![github-automerge-graph](/images/uploads/github-automerge-graph.png)

## Setup

The action relies on the branch that opens the PR to remain in place so that the subsequent merges can still occur. The option for Automatically delete head branches must be deselected in order for the action to work properly.

