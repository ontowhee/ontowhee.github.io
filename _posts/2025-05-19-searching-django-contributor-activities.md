---
layout: post
title:  "Searching Django Contributor Activities"
date:   2025-05-19
---

The search tab on Django's Trac ticket management system is a powerful tool. It can search for git commits that were merged into the main or release branches. It can search for content on the wiki page. It can also search on multiple fields on tickets.

# Searching On Names

The search can be used by contributors looking for tickets mentioning specific phrases, or it can also be used by working groups who are looking for activities from contributors.

![Django Trac Search For A User](/assets/images/django-trac-search-user.png)

In this [video](https://youtu.be/NaHzmw31aoA), I walk through the case of looking for activities from a contributor. The search is able to match on search terms across multiple fields including summary, description, reporter, owner, cc, and comment contents.

However, the results of one search may not provide the full picture about a contributor. For example, searching on the user display name returns different results than searching on the username. This means we have to try different search terms before we can start to get a complete story. We have to be aware of the nuances between username and display name in the first place! That is not quite apparent in the web interface.

Also, the presentation of the results are very raw and not easy to digest. A better mechanism will save a lot of time for looking up a contributor.

# Looking Up A Django Contributor Profile
I'm working on a tool, and one of the features allows you to search for a "Django Contributor Profile". It provides a summary of the contributor's activities and also lists the tickets they have participated in. Here's a screenshot of a contributor's profile.

![Django Contributor Profile](/assets/images/django-contributor-profile.png)

Want to access the tool? Send me a private message on discord and I'll share the link.
