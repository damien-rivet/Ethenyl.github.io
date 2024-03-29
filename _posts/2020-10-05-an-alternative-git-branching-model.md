---
title: An alternative take on a very "Successful Git branching model"
author: Damien Rivet
excerpt: "A short and concise article about implementing a standard GitFlow process and customizing it in order to take it to the next level."
categories:
    - Processes
tags:
    - Git
    - GitFlow
---

![image-center](/assets/images/icon-git.png){: .align-center}

Let me render to Caesar the things that are Caesar's 🕊, this post is heavily based on the wonderful work of [Vincent Driessen](https://nvie.com) on [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model), which you should already know by now.

I came to learn about his post during the summer of 2016 when I joined for the first time a team of more than 150 people. As obvious as it is, working within such a vast crowd of heteroclite developers requires a very strict **Git** discipline and boy… 😥 saying that I was not ready for it is a massive understatement!

During my first couple of weeks, I learned much more than during the past two years both in terms of best practices enforcement and continuous integration. Beforehand, I never had the opportunity nor the knowledge to squash commits together or rebase a branch that was behind one of the main branches.

I needed to quickly get up to speed and the best way was to learn from my colleagues and try to document myself on the **Git** flow they were using. That's when I started reading the source post and changing my ways.

It is only when I reached a certain point that I started to feel like something was missing, that some corner cases weren't taken into account.

During those last years, I moved from one job to another and worked on multiple projects at the same time with different team sizes, which led me to start experimenting with a different branching model.

And after this experimentation period, I feel that this branching model is now mature enough to release it and start getting some feedback from more experienced people.

## Purpose

The goal of this post is to expand the original model by covering advanced topics like:

- Good practices for commits
- Branch protections
- Parallel feature branches
- Semantic versioning
- Beta versions

Even thought this post is targeted at both **Git** newcomers & veterans alike, I am not going to lecture you on the basic usage and concepts of **Git** like using the command line interface (CLI) or creating a pull request (PR).

For the sake of clarity, I will also not stick to a specific hosting platform (GitHub, GitLab, BitBucket, etc.) and to a specific stack (front, back, mobile, etc.) although I have tested thoroughly this model with iOS Apps and Frameworks.

## Overview

The following sections will each be dedicated to a specific aspect of working with **Git** or with a specific branch.

Those sections are ordered to follow the actual flow explained by this post.

Here is the legend for the images enclosed in this post:

| Icon    | Description                                                        |
| ------- | ------------------------------------------------------------------ |
| 🔒      | A branch with permissions (ex: enforcing comment resolution)       |
| 🛡       | A protected branch that cannot be deleted or rewritten             |
| `X 🛠 Y` | A `X` tag that triggers an automated build for the `Y` environment |

## Commit subject and body 💬

Any commit within your **Git** tree must be identifiable at a quick glance, there's nothing more frustrating than looking for a very specific change and finding it hidden in a commit with a subject like `Fix issue, minor tweaks` 😥

Others have written thorough posts on how to write a "good" commit, feel free to check them out:

- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit)
- [Anatomy of a "Good" commit message](https://medium.com/@andrewhowdencom/anatomy-of-a-good-commit-message-acd9c4490437)

If the branch you are going to create is linked to a specific ticket in your team's issue tracking software, it is critical that you apply the following tips.

First, add the ticket key after the mandatory prefix in your branch name:

> feature/ABC-123-do-this-and-do-that
>
> bugfix/DEF-456-and-fix-this

Second, all commits done in this branch must include the ticket's key in their subject:

> [ABC-123] Do this
>
> [ABC-123] And do that

This will ensure that by giving a quick look at a branch name is more than enough to know what is actually going on.

And naming the commits accordingly will also let most of the issue tracking softwares pickup the commits. As a bonus, it will also let you know how to find more details about the actual feature or fix that is being implemented without asking around.

Here's a few good examples:

> [ABC-123] Implement new behaviour for the main entry point
>
> [DEF-456] Fix memory leak issue with the database manager

And a few bad ones:

> [ABC-123] Implemented new workflow, fixed issues, removed typos
>
> Add the fix for the database manager that seemed perfectly fine last time I checked
>
> [FIX] Fix things

Most sections will contain a TL;DR; part if you just want to scroll through the post.

```
TL;DR;

- Branch name must be prefixed (feature, bugfix, release, hotfix)
- Branch name must contain ID from issue tracking software (if any)
- Commit subject must contain ID from issue tracking software (if any)
- The first letter of the subject should be capitalised
- The subject must be written using the imperative
- The subject must stay below 70 characters
- The body must describe what has been done, without technical details
```

## Production branch 🚚

This peculiar branch is the one where every single commit should represent a stable version of your product.

It is the first branch that you will create when starting a new project, it is also the root of all the work that will be done and the first step in a new journey that will (or will not) change people's life.

Every time a new release is made on the production branch, the commit it originated from must be tagged accordingly but more on that process later.

```
TL;DR;

- Created with the repository
- Named master | main | production | release | stable
- PR are mandatory
```

## Development branch 🛠

This unique branch is where the actual magic happens, it will serve as the root of all your features and bug fixes.

Every single feature branch will be targeted at the development branch in order to make the new stuff available to everyone. Where is the fun in working all alone?

In terms of continuous integration, each time a PR is accepted and a branch is merged into this branch, a new deliverable should be generated by your CI (Continuous Integration) and deployed on an internal platform. The artefact should later be used by the Q&A team to validate the integration of all the features and bug fixes.

```
TL;DR;

- Created from production branch right after the initial commit
- Named develop | development
- PR are mandatory
```

## Branch permissions & protection 🔒

Before moving on to the other branch patterns, I would like us to take a few steps back and reflect on the two branches we just went through.

By default, all branches can be pushed to, their history rewritten and even deleted.

We are all humans, making mistakes is part of who we are, so it is bound to happen that some day, someone might forget to checkout the right branch, target the right branch on a PR or even delete a critical branch after merging a PR.

So, to prevent any mishap, branch permissions & protections must be put in place on both the production and the development branches.

The minimum would be:

> No push allowed 💪

> No history rewrite ✍️

> No deletion 🗑

**Those must be applied to both owners & maintainers as well** 😉

By default, most SCM platforms will enforce those three protections but feel free to poke at the settings and tweak them to match your team's actual needs.

You can also enforce PR validation requirements like the minimum number of reviewers or checking that all discussions are resolved before allowing the PR to be merged.

## Enforcing good practices with Git hooks 🪝

If you need to make sure that everyone is following the conventions regarding branch names, there is a way to make it automatically happen.

Git supplies us with hooks that will be called at different moments of a commit lifecycle, whether it is on the developer's machine or on the Git repository.

There are multiple client hooks but one of the most used is the `pre-commit` and, as the name implies, it will be fired even before a commit is made offering the developer a chance to fix what has been flagged as an issue (ex: unknown branch pattern, missing changelog entry, etc.).

But client-side hooks work only locally and need to be duplicated for each developer.

Server-side hooks, and mainly the `pre-receive` hook, can be used to make sure that everyone is at the same page regardless of their own setup.

Here is the [official documentation](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) on Git hooks.

It is also important to note that some Git platform (like GitLab) offers the option to enforce branch naming

3rd party solutions like [Danger](https://github.com/danger/danger) are also great at helping with this.

## Feature branches 📘

Those branches are the one that will come and go as you add more and more content to your project and you merge outstanding PRs.

Each one of those might contain a very small tweak to an existing feature, a whole new set of features or a complete refactoring of your project.

All of them start from the development branch and will be merged into it as soon as everything is validated by your team.

![image-center](/assets/images/git_branching_model_feature_branch.png){: .align-center}

### Parallel feature branches

What about long-living feature branches that require parallel work? 🤔

The solution is to treat each long-living branch as a development branch and start new features & bugfix directly from them.

Newly created PR will need to target the long-living branches instead of the development branch.

Afterward, it's a matter of rigorousness between the different team members as you will need to thoroughly rebase your long-living branch on the development branch at regular intervals.

Rebasing a long-living branch might be too much of a daunting task, even for seasoned veterans, so it is fine to merge the development branch into them.

This will allow them to keep up with the latest evolutions and ease the final merge.

![image-center](/assets/images/git_branching_model_parallel_features_branch.png){: .align-center}

<br />

```
TL;DR;

- Created from development branch
- Named feature/{ticket}{short-summary}
- Merged into development branch
```

## Merge strategies 🔩

Let's take a few other steps back for a minute and let me entertain you for a little while about merge strategies.

### Explicit merge with commit

A merge commit is created upon merging a branch in another one, this is the default merge strategy and the one I encourage you to use.

- **Pros 👍**
  - History is preserved
  - Easy to revert
- **Cons 👎**
  - Heavy on the eye
  - Requires local squashing discipline to make it efficient

### Squash merge with commit

All commits are squashed into a single new commit before the PR is merged.

- **Pros 👍**
  - Single linear commit
  - Easy to revert
- **Cons 👎**
  - History is lost

### Rebase | fast-forward merge

All commits are applied on the target branch after the PR is merged.

- **Pros 👍**
  - Linear commits
  - History is preserved
- **Cons 👎**
  - Hard to revert

<br />

It goes without saying that my preference goes to the good old explicit merge as there are no real drawbacks.

And if it is not possible to go with it, squash merge are also fine even if the branch history is lost in the process.

![image-center](/assets/images/git_branching_model_merge_strategies.png){: .align-center}

### Going further

Here are a few posts on merge strategies that are worth the read:

- [Types of Git Merge Strategies](https://www.atlassian.com/git/tutorials/using-branches/merge-strategy)
- [Merge strategies in Git](https://www.geeksforgeeks.org/merge-strategies-in-git)
- [Merge strategies](https://www.w3docs.com/learn-git/git-merge-strategies.html)

## Bugfix branches 🐛

Similarly to feature branches, branches dedicated to bug fixing can be created from the development branch but can also from feature branches, especially the one dedicated to major features associated with long development lifecycles.

![image-center](/assets/images/git_branching_model_bugfix_branch.png){: .align-center}

<br />

```
TL;DR;

- Created from development & feature branches
- Named bugfix/{ticket}{short-summary}
- Merged into development & feature branches
```

## Semantic versioning 🖇

Before the next part, a small explanation on semantic versioning (SemVer) is required in order to comprehend all the subtleties of releasing a new stable version or hurling a hotfix toward your users.

Like most software available nowadays, it is mandatory that you keep track of the versions that are released into the wild. Every single release that you and your team perform must be identified by a version number following the SemVer pattern.

The actual pattern is pretty simple to follow:

```
<major>.<minor>.<patch><pre-release>
```

Where major, minor and patch are positive numbers and pre-release is a string of characters that starts with a hyphen (-), here's a few examples:

> 0.1.0
>
> 1.0.42
>
> 2.0.1-beta-new-layout
>
> 458.34.20200101

Here's the [official documentation](https://semver.org) for the avid readers among you.

You will need this information each time a bug is reported, otherwise, it will be hard to track when & where a regression appeared.

It will also be useful for the users of your deliverables so they can pinpoint which exact release they need in order to use a specific feature.

## Release branches 📦

Once your development branch reaches a point where all the expected features and bug fixes are implemented, it might be the right time to perform a release.

A release branch can also be used to perform last minute bug fixes (that you may or may not cherry pick into the development branch) and feature implementation.

Preparing a release is a straightforward process:

1. Create a release branch from the development branch (ex: `release/1.2.3`)
1. Perform a version bump within your project to `1.2.3`
1. Update the `CHANGELOG.md` file to include all the additions, changes and fixes
1. Push your changes with a commit named `Release 1.2.3`
1. Create a PR targeted at the production branch
1. Once merged, tag the merge commit with `1.2.3` (see below for handling tags)
1. Generate a new version manually or let the CI do the heavy lifting
1. Share this version with the Q&A team and let them test it with their beloved Non-Regression Testing (NRT) campaigns 😈

A small reminder, **it is strictly forbidden to merge the development branch into a release branch** as it might introduce unwanted features and potential regressions.

After the testing phase comes the best and most stressful moment in a developer's life, the public release but this is your story and yours only 🚀

As for handling beta versions, everything will be explained in a dedicated section below 😉

![image-center](/assets/images/git_branching_model_release_branch.png){: .align-center}

<br />

```
TL;DR;

- Created from development branch
- Named release/{version}
- Merged into production branch
- Induces back merge (see below)
```

## Back merges ⏳

Performing a back merge is a pretty straightforward process, once a new stable version or a hotfix has been released into production, the production branch needs to be merged back into the development branch.

This will ensure that all tags and hotfixes are back ported into the development branches so nothing is lost in translation.

## Tags 🏷

Tags are like any other branches, the main difference is that nothing happens on those, they are just markers on specific commits to express a state of your project at a given time.

The main usage of a tag is to keep a lock of your project at a specific commit, this is critical for production release as it allows you to mark the merge commit that was the result of merging a release branch into the production branch.

If you want to know a little more about **Git** tags, follow this [link](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

And they can also be used for something else, managing beta versions 👇

## Beta versions ☢️

What happens when you and your team are ready to release a new version but you also want to let some of your esteemed users have a bite of the new features before everyone else?

The solution is to create a beta version and release it to a select few that may help you test your product before performing the actual release itself.

But… should you create a dedicated branch like `beta/my-awesome-beta-for-v0.1.0`? Should you simply tag a specific commit on your release branch and perform a manual release?

Well, in this flow, it is a slight mix of those two, the steps are the following:

1. Create a branch named `release/0.1.0-beta-new-layout` from the commit that you want to test in your release branch
1. Commit & push only a version bump in it, going from `0.1.0` to `0.1.0-beta-new-layout`
1. Tag this new commit with `0.1.0-beta-new-layout-XXX`, where `XXX` is a positive number indicating the iteration for the current beta (like `0.1.0-beta-new-layout-004`, leading zeros are not mandatory, it's just easier on the eyes) and push it to origin
1. Delete the beta branch (the tag will hold the beta version like a branch would)
1. Using this commit, perform a manual release or configure your CI to handle beta tags to generate a new deliverable
1. Share this version with the Q&A team and let them test it (it can also be shared with a restricted selection of customers that are willing to help)

Those small steps will allow you to safely release new test versions without altering the actual content of the release. Moreover, the tags will also keep those versions alive for regression testing later on.

Afterward, it is up to you and your team to decide whether you keep those beta versions or not, a good practice would be to keep at least two versions behind the version in production.

### Bug fixes in beta version

If a bug is found in a beta version, a bugfix branch can be created from the HEAD of the release branch.

This bugfix can later be merged into the release branch in preparation for another beta version release.

For critical bug fixes, it is also possible to merge them into the development branch if waiting for the next back merge is not an option.

![image-center](/assets/images/git_branching_model_beta_branch.png){: .align-center}

<br />

```
TL;DR;

- Created from release branch
- Named release/{version}{beta-version}
- Contains a single tagged commit
- Never merged
```

## Hotfix branches 🚒

Unlike bugfix branches, hotfix branches can only be created from the production branch, their sole purpose is to fix a critical issue that found it's way into production 😈

And like any release branch, a hotfix branch must induce a version change and include changes to the `CHANGELOG.md` file to explain what has been done since the last release / hotfix.

![image-center](/assets/images/git_branching_model_hotfix_branch.png){: .align-center}

<br />

```
TL;DR;

- Created from production branch
- Named hotfix/{version}
- Merged into production branch
- Induces back merge
```

## Conclusion

As I am well aware that each project is unique, I am also aware that this particular **Git** branching model might not suit your exact needs. And that's perfectly fine, I never claimed to offer you THE silver bullet that would solve all your issues 😏

I hope that some of you have found a way to improve their day-to-day workflows with this simple yet effective flow.

Below is an overview of the whole branching model:

![image-center](/assets/images/git_branching_model_global.png){: .align-center}

That's all folks!

## Contribute

In order to tweak / improve this post for everyone, do not hesitate to ask questions using the comments section below or **Twitter**.

This post will be updated as new topics are raised and solutions are found.

## Q & A

None for the moment 😉

## Glossary

| --- | --- |
|**TL;DR;**| Too Long; Didn't Read;|
| **SCM** | Source Code Management |
| **PR** | Pull Request, also known as Merge Request in some SCM |
| **CI** | Continuous Integration |
| **SemVer** | Semantic Versioning |
| **NRT** | Non-Regression Testing (commonly named Regression Testing) |
