# Simplest git branching model

This is a practical, lightweight, single-branch model that makes it easy to handle simple, and less simple, situations.

It is similar to Trunk-Based development workflow with some differences listed in [this chapter](#differences-with-trunk-based-development-tbd).

**TLDR**
```
    Single main branch
    One feature branch per task
    One dev per feature branch
    One PR per feature branch
    Delete feature branches after merge
```

Some more details now!

## Single branch model
* A branch `main` stores the latest version and tags.
* Repository users must take care of checking out the tag or commit that best fits their needs.

```mermaid
gitGraph
   commit id: "ONE" tag: "v1.0.0"
   commit id: "TWO"
   commit id: "THREE"
   commit id: "FOUR" tag: "v1.1.0"
```

## Adding features

### Adding a feature branch
* Add a `feature` branch checked out from `main`.
* Only you (the developer) should use it.

### Publishing the feature
* Rebase your branch onto `main`, like `git checkout feature && git rebase main`
* You are responsible for the conflict management.
* Publish a Pull Request from `feature` to `main`.

## Merging features
* You (the software manager) should merge the feature branch to `main`, manually or using a [GitHub](https://github.com/) Pull Request or a [GitLab](https://gitlab.com/) Merge Request.
* :x: This merge should probably not be done with option `--fast-forward`, which will inevitably lead to unreadable history.
* Delete the feature branch. Modifying sources within the same ticket should be done in a **new branch**.
* Put a new tag on `main` if needed.

```mermaid
---
title: Merge
---
gitGraph
   commit id: " " tag: "v1.0.0"
   branch feature1
   commit id: "A"
   commit id: "B"
   checkout main
   merge feature1 id:"Merge feature1"
   branch feature2
   commit id: "1"
   commit id: "2"
   checkout main
   merge feature2 id:"Merge feature2" tag: "v1.1.0" 
```

## Merging hotfixes
In this model hotfixes are treated as features.

## Complex feature treated by more than one developer
* Still one branch per developer.
* Devs can checkout from other dev branches.
* Merges must always be done back onto their base branch. In the following case, `feature_dev2` couldn't directly be merged back to `main`.

```mermaid
---
title: Complex feature
---
gitGraph
    commit id: "ONE"
    branch feature_dev1
    checkout main
    checkout feature_dev1
    commit id:"A"
    branch feature_dev2
    commit id:"1"
    checkout feature_dev1
    commit id:"B"
    checkout feature_dev2
    commit id:"2"
    checkout feature_dev1
    merge feature_dev2 id: "Merge from dev2"
    checkout main
    merge feature_dev1 id:"Merge part 1"
    branch feature_dev3
    commit id: "a"
    commit id: "b"
    checkout main
    merge feature_dev3 id: "Merge part 2"
```

## Maintaining several `main` branches
A common case is maintaining multiple major releases.
* Create a main branch per release.
* Keep separated feature branches for each release, even if the feature and change are the same. `git cherry-pick` can be your friend here.
* Publish one PR per release.

```mermaid
---
title: Maintain several releases
---
gitGraph
    commit id: "ONE" tag: "v1.0"
    checkout main
    branch release-v1
    commit id: "one" tag: "1.1"
    branch feature-for-v1
    checkout main
    commit id: "TWO" tag: "v2.0"
    checkout main    
    branch feature
    commit id: "A"
    checkout feature-for-v1
    commit id: "A cherry-pick"
    checkout feature
    commit id: "B"
    checkout feature-for-v1
    commit id: "B cherry-pick"
    checkout release-v1
    merge feature-for-v1 tag: "v1.1"
    checkout main
    merge feature tag: "v2.1"
```

## DONTs

* :x: Avoid bubble merges
    ```mermaid
    ---
    title: Bubble merge
    ---
    gitGraph
        commit id: "ONE"
        branch feature1
        checkout main
        commit id:"TWO"
        branch feature2
        checkout feature1
        commit id:"A"
        checkout feature2
        commit id:"1"
        checkout feature1
        commit id:"B"
        checkout main
        merge feature1 id:"Merge feature1"
        checkout feature2
        commit id:"2"
        checkout main
        merge feature2 id:"Merge feature2"
    ```

    To avoid this please consider rebasing `feature2` onto main, like `git checkout feature2 && git rebase main` :white_check_mark:

    Don't make the mistake of merging `main` back to `feature2`, like `git checkout feature && git merge main` :x:

    This will still lead to a bubble merge.

## Differences with Trunk-Based Development (TBD)
* Branches are larger than under TBD, they can last as long as a feature is not developed ; complex features involving multiple developers are also not handled this way in TBD.
* Fast-forwards are heavily used in TBD, which is not the case here.
* Multiple release branches is not TBD.

## Credits
* This page was born from the desire to free ourselves from more complex models like Git Flow, see [What is Git Flow](https://www.gitkraken.com/learn/git/git-flow) or popular Vincent Drissen's [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/), which I have nevertheless used for years (thank you Vincent), or [GitLab Flow](https://about.gitlab.com/topics/version-control/what-is-gitlab-flow/).
* It also gives more than [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
* Thanks to [jbenet](https://github.com/jbenet) for [this gist about a simple git branching model](https://gist.github.com/jbenet/ee6c9ac48068889b0912)
* Diagrams are done with [mermaid](https://mermaid.js.org/), a great tool.
