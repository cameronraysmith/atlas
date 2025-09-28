# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Atlas is a language-agnostic database schema management tool written in Go. It provides two primary workflows:
- **Declarative**: Compares current database state to desired state (defined in HCL/SQL/ORM) and generates migration plans
- **Versioned**: Automatically plans schema migrations from user-defined schemas

## Development Commands

### Building
```bash
# Build the atlas CLI binary
go build -o atlas ./cmd/atlas

# Run tests
go test ./...

# Run tests for a specific package
go test ./sql/migrate/...

# Run tests with coverage
go test -cover ./...

# Run a single test
go test -run TestName ./path/to/package
```

### Linting
```bash
# Run golangci-lint (ensure it's installed)
golangci-lint run

# Run with automatic fixes
golangci-lint run --fix
```

### Common Development Tasks
```bash
# Format code
go fmt ./...

# Tidy dependencies
go mod tidy

# Update dependencies
go get -u ./...

# Generate code (if any generators are present)
go generate ./...
```

## Architecture Overview

### Core Packages

**`/sql`** - Database abstraction layer
- `/sql/schema` - Core schema types (Table, Column, Index, etc.)
- `/sql/migrate` - Migration planning and execution
- `/sql/mysql`, `/sql/postgres`, `/sql/sqlite` - Database-specific implementations

**`/schemahcl`** - HCL schema definition language
- Parsing and evaluation of Atlas HCL files
- Type specifications and resource definitions

**`/cmd/atlas`** - CLI implementation
- `/internal/cmdapi` - Command API and root commands
- `/internal/migrate` - Migration-specific commands
- `/internal/docker` - Docker driver support

**`/atlasexec`** - Programmatic Atlas client
- Go client for executing Atlas commands programmatically
- Used by integrations (Terraform, GitHub Actions, etc.)

### Key Concepts

1. **Driver Pattern**: Each database has its own driver implementing the `migrate.Driver` interface
2. **Schema Inspection**: Databases are inspected to create `schema.Realm` representations
3. **Diffing**: Changes between schemas are calculated as `schema.Changes`
4. **Planning**: Changes are converted to executable SQL statements in `migrate.Plan`
5. **HCL Integration**: Schemas can be defined in HCL and converted to/from database schemas

### Database Support
- MySQL/MariaDB
- PostgreSQL
- SQLite
- SQL Server
- TiDB
- CockroachDB
- ClickHouse
- Redshift

### Testing Strategy

Tests are organized by:
- Unit tests alongside implementation files (`*_test.go`)
- Integration tests in `/internal/integration/`
- End-to-end tests in `/atlasexec/internal/e2e/`

Run database-specific integration tests with Docker:
```bash
# Requires Docker to be running
go test ./internal/integration/postgres_test.go
go test ./internal/integration/mysql_test.go
```

## Key Files to Understand

- `cmd/atlas/main.go` - CLI entry point
- `sql/migrate/migrate.go` - Core migration interfaces
- `sql/schema/schema.go` - Core schema types
- `schemahcl/spec.go` - HCL specification types
- `atlasexec/atlas.go` - Programmatic client

## Documentation Locations

The project has limited inline documentation, with markdown files in specific locations:

- `/README.md` - Main project documentation with installation and usage examples
- `/atlasexec/README.md` - Documentation for the Go SDK/client library
- `/internal/integration/README.md` - Integration testing guide with Docker setup
- `/cmd/atlas/internal/sqlparse/sqliteparse/README.md` - SQLite parser documentation

Primary documentation is hosted externally at https://atlasgo.io

## Important Notes

- The codebase uses Go 1.24.6+
- Heavy use of interfaces for database abstraction
- Docker URLs (e.g., `docker://mysql/8`) are used for dev databases in tests
- The `atlas` command is the primary user interface
- Error handling follows Go idioms (return error as last value)