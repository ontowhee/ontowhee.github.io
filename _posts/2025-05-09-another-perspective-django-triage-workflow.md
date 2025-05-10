---
layout: post
title:  "Another Perspective Of The Django Triage Workflow"
date:   2025-05-09
---

## The Cost Of Wrong Triage Stages

Sometimes the Django development process can be confusing because the ticket flags are not appropriately set. Here are a few scenarios that contributors may find themselves in:

- A PR author addresses feedback from a review, but does not unset the `Patch Needs Improvement`, `Needs Docs`, and `Needs Tests` flags to get the ticket back into the review queue.
- A merger gets a message from a PR author asking why a ticket isn't being reviewed. The merger replies to explain the development process and remind the PR author to unset the `Patch Needs Improvement`, `Needs Docs`, and `Needs Tests` flags.
- A reviewer starts to review a ticket from the review queue, but after poring over Trac and Github, determines that the ticket should be in the `Ready For Checkin` stage.
- A bug reporter gets confused and feels shutdown when a ticket they had just opened is marked as `Closed` with the resolution set to `Needs Info`.

In each scenario, the ticket is in the wrong stage, or the contributor's understanding is not aligned with the intention of the stage. There's an overhead for all contributors involved.

Without changing any of the underlying mechanisms, where can we start to help contributors better understand the expectations in the development process so they can collaborate more smoothly? Let's start by understanding the current workflow.

## Current Triage Workflow

Below is a diagram taken from the Django contributing documentation depicting the current [triage workflow](https://docs.djangoproject.com/en/5.2/internals/contributing/triaging-tickets/#triage-workflow).

![Django Triage Workflow](/assets/images/django-triage-workflow.png)

In this diagram, there is a lane for "Open tickets" consisting of 3 triage stages: `Unreviewed`, `Accepted`, and `Ready for Checkin`. There is another lane for "Closed tickets" consisting of different states for the resolution property of a ticket (`duplicate`, `wontfix`, `invalid`, `needsinfo`, `worksforme`, and `fixed`).

It is a good starting point. It may seem simple and straightforward at first, until you get into the contribution process and start to experience some miscommunications and uncertainty. The documentation itself acknowledges the [big gray area](https://docs.djangoproject.com/en/5.2/internals/contributing/triaging-tickets/#triage-workflow?:~:text=The%20big%20gray%20area) of the "Accepted" stage.

## Another Perspective Of The Triage Workflow

Here is a different take on the triage workflow diagram. Note that it doesn't change the underlying mechanisms of the triage stages.

![Proposed Triage Workflow](/assets/images/proposal-triage-workflow.png)

It introduces two minor conceptual adjustments:
- Explicitly name the 3 distinct stages for Accepted stage: "Needs Patch", "Needs Review", and "Waiting On Author".
- Organizes the triage stages along a horizontal axis based on the ticket's progress. That is, how much work remains in order to get the ticket to the finish line.

### 3 Distinct Stages For Accepted Stage

Despite mentioning the big gray area of the Accepted stage, the contributing documentation already does a great job of outlining 3 distinct stages. All that was missing was having explicit names for them:
- "Needs Patch"
- "Needs Review"
- "Waiting On Author"

These names are chosen because they make it clear what action needs to happen and which contributor role is responsible for taking that action to move the ticket forward. Having distinct stages and explicit names makes it easier to understand where a ticket stands at a glance. It frees the contributor from the cognitive load of computing 4 booleans and makes it easier to move the ticket into the correct queue.

Imagine being a PR author who has just finished addressing feedback from a review. The author can now go to the ticket and adjust the triage stage from "Waiting On Author" to "Needs Review". The ticket lands in the review queue. Having explicit stages is so much easier compared to the current workflow, where the author needs to remember to unset the flags for `Patch Needs Improvement`, `Needs Docs`, and `Needs Tests`.

Similarly, imagine being a reviewer who has just finished adding feedback to a PR. The reviewer can change the triage stage from "Needs Review" to "Waiting on Author". They will still need to consider setting the `Needs Docs` and `Needs Tests` flags. However, I think having a triage stage name of "Waiting On Author" provides a much better experience. It creates the image of tossing the potato back to the author, which is more intuitive compared to toggling the value of `Patch Needs Improvement`.

### Depicting Progress

Mapping the triage stages along a horizontal axis helps depict how far along the ticket is to the finish line. The `Needs Info` resolution state is moved out of the "Closed tickets" section. It is considered to have a lower progress status, so it is placed further towards the left on the axis.

With the existing workflow, when a maintainer marks a ticket as `Closed` and `Needs Info`, a bug filer might assume that the maintainer is unfairly suggesting that the progress is 100% ("Closed"). However, in this new diagram, the progress for the ticket can now be interpreted as closer to 10%, and the bug filer can feel that this is a more fair assessment and be more cooperative.

## Thoughts?

Does the new view of the triage workflow help you better understand what you need to do to move a ticket forward?

This may not be perfect, and it may be an oversimplification of the Django triage workflow, but hopefully it brings some clarity and reduces the confusion and communication overhead.

A few questions to entertain:

- Instead of doing mental gymnastics with the flags `Has Patch`, `Patch Needs Improvement`, `Needs Documentation`, and `Needs Tests`, what if contributors can easily glance at a ticket and know which stage it is in?
- What if contributors can easily move a ticket from one stage to another?
- What if contributors can easily pull up the list of tickets in each stage on the "View Tickets" tab, instead of having to switch to the "Reports" tab or find in their bookmarks?
- What low hanging fruits can be built into the UI that makes navigating the contributor process more intuitive? What would cut down on the overhead for contributors?

[Next post coming soon...]
