---
name: pino-setup-and-configuration
description: Installing and configuring Pino with pino-pretty for dev and structured JSON for production
metadata:
  tags: pino, setup, configuration, pino-pretty
---

## Installation

```bash
pnpm add pino
pnpm add -D pino-pretty
```

## Singleton Logger

Always create a single logger instance and export it. Never instantiate Pino inline.

```ts
// lib/logger.ts
import pino from 'pino'

const logger = pino({
  level: process.env.LOG_LEVEL ?? (process.env.NODE_ENV === 'production' ? 'info' : 'debug'),
  transport:
    process.env.NODE_ENV !== 'production'
      ? {
          target: 'pino-pretty',
          options: {
            colorize: true,
            translateTime: 'SYS:HH:MM:ss',
            ignore: 'pid,hostname',
          },
        }
      : undefined, // structured JSON in production
})

export default logger
```

Import this everywhere:

```ts
import logger from '@/lib/logger'
```

## Environment Variables

Add to your `.env`:

```
LOG_LEVEL=debug   # override log level if needed
```

## Dev Output (pino-pretty)

In development you'll see readable, colorized output:

```
[12:34:56] INFO: Server started
    port: 3000
[12:34:57] INFO: User logged in
    userId: "abc123"
```

## Production Output (structured JSON)

In production, Pino outputs newline-delimited JSON - ready for any log aggregation service (Datadog, Logtail, Grafana Loki, etc.):

```json
{"level":30,"time":1710000000000,"msg":"User logged in","userId":"abc123"}
```

## Next.js Note

Pino runs only in server-side code (API routes, Server Actions, Route Handlers). Do not import the logger in client components - it will fail at runtime. For client-side errors, use a separate error boundary or a browser-safe logging solution.
