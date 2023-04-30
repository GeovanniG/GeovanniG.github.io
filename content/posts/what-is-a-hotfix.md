---
title: "What Is a Hotfix and How are Hotfixes Deployed"
date: 2023-04-29T00:00:00-07:00
draft: true
---

Software has bug and not all bugs are equal. Some bugs are mildly nuisances while others are critical and need to be resolved quickly. This is where hotfixes comes in.

So, what is a hotfix you may be asking? A hotfix is a sudden release to address a known bug in production. It does not follow a normal release cycle of say a sprint, where we normally release on a set schedule. Instead, hotfixes are released as soon as a patch has been implemented to address the bug.

## Getting Hotfix Ready For Deployment

Hotfixes take many forms. In my time as a developer, I have noticed that most hotfixes congregate around releases. When a new release is deployed, bug come along for the ride. 

To begin our discussion of hotfix deployment and branching, let us assume that a critical bug has been discovered in production. The alarms have sounded and we are in charge of resolving this issue. For simplicity, let's assume we found the fix and are ready to merge our changes. The question then becomes: How do we get this hotfix out in production while keeping all our environments in sync?

## Git Branching Strategy

The answer to this question is not as straight forward as you might think. There are [4 git branching](https://www.flagship.io/git-branching-strategies/) strategies, and depending on the strategy your organization uses, we have different approaches to deploy this hotfix. As for the branches strategies, we have: 
* GitFlow
* GitHub Flow
* GitLab Flow
* Trunk-based Development

We will discuss these branching strategies as they relate to hotfixes.

## Deploying Hotfixes into GitHub Flow or Trunk-based Development Strategies

When following GitHub Flow or Trunk-based Development, we have a `main` that should be ready for deployment at all times. In these strategies, a hotfix is nothing more than a feature branch. We get the hotfix tested merged and deployed. 

In these strategies, a hotfix is synonyms with a feature branch. In these strategies deploying hotfixes is as simple as it gets.      

## Deploying Hotfixes into GitLab Flow or GitFlow Strategies

On the other hand, GitLab Flow and GitFlow have a much more complicated process for releasing hotfixes. In these strategies, hotfixes do not follow the normal workflow. We cannot just merge into `main` (dev environment) and calling it a day as we do with the other strategies. In these flows, there are more branches to maintain and keep in sync. 

Let's say we have 3 branches:
* `main` (in dev environment)
* `staging-v-2` (in staging environment)
* `prod-v-1` (in production environment)

Merging into `main` is not enough because that will only solve the issue in dev. We must get our changes into `prod-v-1`. 

To merge our changes into `prod-v-1`, we must first branch off of `prod-v-1`; let's call this branch `hotfix/1/prod-v-1`. Once we have our changes in `hotfix/1/prod-v-1`, we are nearly half way there. The next step involves merging `hotfix/1/prod-v-1` into:

* `main` (in dev environment)
* `staging-v-2` (in staging environment)
* `prod-v-1` (in production environment)

Once we have completed the above, all three of our environment will be in sync and have the hotfix change.

## Summary

Hotfixes can be anomalies in deployments. In certain git branches strategies, getting a hotfix into production takes work.