# Examples

Concrete before/after for each Vibe Engineer principle. The same scenario shown two ways: how an undisciplined Vibe Engineer would prompt it, versus how a disciplined one would.

---

## 1. Hypothesis before help

### Example 1.1 — A failing test

**Scenario**: `npm test` fails on `OrderService.test.ts` with a cryptic error.

**Undisciplined**:

> "the order test is failing, fix it"

The AI looks at the error, guesses at the most plausible failure mode, makes a change that compiles and passes a test, and ships you a green build that may or may not be correct.

**Disciplined**:

> "OrderService.test.ts is failing with `Cannot read properties of undefined (reading 'id')` at line 47. The setup mocks the OrderRepository; line 47 is `expect(orders[0].id).toBe('order-1')`. My hypothesis: the mock isn't returning the test fixture — either the mock isn't being applied, or the fixture has the wrong shape. Diagnose, don't fix yet."

The AI now has a specific hypothesis to test. It traces the mock setup, identifies that the spy was set on the wrong instance, and reports the root cause. You then approve the fix.

**Why the second is better**: the disciplined prompt put a hypothesis in front of the AI. The AI's job became validation, not invention.

### Example 1.2 — A flaky CI run

**Undisciplined**:

> "the CI is flaky, the integration tests randomly fail"

**Disciplined**:

> "Integration tests `payment.spec.ts` fail ~1 in 5 CI runs. The failure is always on `test('refunds settle within 10s')` and the error is timeout. My hypothesis: the test is racing the webhook delivery — the assertion fires before the server has processed the inbound webhook. Confirm by running with verbose logs, or propose an instrumentation that proves/disproves the race."

The second version gives the AI a specific hypothesis to confirm or refute, plus the next step. The first version invites a guess that might address something else (network instability, dependency version, etc.).

---

## 2. Scoped prompts

### Example 2.1 — A refactor

**Undisciplined**:

> "clean up the auth module, it's getting messy"

The AI looks at the auth module, picks the messiest-looking parts by its own judgment, and proposes large changes that may not match your priorities.

**Disciplined**:

> "In `src/auth/session.ts`, `Session.acquire()` (lines 42-89) has nested try/catch with 4 levels of fallback. The fallback for `RefreshTokenExpired` (line 73-78) is the one that's confusing on PR review — multiple callers say it's hard to reason about. Refactor just that fallback path without changing the surface of `Session.acquire()`."

The disciplined prompt names the file, the function, the line range, the specific concern, the constraint (don't change surface), and the success criterion (callers find it easier to reason about).

### Example 2.2 — A bug report

**Undisciplined**:

> "users say the upload is broken"

**Disciplined**:

> "User report: upload to S3 fails with `PermanentRedirect` for files >5MB. Repro: upload file 6.2MB via `POST /api/uploads` with `Content-Type: multipart/form-data`. Backend logs show the multipart parser receiving the file correctly; the S3 `putObject` call returns 301 with `Location: <us-east-1 url>`. The bucket is in `us-west-2`. My hypothesis: the S3 client is configured for the wrong region, or some code path bypasses the region config. Diagnose."

The first prompt invites a guess. The second gives the AI the diagnostic chain — error type, repro, evidence already gathered, hypothesis, ask.

---

## 3. Validate before accepting

### Example 3.1 — Over-engineered fix

**Bug**: A typo in a config file causes `getServiceUrl()` to return `undefined`.

**AI's proposed diff**:

```diff
+ import { ConfigError } from './errors';
+ import { ConfigValidator } from './validators';
+
  function getServiceUrl(): string {
-   return config.serviceUrl;
+   const validator = new ConfigValidator(config);
+   const result = validator.validate('serviceUrl', { required: true, type: 'string', format: 'url' });
+   if (result.errors.length > 0) {
+     throw new ConfigError(`Invalid serviceUrl: ${result.errors.join(', ')}`);
+   }
+   return result.value;
  }
```

**Validation questions**:

1. **Does it over-engineer?** Yes. The user reported a typo. The fix introduces a ConfigValidator class, a ConfigError type, and validation infrastructure none of which existed before.
2. **Does it miss the actual bug?** Yes. The typo is in the config file; this diff doesn't fix the config file, it adds a validator that will now throw because the typo is still there.
3. **Does it match the root cause?** No. Root cause is a typo. The fix should be: correct the typo.

**Reject and iterate**: "The bug is a typo in `config.json` line 14 (`serviceUrll` should be `serviceUrl`). Fix the typo. No new validation infrastructure."

### Example 3.2 — Missing edge case

**Task**: Add an empty-state to the dashboard.

**AI's proposed diff**: adds a check `if (orders.length === 0)`.

**Validation questions**:

1. **Does it over-engineer?** No, it's a clean conditional.
2. **Does it miss an edge case?** Yes. `orders` could be `undefined` if the fetch failed; the proposed code throws on `undefined.length`.
3. **Does it match the root cause?** Sort of, but incompletely.

**Iterate**: "Add the same empty-state branch for `orders === undefined` (fetch failed) and `orders === null` (cold state). Three states: undefined → 'loading or error', empty array → 'no orders yet', populated array → render."

---

## 4. Reject working-but-wrong

### Example 4.1 — Try/catch that swallows

**Bug**: A function intermittently throws `TypeError: Cannot read 'name' of undefined`.

**AI's proposed fix**:

```diff
- function formatUser(user) {
-   return `${user.name} (${user.id})`;
- }
+ function formatUser(user) {
+   try {
+     return `${user.name} (${user.id})`;
+   } catch {
+     return 'Unknown user';
+   }
+ }
```

**Test result**: Tests pass.

**Why this is wrong**: The fix doesn't fix the bug; it hides it. Now when `user` is `undefined`, the UI shows "Unknown user" without telling anyone why. The root cause (some caller passing `undefined`) is never addressed, and the bug propagates silently to every other place that uses the same caller.

**Reject**: "Don't try/catch around the symptom. Find the caller that passes undefined. The repro is the failing trace — start there. We want to know why `user` is undefined, not to paper over it."

### Example 4.2 — Defensive masking

**Bug**: A database query returns null occasionally when it shouldn't.

**AI's proposed fix**:

```diff
- const order = await db.findOrder(orderId);
+ const order = await db.findOrder(orderId) ?? { id: orderId, status: 'unknown' };
```

**Tests pass.**

**Why this is wrong**: `findOrder` returning null is a contract violation. The fix invents a fake order object that downstream code treats as real. The downstream code now operates on synthetic data and produces wrong results that look right.

**Reject**: "The fix masks a contract violation. `findOrder` is contracted to return Order or throw NotFoundError. If it's returning null silently, fix `findOrder`. If the order can legitimately not exist, the caller needs explicit handling for that case — not a synthetic placeholder."

### Example 4.3 — Renaming the symptom

**Bug**: A variable `userIsLoggedIn` is true when the user is actually logged out.

**AI's proposed fix**: Rename the variable to `userSessionState`, update all references.

**Why this is wrong**: The variable holds wrong state. Renaming doesn't change the state. The bug is in whatever sets `userIsLoggedIn`; the fix should trace the assignment and correct it.

**Reject**: "The rename doesn't fix anything. The bug is that the variable holds wrong state. Find the code that sets it (probably in auth event handlers), and fix the state logic."

---

## 5. AI output is a draft, not a verdict

### Example 5.1 — A confident diff that's actually wrong

The AI proposes a "clean" refactor with a 50-line diff. Tests pass.

**Undisciplined**: Stage all, commit.

**Disciplined**:

1. Read the full diff.
2. Note: the refactor moved `validateOrder` from `OrderService` to a new `OrderValidator` class. Old callers updated to use the new class.
3. Check: are there callers in tests, integration scripts, or other modules that weren't in the diff?

```bash
grep -r 'OrderService\.validateOrder' --include='*.ts'
grep -r 'validateOrder' tests/
```

4. Result: integration tests in `e2e/orders.spec.ts` still call `OrderService.validateOrder`. The AI's diff missed them.

5. Confidence corrected. Either: roll back, or extend the diff to update the missed callers.

The AI didn't lie; it didn't have those files in context. Verifying caught the gap.

### Example 5.2 — Reading what it did, not just whether it succeeded

You ask the AI to add a migration that adds a column.

**Undisciplined**: Migration runs successfully → assumed correct.

**Disciplined**:

```sql
-- Inspect the actual migration the AI wrote
cat migrations/2026_05_24_add_orders_priority.sql
```

```sql
ALTER TABLE orders ADD COLUMN priority INT DEFAULT 1;
UPDATE orders SET priority = 1 WHERE priority IS NULL;
```

The `UPDATE` is redundant — `DEFAULT 1` already populates new rows; existing rows don't need backfill because they wouldn't have NULL priority before the column existed. The migration ran fine, but the AI added a no-op UPDATE that locks the table.

You spotted it because you read the migration, not just whether it succeeded. For a 50M-row table that no-op UPDATE is a real production incident.

---

## A note on cost

Every principle above costs you time. Hypothesis-first is slower than ask-and-go. Scoped prompts take longer to write than vague ones. Validation takes longer than accepting. Rejecting working-but-wrong sometimes feels precious when tests pass.

The cost compounds the other way too: the bug you didn't catch becomes a production incident. The over-engineered abstraction becomes the thing the next developer has to undo. The working-but-wrong fix becomes the bug pattern propagated through the codebase.

The Vibe Engineer's discipline is choosing the longer up-front cost because the downstream cost of skipping it is higher.

## Further reading

- [`andrej-karpathy-skills`](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the companion principles for Claude's side of the discipline
- [`hitchhikers-guide-to-vibe-engineering`](https://github.com/HermeticOrmus/hitchhikers-guide-to-vibe-engineering) — the long-form book, with chapters on practice, pitfalls, and case studies
- Karpathy on LLM coding pitfalls: https://x.com/karpathy/status/2015883857489522876
- Google's Code Comprehension Interview: https://velocode.ai/blog/google-code-comprehension-interview
