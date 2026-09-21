# Authentication failure and recovery

Read this reference after an MCP tool authentication rejection, but only after excluding the dedicated ToB business-permission condition. A business-channel response with `code == 20105` means the `sk_` credential was accepted but business flight-booking access is not enabled; keep the configured MCP credential and follow the permission rule in `SKILL.md`, even if its transport status or message also resembles an authentication failure. For every other underlying HTTP 401, `unauthorized`, or `invalid_token` result, authentication recovery takes precedence over transport retries and ordinary retry requests.

## Stop and replace the rejected connection credential

Stop the affected operation and do not retry it automatically. Tell the user to remove or replace the rejected secret `X-Skill-Token` header in the MCP client and reconnect. Do not ask the user to paste the credential into the conversation, and do not read or write a local token file.

Never reuse the rejected connection, send an empty or guessed credential, silently switch channels, change the MCP endpoint, or fall back to direct HTTP during the same recovery attempt. Do not search another installation, workspace, archive, backup, environment variable, shell history, previous message, or account for replacement credentials. A request to hurry, accept risk, change the itinerary, or “try once” does not authorize another call with the rejected credential.

Public airport lookup remains available through `search_airports` without a credential. A public lookup or `check_skill_update` call does not prove flight authorization and does not resume the failed protected operation.

## Obtain and configure a replacement credential

Direct a personal-channel user to <https://auth.journione.ai> to obtain a new `uk_` credential. Direct a business-channel user to <https://tourmind.com/user/skill-token> to obtain a new `sk_` credential; if they do not have a business account, provide <https://tourmind.com/admin/skillSignup>. If the rejected credential's channel is unknown, present both choices without selecting one for the user.

The user must configure the complete replacement as the MCP connection's secret `X-Skill-Token` header and reconnect. The prefix selects the channel, but configuring the credential does not prove authorization and does not by itself authorize any tool call. Never accept it as an ordinary tool argument.

Do not invent a validation tool, call a server-internal verifier, or automatically issue a protected tool call solely because the MCP client reconnected. The replacement may be used for the next protected operation only when the user explicitly requests or approves that operation and its normal input validation, quotation freshness, channel, review, and confirmation requirements are satisfied. If the user explicitly asks to test the replacement, use only a documented read-only protected tool with complete valid inputs; never use `create_booking` or `create_payment` as a credential test.

Treat the official MCP tool response as authoritative. First exclude the exact ToB `code == 20105` permission condition, which keeps the configured credential. If the response instead returns another authentication rejection, tell the user to remove or replace the rejected connection credential and restart this recovery flow. Otherwise, handle the response under its normal success, business-error, or transport-error rules without claiming more authorization than the response establishes.

## Resume the business workflow safely

Replacing the MCP connection credential invalidates pre-order quotations, verification sessions, passenger/contact confirmations, and payment context as required by `SKILL.md`. Obtain approval for a fresh search and repeat verification before any later booking. An existing order remains bound to its creation channel and requires a matching-prefix credential authorized to access it; never probe the other channel.

Never automatically replay `create_booking` or `create_payment` after authentication recovery. If an earlier creation result was ambiguous, reconcile it before considering another creation. Any later booking or payment creation must satisfy the complete current review and explicit-confirmation flow.

## Customer-facing response

Use the [authentication recovery template](response_templates.md#authentication-recovery). Select its operation outcome, authorization path, and order-state variables from actual evidence. If an existing order is known, retain its confirmed number and explain which follow-up action stopped. If an earlier creation is uncertain, retain that uncertainty. Do not show raw authentication errors or any credential fragment.
