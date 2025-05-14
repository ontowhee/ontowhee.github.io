---
layout: post
title:  "Dataclass For Django Custom Command Arguments"
date:   2025-05-14
---

## Dataclasses

I was cleaning up a custom Django command yesterday and ended using a dataclass as a way to keep the code DRY.

If dataclasses are new to you, don't worry. Dataclasses are similar to classes, but they have boilerplates that handle many common method and attribute definitions for you. For example, instead of you having to explicitly define the constructor and assign attributes such as below,

```python
class MyClass:
    def __init__(self, param1: str, param2: int, ...):
        self.param1 = param1
        self.param2 = param2
        ...
```

this code is already defined and hidden away. You just need to define the fields.

```python
@dataclass
class MyClass:
    param1: str
    param2: int
    ...
```

There are many useful features of dataclasses. In my specific use case, I leveraged a dataclass for two purposes:
- To group of a set of variables and logic that are commonly used together.
- To centralize default value definitions.

## The Original Custom Command

My custom command allows me to scrape a website. It takes in arguments from the command line and builds the url query string for the website. I can run a command like this:

```bash
python manage.py scrape --start-date=2025-05-01 --end-date=2025-05-14 --max-items-per-page=20
```

None of the arguments are required, but they have default values whose constants are defined near the top of the file.

```python
from django.core.management.base import BaseCommand


BASE_URL = "https://code.djangoproject.com/query"
DATE_STR = (datetime.now() - timedelta(days=3)).strftime("%Y-%m-%d")
MAX_ITEMS_PER_PAGE = 100
START_PAGE = 1

...
```

The arguments are registered with their default values by invoking `parser.add_argument(..., default=<CONSTANT>, ...)`.

```python
...

class Command(BaseCommand):

    def add_arguments(self, parser):
        parser.add_argument("--start-date", default=DATE_STR, help="Start date in format YYYY-mm-dd")
        parser.add_argument("--end-date", default="", help="End date in format YYYY-mm-dd")
        parser.add_argument("--start-page", default=START_PAGE, type=int)
        parser.add_argument("--max-items-per-page", default=MAX_ITEMS_PER_PAGE, type=int)

...
```

When the command is run, it instantiates an object for the scraper class using the values from the command line arguments.

```python
...

class Command(BaseCommand):
	...

    def handle(self, *args, **options):
        scraper = TracScraper(
            options["start_date"],
            options["end_date"],
            options["start_page"],
            options["max_items_per_page"],
        )
        scraper.scrape()

...
```

The constructor for the scraper class consists of keyword arguments which are used to initialize the object attributes. The default values for the keyword arguments are coming from the constants defined near the top of the file. The constructor also includes logic to build the url query string from the attributes.

```python
...

class TracScraper:
    def __init__(self, start_date=DATE_STR, end_date=DATE_STR, start_page=START_PAGE, max_items_per_page=MAX_ITEMS_PER_PAGE):
        self.start_date = start_date
        self.end_date = end_date
        self.start_page = start_page
        self.max_items_per_page = max_items_per_page

        query = f"changetime={self.start_date}..{self.end_date}&start_page={self.start_page}&max_items_per_page={self.max_items_per_page}"
        self.url = f"{BASE_URL}?{query}"

    def scrape(self):
        ...
```

This seems typical for a custom command. However, there are a few places of redundancy.
- While global constants are used to define the default values, referencing them in the function signature of `TracScraper.__init__()` can feel unwieldy and repetitive when there is a much larger number of command line arguments.
- When another scraper class is introduced for the same website, not only will it likely have a similar function signature for its constructor class, but it will also need the logic for building the url.

## Using Dataclass

I now use a dataclass that includes a field for each command line argument. It also includes the `base_url`. The global constants near the top of the file have been removed and their values are set directly on the fields.

```python
from dataclasses import dataclass
from django.core.management.base import BaseCommand


@dataclass
class Params:
    start_date: str = (datetime.now() - timedelta(days=3)).strftime("%Y-%m-%d")
    end_date: str = ""
    max_items_per_page: int = 100
    start_page: int = 1

    base_url: str = "https://code.djangoproject.com/query"

    ...

```

I add a field for `start_url` that defaults to an empty string. Then in the `__post_init__()` method, I included the logic that builds and assigns the query string to `start_url`.

```python
...

@dataclass
class Params:
    ...

    start_url: str = ""

    def __post_init__(self):
        query_params = f"changetime={self.start_date}..{self.end_date}&max={self.max_items_per_page}&page={self.start_page}"
        self.start_url = f"{self.base_url}&{query_params}"

...
```

When registering the command line arguments, an instance of the `Params` dataclass is created with all the fields assuming their default values.

```python
class Command(BaseCommand):

    def add_arguments(self, parser):
        # Instantiate a dataclass where all fields assume their default values.
        p = Params()

        parser.add_argument("--start-date", default=p.start_date, help="Start date in format YYYY-mm-dd")
        parser.add_argument("--end-date", default=p.end_date help="End date in format YYYY-mm-dd")
        parser.add_argument("--start-page", default=p.start_page, type=int)
        parser.add_argument("--max-items-per-page", default=p.max_items_per_page, type=int)

```

When the command is run, it instantiates an object for the dataclass using the values from the command line arguments, then passes this `Params` object to instantiate the scraper class.

```python
class Command(BaseCommand):
    ...

    def handle(self, *args, **options):
        p = Params(
            options["start_date"],
            options["end_date"],
            options["start_page"],
            options["max_items_per_page"]
        )
        scraper = TracScraper(p)
        scraper.scrape()
```

The constructor for the scraper class now consists of just the `Params` dataclass. It no longer needs to be concerned about defining the function signature with keyword arguments and their corresponding default values. It also doesn't need to be concerned about the logic for building the url query string. All the variables are grouped together, including the `start_url` variable derived from the command line arguments.

```python
class TracScraper:
    def __init__(self, params: Param):
        self.params = params

    def scrape(self):
        ...
```


## Full Code

Here is the original custom command file:

```python
from django.core.management.base import BaseCommand


BASE_URL = "https://code.djangoproject.com/query"
DATE_STR = (datetime.now() - timedelta(days=3)).strftime("%Y-%m-%d")
MAX_ITEMS_PER_PAGE = 100
START_PAGE = 1


class Command(BaseCommand):

    def add_arguments(self, parser):
        parser.add_argument("--start-date", default=DATE_STR, help="Start date in format YYYY-mm-dd")
        parser.add_argument("--end-date", default="", help="End date in format YYYY-mm-dd")
        parser.add_argument("--start-page", default=START_PAGE, type=int)
        parser.add_argument("--max-items-per-page", default=MAX_ITEMS_PER_PAGE, type=int)

    def handle(self, *args, **options):
        scraper = TracScraper(
            options["start_date"],
            options["end_date"],
            options["start_page"],
            options["max_items_per_page"],
        )
        scraper.scrape()

class TracScraper:
    def __init__(self, start_date=DATE_STR, end_date=DATE_STR, start_page=START_PAGE, max_items_per_page=MAX_ITEMS_PER_PAGE):
        self.start_date = start_date
        self.end_date = end_date
        self.start_page = start_page
        self.max_items_per_page = max_items_per_page

        query = f"changetime={start_date}..{end_date}&start_page={start_page}&max_items_per_page={max_items_per_page}"
        self.url = f"{BASE_URL}?{query}"

    def scrape(self):
        ...
```

Here is the updated file in one piece after using the dataclass:

```python
from dataclasses import dataclass
from django.core.management.base import BaseCommand


@dataclass
class Params:
    start_date: str = (datetime.now() - timedelta(days=3)).strftime("%Y-%m-%d")
    end_date: str = ""
    start_page: int = 1
    max_items_per_page: int = 1000

    base_url: str = "https://code.djangoproject.com/query"
    start_url: str = ""

    def __post_init__(self):
        query_params = f"changetime={self.start_date}..{self.end_date}&max={self.max_items_per_page}&page={self.start_page}"
        self.start_url = f"{self.base_url}&{query_params}"


class Command(BaseCommand):

    def add_arguments(self, parser):
        p = Params()
        parser.add_argument("--start-date", default=p.start_date, help="Start date in format YYYY-mm-dd")
        parser.add_argument("--end-date", default=p.end_date, help="End date in format YYYY-mm-dd")
        parser.add_argument("--start-page", default=p.start_page, type=int)
        parser.add_argument("--max-items-per-page", default=p.max_items_per_page, type=int)

    def handle(self, *args, **options):
        p = Params(
            options["start_date"],
            options["end_date"],
            options["start_page"],
            options["max_items_per_page"]
        )
        scraper = TracScraper(p)
        scraper.scrape()

class TracScraper:
    def __init__(self, params: Param):
        self.params = params

    def scrape(self):
        ...
```


# Final Thoughts

What are your thoughts on this particular use case for grouping command line arguments into a dataclass?

I think it is interesting and it also seems like a natural use case. I like how much easier it is to create the scraper class without worrying about redefining the default values for the keyword arguments. It makes the function signature much shorter. It also reduces the likelihood of assigning the wrong default values and having them be out of sync between the scraper class and the command line argument registry.

How have you used dataclasses? What do you like about them?
