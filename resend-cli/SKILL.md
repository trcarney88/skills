---
name: resend-cli
description: Use the Resend CLI for safe read-only email operations while refusing and blocking all email sending workflows.
compatibility: opencode
---

# Resend CLI Skill

Use this skill when working with the `resend` CLI. The CLI is useful for inspecting Resend account state, domains, logs, webhooks, templates, contacts, and email metadata.

## Non-Negotiable Safety Rule

Never send, schedule, batch-send, broadcast-send, or trigger email delivery through Resend.

You must refuse any request that would run or assist with a command that sends or triggers email delivery, including but not limited to:

- `resend emails send`
- `resend emails batch`
- `resend broadcasts send`
- `resend broadcasts create --send`
- `resend events send`

Do not work around this restriction by using `curl`, SDKs, direct HTTP calls, API clients, custom scripts, automations, or any other tool to trigger email delivery.

When refusing, be brief and offer a safe alternative, such as drafting email content, reviewing templates, checking domains, inspecting logs, or validating webhook configuration.

## Safe CLI Defaults

- Prefer `--json` for commands used by agents or scripts.
- Use read-only commands whenever possible.
- Do not print, log, or expose full API keys.
- Do not store API keys in project files, command files, logs, shell history, or generated documentation.
- Prefer existing authentication from `RESEND_API_KEY` or saved credentials.
- Use `resend doctor --json` to diagnose CLI/auth/domain status without changing resources.

## Allowed Inspection Commands

These command families are generally safe for read-only inspection:

```bash
resend whoami --json
resend doctor --json
resend domains list --json
resend domains get <id> --json
resend emails list --json
resend emails get <id> --json
resend emails receiving list --json
resend emails receiving get <id> --json
resend logs list --json
resend logs get <id> --json
resend webhooks list --json
resend webhooks get <id> --json
resend templates list --json
resend templates get <id> --json
resend broadcasts list --json
resend broadcasts get <id> --json
resend contacts list --json
resend contacts get <id> --json
resend segments list --json
resend segments get <id> --json
resend topics list --json
resend topics get <id> --json
resend events list --json
resend events get <id> --json
resend automations list --json
resend automations get <id> --json
```

Before running a command that creates, updates, deletes, forwards, publishes, verifies, listens, opens a browser, or otherwise changes remote state, explain the intended effect and ask for confirmation unless the user explicitly requested it.

## Webhooks

Webhook inspection is safe:

```bash
resend webhooks list --json
resend webhooks get <id> --json
```

`resend webhooks listen` creates a temporary webhook and streams events. Treat it as state-changing and ask first. It must not be used to create a workflow that sends email.

## Templates And Drafts

It is allowed to draft email copy, review HTML, inspect templates, and suggest improvements.

It is not allowed to publish, send, schedule, or trigger delivery of that content through Resend.

## Refusal Template

If asked to send or trigger email delivery, respond with:

"I can't send or trigger emails with Resend from OpenCode. I can draft the email, inspect templates/logs/domains, or prepare a command for you to run manually outside OpenCode."
