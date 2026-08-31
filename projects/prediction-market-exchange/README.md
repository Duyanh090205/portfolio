# Prediction-Market Exchange

A working exchange, built from the matching engine up. Not a backtest of one, not a
simulation — a central limit order book with price–time priority, a margin engine, and
atomic settlement, running behind a real-time web front end.

**[Repository](https://github.com/Duyanh090205/prediction-market-exchange)**

> **Authorship.** I designed and built the matching engine, margin engine and settlement
> layer — 88% of the codebase by line count. Deployment configuration and the SSO
> integration were contributed by teammates.

<!-- VISUAL SLOT 2 - short clip or screenshot of the order book matching a trade.
     This is the single highest-value visual in the whole portfolio: it shows the
     exchange is real. Produce by running the stack locally per the repo setup. -->

---

## What it does

Players trade binary spread contracts: OVER or UNDER a strike price, with fixed ±size
payouts. An administrator opens a contract on a question with one real numerical answer,
market makers post two-sided quotes, players hit them or sweep the book, and the contract
settles when the real answer is entered.

The design constraint that shaped everything: **orders execute instantly and atomically,
with no manual confirmation step anywhere in the path.** That is what makes it an
exchange rather than a message board with a ledger attached, and it is also what makes
the concurrency and margin problems real.

## The three things worth reading the code for

### 1. Matching engine

Price–time priority over a live order book. LIMIT orders hit a specific resting quote by
ID; MARKET orders sweep multiple price levels with slippage protection via a limit price.
Multi-level sweeps are atomic — a market order that crosses four quotes either fills
against all four or none.

Concurrent fills are the hard part. Two players hitting the same quote simultaneously
must not both fill against it. The engine takes `SELECT FOR UPDATE` row locks inside the
transaction, so the second order sees the quote already consumed rather than a stale
snapshot of it.

### 2. Margin engine, checked twice

Margin is reserved against **worst-case loss** on the position, not against notional and
not against the current mark.

The part worth talking about is that the check runs **twice**: once when the order is
submitted, and again inside the execution transaction. The first check is the fast
rejection path. The second exists because the first one is stale by the time it matters —
between submission and execution, the same user's other orders may have filled and
consumed the balance the first check approved against. Only the check that runs inside
the transaction, against locked rows, is binding.

### 3. Atomic settlement

When a contract settles, every position closes, every balance moves, and realized P&L
lands on the leaderboard in a single database transaction. There is no intermediate state
where some users have been paid and others have not. Settlement is the one operation
where partial failure cannot be fixed by retrying, so it gets the strictest transactional
guarantee in the system.

## Correctness

**103 unit tests across 13 files**, concentrated on matching, margin and P&L — the three
places where a bug silently moves money instead of throwing an error.

Supporting hardening: UUIDv7 idempotency keys on every state-changing operation, CSRF
protection, per-IP login rate limiting, bcrypt at factor 12, structured logging, and an
API-layer rule blocking the administrator from trading at all.

## Stack

Roughly 15.6k lines. Next.js 15 (App Router), PostgreSQL, Prisma, Socket.IO over a custom
Node server for real-time book and trade updates, Tailwind.

Eleven database tables. Migrations are additive-only by policy — no `DROP TABLE` or
`DROP COLUMN` in new migrations without an explicit maintenance window, because the
database holds settled trade history that cannot be reconstructed.

---

## Deployment status, stated plainly

The original hosted deployment has been retired and its domain released. I no longer have
access to that hosting account, so I cannot guarantee the service stays up, and a link
that intermittently fails is worse than no link at all.

There is also a real bug in the retired deployment worth naming, because diagnosing it is
more interesting than hiding it: the login flow built its return URL from the **runtime
bind address** rather than from a configured public URL. In production that resolved to
`0.0.0.0:3000`, so even a successful login redirected the user to a host that does not
exist. The fix is a public-URL environment variable. The lesson is that anything deriving
a user-visible URL from the socket it happens to be listening on will work perfectly in
local development and fail only in production.

Until a redeployment on infrastructure I control, the repository and its test suite are
the artifact, and the setup instructions in the repo bring the full stack up locally.
