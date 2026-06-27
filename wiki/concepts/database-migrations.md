---
sources: [summaries/top_level.md]
brief: Database migrations are versioned, incremental changes to a database schema managed programmatically over time.
---

# Database Migrations

Database migrations are the practice of managing incremental, reversible changes to a database schema in a version-controlled, programmatic way. Rather than manually altering database structures, migrations provide a structured history of schema evolution that can be applied, rolled back, and shared across development teams and environments.

## Core Concepts

- **Versioned Schema Changes** — Each migration represents a discrete, ordered change to the database schema (e.g., adding a table, dropping a column, creating an index).
- **Reversibility** — Well-designed migrations include both an `upgrade` path (applying the change) and a `downgrade` path (reverting it).
- **Reproducibility** — Migration scripts can be re-run in any environment (development, staging, production) to bring a database schema to a known state.
- **Auditability** — The migration history serves as a changelog for the database schema, supporting team collaboration and debugging.

## Alembic

**Alembic** is the most widely used database migration tool in the Python ecosystem. It is tightly integrated with SQLAlchemy, the Python SQL toolkit and ORM. Key features include:

- Auto-generation of migration scripts by comparing ORM models to the current database state
- Support for branching and merging migration histories
- Environment-aware configuration (e.g., different connection strings per environment)
- Integration with popular Python web frameworks and project structures

See [[summaries/top_level]] for the source reference to Alembic.

## Relationship to Other Concepts

- [[concepts/alembic]] — The specific tool most associated with database migrations in Python projects
- [[concepts/python-project-structure]] — Migration scripts are typically organized as part of a Python project's directory structure
- [[concepts/python-environment-management]] — Migration tooling often depends on environment configuration for database connection strings
- [[concepts/database-migrations]] — This page

## Best Practices

1. **Keep migrations small and focused** — Each migration should represent a single logical change.
2. **Always include a downgrade path** — Ensures rollback is possible in production incidents.
3. **Never edit applied migrations** — Once a migration has been applied in any environment, treat it as immutable.
4. **Test migrations in CI** — Apply and roll back migrations as part of automated testing pipelines.
5. **Use descriptive migration names** — Names should convey the intent of the change (e.g., `add_users_email_index`).

## Common Use Cases

- Adding or removing columns from tables
- Creating or dropping tables and indexes
- Renaming schema objects
- Seeding reference data alongside schema changes
- Migrating data from one structure to another (data migrations)
