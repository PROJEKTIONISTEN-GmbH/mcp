# Reporting a security problem

Write to **<support@projektionisten.de>** with `security` in the subject line.

Please do **not** open a public issue for anything that could be exploited
before it is fixed.

## What helps us act quickly

* Which endpoint — `https://ai.projektionisten.eu/mmcp` or
  `https://ai.projektionisten.eu/tmcp`.
* What you observed, and what you expected instead.
* The smallest request that shows it, and roughly when you sent it (with the
  time zone), so we can find it in our own records.
* **Never send us a credential.** Not yours, not one you found. If you believe a
  credential has leaked, say so and we will revoke it; do not paste it.

## What we do

We acknowledge a report within a few working days and tell you what we found.
If the problem affects accounts other than your own, we say so when we tell you
what we changed.

## Testing against the live endpoints

Reading is fine: connect, list the tools, ask questions. Please do not run load
tests, and please do not try to reach other accounts' data — if you think you
can, tell us instead and we will look with you.

Both servers are read-only: every tool they offer queries a source and changes
nothing. There is nothing here to delete, overwrite or buy.
