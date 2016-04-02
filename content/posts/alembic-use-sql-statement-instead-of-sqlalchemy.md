+++
date = "2016-04-03T04:35:20+08:00"
draft = true
title = "alembic use sql statement instead of sqlalchemy"

+++

I know it's pretty popular to use the wonderful `sqlalchemy` ORM
for python, especially combined with `alembic`, but sometimes
you just want to type plain SQL request.

And actually it's pretty easy, after creating your migrations
just do:

```python

"""your description

Revision ID: 0c8ab27d3408
Revises: d58bd4ed2b03
Create Date: 2016-03-18 13:13:24.228036

"""
# revision identifiers, used by Alembic.
revision = '0c8ab27d3408'
down_revision = 'd58bd4ed2b03'
branch_labels = None
depends_on = None

from alembic import op


def upgrade():
    conn = op.get_bind()
    conn.execute("DROP TABLE whatever")


def downgrade():

    conn = op.get_bind()
    conn.execute("CREATE TABLE IF NOT EXISTS whatever (id smallint)")

```
