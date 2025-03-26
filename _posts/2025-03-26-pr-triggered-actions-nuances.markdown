---
layout: single
title:  "`pull-request`-triggered GitHub Actions ***don't*** run against the latest commit in your head branch"
date:   2025-03-26 21:00:00 +0000

---

When you use the [checkout action](https://github.com/actions/checkout) in a GitHub workflow, the default behavior is to fetch a single commit from the current repo and put it in the location specified by `$GITHUB_WORKSPACE`. The specific commit that's chosen is the one pointed to by the variable `$GITHUB_SHA`. But what is `$GITHUB_SHA` set to?

Well, it depends on the type of event that's triggered the workflow. [Here](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows) you can see a list of action triggers and the corresponding value that `$GITHUB_SHA` is set to.

For instance, when the action is triggered by a [push event](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#push), `$GITHUB_SHA` is set to the tip commit of the branch that you pushed. Simple enough!

So, what about `pull-request` triggers? I always assumed that it was the tip commit of the branch you want to merge (i.e. the head branch). It turns out that's wrong! In fact, when a pull request event occurs, a new temporary *pull request merge commit*  is created behind the scenes that includes the base branch commits and the head branch commits stacked on top. The `$GITHUB_SHA` variable is then set to this pull request merge commit.

What this effectively means is that the actions you see running in your PR are testing against what it *would look like* if you merged your head branch (i.e. the thing you want to merge) into the base branch (i.e the thing you want to merge into) *at the moment the actions are triggered*. 

Why is this important? Well, every time that a pull request action is triggered, the temporary merge branch is updated with commits in the head branch *and* any changes that may have been made in the base branch. So, even if you don't merge the base branch back into your head branch, they will be there when the pull request action is triggered again (e.g. by a change in your head branch)!

Something to bear in mind when reviewing a PR is that pull request triggered actions only run (by default) when:

1. a PR is first opened
2. a PR is reopened, after being closed
3. a PR's head branch was updated

Importantly, changes to the base branch ***do not*** cause the actions in the PR to run again. So, just because your PR checks passed previously doesn’t mean they’ll pass if you run them again, especially if there have been changes to the base branch.