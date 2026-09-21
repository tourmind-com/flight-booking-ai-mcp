# TourMind Flight MCP Product Contract

This document defines the user-visible MCP connection, companion-Skill workflow, and minimum server capability for TourMind Flight Booking. The MCP endpoint is hosted by the existing TourMind flight service; it does not require a separate backend or duplicate the flight business logic.

## User connection

Public connection for airport lookup and the ToC update check:

```json
{
  "mcpServers": {
    "tourmind-flight": {
      "url": "https://airxapi.hlzinterface.cn/skill/flight/v1/mcp",
      "type": "streamableHttp"
    }
  }
}
```

Connection for live flight search and other protected tools:

```json
{
  "mcpServers": {
    "tourmind-flight": {
      "url": "https://airxapi.hlzinterface.cn/skill/flight/v1/mcp",
      "type": "streamableHttp",
      "headers": {
        "X-Skill-Token": "uk_or_sk_credential"
      }
    }
  }
}
```

The MCP connection contains endpoint and authentication settings only. Keep the credential secret and do not place the companion Skill version in connection headers.

## Credential channels

| Credential state | Channel | Available operations |
|---|---|---|
| No credential | Public / ToC update | `search_airports` and the public `check_skill_update` path only |
| Begins with `uk_` | Personal (ToC) | All documented tools, subject to normal workflow checks |
| Begins with `sk_` | Business (ToB) | All documented tools, subject to account permissions and normal workflow checks |
| Any other value | Invalid | No protected operation |

Personal users obtain a `uk_` credential at <https://auth.journione.ai>. Business users obtain an `sk_` credential at <https://tourmind.com/user/skill-token>; business-account registration is available at <https://tourmind.com/admin/skillSignup>.

The credential is supplied only through the MCP connection's `X-Skill-Token` header. It must not appear in tool arguments, URLs, results, logs, prompts, screenshots, commits, or issue reports. A changed credential requires the MCP client to reconnect and invalidates all pre-order quotation, verification, passenger-confirmation, and payment context held by the Agent. Existing orders remain bound to their creation channel.

## User-visible tools

| Tool | Type | User purpose | Authentication |
|---|---|---|---|
| `check_skill_update` | Read | Check whether the installed companion Skill has an update | Public for no credential/ToC; ToB when connected with `sk_` |
| `search_airports` | Read | Resolve cities and airports from a keyword | Public |
| `search_flights` | Read | Search live flight offers | Protected |
| `verify_offer` | Read | Verify the selected offer and obtain a booking session | Protected |
| `create_booking` | Destructive write | Create a booking after explicit confirmation | Protected |
| `query_order` | Read | Query an existing flight order | Protected |
| `create_payment` | Financial write | Create a third-party payment after explicit confirmation | Protected |
| `query_payment` | Read | Query third-party payment status | Protected |

The server preserves the existing request fields, response envelopes, business codes, and validation rules. A flight operation succeeds only when the returned business `code` equals `0`; HTTP success alone is not business success. Tool results expose the existing response as structured content and never expose the connection credential.

## Recommended flows

```text
Update check, only when due:
check_skill_update(current_version)

Flight discovery:
search_airports → confirm route, dates and passenger counts → search_flights

Booking:
select a fresh offer → verify_offer → complete passenger/contact review
→ explicit booking confirmation → create_booking

Payment:
query_order → choose method → complete payment review
→ explicit payment confirmation → query_order again → create_payment

Reconciliation:
query_order / query_payment
```

The companion Skill owns conversation workflow, quotation numbering, the 20-minute verification window, passenger-age and composition checks, complete booking/payment confirmation, retry limits, Stripe-fee disclosure, and recovery after ambiguous creation results. The MCP server owns input validation, authorization, current flight data, booking/payment execution, and authoritative results.

`create_payment` accepts only `order_no` and `payment_method`. The caller does not send `return_url`, amount, currency, fee, agent, or operator values; the existing service derives or supplies them.

## Authentication and permission behavior

- `search_airports` remains usable without a credential.
- `search_flights`, `verify_offer`, `create_booking`, `query_order`, `create_payment`, and `query_payment` require a valid `uk_` or `sk_` connection credential.
- `check_skill_update` uses the public ToC update behavior when no credential or a `uk_` credential is connected, and the ToB update behavior when an `sk_` credential is connected.
- Business `code == 20105` means the `sk_` credential was accepted but business flight-booking access is not enabled. The server must preserve this code. The Agent keeps the credential, permits later user-requested price searches, and stops verification/booking until access is enabled.
- After excluding `code == 20105`, an authentication rejection stops the protected workflow. The user must replace the MCP connection credential and reconnect; the Agent must not automatically replay the failed operation or switch channels.

## Skill version flow

The companion `SKILL.md` declares one version in YAML frontmatter:

```yaml
metadata:
  author: TourMind
  version: "<current-version>"
```

`metadata.version` is the single source of truth for the installed companion Skill version.

1. The Agent calls `check_skill_update(current_version)` on the first use in every new conversation.
2. It calls the tool again when an existing conversation resumes after at least 24 hours of inactivity.
3. It does not call the tool before every workflow operation.
4. No other flight tool receives a Skill version.
5. A failed update check does not block the user's flight task.
6. When an update is accepted and installed, the local `metadata.version` must equal the validated `latest_version`.
7. Updating the companion Skill never changes the MCP connection or its credential.

The update tool returns the existing top-level `skill_update` contract, including `available`, `display_to_user`, `latest_version`, and, when displayed, `message` and `release_source_url`.

## Minimum server capability

The production MCP implementation must:

- expose all eight tools listed above through Streamable HTTP at `https://airxapi.hlzinterface.cn/skill/flight/v1/mcp`;
- reuse the existing TourMind flight service, authorization, validation, business implementation, and response contracts rather than maintaining a second set of flight logic;
- keep `search_airports` public while protecting search, verification, booking, order, and payment tools;
- select the personal or business channel from the connected `uk_` or `sk_` credential and preserve the existing channel-bound order behavior;
- expose `check_skill_update` as read-only and idempotent with one required semantic-version string, `current_version`;
- preserve `code == 20105` as the dedicated business flight-permission result and keep it distinct from invalid credentials;
- mark search, verification, order query, payment query, and update check as read-only; mark booking and payment creation as non-idempotent write operations;
- reject malformed tool arguments with concrete errors and return the existing business envelope in structured content;
- never accept the connection credential as a normal tool argument or return it in any result;
- require no caller-supplied `return_url` or financial amount fields for `create_payment`;
- return concrete errors without inventing airports, offers, prices, sessions, orders, tickets, or payment state.

## Distributed Skill package

```text
skill/flight-booking-ai/
├── SKILL.md
└── references/
    ├── authentication.md
    ├── parameter_guide.md
    ├── passenger_details.md
    └── response_templates.md
```
