---
sources: [summaries/top_level.md]
brief: Alembic is a Python database schema migration tool built on top of SQLAlchemy.
---

# Alembic

Alembic is a lightweight, open-source database migration tool for Python applications. It is closely associated with SQLAlchemy (the popular Python SQL toolkit and ORM) and provides a structured framework for managing incremental, reversible changes to a relational database schema over time.

## Overview

In production software projects, database schemas rarely stay static — tables are added, columns change, indexes are created or dropped. Alembic solves the problem of tracking and applying these changes in a controlled, version-controlled manner. Each migration is expressed as a Python script describing the "upgrade" and "downgrade" steps, allowing teams to move a database schema forward or roll it back reliably.

## Key Features

- **Migration scripts** — Each schema change is captured in a timestamped Python file with `upgrade()` and `downgrade()` functions.
- **Auto-generation** — Alembic can compare the current database state to SQLAlchemy model definitions and auto-generate migration scripts.
- **Branching and merging** — Supports parallel migration branches, useful in multi-team or multi-feature development workflows.
- **Environment configuration** — Uses an `env.py` file and `alembic.ini` for flexible connection and behavior configuration.
- **Integration with SQLAlchemy** — Leverages SQLAlchemy's engine and metadata abstractions directly.

## Relationship to Database Migrations

Alembic is a primary implementation of the [[concepts/database-migrations]] pattern in the Python ecosystem. It provides the tooling that makes incremental, auditable schema evolution practical at scale.

## Related Concepts

- [[concepts/database-migrations]] — The general pattern Alembic implements.
- [[concepts/python-project-structure]] — Alembic configuration and migration directories are typically part of a Python project's layout.
- [[concepts/python-environment-management]] — Alembic is installed and managed as a Python package dependency.
- [[concepts/yaml-configuration]] — Alembic uses INI-style configuration, often alongside broader project config patterns.

## Source

- [[summaries/top_level]] — The source document referencing Alembic.
