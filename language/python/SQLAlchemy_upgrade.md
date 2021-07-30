# SQLAlchemy Upgrade

## Alembic

### Alembic Install
pip install alembic

### Usage
- alembic init alembic
- set alembic.init sqlalchemy.url = driver://user:pass@localhost/dbname
- sqlalchemy.url = postgresql://postgres:123456@localhost:5432/testdb
- Modify env.py target_metadata = None

'''python

from model.price import Price
from model.user import User
from model.teacher import Teacher
from model import BaseModel
target_metadata = BaseModel.metadata
'''

- alembic revision --autogenerate --rev-id 0001
- alembic upgrade head
