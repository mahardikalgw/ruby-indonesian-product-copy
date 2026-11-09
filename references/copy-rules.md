# Copy rules: glossary, parity, mail and CSV

## The glossary is code

```ruby
module Glossary
  STATE_KEYS = %w[open in_progress waiting_requester waiting_third_party resolved closed cancelled].freeze
  ROLE_KEYS  = %w[requester agent team_leader supervisor admin].freeze

  def self.state_label(key) = I18n.t("helpdesk.status.#{key}")
  def self.role_label(key)  = I18n.t("helpdesk.role.#{key}")
end
```

The glossary is the single place a status or role becomes a word. The queue, the ticket page, the mail,
the CSV and the assistant all read it, so "Sedang ditangani" cannot become "Diproses" on one screen and
stay "Sedang ditangani" on another.

## The parity test

Labels drift silently, so pin them:

```ruby
class GlossaryTest < ActiveSupport::TestCase
  test "every status key has a label" do
    Ticket::STATUSES.each do |status|
      label = I18n.t("helpdesk.status.#{status}")
      refute label.start_with?("translation missing"), "missing label for #{status}"
      refute_equal status, label, "status #{status} has no human label"
    end
  end

  test "every role key has a label" do
    User::ROLES.each do |role|
      refute_equal role, I18n.t("helpdesk.role.#{role}")
    end
  end

  test "the assistant vocabulary matches the glossary" do
    assert_equal Glossary::STATE_KEYS.map { |key| Glossary.state_label(key) },
                 Assistant::Vocabulary.canonical_states
  end
end
```

That last test is the one that matters when an assistant is in the product: a model will invent
"pending", and the only way to know it cannot is to assert the vocabulary it is allowed to use is the same
list the UI uses.

## Mail and CSV

Both are requester-facing and both are generated outside the page, so they read the locale file directly:

```yaml
id:
  helpdesk:
    mail:
      ticket_created:
        subject: "Permintaan Anda #%{ticket_number} telah kami terima"
        greeting: "Halo %{name},"
        body: "Kami sudah menerima permintaan Anda dan akan menindaklanjutinya."
        cta: "Lihat permintaan"
    exports:
      tickets:
        columns:
          number: Nomor
          title: Judul
          status: Status
          last_activity: Aktivitas terakhir
```

- A CSV header is copy. It comes from the locale file, never from `attribute.humanize`, so the export reads
  like the screen.
- A mail never includes an internal field. The mailer renders the requester payload, so the isolation that
  protects the page protects the inbox too.
- Interpolate values; never build a sentence by concatenating fragments (`"#{a} #{b}"` breaks in every
  language that reorders them).

## Number and date formatting

```ruby
I18n.l(time, format: :short)          # 9 Nov 2026 14.05
number_with_delimiter(1234)           # 1.234
```

On the client, use one helper in `lib/format.ts` — `Intl.NumberFormat('id-ID')` and
`Intl.DateTimeFormat('id-ID')` — and call it from every component. A component that formats inline is a
component that will be reformatted differently next month.

Dates travel as ISO 8601 strings and are formatted at the edge. A `Time` object in props makes the
timezone a client decision.

## Empty and error states

The server sends the copy, so it is translatable and testable:

```ruby
empty_state: {
  title: I18n.t("helpdesk.tickets.empty_title"),      # "Belum ada tiket"
  body: I18n.t("helpdesk.tickets.empty_body"),        # "Permintaan yang Anda buat akan muncul di sini."
  action_label: (I18n.t("helpdesk.tickets.create") if can_create),
  action_href: ("/tickets/new" if can_create)
}
```

```yaml
id:
  helpdesk:
    errors:
      "403":
        title: "Anda tidak memiliki akses"
        body: "Akun Anda tidak diizinkan membuka halaman ini."
      "404":
        title: "Halaman tidak ditemukan"
        body: "Alamat yang Anda buka tidak tersedia."
```

One shared error page renders all four statuses, so an error never invents its own wording.

## Refusal copy

A refusal states **why**, next to the control that caused it:

```
"Alamat surel Anda di luar domain institusi."
"Akun Anda tidak aktif. Hubungi administrator."
"Permintaan tidak dapat dibuka. Masuk untuk melihat permintaan Anda."
```

Rules:

- Never an id, a count, or another person's name (see `scoped-authorization`).
- Never "Terjadi kesalahan" for a refusal the application already understands.
- Never blame the user for a policy ("Anda salah domain") — state the policy.

## The placeholder inventory

Keep one list of everything not built, and render it as non-interactive:

```tsx
const PENDING = { label: 'Laporan', note: 'segera' }   // rendered as a disabled nav item, never a Link
```

A product that ships a navigation entry to a 500 teaches users to distrust the navigation. The marker is
honest and takes one line; the alternative is a support ticket about a broken page.
