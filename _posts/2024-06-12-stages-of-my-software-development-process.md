---
layout: post
title:  "5 Stages Of My Software Development Process"
date:   2024-06-12
---

It’s simple and straightforward. However, it is not second nature to me. I have to write it out in order to follow it.

It became very important for me to know the process very well when the tech lead on the team gave me more responsibilities on a recent project. I had to take the helm and drive the process forward.

This is what I individually use on my current team. Whether I am working on a task on my own or pairing with my coworkers, I reference these 5 stages. It serves as my personal guard rails. It helps me ask the right questions, use the right medium, at right time!

Breaking the process into these 5 stages helps me and my team get on the same page. We are able to collaborate efficiently. We are able achieve successful and satisfying deliveries.

The 5 Stages:

1. Grooming
2. Align With Stakeholders
3. Active Development
4. Code Review
5. QA

(I’m only discussing stages that I am directly involved in as a software engineer on my team. Stages prior to Grooming are owned by the product team. Stages after QA are owned by the release team. Those are not covered here.)

## Stage 1: Grooming

The purpose of this stage is for me to establish a good grasp of the project and provide a document that I can share with my team.

### Goal

- Read the product ticket. Go through the design documents.
- Explore the code.
- Prepare notes to share with the team.

### Allocated time

- 1 day for small projects. 2-3 days for medium projects. 4+ days for larger projects.

### Deliverables

By the end of this stage, I create what I call a “Grooming document” containing the following sections:

- Brief description of the problem in my own words
- Brief description of the technical requirements
- Proposal for implementation
    - Highlight key existing code patterns, components, and functional behaviors that will be involved in this project
    - Highlight key code components that need to be added, removed, or modified
    - Breakdown tasks. Add implementation details as necessary.
    - List of test cases to cover both the product and technical requirements
- List of questions for:
    - Product and design
    - Code
    - Scope of work and technical feasibility given time and resource constraints
- List of unknowns that would require further deep dive

The Grooming document does not need to be lengthy or professionally written. It is a set of notes used internally for the team. I don’t require my coworkers to read them thoroughly. I walk through the document with them during the Align With Stakeholders stage.

It’s a balancing act to achieve notes that are detailed and accurate without spending too much time writing them up. To accomplish this balance, I have to determine which areas need deeper dive at this early stage, and which areas can be briefly mentioned with the expectation that they will be explored later. I don’t get it right all the time, and that’s ok.

Once I have prepared my notes, I am ready to have a meeting. I reach out to the product manager and tech lead. They schedule the meeting and invite all the engineers.

This is my personal favorite part of software engineering. I love exploring the existing code base and thinking through ideas for changing the code. I also love organizing my thoughts to communicate them clearly to my coworkers. To me, this is the bulk of software development. I love it because it allows me to invest time upfront planning the project, so I can execute smoothing in the next 4 stages!

## Stage 2: Align With Stakeholders

Now that I have prepared the Grooming document, I share my findings with my team, and we discuss the details to make sure the proposed implementation is viable and meets the product’s needs.

### Goals

- Get answers to questions.
- Discuss technical details with engineers to ensure the proposed implementation is viable. Answer questions to help them get up to speed on the problem. Also allows them to challenge the proposal.
- Refine the problem and solution. Make adjustments to project requirements and scope as necessary.

### Allocated time

- 30 mins - 1 hour meeting. Larger projects may need multiple sessions.

### Deliverables

- Grooming document updated with questions answers. Note down decisions and any adjustments to the project.
- Create tickets for the tasks. These tasks are now ready to be picked up by engineers on the team, including myself.

For questions that do not have answers, the team will make a decision together on how to work around it, or potentially exclude it, from the project. That is, remove it from the scope of work. This decision can sometimes happen during the meeting, or sometimes the product manager or tech lead or designer need extra time to think, and we follow up another time.

Towards the end of the meeting, I like to ask the product manager for the expected start date. Sometimes the project is expected to start immediately. Other times, the project is queued up and scheduled to be developed at later date. This gives me a clear sense of the priority, and I can allocate my time for my ongoing projects accordingly.

Once the team has finished the alignment meeting, it’s mainly about execution from here! So, let’s go!

## Stage 3: Active Development

Now I dive into code. The tasks have been broken down. I can start picking them up.

### Goals

- Pick up and work through project tasks.
    - Write code
    - Write tests
    - Write docstrings
- Handle unexpected items
    - Communicate these items to the team immediately
    - Methodically work through items
    - Iterate with additional grooming and aligning if necessary.

### Time Allocation

- Varies depending on the project. I’m not specifying a time here. It’s easier to estimate each task once they have been broken down, and then add them up to estimate the overall project’s time. Typically, the team categorizes projects loosely as "small rock", "medium rock", and "large rock".

### Deliverables

- PR ready for review

My team strives to iterate on our code development. We have 3 rough phases within the Active Development stage:

1. Implement the functionality
2. Connect the frontend and backend
3. Polish up details for UI or edge cases

Following these phases allows us to get the core functionality in place, then worry about polishing details later.

Sometimes, unexpected items pop up during Active Development. Usually this happens when I decide not to dive deeper during the Grooming stage and relied on assumptions that turn out to be incorrect. When this happens, I’ll take a step back and enter a mini Grooming and Aligning session to hammer out the details. I’ll dive into the code to find out more about this unexpected item. If it requires a decision from the project leaders, I’ll discuss my findings with the product manager, tech lead, or designer.

It is normal for there to be unexpected items, but I have to keep on top of them by letting my team be aware of my progress. That way, there will be no surprises for why things are taking a bit longer than expected.

If I have really good intuition, these unexpected items would have been identified upfront during the Grooming and Aligning stages. I would have been able to allocate time I need to explore these items before Active Development even begins! However, I’m not perfect, so I’ll miss some details every now and then, and that’s ok. As long as I keep my team aware, and I take the time to re-enter grooming and aligning stages, I’ll be able to get unblocked and continue with development!

Once development is done, I send the code into review!

## Stage 4: Code Review

The Code Review stage is where I get feedback on my code. This is done to improve the code quality and ensure it meets the standards for the team.

### Goals

- Check for correctness of the functionality.
- Check that it meets technical requirements and aligns with the team’s choice of patterns, components, and packages to utilize.
- Improve code quality by pointing out areas that can be refactored
- Reviewer will provide feedback. Weigh their feedback and incorporate the suggestions into the code.

### Check list

Our team has a checklist when we open a PR that is ready for review. This is a simplified version:

- Fill out the PR description with details on what code changes were made and the decisions that went into it.
- Include screenshots to illustrate the changes. For UI changes, include before and after images. Alternatively, record a video to demonstrate the functionality.
- Make sure tests are all passing
- Double check if migration files have been included
- Add a release note

### Time Allocation

- Varies depending on project and availability of reviewers.

### Deliverables

- All requested changes have been addressed
- All tests pass
- Reviewer’s approval

Ok, I know I said earlier that, “once development is done, I send the code into review!” This is not completely true. Sometimes I’ll send a draft PR into review. This allows me to get quick feedback and make sure I am on the right path. I’ll do this when I think my implementation could use an extra pair of eyes. The tests do not need to pass in this situation. I’ll tell the reviewer, “This is just a draft! The tests are not passing yet. I just need your eyes to make sure I’m on the right path!” The reviewer knows exactly what to look for in the code and offer a quick review.

If all the previous stages were done well, code review should be a breeze with just minor comments. The details of the implementation should have been discussed and agreed upon during the Grooming and Aligning stages.

Any grooming or aligning that is happening during the code review is a big red flag. If this happens, I have failed to follow the process! I have failed as an engineer. I’m embarrassed to admit that it has happened before (recently). The failure is motivating me to write and publish this blog entry. I can have an explicit process to follow and avoid such mistakes in the future.

Once the PR gets passed Code Review, it is ready to undergo QA!

## Stage 5: QA

On our team, the product manager is in charge of manual QA, which is the final step before release. My role here is to prepare the sandbox for the product manager and provide some QA steps.

### Goals

- Deploy code to sandbox and prepare data on the sandbox
- Write steps for QA for special scenarios
- Work with product manager to ensure the implementation matches the product requirements

### Time Allocation

- Varies by project size. Can be 5 minutes for small projects, or 1 hour for a large one. Can be multiple sessions if bugs are found.

### Deliverables

- Meets all items outlined in the project "acceptance criteria"
- Product manager’s approval

If there are cases that the product manager has not yet considered — not yet outlined in the product requirements or acceptance criteria — then I will write them up, update the ticket, and notify the product manager. These cases tend to involve more technical details, such as inspecting specific data records or ensuring a specific order of events. We conduct manual QA on these cases instead of writing automated tests because we want to confirm the code’s behavior in a real, live, complex system that our automated tests are not set up to handle.

QA can be laborious, especially when our team is conducting it manually. However, once there is a process in place, and you follow it well, it can get easier and easier. Ideally, more of the QA would be automated, but our team is not quite at there yet! For now, we invest our time to ensure high quality and minimize the bugs. Keep in mind, we’re also balancing quality with delivery time. Lots of balancing acts! I love the challenge.

Once I get QA approval and Code Review approval, the code is ready to merge! I can take a moment to breathe and celebrate with the team. Then I’ll repeat the 5 stages with the next project.

## Appreciation

It took me quite some time to get used to working with a well-defined process. Part of the reason is that there was no well-defined process when I first joined the team. The team was growing and still learning the ropes.

Generally, I think the software development process has always been more or less the same, but it wasn’t explicitly laid out. It was implied that all engineers knew exactly what was expected from the product manager, tech lead, and designer. However, that is rarely true, and very commonly the source of team failures.

The other part of the reason is that I resented any form of process. I didn’t want to follow steps and rules. I used to think that I was a much better engineer before I joined this team. I thought I was above it all, that I could work more efficiently without a process. I didn’t realize how wrong I was until I kept repeating my mistakes. My projects would drag on and on, and many times it seemed like there was no end in sight. I was miserable. The product manager, tech lead, and my engineering teammates were not happy either. It wasn’t until I was given more responsibilities that I learned to follow the process more diligently.

Now I appreciate the process! It is what makes me a better engineer. Each stage helps me get into the right mindset for the tasks that I need to perform. I have the tools and structure to get unblocked when something unexpected comes up. It helps me plan projects, which improves how I manage my time. It helps me break down my tasks, which helps me execute efficiently instead of getting overwhelmed with implementing too many things at once. I know how and when to communicate with the product manager, tech lead, and designer on the team.

Overall, my 5 stage software development process helps reduce the risks of the project getting out of hand. The bulk of the work is done when there is dedicated time for planning and collaborating in the first two stages. The entire team is fully present to participate in giving feedback. The last 3 stages will then follow pretty smoothly. It’s very effective, and allows our team to deliver success! I have so much appreciation for it now.

What does your software development process look like? What part was a game changer for you in your engineering career?
