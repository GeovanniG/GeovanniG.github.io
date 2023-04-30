---
title: "What Is a Hotfix and How are Hotfixes Deployed"
date: 2023-04-30T00:00:00-07:00
draft: false
---

Software has bugs and not all bugs are created equal. Some bugs are mild nuisances while others are critical and need to be resolved quickly. This is where hotfixes comes in.

A hotfix is a sudden release to address a known bug in production. The bug is usually severe in nature and needs to be resolved as quickly as possible. The release of this patch does not follow a normal release cycle of say a sprint, where we normally release on a set schedule. Instead, hotfixes are released as soon as a patch has been implemented to address the known vulnerability.

Why should we care about hotfixes? In your career, you will be asked to handle many of these stressful situations. Knowing the steps to successfully deploying a hotfix will help ease the stress.

## Getting Hotfix Ready For Deployment

To begin our discussion of hotfix deployment and branching, let us assume that a critical bug has been discovered in production. The alarms have sounded and we are in charge of resolving this issue. For simplicity, let's assume we found the fix and are ready to merge our changes. The question then becomes: How do we get this hotfix out in production?

## Git Branching Strategy

The answer to this question is not as straight forward as you might think. There are around [4 git branching](https://www.flagship.io/git-branching-strategies/) strategies out in the industry, and depending on the strategy your organization uses, we have different approaches to deploying this hotfix. 

As for the branches strategies, we have: 

* GitFlow
* GitHub Flow
* GitLab Flow
* Trunk-based Development

Let's discuss these branching strategies as they relate to deploying a hotfix.

## Deploying Hotfixes using GitHub Flow or Trunk-based Development Strategies

When following GitHub Flow or Trunk-based Development, we have a `main` branch that is ready for deployment at all times. These strategies are based on feature branches, so, in this case, a hotfix is nothing more than a feature branch. 

In these strategies, a hotfix is synonyms with a feature branch. and deploying a hotfix is as simple as it gets. We create a feature branch for the hotfix, get it tested, merged and deployed. That's it!

The more interesting case occurs when dealing with more sophisticated branching strategies.

## Deploying Hotfixes using GitLab Flow or GitFlow Strategies

On the other hand, GitLab Flow and GitFlow have a much more complicated process for releasing hotfixes. In these strategies, hotfixes do not follow the normal workflow. We cannot just merge into `main` and calling it a day as we do with the above strategies. In these flows, there are more branches to maintain and keep in sync. 

Let's say we have 3 branches:
* `main` (in dev environment)
* `staging-v-2` (in staging environment)
* `prod-v-1` (in production environment)

Merging into `main` is not enough because that will only solve the issue in dev. We must get our changes into `prod-v-1`. 

To merge our changes into `prod-v-1`, we must first branch off of `prod-v-1`; let's call this branch `hotfix/prod-v-1.1`. Once we have our changes in `hotfix/prod-v-1.1`, we are nearly half way there. The next step involves merging `hotfix/prod-v-1.1` into:

* `main` (in dev environment)
* `staging-v-2` (in staging environment)
* `prod-v-1` (in production environment)

Once we have completed the above, all three of our environment will be in sync and have the hotfix change. Now all that remains is deploying `prod-v-1`, which can take a new name of `prod-v-1.1` to signify that it has been updated.

## Summary

Hotfixes take many forms. In my time as a developer, I have noticed that most hotfixes congregate around releases. When a new release is deployed, bugs come along for the ride. 

Also, I have simplified the case when following GitLab Flow or GitFlow strategies. There will be times when merging `hotfix/prod-v-1.1` into `staging` and `main` will cause a merge conflict. This case is a bit more cumbersome to work with. In future posts, we will address how to deal with merge conflicts and tools available to us to help simplify this process.

As always, thanks for reading!