---
title: "Auth providers"
description: "Configuring SAML and OIDC identity providers for osctrl-api, and the new Auth Providers management UI."
sidebar:
  order: 3
---

[osctrl-api](/components/osctrl-api/) supports three authentication methods at once: local password, OIDC, and SAML 2.0. This page covers the parts specific to identity providers — the base `saml:`/`oidc:` YAML fields are documented in [Configuration](/configuration/#saml-configuration).

## Environment variables reference

Every YAML key under `saml:`/`oidc:` has a matching environment variable (`SAML_*`/`OIDC_*`), used when running without `--config` (e.g. containers, or a hand-rolled invocation).

### OIDC

| Variable | Required | Description |
|----------|----------|-------------|
| `OIDC_ENABLED` | yes | Set `true` to enable the OIDC login surface |
| `OIDC_ISSUER_URL` | yes | Issuer URL (realm root); `/.well-known/openid-configuration` is appended automatically |
| `OIDC_CLIENT_ID` | yes | Client ID registered with the IdP |
| `OIDC_CLIENT_SECRET` | yes | Client secret |
| `OIDC_REDIRECT_URL` | yes | Must match the IdP's allowed callback and end with `/api/v1/auth/oidc/callback` |
| `OIDC_SCOPES` | no | Comma-separated list (default: `openid,profile,email`) |
| `OIDC_USERNAME_CLAIM` | no | id_token claim used as the osctrl username (default: `preferred_username`; see [Username rules](#username-rules)) |
| `OIDC_GROUPS_CLAIM` | no | id_token claim containing group memberships (default: `groups`) |
| `OIDC_REQUIRED_GROUPS` | no | Comma-separated group names; login is denied unless the user belongs to at least one |
| `OIDC_JIT_PROVISION` | no | Set `true` to auto-create osctrl users on first login (as non-admin) |
| `OIDC_LINK_LOCAL_ACCOUNTS` | no | Set `true` to let an OIDC login claim an existing local password account with the same username (see [Linking existing local accounts](#linking-existing-local-accounts)) |
| `OIDC_USE_PKCE` | no | Set `true` to enable PKCE (S256) for the authorization code flow |

### SAML

| Variable | Required | Description |
|----------|----------|-------------|
| `SAML_ENABLED` | yes | Set `true` to enable the SAML login surface |
| `SAML_IDP_METADATA_URL` | yes | URL to the IdP's SAML metadata XML — fetched once at startup |
| `SAML_ENTITY_ID` | yes | SP Entity ID — must match what the IdP has registered (typically the metadata URL) |
| `SAML_ACS_URL` | yes | Assertion Consumer Service URL — must end with `/api/v1/auth/saml/acs` |
| `SAML_USERNAME_ATTRIBUTE` | no | SAML attribute whose value becomes the osctrl username; empty = use NameID |
| `SAML_JIT_PROVISION` | no | Set `true` to auto-create osctrl users on first login (as non-admin) |
| `SAML_LINK_LOCAL_ACCOUNTS` | no | Set `true` to let a SAML login claim an existing local password account with the same username |
| `SAML_FORCE_AUTHN` | no | Force re-authentication at the IdP on every login (default: `true`) |
| `SAML_SIGNING_CERT` | no | Path to PEM certificate for signing AuthnRequests |
| `SAML_SIGNING_KEY` | no | Path to PEM RSA private key for signing AuthnRequests |
| `SAML_LOGOUT_URL` | no | IdP session-termination URL; returned to the frontend so it can end the IdP session on logout |

## Username rules

A username resolved from a SAML attribute or OIDC claim may take either shape:

* **Plain handle** — `^[a-zA-Z0-9_-]{1,64}$`
* **Email address** — `local@domain.tld`, up to 254 characters, stored **lowercased**

| IdP claim/attribute | Typical value | Passes? |
|---------------------|---------------|---------|
| `preferred_username` | `alice` | yes |
| `nickname` | `alice` | yes |
| `email` | `alice@example.com` | yes — stored as `alice@example.com` |
| NameID (email format) | `Alice@Example.com` | yes — stored as `alice@example.com` |
| `sub` (Auth0) | `auth0\|6a0a...` | **no** — contains `\|` |
| `sub` (Keycloak) | `a1b2c3d4-...` | yes — a 36-character UUID fits the plain shape |

An `email` claim/attribute is only accepted as a username when the IdP also asserts it's verified (`email_verified: true` for OIDC); an unverified address falls back to the provider's subject identifier instead.

Changing the username claim on a live deployment creates **new** accounts rather than renaming existing ones — the username is the identity. Migrate deliberately.

## Linking existing local accounts

A federated login whose resolved username matches an existing **local password account** is refused by default — otherwise, anyone who can make the IdP assert a given username (e.g. `admin`) would inherit that account's privileges. To let the IdP adopt accounts you pre-created locally, opt in per provider with `linkLocalAccounts: true` (or `OIDC_LINK_LOCAL_ACCOUNTS` / `SAML_LINK_LOCAL_ACCOUNTS`).

Enable it only when you trust the IdP to be authoritative over usernames. Linking never grants extra privileges — a non-admin local account stays non-admin — and once the account is stamped on first federated login, the flag can be turned back off; already-linked accounts keep working.

## Multi-factor authentication

MFA (TOTP, WebAuthn passkeys, and recovery codes) applies only to local password logins — federated logins are the identity provider's responsibility and are exempt. See [Configuration](/configuration/#service-configuration) for the `mfaRequired`/`mfaIssuer`/`mfaRPID`/`mfaOrigins` settings.

## Running OIDC and SAML simultaneously

Both protocols can be enabled at once; the login page shows a separate button for each, and they can point at the same IdP or different ones. A user originally provisioned via one protocol can later log in via the other, as long as the resolved username matches — the session's authentication method is tracked per login, not per user.

## Auth Providers management UI

The **Auth Providers** page in [osctrl-frontend](/components/osctrl-frontend/#auth-providers) — gated by `service.authProvidersEnabled` (default `true`) — lets an operator review, test, fetch IdP metadata for, and edit these SAML/OIDC settings from the UI. It's a live editor and test bench for the single active SAML config and single active OIDC config described above, not a way to run multiple concurrent SAML/OIDC providers yet.

## Per-IdP setup and troubleshooting

For step-by-step setup (Keycloak, Auth0, Okta, Microsoft Entra ID), common per-IdP gotchas (e.g. Auth0 defaulting to HS256 token signing, Okta/Entra emitting email-shaped username claims), logout/IdP-session-termination behavior, and a troubleshooting guide for failed logins, see the [full identity provider guide](https://github.com/jmpsec/osctrl/blob/master/docs/auth-providers.md) in the osctrl repository.
