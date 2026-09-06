# Flask-SQLAlchemy Serialization Lab

## Description

A simplified e-commerce backend built with Flask, SQLAlchemy, and Marshmallow.
It models customers, the items they purchase, and the reviews they leave for
those items, and it serializes that relational data into JSON-friendly
dictionaries for use by a front end or external API consumer.

**Domain model**

- A `Customer` has many `Review`s.
- An `Item` has many `Review`s.
- A `Review` belongs to a `Customer` and belongs to an `Item` — it is the join
  between the two.
- A `Customer` has many `Item`s *through* `Review`s, exposed via an
  `association_proxy` named `items`.

![customer review item erd](/assets/sqlalchemy_lab_2_erd.png)

## Tech Stack

- Flask / Flask-SQLAlchemy / Flask-Migrate
- Marshmallow (for schema-based serialization)
- SQLite
- pytest

## Setup

Fork and clone the repo, then from the project root:

```console
$ pipenv install
$ pipenv shell
$ cd server
```

Create the database tables:

```console
$ flask db init
$ flask db migrate -m "initial migration"
$ flask db upgrade head
```

Seed the database with sample customers, items, and reviews:

```console
$ python seed.py
```

## Usage

### Inspecting relationships

From `server/`, open a Flask shell:

```console
$ flask shell
>>> from models import *
>>> customer1 = Customer.query.filter_by(id=1).first()
>>> customer1.reviews
[<Review 1, ...>, <Review 3, ...>]
>>> customer1.items          # association proxy: items via reviews
[<Item 1, Laptop Backpack, 49.99>, <Item 2, Insulated Coffee Mug, 9.99>]
```

### Serializing models

Each model has a corresponding Marshmallow schema (`CustomerSchema`,
`ItemSchema`, `ReviewSchema`). Nested relationships are included with
`fields.Nested`, and the reciprocal side of each relationship is excluded
from the nested output to avoid infinite recursion:

```console
>>> import json
>>> json.dumps(CustomerSchema().dump(customer1))
```

```json
{
  "id": 1,
  "name": "Tal Yuri",
  "reviews": [
    {
      "id": 1,
      "comment": "zipper broke the first week",
      "item": { "id": 1, "name": "Laptop Backpack", "price": 49.99 }
    }
  ]
}
```

## Testing

Run the full test suite from the project root:

```console
$ pytest
```

![passing test run](/assets/test_run_screenshot.png)

## Project Structure

```
server/
├── app.py                 # Flask app + Flask-Migrate setup
├── models.py               # Customer, Item, Review models + Marshmallow schemas
├── seed.py                 # Sample data
├── migrations/              # Alembic migration history
└── testing/                 # pytest suite (models, association proxy, serialization)
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

See [LICENSE.md](LICENSE.md).
