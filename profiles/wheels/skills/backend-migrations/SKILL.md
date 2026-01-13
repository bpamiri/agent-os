---
name: Backend Migrations
description: Create and manage database migrations for schema changes. Use when creating tables, adding columns, modifying indexes, or managing foreign keys. Apply when working on files in migrator/migrations/ or db/migrate/ directories. Also use when generating migration files, running database versioning commands, adding seed data, or troubleshooting migration failures. Essential for any database schema modification task.
---

# Backend Migrations

## When to use this skill:

- When creating new database tables or modifying existing schemas
- When adding, removing, or modifying columns in tables
- When creating or dropping indexes and foreign keys
- When working on files in `app/migrator/migrations/` or `db/migrate/` directories
- When running migration commands (`wheels dbmigrate latest`, `wheels dbmigrate reset`)
- When adding seed data or initial records to tables
- When troubleshooting migration failures or rollback issues
- When designing many-to-many relationships with join tables
- When adding timestamps (createdAt, updatedAt, deletedAt) to tables
- When ensuring cross-database compatibility (MySQL, PostgreSQL, H2, SQL Server)

This Skill provides Claude Code with specific guidance on how to adhere to coding standards as they relate to how it should handle backend migrations.

## Instructions

For details, refer to the information provided in this file:
[backend migrations](../../../agent-os/standards/backend/migrations.md)
