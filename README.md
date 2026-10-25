# Who Owe Me?

A private, offline-friendly ledger of who owes you money. Built for traders, tailors, landlords and anyone who sells on credit and tracks it in a notebook.

Live: https://who-owe-me.vercel.app/

No account. No backend. No analytics. Everything is stored in your browser's IndexedDB and never leaves the device.

## Stack

A single HTML file with inline CSS and vanilla JavaScript. There is no build step. The page loads DM Sans from Google Fonts and Lucide from its CDN. Lucide currently uses `@latest`, so pin the dependency before treating the deployment as a production-hardened build.

| Piece | Source | Why |
| --- | --- | --- |
| CSS | Inline stylesheet | Design tokens, responsive layouts, custom dropdown, calendar and bottom-sheet styles |
| Vanilla JavaScript | Inline script, one IIFE | Local state, ledger calculations, IndexedDB persistence and UI interactions |
| Lucide | `https://unpkg.com/lucide@latest` | Interface icons rendered with `lucide.createIcons()` |
| DM Sans | Google Fonts | A consistent sans-serif family across headings, forms, labels and body copy |

The app has no framework or runtime build tooling. The README is documentation and is not required to run the app.

## Run locally

```bash
npx serve .
```

Open the printed URL. IndexedDB needs `http(s)://` or `localhost`, not `file://` in some browsers.

## Deploy

Vercel: import the repo, framework preset "Other", no build command, output directory `.`. Nothing else to configure.

## How the data works

Three IndexedDB stores: `people`, `entries`, `meta`.

- **Append-only ledger.** An entry is a `debt`, a `payment`, or a `void` that points at another entry. Balances are always computed from entries by the pure `balances()` function. No balance is ever stored or mutated, so a wrong edit cannot silently corrupt someone's debt, and every change leaves a trail.
- **Voiding.** Mistakes are voided, not deleted. A voided entry stays in history, struck through. The toast Undo is the only time an entry is physically removed, and only the void marker you just created.
- **Integer minor units.** Amounts are stored as integers (kobo, cents, pence). Decimal places come from the currency itself via `Intl`, so JPY has 0 and KWD has 3. Parsing is string based, never `parseFloat`.
- **Multi-currency.** Any ISO currency the browser knows. Totals are shown per currency and are never converted or summed across currencies, because that would need exchange rates and would be wrong the moment it was rendered.
- **Overpayment is blocked.** A payment cannot exceed what is owed in that currency.

## Backup

Browser storage can be wiped by clearing site data, by the browser evicting storage under pressure, or by losing the phone. The app requests persistent storage, but treat that as a hint, not a guarantee.

- Export downloads a JSON file. Import validates every field and merges by entry id, so importing the same file twice changes nothing.
- A banner appears on the home screen once your last backup is more than 7 days old.

## WhatsApp reminders

"Remind" opens `wa.me/<number>?text=...` with the person's name and current balance prefilled. It is a single, user-initiated message per tap. There is no bulk sending and no automation.

Numbers are normalized: `+` and `00` prefixes are stripped, and a leading `0` is treated as Nigeria (`234`). For any other country, enter the number with `+` and the country code.

## Design and motion notes

- Warm cream surfaces, near-black ink text, muted labels, coral-orange accents, and occasional soft pastel panels create the visual identity.
- DM Sans is used throughout. Display headings remain large and expressive through scale, weight, spacing and line height rather than a serif typeface. The wordmark has no decorative dot or pill.
- Rounded cards and controls use consistent radii, subtle borders and restrained shadows. Orange is reserved for primary actions, selected dates, focus states and a small number of important accents.
- Motion uses short ease-out transitions, restrained entrance effects and press feedback. Avoid `transition: all`.
- `prefers-reduced-motion` is respected. Motion is reduced when the system preference is enabled.
- Money uses tabular numerals. Inputs are 16px to stop iOS zooming on focus. Safe-area insets and `100dvh` are handled, with layouts adapted for narrow phones, larger screens and desktop.
- Currency selection uses a searchable custom dropdown. Date selection uses a custom calendar with month navigation, a selected-date state and a Today shortcut.

## Known limits

1. **External assets need a network connection.** DM Sans and Lucide load from external CDNs. The ledger data remains local, but fonts and icons may not load offline or when those services are unavailable.
2. **No offline-first shell.** There is no service worker or web manifest. The page itself needs to be loaded over the network unless the browser cache serves it.
3. **Lucide is not pinned.** The script currently loads `lucide@latest`. Pin a tested version and consider adding Subresource Integrity before treating the deployment as production-hardened.
4. **Sorting by "Most owed"** compares totals across currencies by raw major-unit value, which is approximate when someone owes in several currencies.
5. **No people management.** Settled people drop off the home list but are kept in storage. There is no rename, archive or hard-delete yet.
6. **Single device.** No sync. Use Export and Import to move data.
