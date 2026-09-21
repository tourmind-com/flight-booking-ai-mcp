# TourMind Flight Booking MCP Product Package

This repository contains the MCP connection metadata, product contract, and user-installable companion Skill for TourMind Flight Booking. The remote MCP endpoint is part of the existing TourMind flight service; this repository does not contain a second server implementation or deployment.

## Package contents

```text
flight-booking-ai-mcp/
├── .github/workflows/publish-mcp.yml
├── README.md
├── TourMind MCP-FORMAT.md
├── server.json
└── skill/
    └── flight-booking-ai/
        ├── SKILL.md
        └── references/
            ├── authentication.md
            ├── parameter_guide.md
            ├── passenger_details.md
            └── response_templates.md
```

The companion Skill preserves the current TourMind flight workflow, including airport resolution, passenger validation, live search, offer verification, booking confirmation, order lookup, payment confirmation, and failure recovery. It changes only the transport: the Agent calls MCP tools instead of the HTTP endpoints directly.

## MCP connection

Production endpoint:

```text
https://airxapi.hlzinterface.cn/skill/flight/v1/mcp
```

The connection may be created without a credential for public airport lookup and the public ToC update check. Flight search, verification, booking, order, and payment tools require an `X-Skill-Token` connection header containing either a personal `uk_` credential or a business `sk_` credential.

- Personal users obtain a `uk_` credential by signing in at <https://auth.journione.ai>.
- Business users obtain an `sk_` credential at <https://tourmind.com/user/skill-token>. A business account can be requested at <https://tourmind.com/admin/skillSignup>.

Configure the credential as a secret MCP connection header and reconnect. Do not place it in tool arguments, prompts, logs, screenshots, commits, or issue reports.

## Tools

| Tool | Purpose | Credential |
|---|---|---|
| `check_skill_update` | Check the companion Skill version | Public for no credential/ToC; ToB when connected with `sk_` |
| `search_airports` | Resolve cities and airports | Public |
| `search_flights` | Search live flight offers | Required |
| `verify_offer` | Verify a selected offer and obtain a booking session | Required |
| `create_booking` | Create a booking after explicit confirmation | Required |
| `query_order` | Query an existing flight order | Required |
| `create_payment` | Create a third-party payment after explicit confirmation | Required |
| `query_payment` | Query third-party payment status | Required |

See [TourMind MCP-FORMAT.md](TourMind%20MCP-FORMAT.md) for the complete product contract and server requirements.

## Versioning and publishing

`server.json` contains the MCP Registry package version. The companion Skill declares its installed version in `metadata.version`; the two values are released together.

The included GitHub Actions workflow validates and publishes `server.json` to the MCP Registry when a `v*` tag is pushed or the workflow is started manually. Because the server name uses the reverse-DNS namespace `cn.hlzinterface.airxapi`, configure the repository secret `MCP_PRIVATE_KEY` with the Ed25519 private key matching the MCP Registry DNS verification record for `airxapi.hlzinterface.cn` before publishing.

## Related Skill repository

The direct-HTTP hotel and flight Skills are distributed from [TourMind Booking Skills](https://github.com/tourmind-com/Tourmind-Booking-Skills). This MCP package keeps the flight workflow aligned with that source while using the connected MCP server for all TourMind operations.
