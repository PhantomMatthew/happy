# AGENTS.md - Happy Coder Monorepo

Guidelines for AI coding agents working in this repository.

## Project Overview

**Happy Coder** is a mobile and web client for Claude Code and Codex with end-to-end encryption. This is a Yarn 1 workspaces monorepo.

### Packages

| Package | Description | Tech Stack |
|---------|-------------|------------|
| `happy-app` | Mobile (iOS/Android) + Web client | Expo, React Native, Unistyles |
| `happy-cli` | CLI wrapper for Claude Code/Codex | Node.js, TypeScript, Socket.IO |
| `happy-server` | Backend sync server | Fastify 5, Prisma, PostgreSQL |
| `happy-agent` | Remote agent control CLI | Node.js, TypeScript |
| `happy-wire` | Shared wire types & Zod schemas | TypeScript, Zod |

---

## Build, Lint, Test Commands

### Root Level (Monorepo)

```bash
yarn cli --help          # Run CLI from repo root
yarn web                 # Start web app
yarn release             # Release packages (maintainers)
```

### Per-Package Commands

```bash
# From package directory or use: yarn workspace <package-name> <command>

# TypeScript check
yarn build               # tsc --noEmit (typecheck only for most packages)
yarn typecheck           # Explicit typecheck

# Testing
yarn test                # Run vitest

# Development
yarn start               # Start server/app
yarn dev                 # Development mode with hot reload

# Package-specific
# happy-cli:
yarn cli                 # Run CLI directly with tsx

# happy-server:
yarn migrate             # Run Prisma migrations (DO NOT run yourself)
yarn generate            # Generate Prisma client
yarn db                  # Start local PostgreSQL in Docker
```

### Running a Single Test

```bash
# Run specific test file
yarn vitest run path/to/file.test.ts

# Run tests matching pattern
yarn vitest run -t "test name pattern"

# Watch mode
yarn vitest path/to/file.test.ts
```

---

## Code Style Guidelines

### Indentation & Formatting

- **4 spaces** for indentation (not 2 spaces, not tabs)
- Use **yarn** instead of npm for all package management
- TypeScript strict mode is enabled everywhere

### Imports

- **Always use absolute imports** with `@/` alias
- `@/*` maps to `./sources/*` (or `./src/*` in CLI)
- **ALL imports must be at the top of the file** - never import mid-code

```typescript
// ✅ Correct
import { logger } from '@/utils/log';
import { something } from '@/modules/feature';

// ❌ Wrong
import { something } from '../utils/log';  // Use @/ alias
```

### TypeScript Conventions

- **Strict typing**: No untyped code, explicit parameter and return types
- **Prefer interfaces over types**
- **Avoid enums** - use maps instead
- **Avoid classes** - prefer functional and declarative patterns
- Use descriptive variable names with auxiliary verbs: `isLoading`, `hasError`, `canEdit`

```typescript
// ✅ Prefer interfaces
interface UserConfig {
  name: string;
  enabled: boolean;
}

// ❌ Avoid enums
const Status = {
  Active: 'active',
  Inactive: 'inactive',
} as const;
```

### Naming Conventions

- **Directories**: lowercase with dashes (e.g., `components/auth-wizard`)
- **Files**: camelCase for utilities, PascalCase for components
- **Utility functions**: Name file and function the same way
- **Test files**: `.spec.ts` or `.test.ts` suffix

### Error Handling

- Graceful error handling with proper error messages
- Use `try-catch` blocks with specific error logging
- Use AbortController for cancellable operations
- Handle process lifecycle and cleanup carefully

```typescript
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  logger.error('Operation failed', { error });
  throw error;
}
```

### Async/Await

- Always use async/await over raw Promises
- Handle errors explicitly

---

## Package-Specific Guidelines

### happy-server

- **Framework**: Fastify 5 with Zod validation
- **Database**: PostgreSQL via Prisma ORM
- **Validation**: Always validate inputs using Zod
- **Routes**: Located in `/sources/apps/api/routes`
- **Transactions**: Use `inTx` wrapper for DB operations
- **Events**: Use `afterTx` to emit events after transaction commits
- **Idempotency**: Design all operations to be idempotent

```typescript
// Never create migrations yourself
// Only run: yarn generate (when new types needed)
```

### happy-app (React Native/Expo)

- **Styling**: Always use `StyleSheet.create` from `react-native-unistyles`
- **Routing**: Use expo-router API, not react-navigation directly
- **UI Components**: Use `Item` and `ItemList` from `@/components` first
- **Avatars**: Always use `Avatar` component
- **Modals**: Never use React Native `Alert` - use `@/modal` instead
- **Layout**: Apply width constraints from `@/components/layout` to ScrollViews
- **Hotkeys**: Use `useGlobalKeyboard` (web only)
- **Hooks**: Use `useHappyAction` from `@/hooks/useHappyAction.ts` for async operations

```typescript
// Styling pattern
import { StyleSheet } from 'react-native-unistyles';

const styles = StyleSheet.create((theme, runtime) => ({
  container: {
    flex: 1,
    backgroundColor: theme.colors.background,
    paddingTop: runtime.insets.top,
  },
}));
```

### happy-cli

- **Claude SDK**: Uses `@anthropic-ai/claude-code` SDK
- **Logging**: All debugging to file logs (not console) to avoid disturbing Claude sessions
- **Encryption**: End-to-end encryption using TweetNaCl
- **Session persistence**: Sessions resume across restarts

### happy-wire

- **Purpose**: Shared wire types and Zod schemas
- **Build**: Must build before dependents (`yarn workspace @slopus/happy-wire build`)
- **Changes**: Prefer additive changes for backward compatibility

---

## File Operations

- **NEVER create files unless absolutely necessary**
- **ALWAYS prefer editing existing files** over creating new ones
- **NEVER proactively create documentation files** (*.md, README) unless explicitly requested

---

## Testing

- **Framework**: Vitest
- **Test location**: Colocated with source files (`.test.ts` or `.spec.ts`)
- **Utility functions**: Write tests BEFORE implementation
- **happy-cli tests**: No mocking - tests make real API calls

```bash
# Run all tests
yarn test

# Run single test file
yarn vitest run path/to/file.test.ts

# Watch mode
yarn vitest path/to/file.test.ts
```

---

## Environment Variables

### happy-cli
- `HAPPY_SERVER_URL` - Custom server URL
- `HAPPY_HOME_DIR` - Custom home directory for Happy data

### happy-server
- `DATABASE_URL` - PostgreSQL connection URL
- `REDIS_URL` - Redis connection URL
- `HANDY_MASTER_SECRET` - Master secret for auth/encryption

---

## Security & Privacy

- All communications use end-to-end encryption (TweetNaCl/libsodium)
- Private keys stored with restricted permissions
- Challenge-response authentication prevents replay attacks
- Never log sensitive data

---

## Quick Reference

```bash
# Type checking
yarn typecheck

# Run tests
yarn test
yarn vitest run path/to/file.test.ts

# Development
yarn dev                  # Start in dev mode
yarn start                # Start server/app

# Build
yarn build                # TypeScript check + build if applicable

# Monorepo
yarn workspace <pkg> <cmd>  # Run command in specific package
```
