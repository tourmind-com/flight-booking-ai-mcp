# Authentication failure and recovery

Read this after a tool authentication rejection, excluding machine-readable `verify_offer` business `code == 20105`. That code means authorization changed: keep the configured connection credential, discard old quotations, and follow [token-change recovery](parameter_guide.md#token-change-recovery) for either channel. Other underlying HTTP 401, `unauthorized`, or `invalid_token` results invoke this recovery; never downgrade a rejected-token search to anonymous search.

## Stop and replace the rejected connection credential

Mark the active `tourmind-flight` connection rejected, stop the affected operation, and do not retry. Direct the user to remove or replace its secret `X-Skill-Token` header through the host's connection settings and reconnect. Do not claim to have cleared settings you cannot modify. Never ask the user to paste a credential into chat, pass it as an argument, or access a local credential file, environment variable, log, backup, history, or another connection's secret.

Never reuse the rejected connection, send an empty or guessed credential, silently switch channels, change the MCP server, fall back to direct HTTP, or retry anonymously during this recovery. A request to hurry, accept risk, alter the itinerary, or try once does not authorize another call through the rejected binding. Public airport lookup remains available without a credential. Tokenless search is available only when no credential is configured and no authentication-recovery stop is active; removing a rejected header alone does not authorize resumption. Public lookup and update checks do not prove authorization or resume the failed operation.

## Obtain and securely configure a replacement

Direct a personal user to <https://auth.journione.ai> for a `uk_` credential. Direct a business user to <https://tourmind.com/user/skill-token> for an `sk_` credential, or <https://tourmind.com/admin/skillSignup> if they need a business account. If the rejected channel is unknown, show both paths without choosing for the user. Use trusted nonsecret connection metadata or the user's stated channel; never inspect the secret to determine its prefix.

The user configures the complete replacement as the connection's secret `X-Skill-Token` header and reconnects. Neither its stated format nor successful configuration proves authorization or permits a call. Track only trusted nonsecret generation/reconnect evidence and the user's confirmed configuration change, as specified in `SKILL.md`; never hash, compare, or retain secret material or fragments.

Do not invent a validation tool, call an internal verifier, or issue a protected request solely because the client reconnected. Use a replacement only for the next operation the user explicitly requests or approves and only after all normal input, freshness, channel, review, and confirmation checks. If the user explicitly asks for a credential test, use only a documented read-only protected operation with complete valid inputs; never `create_booking` or `create_payment`.

Treat the official tool response as authoritative. First exclude verification 20105, which keeps the current credential and permits one bounded refresh search followed by customer reselection. Another authentication rejection marks the replacement connection rejected and restarts this flow. Otherwise apply the normal success, business-error, or transport-error rules without claiming authorization beyond what the response establishes.

## Resume the workflow safely

A changed credential invalidates pre-order quotations, verification sessions, passenger/contact confirmations, and booking confirmations. Preserve known orders and uncertain creation/payment evidence. Once recovery permits the requested operation, a known new `sk_` followed by a verification request authorizes one fresh search directly; show new results and wait for selection. Otherwise obtain approval for a fresh search. Uncertain credential continuity also requires discarding pre-order context and user approval for a new search unless verification 20105 applies. Verify a newly selected quotation before booking. Existing orders remain bound to their creation channel and require authorized credentials for that order; never probe another channel.

Never replay booking or payment creation after recovery. Reconcile ambiguous prior creation first. Later creation requires the complete current review and separate explicit confirmation.

## Customer-facing response

Use the [authentication recovery template](response_templates.md#authentication-recovery), choosing operation outcome, authorization path, and order state from evidence. Retain a confirmed order number and explain which follow-up stopped; preserve uncertainty after ambiguous creation. Do not show raw authentication errors or any credential fragment.
