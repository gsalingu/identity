## Testing Requirements (MANDATORY)

**CRITICAL**: No work is considered complete without comprehensive tests passing. This is non-negotiable.

### Testing Mandate

1. **Before any commit**: Run `make test-all` and ensure ALL tests pass
2. **Before any PR**: Run `make ci` (lint + test + build) - must pass completely
3. **Every deliverable MUST include tests** covering the new/changed functionality
4. **Every deliverable MUST include an end to end test suite** covering the new/changed functionality

### What Tests Are Required

For **backend code**:
- Unit tests for all service functions
- Integration tests for all API endpoints
- Error case coverage (validation failures, not found, unauthorized, etc.)

For **frontend code**:
- Component tests for all new components
- Hook tests for custom hooks
- User interaction tests (clicks, form submissions, etc.)

### Running Tests

```bash
# Run ALL tests (backend + frontend) - REQUIRED before commits
make test

# Run backend tests only
make test-backend

# Run frontend tests only
make test-frontend

# Run specific test file
make test-backend -- --test users_service
make test-frontend -- --grep "UsersPage"

# Watch mode for development
make test-backend-watch
make test-frontend-watch

# Full CI pipeline (lint + test + build)
make ci
```

### Test Coverage Expectations

- New services: 80%+ line coverage
- API endpoints: 100% happy path + common error paths
- Frontend components: Key user interactions tested
- Bug fixes: Include regression test that would have caught the bug

---

## Git Workflow

```bash
# Create feature branch
git checkout -b feature/module-name

# Commit format
git commit -m "feat(module): description

- Detailed change 1
- Detailed change 2"

# Push and create PR
git push -u origin feature/module-name
gh pr create --title "feat(module): description" --body "..."
```


## Project Overview
You can find the full project description in docs/project/PROJECT.md

## CRITICAL: Always Use Makefile Commands

**MANDATORY**: All CI/CD setup and operations MUST use Makefile commands:
- Any part of the CI/CD chain should be a make command
- Always use make commands to run/setup CI/CD tasks
- Create missing make commands when you discover you need one
- NEVER run raw npm, cargo, or other commands directly for CI/CD tasks

**IMPORTANT**: Use Makefile targets for all common operations. Run from the project root directory.

### Build Commands
```bash
make build       # Build the backend (cargo build)
make check       # Check compilation without building (cargo check)
make fmt         # Format code (cargo fmt)
make fmt-check   # Check formatting without modifying (cargo fmt --check)
make lint        # Run linter (cargo clippy with all targets/features)
make fix         # Auto-fix warnings (cargo fix)
make pre-commit  # Run all pre-commit checks (fmt + lint) - RUN BEFORE COMMITTING
```

### Test Commands
```bash
make test           # Run unit tests (auto-manages test DB)
make test-all       # Run all tests including integration tests
make test-e2e       # Run all E2E tests (API + UI)
make test-e2e-api   # Run backend API E2E tests only
make test-e2e-ui    # Run frontend UI E2E tests (Playwright)
```

### Frontend Commands
```bash
make frontend-install   # Install frontend dependencies
make playwright-install # Install Playwright browsers for E2E testing
make frontend-build     # Build frontend for production
make frontend-dev       # Start frontend dev server
```

### Database Commands
```bash
make db-up       # Start database containers
make db-down     # Stop database containers
make db-migrate  # Run database migrations
make db-revert   # Revert all database migrations
```

### Other Commands
```bash
make clean       # Clean build artifacts
make help        # Show all available commands
```

## Database Migrations (Manual)

**CRITICAL**: Check your current directory first with `pwd`, then adjust paths accordingly

### Project Structure
- **Root directory**: `~/git/<REPOSITORY>`
- **Backend directory**: `~/git/<REPOSITORY>/backend/`
- Note: When in root, the backend is at `./backend/`
- Note: When in backend, you're already in the right place

### Running Migrations
```bash
# Preferred (from root):
make db-migrate

# Manual (from root):
cd backend && source "$HOME/.cargo/env" && diesel migration run

# Manual (from backend/):
source "$HOME/.cargo/env" && diesel migration run
```

### Reverting Migrations
```bash
# Preferred (from root):
make db-revert

# Manual (from root):
cd backend && source "$HOME/.cargo/env" && diesel migration revert --all
```

### Creating New Migrations
```bash
# If you're in the root directory:
cd backend && source "$HOME/.cargo/env" && diesel migration generate <migration_name>

# If you're already in backend/:
source "$HOME/.cargo/env" && diesel migration generate <migration_name>
```

### Running Cargo Commands
```bash
# Preferred (from root):
make build
make check
make test

# Manual (from root):
cd backend && source "$HOME/.cargo/env" && cargo build
cd backend && source "$HOME/.cargo/env" && cargo check
cd backend && source "$HOME/.cargo/env" && cargo test
```

### Important Notes
- The `.env` file is in the root directory
- ALWAYS run `pwd` first to know where you are
- Prefer Makefile targets over manual commands
- Adjust cd commands based on your current location
- Source the Cargo environment to ensure tools are available

## CRITICAL: OpenAPI specs for frontend development
- When you're developing / updating a frontend feature leverage the openapi schema you can generate with `make generate-api` 

## CRITICAL: E2E Testing Rules

**NEVER mock the backend for internal dependencies in UI E2E tests.**
- The purpose of E2E tests is to verify real integration between frontend and backend
- E2E tests must run against the actual backend API
- Only mock external third-party services (not our own backend)
- If an E2E test fails due to backend issues, fix the backend or test infrastructure, don't mock

## CRITICAL: Project specifications

Always refer to docs/PROJECT.md when you need details about specifics of the project you're working on

## DEFAULT behavior when I ask you to "resume working on the project"

You will read the docs/PLAN.md
As soon as a milestone does not have dependencies in status "todo" in the main branch it can be worked on
You will spawn sub agents able to work in their own isolated environments if they are not already working on a task which can be worked on
When you're done working on a milestone, push and open a pull request
