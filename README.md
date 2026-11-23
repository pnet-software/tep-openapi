# Telco Exposure & Programmability (TEP) API

The contract of the TEP API: the OpenAPI specification that every TEP element manager
serves and every TEP client is generated from.

TEP exposes a 5G core network to the applications that run on it:

- **Subscribers**: provision them (identity, credentials and the data networks they may
  use), and read what the core currently knows of each: real-time status, the cell it is
  in, and its latest SIM and device swap.
- **Quality Profiles**: the QoS levels the core offers. Read-only.
- **Cells**: the radio cells of the network and their coverage areas.
- **AS sessions with QoS**: the 3GPP TS 29.122 AsSessionWithQoS API, for an application
  server to give a UE's traffic one of the quality profiles.

## Files

| File | What it is |
| --- | --- |
| [`tep-api.openapi.yaml`](tep-api.openapi.yaml) | The TEP API (OpenAPI 3.0.3). The contract. |
| [`TS29122_AsSessionWithQoS_Bundle.openapi.yaml`](TS29122_AsSessionWithQoS_Bundle.openapi.yaml) | 3GPP TS 29.122 V17.8.0 AsSessionWithQoS, self-contained. `tep-api.openapi.yaml` references it for the AS session operations and their models. |

Keep the two files side by side: `tep-api.openapi.yaml` references the bundle by relative path.

The bundle is the 3GPP spec with the components it uses from TS 29.122 CommonData,
TS 29.512, TS 29.514 and TS 29.571 copied in, so that tools resolve one file instead of
a chain of five. The 3GPP content is unchanged. Don't edit it by hand.

The spec is OpenAPI 3.0, not 3.1, like the 3GPP files it references: in a 3.1 document,
swagger-parser (and so openapi-generator) inlines external references, and the 3GPP
models lose their names.

## Trust model

TEP is a **trusted internal service** behind network-level access control (private
network, service mesh, mTLS at the ingress, an API gateway).

- The API has **no authentication**.
- It returns subscriber credentials (**K, OPc**) and identifiers (**IMSI, MSISDN**) in
  plaintext.

**Do not expose a TEP deployment to the public internet or to untrusted clients.** If you
need external access, put an authenticating gateway in front of it.

## Conventions

The spec's description covers these in full. In short:

- **Base URL**: every path is relative to `{apiRoot}/tep-api/v1`.
- **Paging**: list operations take `limit` (1 to 500, default 50) and `offset`, and
  answer with the page's `items` and the `total`.
- **Correlation**: every operation takes an optional `x-correlator` header and echoes it
  back, errors included.
- **Errors**: RFC 9457 problems (`application/problem+json`).
- **No phone numbers in URLs**: lookups by phone number take it in the request body, as a
  `TepDevice`.
- **Subscriber id**: the `id` in `/subscriber/{id}` is an opaque string the element
  manager assigns, stable through SIM swaps and number changes.

## Versioning

The spec follows [Semantic Versioning](https://semver.org) through `info.version`: a change
that can break clients takes a new major version.

## License

Copyright 2026 P-NET New Generation Emerging Networks & Verticals Private Company.

Licensed under the [Apache License, Version 2.0](LICENSE). You may use, modify and redistribute this work, provided you keep the copyright and license notices and include the [NOTICE](NOTICE) file in what you distribute. The 3GPP API specifications in this repository keep their own copyright; see [NOTICE](NOTICE).
