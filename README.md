# Indonesian product copy

Keep requester-facing copy in Indonesian and server-owned — strings in one client file and one locale file, status and priority labels sent as props, refusals that state the reason next to the action, and unbuilt features marked as such instead of linking nowhere. Use when adding user-visible text, a label, a notification, an empty state, or an error message.

One skill, one repository. Part of a twelve-skill pack extracted from a production Rails 8 +
Inertia + React helpdesk monolith — the opinionated layer no public Rails or shadcn skill can know.
Version `2.0.0` tracks the source constitution.

## Install

```jsonc
// skills-lock.json
{
  "version": 1,
  "skills": {
    "indonesian-product-copy": {
      "source": "mahardikalgw/indonesian-product-copy",
      "sourceType": "github",
      "skillPath": "SKILL.md"
    }
  }
}
```

Or clone and symlink:

```bash
git clone git@github.com:mahardikalgw/indonesian-product-copy.git
ln -s "$PWD/indonesian-product-copy" .claude/skills/indonesian-product-copy
```

## When to activate

- Adding any user-visible string: heading, label, button, helper text, empty state, error, notification.
- Adding a status, priority, role, or category label.
- A mailer or CSV needs text the page already shows.
- A feature is not built yet and appears in navigation.

## What's inside

```
indonesian-product-copy/
  SKILL.md
  references/copy-rules.md
```

`SKILL.md` is a router (kept under 250 lines) — the depth lives in `references/`, and the
copy-in code lives in `assets/`.

## Validate

```bash
./bin/check-skill
```

Checks frontmatter, the activation section, link resolution, orphaned references, script
executability, leftover placeholders, and — when the source app is checked out next to this
repository — that vendored scripts still match it byte for byte. Runs in CI on every push.

## Related skills

- [`inertia-vertical-slice`](https://github.com/mahardikalgw/inertia-vertical-slice) — route → skinny controller → query/service → typed props → Inertia page → test
- [`scoped-authorization`](https://github.com/mahardikalgw/scoped-authorization) — scopes, 404-not-403, explicit public opt-outs, isolation by construction

## Licence

MIT — see [LICENSE](LICENSE).
