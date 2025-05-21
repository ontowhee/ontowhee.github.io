---
layout: post
title:  "Searching Django Contributor Activities"
date:   2025-05-19
---

The search tab on Django's Trac ticket management system is a powerful tool. It can search for git commits that were merged into the main or release branches. It can search for content on the wiki page. It can also search on multiple fields on tickets.

In this [video](https://youtu.be/NaHzmw31aoA), I walk through the specific use case of looking for activities from a contributor. I also demonstrate a [tool](https://django-contributor.ontowhee.com/trac/contributor/) that could make it easier to conduct such searches.

## Searching On Names

The search can be used by contributors looking for tickets mentioning specific phrases, or it can also be used by working groups who are looking for activities from contributors.

Here's a screenshot of looking up a contributor by their name.

![Django Trac Search For A User](/assets/images/django-trac-search-user.png)

The search is able to match on search terms across multiple fields including summary, description, reporter, owner, cc, and comment contents.

However, the results of one search may not provide the full picture about a contributor. For example, searching on the user display name returns different results than searching on the username. This means we have to try different search terms before we can start to get a complete story. We have to be aware of the nuances between username and display name in the first place! That is not quite apparent in the web interface.

Also, the presentation of the results are very raw and not easy to digest. A better mechanism will save a lot of time for looking up a contributor.

## Looking Up A Django Contributor Profile

I'm working on a tool, and one of the features allows you to search for a "Django Contributor Profile". Here's a screenshot of a contributor's profile.

![Django Contributor Profile](/assets/images/django-contributor-profile.png)

It provides a summary of the contributor's activities. The timeline chart gives a sense of how the contributor has been participating on Trac in the recent times. There is also information on which Triage Stage and which component (contrib.admin, Migrations, Testing Framework, etc.) they are most engaged in. This gives an idea of the contributor's main roles and areas of expertise.

There is a table with a list of tickets. For each ticket, it shows whether the contributor was the reporter, owner, and how many comments they authored on that ticket.

## Next Steps

We can get a good sense of a contributor's involvement with Django. The next step would be to provide access to information on the component and community levels.
