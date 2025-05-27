---
layout: post
title:  "Creating A Temporal Table To Track Django Development Process"
date:   2025-05-26
---

# Temporal Table

Temporal data refers to data that is associated with time periods. A time period consists of a start time and an end time.

A temporal table is a database table that stores time periods. There are two types of temporal tables. One is application-time versioned table, the other is system-time versioned table. The difference is the meaning behind the stored time value.

In application-time, the stored time value reflects the application state. For example, it can be the activation and deactivation dates of a membership to an organization. Application-time is useful for tracking the history of the application state.

In system-time, the time values reflect the moment the data is modified. System-time is useful for auditing purposes.

### SQL\:2011 Temporal Features

SQL:2011 introduced the standards for [temporal features](https://cs.ulb.ac.be/public/_media/teaching/infoh415/tempfeaturessql2011.pdf). These standards include defining a time period using the `PERIOD` declaration, DML for `UPDATE` and `DELETE`, period predicates such as `OVERLAPS`, `CONTAINS`, and `EQUALS` to compare time periods and use them in conditional expressions for filtering and ordering.

It also includes table constraints to ensure the time period columns are an ordered pair of datetime/date fields that satisfy the bounds of the time interval when a `PERIOD` is declared. It also describes primary key and foreign key constraints applied to the time periods.

PostgreSQL does not yet implement temporal tables. There is ongoing work in [Patch #5660](https://commitfest.postgresql.org/patch/5660/) to add support for application-time.

We won't be using built-in database temporal features. Instead, we will be creating a temporal table by adding the ordered pair of time columns. We will manage the data in the application layer.

# Trac

Django uses [Trac](https://trac.edgewall.org/wiki/TracTickets) for ticket management. When we visit a ticket page, we see two main sections: a ticket details section, and a change history section.

Figure 1 is a screenshot of ticket [#12090](https://code.djangoproject.com/ticket/12090). It shows the current state of the ticket, including the fields Reported By, Owned By, Cc, Triage Stage, Has Patch, etc.

![Figure 1: Image Of Ticket Change History](/assets/images/trac_ticket_detail.png "Figure 1: Django Ticket Detail")

The "Change History" section shows how the ticket has been modified over time. In Figure 2, we can see the values of ticket fields such as Status, Triage Stage, and Cc transition from their old value to new value. This section also includes comments and attachments that were uploaded to the ticket.

![Figure 2: Image Of Ticket Change History](/assets/images/trac_ticket_change_history.png "Figure 2: Django Ticket Change History")

A look into the [Trac schema](https://github.com/edgewall/trac/blob/trunk/trac/db_default.py) shows two tables, `ticket` and `ticket_changes`. These tables correspond to the ticket details and change history sections. Here are their schemas:

```python
...

Table('ticket', key='id')[
	Column('id', auto_increment=True),
	Column('type'),
	Column('time', type='int64'),
	Column('changetime', type='int64'),
	Column('component'),
	Column('severity'),
	Column('priority'),
	Column('owner'),
	Column('reporter'),
	Column('cc'),
	Column('version'),
	Column('milestone'),
	Column('status'),
	Column('resolution'),
	Column('summary'),
	Column('description'),
	Column('keywords'),
	Index(['time']),
	Index(['status'])],
Table('ticket_change', key=('ticket', 'time', 'field'))[
	Column('ticket', type='int'),
	Column('time', type='int64'),
	Column('author'),
	Column('field'),
	Column('oldvalue'),
	Column('newvalue'),
	Index(['ticket']),
	Index(['time'])],

...
```

Notice that the `ticket_changes` table is not a temporal table. It only contains one time field. That is, it captures a point in time, rather than an interval of time. This makes it difficult to query for information such as the duration of something, or to know if something was valid at a given time. However, it is a log of data changes, and we can use it to reconstruct the temporal data.

# Creating Temporal Table For Triage Stages

Trac data is acquired by downloading the html pages for a ticket. Data for the ticket details section is extracted using BeautifulSoup. Data for the "Change History" section is extracted from a javascript variable called `changes`.

We are building a Django app. We create a `Ticket` model to store the ticket details. Additionally, we add a JSONField called `changes` to store the data as-is from the javascript `changes` variable .

```python
from django.db import models


class Ticket(models.Model):
    cc = models.TextField()
    changetime = models.DateTimeField(null=True, default=None)
    changes = models.JSONField(null=True, default=None)
    component = models.CharField(max_length=128)
    description = models.TextField()
    easy = models.BooleanField(default=False)
    has_patch = models.BooleanField(default=False)
    keywords = models.TextField()
    needs_better_patch = models.BooleanField(default=False)
    needs_docs = models.BooleanField(default=False)
    needs_tests = models.BooleanField(default=False)
    owner = models.CharField(max_length=256)
    reporter = models.CharField(max_length=256)
    resolution = models.CharField(max_length=128)
    severity = models.CharField(max_length=64)
    stage = models.CharField(max_length=64)
    status = models.CharField(max_length=64)
    summary = models.CharField(max_length=256)
    time = models.DateTimeField(null=True, default=None)
    type = models.CharField(max_length=64)
    ui_ux = models.BooleanField(default=False)
    version = models.CharField(max_length=10)
```

Next, we create a QueueHistory model, and the underlying table for this model is our application-time versioned table. We add two datetime fields, `valid_from` and `valid_to`, and together they create the time period. The other fields in the QueueHistory model hold the information about the state of the development process during this application-time period.

```python
class QueueHistory(models.Model):
    ticket = models.ForeignKey(Ticket, on_delete=models.CASCADE)
    queue = models.CharField(max_length=30)
    author = models.CharField(max_length=128)
    has_patch = models.BooleanField(default=False)
    needs_better_patch = models.BooleanField(default=False)
    needs_docs = models.BooleanField(default=False)
    needs_tests = models.BooleanField(default=False)
    resolution = models.CharField(max_length=128)
    stage = models.CharField(max_length=64)
    status = models.CharField(max_length=64)
    valid_from = models.DateTimeField()
    valid_to = models.DateTimeField()
```

PostgreSQL has [tstzrange](https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-BUILTIN%23:~:text=tstzrange) and [daterange](https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-BUILTIN#:~:text=daterange) data types, and Django supports these data types with the fields [DateTimeRangeField](https://docs.djangoproject.com/en/5.2/ref/contrib/postgres/fields/#datetimerangefield) and [DateRangeField](https://docs.djangoproject.com/en/5.2/ref/contrib/postgres/fields/#daterangefield). We won't be using this field type. We'll be sticking to creating two separate datetime fields. The decision behind this lies in having the flexibility to switch between PostgreSQL and SQLite (which could come in handy if we choose to use Datasette or another tool later).

You may be wondering why this model is called QueueHistory instead of StageHistory. We'll explain more in the next section, but ideally the queue and stage would be synonymous.

# Insights For The "Big Gray Area"

We are building a standalone project, completely independent from Django's Trac system. This provides flexibility to experiment with new stages.

With the current Django Triage Workflow, there are 3 triage stages:
- Unreviewed
- Accepted
- Ready For Checkin

There are also ticket `resolution` values to indicate why a ticket is closed: duplicate, wontfix, invalid, worksforme, needsinfo, fixed.

In our experiment, we will refer to stages as queues for two reasons: 1) avoid confusion with the original triage stages, and 2) ultimately we want queues to be a feature in the UI of the ticket management system.

We break the original Accepted stage into 3 new queues. We also separate the Needs Info resolution into its own queue, and we add Someday/Maybe as a queue as well.

Our queues are:
- Unreviewed
- Needs Info
- Someday/Maybe
- Needs Patch
- Needs Review
- Waiting On Author
- Ready For Checkin
- Closed
- Unknown

We define constants for our queue labels.

```python
# constants.py

class QueueLabel:
    UNREVIEWED = "Unreviewed"

    # Waiting On Info
    NEEDS_INFO = "Needs Info"
    SOMEDAY_MAYBE = "Someday/Maybe"

    # Accepted
    NEEDS_PATCH = "Needs Patch"
    NEEDS_REVIEW = "Needs Review"
    WAITING_ON_AUTHOR = "Waiting On Author"
    READY_FOR_CHECKIN = "Ready For Checkin"

    # Closed
    CLOSED = "Closed"

    # When combination of fields do not correspond to any known queue
    UNKNOWN = "Unknown"
```

We define a function to calculate the queue value given a snapshot of the ticket state as determined by the following fields: `stage`, `status`, `has_patch`, `needs_better_patch`, `needs_docs`, and `needs_tests`.

```python
def calculate_queue_value(snapshot):
    def check_values(snapshot, **kwargs):
        boolean_fields = {"has_patch", "needs_better_patch", "needs_docs", "needs_tests"}
        string_fields = {"stage", "status"}
        return all([
            snapshot[k] is v
            for k, v in kwargs.items()
            if k in boolean_fields
        ]) and all([
            snapshot[k].lower() == v.lower()
            for k, v in kwargs.items()
            if k in string_fields
        ])

    if check_values(snapshot, stage="Unreviewed"):
        return QueueLabel.UNREVIEWED

    if check_values(snapshot, has_patch=False, status="Closed", resolution="needsinfo"):
        return QueueLabel.NEEDS_INFO

    if check_values(snapshot, status="Closed"):
        return QueueLabel.CLOSED_FIXED

    if check_values(snapshot, stage="Someday/Maybe"):
        return QueueLabel.SOMEDAY_MAYBE

    if check_values(snapshot, stage="Ready for checkin"):
        return QueueLabel.READY_FOR_CHECKIN

    if check_values(snapshot, stage="Accepted", has_patch=False):
        return QueueLabel.NEEDS_PATCH

    if check_values(snapshot, stage="Accepted", has_patch=True, needs_doc=False, needs_tests=False, needs_better_patch=False):
        return QueueLabel.NEEDS_REVIEW

    if check_values(snapshot, stage="Accepted", has_patch=True, needs_better_patch=True):
        return QueueLabel.WAITING_ON_AUTHOR

    return QueueLabel.UNKNOWN
```

# Reconstructing Queue History

To reconstruct the queue history, we loop through each change item from the `Ticket.changes` field to gather the timestamps to fill in the `valid_from` and `valid_to` values, and we gather the old and new values of fields to get the ticket state snapshots.

```python
def reconstruct_queue_history(data):
    """
    Creates a list of dicts with the following fields:
        "ticket_id",
        "author",
        "queue",
        "has_patch"
        "needs_better_patch",
        "needs_docs",
        "needs_tests",
        "resolution",
        "stage",
        "status",
        "valid_from",
        "valid_to",
    These dicts are used to create QueueHistory.
    """
    # Fields that impact queue
    fields = [
        "has_patch",
        "needs_better_patch",
        "needs_docs",
        "needs_tests",
        "stage",
        "status",
    ]

    # Only consider the changes that have modified fields.
    # Skip changes where the fields being modified do not impact the ticket's queue.
    changes = []
    for change in data["changes"]:
        modified_queue_fields = set(fields) & set(change["fields"].keys())
        if len(modified_queue_fields):
            changes.append(change)
    changes = sorted(changes, key=lambda elem: elem["date"], reverse=True)
    changes.append({
        "date": data["ticket_details"]["time"],
        "author": data["ticket_details"]["reporter"],
    })

    # Work backwards to reconstruct the history of the field values.
    cur_values = {
        field: data["ticket_details"][field] for field in fields
    }
    cur_values["ticket_id"] = data["ticket_details"]["id"]
    cur_values["valid_from"] = changes[0]["date"]
    cur_values["valid_to"] = datetime.max
    cur_values["author"] = changes[0]["author"]
    queue_history = [{k: v for k, v in cur_values.items()}]

    # Pair the changes to get the date range.
    for change, prev_change in zip(changes, changes[1:]):
        cur_values["author"] = change["author"]
        cur_values["valid_from"] = prev_change["date"]
        cur_values["valid_to"] = change["date"]
        modified_queue_fields = set(fields) & set(change["fields"].keys())
        for field in modified_queue_fields:
            old_value = change["fields"][field]["old"]
            cur_values[field] = old_value

        # Add to history
        queue_history.append({k: v for k, v in cur_values.items()})

    queue_history = sorted(queue_history, key=lambda elem: elem["valid_from"])
    return queue_history
```

The QueueHistory objects are created for a ticket and saved to the database. We can run a query to retrieve the QueueHistory items, order by `valid_from`, and render the data in a chart.

```python
# views.py
def ticket_detail(request, pk):
    item = Ticket.objects.filter(pk=pk).first()
    queue_histories = []
    if item is not None:
        queue_histories = QueueHistory.objects.filter(
            ticket=item
        ).annotate(
            valid_to_cleaned=Case(
                When(
                    valid_to__year=9999,
                    then=datetime.now() + timedelta(days=1),
                ),
                default=F("valid_to"),
            ),
        ).order_by("valid_from")

    template = "trac/ticket_detail.html"
    return render(request, template, {"item": item, "queue_histories": queue_histories})
```

We use ChartJS to create a Gantt chart with time on the x-axis and the queue labels on the y-axis. We no longer have a big gray area. In the example below, we can see how that the ticket transitions between Waiting On Author and Needs Review eight times before it lands on Ready For Checkin. We can also see the how much time the ticket spends in each queue.

![Figure 3: Queue Timeline For Ticket 36917](/assets/images/ticket_34917_queue_timeline.png "Figure 3: Queue Timeline For Ticket 36917")

This gives us a clearer picture of what goes on during the development process.
# Next Steps

We'll want to aggregate the data into reports on the component level and queue level.
