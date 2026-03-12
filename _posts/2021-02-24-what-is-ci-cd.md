---
layout: post
title: "What is CI/CD?"
excerpt: "In this day and age, it very important to have a good understanding of what CI/CD is and why it is needed."
date: 2021-02-24
tags: [CI/CD, Basics]
feature_image: __GHOST_URL__/content/images/2021/02/shawn-ang--S15r4VsQhY-unsplash.jpg
---

In this day and age, it very important to have a good understanding of what CI/CD is and why it is needed. It is even more important to use that practice in your software production process.

In this article, I will try to explain what CI/CD is, why it is important and what are the benefits of implementing this practice.

## What is CI?

CI is the practice of **Continuous Integration**. But, what is integration then? Code integration is the act of merging (or integrating) your code in the mainline of your code repository. This may sound trivial but if not done right it can turn into a nightmare. Imagine a 10 team project. After 6 months of working on different features, they decide to integrate all their code into the main branch. And then, chaos.

Historically, the integration phase has been a true pain point. It could last days, weeks or even months and it could make projects extremely late or make the fail completely. Luckily, some smart people in the '90s decided that it is best if code integration is performed daily. Ney, **multiple times a day**.

That is how continuous integration was born. The term is attributed to Grady Booch but it was heavily popularised by Kent Beck's XP where it is one of the main practices. During the years, [Martin Fowler](https://twitter.com/martinfowler) has been a big advocate of CI. Other vocal proponents of CI are [Jez Humble](https://twitter.com/jezhumble), [Dave Farley](https://twitter.com/davefarley77) and [Nicole Forsgren](https://twitter.com/nicolefv), to name just a few.

### Is it only code integration?

Simply integrating your code won't get you anywhere. It is not enough. The second thing you need is to automatically build and test your code to prove that the code you just merged worked and that it didn't break anything that worked before. At this point, you would like to run all the checks you have and all types of tests. Code quality, security scans, unit tests, integration tests, acceptance tests, the whole shebang.

### What is the benefit?

The main benefit of continuous integration is that you get to discover problems early. If you are merging multiple times a day you discover problems as early as possible.

But don't forget. To have a proper CI and reap all the benefits you need to merge code multiple times a day and have fully automated build and verification triggered each time you do that.

## What about CD?

CD can stand for two things. **Continuous Delivery** or **Continuous Deployment**. The difference is subtle but significant.

### Continuous Delivery

Continuous delivery builds on CI and takes in one step further. After the build and all the checks have passed a new, potentially deployable version of the software is released. "Potentially" means that it may or may not be released. That is a manual step decided by a human.

This might again sound as trivial as the code integration. But think in scale. If you have a big product with a lot of components, a lot configuration and even more steps in the release process, then release becomes complicated and time consuming. It is so non-trivial that companies even today have special "release engineer" roles. Releases as such are error prone and a huge source of stress.

### Continuous Deployment

With continuous deployment, the automation is taken one step further and each change gets automatically deployed to production. Of course, after all, verification steps have passed.

### What is the benefit?

The biggest benefit of continuous deployment is that it helps bring new changes to the customer continuously. That helps in shortening the feedback loops and finding out much quicker if a feature is valuable to the customer or not. The funny thing is that developers and companies go through so much trouble to understand and implement agile development. While all they need to do is implement continuous deployment. Of course, CD won't solve all your problems. But it will help to increase value creation.

![](__GHOST_URL__/content/images/2021/02/CICDDiagram-2.png)

This pipeline is not the simplest but it still lacks some important steps. Like performance testing. The testing phases often involve deployments of their own to different testing and staging environments.

## Pipelines

The actual implementation of CI/CD, all the steps and checks, is called a pipeline. Pipelines are usually created with the help of specialized applications called Continuous Integration Servers. A few examples are Jenkins, GitLab, GoCD, Bamboo etc. The pipelines are also a visual representation of all the CI/CD steps. In that way, it increases the visibility of the process and provides feedback to the developers.

## Conclusion

CI/CD helps you discover problems early and provides a continuous flow of changes to your customers. By automating all the boring manual work it creates time for the developers to be able to focus on more fun things and on what really matters, development. It is a huge productivity booster. And while maybe continuous deployment is too big of a leap of many, there is absolutely no reason not to have continuous delivery implemented.

So, spin up your CI server and start creating some pipelines.

## Links

Cover photo by [Shawn Ang](https://unsplash.com/@shawnanggg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/shipping?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)