---
name: pino-patterns
description: Pino usage patterns for Node.js and Next.js - child loggers, request context, error logging
metadata:
  tags: pino, patterns, child-loggers, nextjs, nodejs, error-logging
---

## Never console.log

Always replace console methods with the appropriate Pino call:

| Instead of          | Use                    |
|---------------------|------------------------|
| `console.log()`     | `logger.info()`        |
| `console.error()`   | `logger.error()`       |
| `console.warn()`    | `logger.warn()`        |
| `console.debug()`   | `logger.debug()`       |

## Structured Logging

Log objects alongside a message - never concatenate strings:

```ts
// Bad
logger.info('User ' + userId + ' logged in from ' + ip)

// Good
logger.info({ userId, ip }, 'User logged in')
```

## Child Loggers

Use child loggers to attach persistent context to a group of related logs:

```ts
// Attach module context
const log = logger.child({ module: 'auth' })
log.info({ userId }, 'User logged in')
// Output: { module: 'auth', userId: '...', msg: 'User logged in' }
```

## Request Context in Next.js API Routes

Create a child logger per request with a unique request ID:

```ts
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { randomUUID } from 'crypto'
import logger from '@/lib/logger'

export async function GET(req: NextRequest) {
  const requestId = randomUUID()
  const log = logger.child({ requestId, path: req.nextUrl.pathname })

  log.info('Request received')

  try {
    const users = await getUsers()
    log.info({ count: users.length }, 'Users fetched')
    return NextResponse.json(users)
  } catch (err) {
    log.error({ err }, 'Failed to fetch users')
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

## Error Logging

Always pass the error object as `err` key - Pino serializes it correctly:

```ts
try {
  await doSomething()
} catch (err) {
  // Bad - loses stack trace
  logger.error('Something failed: ' + err.message)

  // Good - Pino serializes err with stack, message, type
  logger.error({ err }, 'Something failed')
}
```

## Node.js Server

```ts
// server.ts
import logger from './lib/logger'

const server = createServer()

server.listen(3000, () => {
  logger.info({ port: 3000 }, 'Server started')
})

process.on('unhandledRejection', (err) => {
  logger.error({ err }, 'Unhandled rejection')
  process.exit(1)
})
```

## Log Levels Guide

- `logger.error({ err }, 'msg')` - exceptions, failures, things that need immediate attention
- `logger.warn({ ... }, 'msg')` - recoverable issues, deprecations, unexpected but non-fatal
- `logger.info({ ... }, 'msg')` - business events (user signed up, order placed, server started)
- `logger.debug({ ... }, 'msg')` - detailed development info, not shown in production by default
