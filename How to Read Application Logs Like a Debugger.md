# How to Read Application Logs Like a Debugger

A practical, step-by-step method for reading logs and stack traces to find the real problem quickly, instead of getting lost in noise.

## Find the Actual Failure Point First

Every log has roughly 90% "here's what's happening" INFO lines and maybe one real problem. Skip straight to the signals that matter:

- Lines with `"level":"ERROR"` or `"level":"WARN"`
- Keywords like `FAILED`, `Exception`, `Unauthorized`, `Forbidden`, `refused`, `timeout`

Don't read top to bottom — search or scan for these markers first.

## Read the Message Before the Stack Trace

The message field (or the first line before the stack trace) almost always tells you exactly what broke. For example:

```
401 Unauthorized from POST https://api.groq.com/openai/v1/chat/completions
```

This alone tells you which call, which service, and what HTTP status occurred. For this class of error, you often don't need the stack trace at all — the message is the diagnosis.

## Know What Common HTTP Status Codes Mean

| Code | Meaning | Usual Cause |
| :-- | :-- | :-- |
| 400 Bad Request | Your request was malformed | Wrong field name, bad JSON shape, invalid parameter value |
| 401 Unauthorized | No/invalid credentials | Missing or wrong API key/token |
| 403 Forbidden | Authenticated but not allowed (or blocked) | Wrong permissions, bot-blocking, rate limiting |
| 404 Not Found | Wrong URL/endpoint | Typo in path, wrong port |
| 5xx | Server-side failure | Not usually your bug — check if the service itself is up |

> **Key distinction:** 401 vs 403 is a common point of confusion. 401 means "who are you?" 403 means "I know who you are, still no."

## In a Stack Trace, Find Your Code, Not the Framework's

Most of a stack trace is framework internals (`io.netty...`, `reactor.core...`, `org.springframework...`) — that's just the call path, not the bug. Scan for the first line containing your own package, for example `com.heyhi.*`:

```
at com.heyhi.retrieval.internal.SearxngSearchProvider.search(SearxngSearchProvider.java:30)
```

That's the line in your code that triggered the issue — go look at that file and line directly.

## "Caused by:" Is the Real Root Cause

Java chains exceptions together. The outermost exception is often just a wrapper (e.g., `CompletionException`), while the real cause is nested below:

```
CompletionException
Caused by: HttpClientErrorException$Forbidden: 403 Forbidden
```

> **Rule:** Always scroll to the deepest `Caused by:` — that's the original problem. Everything above it is just "this is how it propagated up."

## Spring Context Startup Failures

If the app fails to start (rather than a runtime request error), look for a pattern like:

```
Error creating bean with name 'X': ... Failed to initialize dependency 'Y' ... Error creating bean with name 'Y'
```

This chains like "Y depends on Z depends on W." Walk down until you hit a `Caused by:` that isn't another `BeanCreationException` — that's the actual root cause (e.g., a DB connection error or a missing config value).

## A Quick Mental Checklist

When something fails, work through these five questions in order:

1. What's the exception type / HTTP status? → Tells you the category of problem.
2. What's the message? → Often tells you exactly what's wrong.
3. Which of my own files/lines are in the trace? → Tells you where to look.
4. What's the deepest `Caused by:`? → Tells you the root cause.
5. Is this config/credentials, code logic, or an external service being down? → Tells you where to fix it.

> **Tip:** When debugging a new raw log, try walking through these five questions yourself first and form a guess before jumping to the answer — this builds the diagnostic instinct faster than just reading the fix.

