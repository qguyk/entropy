# Alembic migrations used in Entropy.

Entropy is using the open source [alembic project](https://alembic.sqlalchemy.org)
to manage structural changes to its internal databases. 

Every time a new version of Entropy requires changes to DB schema or contents, the
Entropy team creates a new alembic migration that implements these changes. 

Migrations are later applied to user databases by the users calling Entropy's 
`entropylab.pipeline.results_backend.sqlalchemy.update_db(path)` function.

*Users should always back up their database to a safe place before upgrading.*

## Entropy Developers

### To create a new empty migration
1. `cd` to `entropy/entropylab/results_backend/sqlalchemy`
2. Run `poetry run alembic revision -m "<short description of migration>"`

The new migration will be created in the `versions` directory next to this README file.

*Be sure to implement both the `upgrade()` and `downgrade()` functions in the new 
migration file and to add tests for them in `tests/test_migrations.py`*

### To autogenerate a new migration from SqlAlchemy metadata changes
If you've made changes to the Entropy's SqlAlchemy metadata, you can automatically 
generate a migration that implements your changes.

You can follow these instructions to get this done:

1. Obtain an Entropy database that does not yet have your changes applied to it.
1. Edit the `alembic.ini` file in the `sqlalchemy` directory (above this README file) 
2. Change the value of `sqlalchemy.url` to point to the DB you've obtained.
3. Run `poetry run alembic revision -m "<short description of migration>"`
4. Be sure to undo your changes in `alembic.ini`.

The new migration will be created and autogenerated in the `versions` directory next
to this README file.

Note that there can be changes to database metadata that [alembic auto-generation 
will not detect](https://alembic.sqlalchemy.org/en/latest/autogenerate.html#what-does-autogenerate-detect-and-what-does-it-not-detect
). 

### Tests
Always review auto-generated migrations and cover them with tests ensure expected 
behavior.

You will need to update a specific test called `test_ctor_ensures_latest_migration()` 
in `entropylab/results_backend/sqlalchemy/tests/test_migrations.py`:

1. Under `entropylab/results_backend/sqlalchemy/tests/db_templates` find the DB template 
file that represents the most recent version (before yours).
2. Make a new DB template file based on the above file. It should contain an empty 
database **after** your migration has been applied to it.
3. Add the file to git.
4. Update the last parameter for the `test_ctor_ensures_latest_migration()` test to be
the name of the file you created.
