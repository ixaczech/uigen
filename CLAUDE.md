# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. It uses Claude AI to generate React components based on user descriptions, featuring a virtual file system, real-time preview, and component persistence for authenticated users.

## Common Commands

### Development
- `npm run dev` — Start the development server with Turbopack (localhost:3000)
- `npm run dev:daemon` — Run dev server in background, logs to logs.txt
- `npm run setup` — Install dependencies, generate Prisma client, and run migrations

### Building & Testing
- `npm run build` — Build for production
- `npm run start` — Start production server
- `npm run lint` — Run ESLint
- `npm run test` — Run Vitest (watches by default)
- `npm run test -- --run` — Run tests once and exit
- `npm run test -- src/components/chat/__tests__/ChatInterface.test.tsx` — Run single test file

### Database
- `npm run db:reset` — Reset database to initial state (destructive)
- The database schema is defined in the @prisma/schema.prisma file. Reference it anytime you need to understand the structure of data stored in the database.

## Architecture

### Virtual File System
The app uses an in-memory virtual file system (not writing to disk) to manage generated code. Key files:
- `src/lib/file-system.ts` — `VirtualFileSystem` class that stores/manages file hierarchy
- `src/lib/contexts/file-system-context.tsx` — React context for file system state and tool interactions
- `src/lib/tools/file-manager.ts` — Tool for create/delete/list operations
- `src/lib/tools/str-replace.ts` — Tool for text replacement within files

Serialization happens when sending messages to Claude via the chat API.

### Chat & AI Integration
- `src/lib/contexts/chat-context.tsx` — Manages message history and file system state using Vercel AI SDK
- `src/app/api/chat/route.ts` — Streaming endpoint that invokes Claude with tool_use, supports continuation
- `src/lib/prompts/generation.tsx` — System prompt guiding Claude on component generation (uses prompt caching via ephemeral)
- `src/lib/provider.ts` — Loads language model (Claude via Anthropic SDK or mock provider if no API key)

Claude operates with maxSteps: 40 (or 4 for mock), using two tools: `str_replace_editor` and `file_manager`.

### Authentication
- `src/lib/auth.ts` — JWT-based session management (7-day expiry) with httpOnly cookies
- `src/middleware.ts` — Validates sessions for protected routes
- `src/components/auth/` — SignUp, SignIn, and AuthDialog components
- Database: SQLite via Prisma, stores users and projects

### UI Components
- `src/components/chat/` — ChatInterface, MessageList, MessageInput, MarkdownRenderer
- `src/components/editor/` — CodeEditor (Monaco), FileTree (virtual FS browser)
- `src/components/preview/` — PreviewFrame (renders generated components in iframe via Babel standalone)
- `src/components/ui/` — Radix UI + Tailwind component library (auto-generated from shadcn)

### Data Models
`prisma/schema.prisma`:
- `User` — Email, hashed password (bcrypt)
- `Project` — Belongs to User (or null for anonymous), stores generated data as JSON

## Key Patterns

### Tool Use
Claude uses tool calls to modify the file system. Responses are streamed from `/api/chat`. Tool-defined schemas:
- `file_manager`: `create` (with content), `delete`, `list`
- `str_replace_editor`: takes oldStr (exact match), newStr, and filePath

### Component Generation Flow
1. User types prompt in ChatInterface
2. Message sent with current file system serialized
3. Claude receives system prompt + context, uses tools to create/modify files
4. File system context updates React state
5. PreviewFrame re-renders via Babel transpilation

### Testing
- Using Vitest + React Testing Library
- Tests colocated with source (e.g., `__tests__/ChatInterface.test.tsx`)
- Environment: jsdom
- Config: vitest.config.mts with tsconfig paths support

## Environment

- **Node.js**: 18+ required
- **Runtime**: Next.js 15 with App Router and Turbopack
- **Database**: SQLite (dev.db locally, managed by Prisma)
- **.env**: Set `ANTHROPIC_API_KEY` for real Claude; leave as placeholder to use mock provider
- **Root path alias**: `@/*` maps to `src/*` (use this for imports in generated code and components)

## Deployment & Persistence

- Anonymous work is tracked in localStorage (via `anon-work-tracker.ts`)
- Authenticated users' projects persist in Prisma database
- Virtual FS is recreated on each message (not persisted between sessions for anon users)

## Notes on Dependencies

- Do not run `npm audit fix` — versions are pinned for compatibility; advisory warnings are low-risk for local-only development
