# Backlog

Priorities use the standard `/dev-cycle` colors:

- 🔴 Critical — broken flows, crashes, or work that blocks all other work
- 🟡 High — important quality bar gaps, prominent UX friction
- 🟢 Medium — polish, smaller bugs, helpful refinements
- 🔵 Future / Maybe — things we've explicitly deferred or might never do

Strike items with `~~text~~` when struck during validation. Check `[x]` when done.

---

## 🔴 Critical

- [ ] **Land the chat sidebar on `development`.** `feat/ai-chat-sidebar` is ~22 commits ahead of `development`. Until it lands, no other work on the editor should happen there — it'll just create merge pain. Decide: fast-forward merge into `development`, or open a PR for review. Verify schema commit on `development` (`8b99707`) is the same one used on the feature branch before merging.
- [ ] **Verify chat sidebar end-to-end in the browser after merge.** PHPUnit + Playwright pass on the feature branch, but no human has walked Compose / Edit / Refine through the new chat UX. Run all three core journeys from `docs/PRODUCT_SENSE.md` against a live wp-env.
- [ ] **Remove the broken `npm run env:stop`.** `package.json` still exposes `env:stop`, which is documented in README as buggy. Either replace the script with the working `docker compose down` form, or remove the script entirely so people don't reach for it.
- [ ] **`OptionsStoreTest` leaks the live API key into stdout** when run with `ANTHROPIC_API_KEY` defined (via `.wp-env.override.json` or any env constant). Happened twice during the chat-sidebar implementation session. Fix is a one-line `$this->markTestSkipped(...)` guard in `setUp` when `defined('ANTHROPIC_API_KEY')`. Worth doing soon — bites anyone who runs `composer test` locally with a key configured.

## 🟡 High

- [ ] **README is out of date.** README still describes the modal-driven Compose / Edit / Refine flows, REST routes (`/v1/compose`, `/v1/edit`, `/v1/refine`, `/v1/jobs/{id}`), and Action Scheduler architecture — all of which were demolished on `feat/ai-chat-sidebar`. After the merge, rewrite README around the chat sidebar + `/chat/*` routes.
- [ ] **`docs/prompts.md` is out of date.** Names `Jobs/ComposeJob::systemBlock()` which no longer exists. The system prompt now lives in `Chat\PromptBuilder`. Refresh the doc.
- [ ] **`docs/privacy.md` doesn't mention chat conversation storage.** The new `chat_conversations` + `chat_messages` tables persist user prompts and assistant replies in the host site's database. Add a clause that explains this and how to clear history.
- [ ] **Loading / error / empty states audit on the chat sidebar.** Walk every state of `ChatSidebar.tsx`: no API key set, mock mode active, rate-limit hit, network failure mid-turn, empty conversation, very long conversation. Each needs the standard set per `docs/STANDARDS.md`.
- [ ] **Streaming-check admin notice — verify wording.** `Activation\StreamingCheck` shows a notice when `fastcgi_finish_request` is missing. Does the copy tell the admin what to do (e.g. "Ask your host to enable PHP-FPM" vs. a fact-only statement)?

## 🟢 Medium

- [ ] **CHANGELOG.md.** We don't have one. We're about to ship a meaningful 0.2.0 (chat sidebar). Start tracking releases — even a flat list of dates + bullet points beats nothing.
- [ ] **Auto-open sidebar discoverability.** `0b8ce07` auto-opens the chat sidebar on first activation. After the user dismisses it once, does it stay dismissed across reloads? Confirm with an E2E run.
- [ ] **Quick-action coverage for all theme block types.** Spec covers paragraph, heading, list, image, generic fallback. Audit `wp-starter-theme`'s registered blocks and confirm there's a reasonable quick-action set for each, or a graceful "Improve / Rewrite" fallback.
- [ ] **Settings page polish — link out to the privacy doc.** The settings screen should link to `docs/privacy.md` (or the hosted version) so admins can read it without leaving the dashboard.
- [ ] **WP-CLI: dump-schema currently exists. Add `wp starter-ai clear-chat-history` for testing.** Useful in dev to reset the conversation store without DROP TABLE.
- [ ] **Rate limit feedback in chat UI.** When rate-limited, the chat should display a clear "You've used 30/30 composes this hour. Resets in 12 min." not a generic error.
- [ ] **Chat-sidebar code-review carryovers.** Minor items deferred during `feat/ai-chat-sidebar` implementation: (1) `src/Chat/VirtualTree.php:179` has a redundant `$entry['attributes'] = $node['attributes']` re-assignment — defensive but obscure, safe to drop. (2) `tests/phpunit/Chat/ConversationStoreTest.php` assigns `$id = $this->store->appendUserMessage(...)` and `startAssistantTurn(...)` but never asserts on the return value — either `assertGreaterThan(0, $id)` or drop the assignment. (3) Class-level docblock on `ConversationStore` should note the silent-`$wpdb`-failure policy (matches existing codebase, but undocumented).

## 🔵 Future / Maybe

- [ ] Multiple conversations per post + conversation list UI (rejected for v1, design spec §"Non-goals").
- [ ] Conversation summarization for long threads.
- [ ] Image attachments / multimodal input.
- [ ] Diff preview before applying block changes (rejected during brainstorming).
- [ ] Cross-post context ("see how I wrote that other article").
- [ ] Streaming tool-call previews ("about to insert a heading…" before the turn completes).
- [ ] Voice input.
- [ ] Per-user system-prompt customization.
- [ ] Multi-provider support (OpenAI, Gemini). Single-provider on purpose for now.
- [ ] Public WordPress.org distribution.
- [ ] Sentry / error telemetry integration so we know when client sites hit production errors.
- [ ] Per-flow `starter_ai_system_prompt` filter (mentioned as v0.2 plan in `docs/prompts.md`, never implemented).
