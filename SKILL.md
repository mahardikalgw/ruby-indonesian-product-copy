---
name: indonesian-product-copy
description: Keep requester-facing copy in Indonesian and server-owned — strings in one client file and one locale file, status and priority labels sent as props, refusals that state the reason next to the action, and unbuilt features marked as such instead of linking nowhere. Use when adding user-visible text, a label, a notification, an empty state, or an error message.
version: 2.0.0
---

# Indonesian product copy

Copy is a **contract**, not decoration. Three rules generate everything below: the requester-facing
language is Indonesian, the server owns any label that also appears in a mail or export, and nothing
claims more than the product does.

## When to activate

- Adding any user-visible string: heading, label, button, helper text, empty state, error, notification.
- Adding a status, priority, role, or category label.
- A mailer or CSV needs text the page already shows.
- A feature is not built yet and appears in navigation.

## 1. Two files, one owner each

| File | Owns | Example |
|---|---|---|
| `app/javascript/lib/copy.id.ts` | client chrome — nav, buttons, section titles, filter labels | `copy.actions.create`, `copy.tickets.countOpen` |
| `config/locales/helpdesk.id.yml` | anything the server also needs — mails, CSVs, error pages, labels sent as props | `helpdesk.status.in_progress`, `helpdesk.errors.403.title` |

The split is decided by **who else needs the text**. A string that appears in an e-mail and on a page
belongs in the locale file and travels to the client as a prop. Two copies of it will diverge on the day
the mail is reworded.

```ts
// app/javascript/lib/copy.id.ts
export const copy = {
  actions: { create: 'Buat permintaan', save: 'Simpan', cancel: 'Batal', reply: 'Balas' },
  tickets: { countOpen: 'Terbuka', countWaiting: 'Menunggu', countResolved: 'Selesai', countTotal: 'Total' },
  nav: { myTickets: 'Tiket saya', queue: 'Antrean', helpdesk: 'Helpdesk' },
} as const
```

```yaml
# config/locales/helpdesk.id.yml
id:
  helpdesk:
    status:
      open: Terbuka
      in_progress: Sedang ditangani
      waiting_requester: Menunggu pemohon
      resolved: Selesai
      closed: Ditutup
      cancelled: Dibatalkan
```

## 2. Labels come from the server

```ruby
# ApplicationController
inertia_share do
  {
    labels: {
      status: Ticket::STATUSES.index_with { |status| I18n.t("helpdesk.status.#{status}") },
      priority: Ticket::PRIORITIES.index_with { |priority| I18n.t("helpdesk.priority.#{priority}") }
    }
  }
end
```

```tsx
const { labels } = usePage<HelpdeskSharedProps>().props
labels.status[ticket.status] ?? ticket.status
```

Never a `const STATUS_LABELS = { open: 'Terbuka', … }` in TypeScript. That is a second source of truth for
a workflow that lives in the database, and it is wrong the moment a status is renamed.

## 3. Never claim more than the product does

| Situation | Wrong | Right |
|---|---|---|
| Feature not built yet | a link to `/reports` that 500s | nav entry marked "segera", not a link |
| An assistant cannot act | "Saya teruskan ke petugas" | "Silakan buat tiket lewat layanan ini untuk penanganan lebih lanjut" |
| A refusal | "Terjadi kesalahan" | the actual reason, next to the action: "Akun Anda tidak aktif." |
| An empty list | an empty table | "Belum ada tiket. Buat permintaan pertama Anda." |
| A demo account | a password in the UI with no notice | labelled "hanya untuk demonstrasi" |

The placeholder rule is the one that gets skipped: `segera` and a disabled control are honest; a broken
link is a bug report waiting to happen.

## 4. Voice

- **Second person, plain.** "Anda" in formal copy, "kamu" only where the product already uses it — pick one
  per surface and keep it.
- **Sentence case.** No ALL CAPS except a two-letter badge; no Title Case On Headings.
- **A verb on every button.** "Simpan", "Kirim", "Balas" — not "OK", not "Submit".
- **Numbers and dates formatted by locale**, not concatenated: `1.234 tiket`, `9 Nov 2026`.
- **No jargon the ticket page does not use.** The vocabulary of the product is the vocabulary of the copy;
  an assistant that says "pending" when the queue says "Menunggu" teaches users a word the product does not
  have.
- **Errors name the field and the fix.** "Judul wajib diisi." — not "Validasi gagal".

## 5. Where each string physically goes

```
app/javascript/lib/copy.id.ts        client chrome: nav, actions, section titles, filter chips
app/javascript/components/*/*.tsx    presentational text with no server counterpart (short)
config/locales/helpdesk.id.yml       server text: mail subjects and bodies, CSV headers, error pages,
                                     status/priority/role labels, notification kinds, flash messages
```

A string in a `.tsx` file that a mailer also needs is a bug: move it to the locale file and send it as a
prop.

## Prohibitions

| Never | Why |
|---|---|
| An English string in requester-facing UI | The product's audience is Indonesian |
| A label map in TypeScript | Two sources of truth; labels come from props |
| Duplicating a locale string in `copy.id.ts` | Diverges on the first reword |
| A placeholder that links nowhere | A 500 for the user, a bug report for the team |
| "Terjadi kesalahan" for a known refusal | The reason already exists; state it |
| Concatenated numbers or dates | Locale separators are not string concatenation |

## References

- [references/copy-rules.md](references/copy-rules.md) — glossary, label parity test, mail and CSV copy
