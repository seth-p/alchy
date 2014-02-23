# alchy

Enhancement library for SQLAlchemy

**Under development**

## Requirements

- `SQLAlchemy>=0.9`

## Installation

Install using pip:

```bash
pip install alchy
```

## Overview

The main components of `alchy` are:

- `Manager`: Session manager class
- `Query`: Enhanced subclass of sqlalchemy's `orm.Query` and default query class for `session.query`
- `ModelBase`: Enhanced base class for models which features a query property
- `make_declarative_base`: Factory function for creating a delcarative base class from `ModelBase` (separated for use outside of `Manager` instance)
- `events`: Support for ORM event listeners defined on model classes

## Quick Start

First, install via pip.

```bash
pip install alchy
```

Then, create our database manager.

```python
import alchy
from sqlalchemy import orm, Column, types, ForeignKey

db = alchy.Manager(config={
    'engine': { 'url': 'sqlite://' }
})
```

Create some declarative model classes.

```python
class User(db.Model):
    __tablename__ = 'user'

    _id = Column(types.Integer(), primary_key=True)
    name = Column(types.String())
    email = Column(types.String())
    level = Column(types.Integer())

    items = orm.relationship('UserItem')

class UserItem(db.Model):
    # when no __tablename__ defined,
    # one is autogenerated using class name
    # like this:
    #__tablename__ = 'user_item'

    _id = Column(types.Integer(), primary_key=True)
    user_id = Column(types.Integer(), ForeignKey('users._id'))
    name = Column(types.String())

    user = orm.relationship('User')
```

Use the manager to create all database tables.

```python
db.create_all()
```

Now, create some records.

```python
# initialize using keyword args
user1 = User(name='Fred', email='fred@example.com')

# ...or initialize using a dict
user2 = User({'name': 'Barney'})

# update using either method as well
user2.update(email='barney@example.org')
user2.update({'email': 'barney@example.com'})

users = [user1, user2]
```

Add them to the database.

```python
# there are several options for adding records

# add and commit in one step using positional args
db.add_commit(user1, user2)

# ...or add/commit using a list
db.add_commit(users)

# ...or separating add and commit calls
db.add(user1, user2)
db.commit()

# ...or with a list
db.add(users)
db.commit()

# ...or separate adds
db.add(user1)
db.add(user2)
db.commit()
```

Fetch model and operate.

```python
user = User(name='Wilma', email='wilma@example.com')

db.add_commit(user)

# fetch from database
user_from_db = User.get(user._id)

# make changes
user_from_db.update(level=5)

# and refresh
user.refresh()

# or flush
user.flush()

# access session which loaded model instance
assert user_from_db.session is db.object_session(user_from_db)

# delete user
user.delete()
db.commit()
```

Query records from the database.

```python
# add some more users
db.add_commit(User(), User(), User(), User(), User())

# there are several syntax options for querying records

# using db.session directly
records = db.session.query(User).all()

# ...or using db directly (i.e. db.session proxy)
records = db.query(User).all()

# ...or via query property on model class
records = User.query.all()
```

Use features from the enhanced query class.

```python
q = User.query.join(UserItem)

# entities
q.entities == [User]
q.join_entities == [UserItem]
q.all_entities == [User, UserItem]

# paging
q.page(2, per_page=2) == q.limit(2).offset((2-1) * 2)

# pagination
page2 = q.paginate(2, per_page=2)
page2.query == q.limit(2).offset((2-1) * 2)
page2.page == 2
page2.per_page == 2
page2.total == User.query.count()
page2.itmes == q.limit(2).offset((2-1) * 2).all()
page2.prev_num == 1
page2.has_prev == True
page2.next_num == 3
page2.has_next == True
page_1 = page2.prev()
page_3 = page2.next()

# searching
# @note: requires simple/advanced search config on models to be defined
# (see `Query` section for details)
q.search('example.com', {'user_name': 'fred', 'item_name': 'shoes'}).all()

# entity loading
User.query.join_eager(UserItem)
User.query.joinedload(UserItem)
User.query.lazyload(UserItem)
User.query.immediateload(UserItem)
User.query.noload(UserItem)
User.query.subqueryload(UserItem)

# column loading
User.query.load_only('_id', 'name')
User.query.defer('email')
User.query.undefer('email') # if User.email undeferred in class definition
User.query.undefer_group('group1', 'group2') # if under groups defined in class

# utilities
User.query.map(lambda user: user.level)
User.query.pluck('level')
User.query.reduce(
    lambda result, user: result + 1 if user.level > 5 else result,
    initial=0
)
```

Utilize ORM events.

```python
class User(db.Model):
    __tablename__ = 'user'

    _id = Column(types.Integer(), primary_key=True)
    name = Column(types.String())
    email = Column(types.String())
    level = Column(types.Integer())

    @alchy.events.before_insert_update
    def validate(self):
        '''Validate model instance'''
        # do validation
```

Finally, clean up after ourselves.

```python
db.drop_all()
```

## ModelBase

TODO

### Instance Methods

#### __init__()

#### update()

#### to_dict()

#### flush()

#### save()

#### delete()

#### expire()

#### refresh()

#### expunge()

### Class Methods

#### get()

#### get_by()

#### simple_search()

#### advanced_search()

### Instance Properties

#### query

#### session

#### strict_update_fields

### Class/Instance Properties

#### attrs

#### descriptors

#### relationships

#### column_attrs

#### columns

### Class Configuration

#### __events___

#### query_class

#### __advanced_search__

#### __simple_search__

## Model Events

TODO

### Attribute Events

#### @set_

#### @append

#### @remove

### Mapper Events

#### @before_delete

#### @before_insert

#### @before_update

#### @before_insert_update

#### @after_delete

#### @after_insert

#### @after_update

#### @after_insert_update

#### @append_result

#### @create_instance

#### @instrument_class

#### @before_configured

#### @after_configured

#### @mapper_configured

#### @populate_instance

#### @translate_row

### Instance Events

#### @expire

#### @load

#### @refresh

## Query

TODO

### Methods

#### map()

#### reduce()

#### reduce_right()

#### pluck()

#### page()

#### paginate()

#### advanced_search()

#### simple_search()

#### search()

### Entity Load Methods

#### join_eager()

#### outerjoin_eager()

#### joinedload()

#### immediateload()

#### lazyload()

#### noload()

#### subqueryload()

### Column Load Methods

#### load_only()

#### defer()

#### undefer()

#### undefer_group()

## Manager

TODO

### Methods

#### __init__()

#### init_engine()

#### init_session()

#### create_all()

#### drop_all()

#### add()

#### add_commit()

#### delete()

#### delete_commit()

#### __getattr__

### Properties

#### metadata

#### engine

## Types

TODO

### DeclarativeEnum

