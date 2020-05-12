# BestDoctor Python styleguide

## Goals

We are developing a huge product with tons of functionality. Moreover, we need to maintain and improve our existing codebase.

In order to make this process as painless as possible, the code needs to be
flexible, loosely coupled, comply with DRY and KISS principles...
The list goes on. Otherwise, developers, technical support or the business
itself will get an unpleasant surprise later down the line.
And you want to avoid that.

In order to get the code that meets our standards,
we need to constantly keep our focus. That in itself can be quite challenging
as it requires a lot more awareness than simply writing code.
We made this styleguide, so it can help us keep track of our solutions
to most common problems, instead of having to reinvent the wheel each
time one of those problems comes up.

This guide contains no rules, only guidelines. We follow these
guidelines in order to write amazing code and not just
for the sake of following them.

It is also very important to us that our code satisfies our team.
That's why the guidelines below represent our personal preferences
that may not align with yours.

## You are not required to follow these guidelines

As a matter of fact, in some cases it is better not to follow this styleguide.
There are situations when it is okay to write `import *`, `except Exception`,
use tab indents or write really long lines of code.
You can do that if you are absolutely certain that it will be beneficial
to your codebase. However, those situations are quite rare.

## The latest Python release

Use 3.8 in production. Use 3.8 in staging. Use 3.8 on every instance.
Think in 3.8 as well.

## PEP8

We love and follow PEP8 except for the maximum line length.
We have set our upper-limit to 120 characters.

## Comments

### No unnecessary comments

There are rules that require writing comments in certain places,
e.g. public modules, classes, methods.

We don't have such rules. We think that you should only write a comment
when something in your code acts in an unexpected manner.

If you can't figure out the purpose of an entity, then you should
consider renaming it. Maybe it will help clear things up.

### Oneline docstrings

A quick way to write a docstring which can fit in one line. Keep the comment
and the quotation marks on the same line and don't forget a full-stop.

Bad:

```python
"""
Statistics on clients' legal entities.
"""
```

Bad:

```python
"""Statistics on clients' legal entities"""
```

Good:

```python
"""Statistics on clients' legal entities."""
```

### Multiline docstrings

Used when you need to provide a description which exceeds one line.
Here's how it works: the first line should be used to describe
what you are talking about. It ends with a full-stop.
The first line and the description are separated by an empty line.
There should be no spaces between the text and the quotation marks.

Bad:

```python
"""
Active service period.
Functions as inteded when user has one or less slots
"""
```

Bad (missing a full-stop):

```python
"""
Active service period

Functions as inteded when user has one or less slots
"""
```

Good:

```python
"""
Active service period.

Functions as inteded when user has one or less slots
"""
```

### Inline comments

You can and you should use them when you need to describe tricky logic,
refer to a ticket or to a wiki article.

You shouldn't use them to make jokes, to describe a certain part of the code or
to divide it into sections. Leave the first two for the water cooler
conversations and the last one – for code refactoring.

Bad:

```python
# this is shit
```

Bad:

```python
# fetching user data
...
# building a context for the report
...
# generating the report
...
```

Good:

```python
# refer to BES-482 for the formula explanations
```

### TODO and FIXME

We use TODO and FIXME to mark something important when we don't want to get
distracted while writing code. We try to remove them before making a commit.
Instead of keeping those marks, we prefer to either fix those parts
immediately or create a ticket to do it later.

There are two reasons for doing so. One is that those marks can stay
in the code for years and no one does anything about it.
And two is that they make an impression that everything is broken.

Keep your code improvement and refactoring tasks in the task tracker,
not in your code.

### Inline comments and urls

Leaving a link without a note in an inline comment is a bad idea.
It's better to write a description because it will help others get the context
without having to follow the link.

Bad:

```python
# https://stackoverflow.com/q/184618/3694363
```

Good:

```python
# empties the cache properly: https://stackoverflow.com/q/184618/3694363
```

## String literals and formatting

Use single quotes for string literals, but avoid using backslashes in strings.

To format a string, use `.format` or f-strings with indexed or
named placeholders. Do not concatenate strings with `+` operator.

F-strings can be used, but you should avoid putting a lot of logic
inside of them.

Bad:

```python
'%s %s' % (first_name, last_name)
```

Bad:

```python
'{} {}'.format(first_name, last_name)
```

Good:

```python
'{0} {1}'.format(first_name, last_name)
```

## Imports

Third-party libraries should be imported on a case by case basis,
i.e. import only the functions and classes that you need.
Do not import the entire module and for the love of god do not use `import *`.
Resolve name collisions with `as` statement.

Stadard library modules should be imported in their entirety.

Imports should be divided into three sections as described in PEP8.
There are no set rules for sorting the imports inside a section.

Bad:

```python
from collections import Counter
```

Bad:

```python
from core import models
```

Good:

```python
import collections
from core.models import Patient
```

## Code inside `__init__.py`

`__init__.py` should only contain imports and nothing else.

## Calls formatting

Do not use line breaks if a call and all of its' arguments can
fit into a single line.

If one line is not enough, but you can fit it in the next line
after the first bracket, then you should do that.

If there are too many arguments or you can't fit a call into two lines,
then use advanced formatting. In that case, you can place several arguments
in the same line if they have similar meaning. For instance, you can combine
`date_from` and `date_to` or `to` and `on_delete`.

Bad:

```python
recieved_date = models.DateField(
    'Receival date', auto_now_add=True,
)  # everything fits into a sinlge line, no need to break it

legal = models.ForeignKey(
    'bestdoctor.ClinicLegalEntity',
    models.CASCADE,
    'registries', 'registry',
    verbose_name='clinic legal entity',
    )  # closing paranthesis has excess indent
```

Good:

```python
recieved_date = models.DateField(
    'Recieval date', auto_now_add=True)

legal = models.ForeignKey(
    'bestdoctor.ClinicLegalEntity',
    models.CASCADE,
    'registries', 'registry',
    verbose_name='clinic legal entity',
)
```

Good:

```python
legal = models.ForeignKey(
    'bestdoctor.ClinicLegalEntity',
    models.CASCADE,
    'registries',
    'registry',
    verbose_name='clinic legal entity',
)
```

## Function calls and arguments

When calling a function, specify argument names if they are unclear from
the name of the function alone.
You may write them even if they are obvious.

Bad:

```python
serializer.is_valid(True)
```

Good:

```python
serializer.is_valid(raise_exception=True)
```

## Commas

Place a comma after the last item in any kind of iterable.
This includes lists, calls, tuples or anything similar.

If it fits in one line then we don't use a comma.
The only exception is a one element `set`.

Bad (forgot a comma in the end):

```python
list_filter = (
    'legal', 'is_finished', 'is_paid', 'client_registry__record__report_date'
)
```

Bad:

```python
Act.objects.create(
    date=today(),
    act_type='tech',
    report_date=report_date,
    is_finished=False
)
```

Good:

```python
list_filter = (
    'legal', 'is_finished', 'is_paid', 'client_registry__record__report_date',
)

Act.objects.create(
    date=today(),
    act_type='tech',
    report_date=report_date,
    is_finished=False,
)
```

## Unused code

We want as few bugs as possible in our code. A great way to get rid of bugs is
to get rid of code. No code means no problems. That's why we remove everything
that can be removed, such as deprecated features, one-time scripts,
pieces of code which have been commented out. All of it gets cut.

It's not difficult to recover it later. Since all commits are tied to a ticket
in the task manager, to find removed code you need to first find the ticket,
then its commits and then reapply them.

If there is something that you don't need this month then cut it.

## Data in settings

We are doing our best to keep all of the data where it belongs.
Project settings is definitely not the right place for it.
Here's an example of how to move a piece of data from the settings to the
database with the help of `BooleanField`:

Before:

```python
# settings.py
NOTIFY_IN_SLACK_COMPANY_IDS = [123, 456, 789]

# code
send_slack_notification(NOTIFY_IN_SLACK_COMPANY_IDS)
```

After:

```python
# Companies now have notify_in_slack checkbox
notify_in_slack_company_ids = Company.objects.filter(notify_in_slack=True).value_list('id', flat=True)
send_slack_notification(notify_in_slack_company_ids)
```

## Rules for fields in Django models

1. The order of fields is defined in
  [flake8-class-attributes-order](https://github.com/best-doctor/flake8-class-attributes-order).
1. `DateTimeField` names should end with `_at`.
1. `DateField` names should end with `_date`.
1. If a field has `choices` attribute, then it should be implemented using `Enum`.
1. Define `__str__` method for each model. Make sure that it does not generate a lot of database queries.
1. Every `ForeignKey` field must have a defined `related_name`.
1. Use `.pk` instead of `.id` to get the model identificator.

## Rules for Django urls

1. Don't use decorators inside an app's `urls.py`. Use  `method_decorator` in
   Class-Based Views for methods defined for their respective HTTP verbs.
   More info about this decorator can be found in
  [oficial Django docs](https://docs.djangoproject.com/en/2.2/topics/class-based-views/intro/#decorating-the-class).
