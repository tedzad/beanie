# Event-based actions

You can register methods as pre- or post- actions for document events.

Currently supported events:
- Insert
- Replace
- SaveChanges
- ValidateOnSave

Currently supported directions:
- Before
- After

Current operations creating events:
- `insert()` and `save()` for Insert
- `replace()` and `save()` for Replace
- `save_changes()` for SaveChanges
- `insert()`, `replace()`, `save_changes()`, and `save()` for ValidateOnSave

To register an action you can use `@before_event` and `@after_event` decorators respectively.

```python
from beanie import Insert, Replace

class Sample(Document):
    num: int
    name: str

    @before_event(Insert)
    def capitalize_name(self):
        self.name = self.name.capitalize()

    @after_event(Replace)
    def num_change(self):
        self.num -= 1
```

It is possible to register action for a list of events:

```python
from beanie import Insert, Replace

class Sample(Document):
    num: int
    name: str

    @before_event([Insert, Replace])
    def capitalize_name(self):
        self.name = self.name.capitalize()
```

This will capitalize the `name` field value before each document insert and replace.

And sync and async methods could work as actions.

```python
from beanie import Insert, Replace

class Sample(Document):
    num: int
    name: str

    @after_event([Insert, Replace])
    async def send_callback(self):
        await client.send(self.id)
```

Actions can be selectively skipped by passing the parameter `skip_actions` when calling
the operations that trigger events. `skip_actions` accepts a list of directions and action names.

```python
from beanie import Insert, Replace, Before, After

class Sample(Document):
    num: int
    name: str

    @before_event(Insert)
    def capitalize_name(self):
        self.name = self.name.capitalize()

    @before_event(Replace)
    def redact_name(self):
        self.name = "[REDACTED]"

    @after_event(Replace)
    def num_change(self):
        self.num -= 1

sample = Sample()

# capitalize_name will not be executed
await sample.insert(skip_actions=['capitalize_name'])

# num_change will not be executed
await sample.replace(skip_actions=[After])

# redact_name and num_change will not be executed
await sample.replace(skip_actions[Before, 'num_change'])
```
