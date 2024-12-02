---
layout: post
title:  "When a ticket description is outdated -- Working through an issue with django-filter upgrade from 10 months ago"
date:   2024-03-31
---

I learned from my rotation as a triage engineer that sometimes, when the author of the ticket documents the issue they encountered, it is possible for that issue to no longer be valid. What it describes no longer holds true by the time you pick it up. That is, the ticket is outdated.

As the title of this blog suggests, the ticket I am writing about falls into this category. Even though the ticket was outdated, and the actual code change was minimal, I feel very proud of the work I did. I am writing up my thought process as I worked through the issue, what I learned, and how I communicated my findings and the resolution to the teams that were impacted.

### The Ticket

After our engineering team’s first technical debt grooming meeting this week, I picked up a ticket from the top of the “groomed” list in the Jira board. The ticket was 10 months old. It was titled “django-filter package upgrade”. At the time the ticket was created, the platform team was working through a major project to bring our company’s application up-to-date with Django’s latest version. There were several versions to upgrade through, and all the dependency packages needed to be updated alongside Django as well.

There were lots of moving parts, and the platform team ran into a problem while updating django-filter. They were able to get the application up to Django 4.2, and they made the decision that it was good enough for now. They didn’t have time to further investigate the issue with django-filter, as their priorities shifted to other projects. So they documented the issue, and the ticket sat untouched for 10 months.

My first experience with package upgrades happened when I attended Django Con 2023 online. The organizers hosted testathons, and I was lucky enough to have attended one session. The host walked through how to upgrade dependency packages for open source projects. I learned what files to start looking for (tox.ini, requirements.txt, CI/CD config files such as circleci.yml, as well as any files documenting the process for contributing). I learned how to bump a version of a dependency and run test suites for the project. I learned how to install the project in edit mode and fix the failing code if anything needs fixing. Or, if the effort is much larger and beyond one’s knowledge, open up the PR anyway with the bumped version for the dependency, and report the issues for the failing tests. Most importantly, I learned to dive right into code and poke around, despite seeing the source code for the first time.

When I picked up the ticket for “django-filter package upgrade”, I was not too familiar with the inner workings of django-filter (and I still am not!). I didn’t know what could be causing the test failures, but I was armed with the experience from the testathon to know what the process of upgrading a package version looks like. I was also armed with the the understanding of the possibility that the ticket’s description might be outdated. I was ready to dive right in and help unblock the platform team.

### The Process

After reading the ticket, these questions came immediately to my mind:

- What is the current package version?
- What version was attempted for upgrade?
- What were the changes between versions?

I looked in the requirements.txt file for our company’s application to see that django-filter was currently pinned to 21.1. The ticket’s description mentions that the upgrade encountered the failing tests when attempting to bump up to version 22.1. That is just one version up from our latest code. I then looked into the changelog for django-filter, which I found here, https://github.com/carltongibson/django-filter/blob/main/CHANGES.rst. I read through the items listed for all versions from 21.1 to the latest 24.2. The major change that stood out to me was the line mentioning changing filter_class to filterset_class and changing filter_fields to filterset_fields.

>
Removed filter_class (use filterset_class) and filter_fields (filterset_fields) that were deprecated in [version 2.0 (2018)]

I didn’t quite understand the implications of this changelog item. I considered my next step. My options were:

- Inspect the source code to understand any structural or implementation changes.
- Or, take a test-driven development approach to run our application test suite and see what happens.

I chose the second option, because I needed more clues, and the results of any failed tests would narrow down the problem and guide me in the right direction. (Hooray to our engineering team for writing quality tests!)

I had the latest version of the application code pulled down, which means it was still using django-filter 21.1. First, I ran the tests as a sanity check and made sure all the tests passed. Then, I bumped django-filter to 22.1 by pinning it in the requirements file and running `pip install -r requirements.txt`. I ran the test suite again. What I found was a bit confusing. The tests that were failing for me were not mentioned in the description. This wasn’t terribly surprising, because it was expected that there were more failing tests than what was documented. However, the tests that _were_ documented as failing were now passing! I couldn’t believe my eyes. What was going on? Shouldn’t they be failing? How did this ticket get written up?

After the initial confusion, more questions came to my mind. Was it a change in django-filter that resolved the issue? Or was it a change in our application that resolved the issue? What would that change be? Was the issue even resolved?

### The Documented Failing Tests

The output of the failing tests that were documented in the ticket looked something like this:

```bash
======================================================================
FAIL: test_api (application.tests.ApiTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/application/tests.py", line 1363, in test_api
    self.assertEqual(response.data['count'], 3)
AssertionError: 6 != 3
```

This particular line is testing the number of items returned by the filter. It was not returning the expected number of items.

I started with the clue from the changelog. I asked myself the following questions: Is our code base using `filter_class` and `filter_fields`, or is it using `filterset_class` and `filterset_fields`? Which filter class in our application code base was part of the failing test?

I did a quick search in my IDE and pulled up the filter class. I saw that the variable being used was `filterset_class`. So our code had already been updated to be compatible with 22.1. Then I wondered, “Did this change occur before or after the ticket reported that django-filter encountered problems during package upgrades?”

I opened up git blame and I placed my cursor on the line for the `filterset_class` to see who changed it and when it was changed. Git blame showed me that the code was changed 5 months ago by a co-worker who was not on the platform team. The ticket was created 10 months ago by the platform team. So this change happened after the ticket was created, and the platform team wasn’t aware. This makes a lot of sense now. It explains why the documented failing tests are now observed to be passing! It also explains why this ticket has remained on the tech debt board.

I wasn’t done yet. Notice that I had pulled down the most recent version of our application code (which had already been updated to use `filterset_class`)? The platform team was working with an older version of the application code. I wanted to confirm with high confidence that the platform team observed the failing tests because the filter class was using `filter_set` as the variable at the time. To do so, I reproduced their set up by temporarily changing the `filterset_class` variable to `filter_set` and ran the tests. The results of the tests matched exactly what was documented! Now I had all the information I needed to confirm how the tests were observed to be failing, and how they are now passing.

### The Undocumented Failing Tests

So far, I addressed one set of failing tests. Now I had to address the other failing tests, the ones that had not been captured in the ticket’s description. The failed tests showed messages that looked like this:

```bash
======================================================================
FAIL: test_query_counts (application.tests.FilterTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/application/tests.py", line 148, in test_query_counts
    self.assertTrue('some_value' in expected_results)
  File ".../lib/python3.9/site-packages/django/test/testcases.py", line 99, in __exit__
    self.test_case.assertEqual(
AssertionError: 25 != 24 : 25 queries executed, 24 expected
Captured queries were:
...
```

It’s not visible here in the stack trace, but the failure happens on a `with self.assertNumQueries(24):` statement that is wrapping around `self.assertTrue()`. The tests were failing because the query counts do not match up with what the expected counts.

Luckily, I recently read through a lot of Django tickets on Trac because I was looking for an open source issue to work on. I remember coming across one ticket regarding a bug in `assertNumQueries`. I was able to quickly opened up Django’s Issues and search for this Trac ticket. It came right up! If you are interested, read more about it here, https://code.djangoproject.com/ticket/23746.

To confirm that our application tests were failing due to the bug in `assertNumQueries`, I dropped the test database and ran the tests again. The tests passed! Then I ran the tests again with REUSE_DB turned on. The tests failed! This behavior matches exactly what is described in the `assertNumQueries` ticket! I was able to conclude that the tests were failing due to a bug in Django, and not due to our application’s code.

### Running All Test Suites With django-filter 24.2

At this point, I hadn’t run all the tests in the application yet, just a targeted group from one part of our application. I also hadn’t checked the latest version of django-filter, which is 24.2. I didn’t have a mechanism at this very moment to efficiently run all the tests locally on my machine and capture all the results for viewing. It was much easier for me to run them all on CircleCI.

So, I bumped up django-filter to the latest version, I committed the requirements.txt file, pushed up the branch, opened a PR, and allowed CircleCI to run all the test suites in the application. After all the tests ran, I checked back, and to my surprise and delight, they all passed! Green check marks down the list!

Once I knew where to look, this issue turned out to be pretty straightforward to address. Even though there was not much to change on my end, other than bumping django-filter version in requirements.txt file, I was really happy to resolve the ticket and unblock the team from upgrading to Django 5.1.

### Report

Finally, I was able to report my findings by posting a comment on the ticket:

- The documented failed tests have been addressed. The created date for the “django-filter package upgrade” ticket along with date of the our application code change explains why the ticket shows the tests as failing, while current test runs show the tests as passing.
- Django’s `assertNumQueries` has a [known bug](https://code.djangoproject.com/ticket/23746) when running tests with REUSE_DB turned on.
- Bumping to 24.2 shows that all tests are passing.
- Conclude that the ticket is resolved and is no longer a blocker for upgrading to Django 5.1.

I tagged the platform team in my comment, and I asked for their review and suggestions on the next steps required to get the code merged into the main branch. They gave me the approval to merge the code. They marked the ticket as Done and moved it off of the technical debt board!

### Summary Of Steps

1. Read the ticket and understand where the author of the ticket is coming from.
2. Attempt to replicate the ticket. Step in the ticket author’s shoes. Pull down the version of the application code and packages at the time of the ticket creation. Compare it to the most recent application code and packages.
3. Debug any failed tests.
4. Implement a fix if it is small and straightforward. Otherwise, document the remaining issues.
5. Report your findings to those who will be impacted. Include any suggestions for the next steps if you have any, or ask for suggestions. Keep the train moving!

### Conclusion

I’m really excited that the platform team is now unblocked by django-filter package. When they are ready to pick up where they left off on package upgrades, they’ll be able to hit the ground running. I’m also excited that I got to leverage my budding familiarity with navigating Django’s Trac ticketing system to reference known bugs in Django. This was crucial in helping me conclude that the test failures we observed were not related to our code base, and we can move forward.

### What I Learned

- Never assume that the problem described in a ticket is accurate.
- Read the changelog for the packages you are upgrading.
- Run your test suites and see what fails. Use that to narrow down the problem and guide you in the right direction.
- Check git blame for the authors and dates of code changes.
- Check git diff for the content of the code changes.
- Django’s `assertNumQueries` has a [known bug](https://code.djangoproject.com/ticket/23746) where the cache is not cleared and the result will vary between runs if you are running with REUSE_DB turned on. The first run will return the correct result, but the subsequent runs might not.
- Confidently report your findings to those who will be impacted and ask for their thoughts and suggestions for how the team can move forward from here.


I hope you enjoyed this article! I enjoyed undergoing the process of resolving the issue and recreating the experience by writing it all up!
