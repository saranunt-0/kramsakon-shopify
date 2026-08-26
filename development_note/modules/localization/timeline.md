# Module: localization
<!-- Everything that decides which language the storefront speaks: the theme's
     locale files, the language picker UI in the header/footer/announcement bar,
     and the script-specific typography that supported a second script. -->

---

## [2026-08-03] Make the storefront English-only, remove Thai

**Type**: `feature`
**Status**: `resolved` (theme side) — see Unverified Items for the Shopify-admin half

### Context

The theme shipped as a bilingual English/Thai storefront:

- `locales/th.json` + `locales/th.schema.json` carried the Thai translations.
- A language picker was rendered in four places (header, footer, announcement
  bar, mobile drawer) via `snippets/language-localization.liquid`.
- Two Thai font families (Noto Serif Thai, IBM Plex Sans Thai) were loaded from
  Google Fonts and appended to both font stacks, with eight `:lang(th)` rules
  correcting letter-spacing, italics and line-height for Thai text.
- Two settings carried Thai copy: the tagline hint in `settings_schema.json`
  and `brand_headline` in `settings_data.json` (`"KRAMSAKON — ครามสกล"`).

Request: English only, Thai removed.

### Work Done

Located every Thai/language-switcher touchpoint before editing rather than
grepping file-by-file mid-change:

1. Scanned all text files for characters in the Thai Unicode block
   (U+0E00–U+0E7F) with a Python walk. A naive `grep` byte-range matched
   binaries (JPEGs, PNGs) and was discarded as a false-positive source — the
   codepoint scan returned exactly four files.
2. Grepped for the language-picker surface area: `enable_language_selector`,
   `available_languages`, `language-localization`, `language-selector`,
   `language_label`.
3. Removed, in dependency order: render sites → snippet → schema settings →
   stored settings in the section-group JSON → CSS.

### Checklist

- [x] `locales/th.json`, `locales/th.schema.json` deleted
- [x] `snippets/language-localization.liquid` deleted
- [x] Language picker removed from `sections/header.liquid`,
      `sections/footer.liquid`, `sections/announcement-bar.liquid`,
      `snippets/header-drawer.liquid`
- [x] `enable_language_selector` removed from all three section schemas and
      from `sections/header-group.json` / `sections/footer-group.json`
- [x] Composite conditionals that mixed country + language rewritten to
      country-only (header `localization_forms`, footer
      `footer__content-bottom-wrapper--center`, announcement-bar grid columns,
      `theme.liquid` localization asset loading)
- [x] Thai font families dropped from the Google Fonts request and from both
      CSS font stacks; all eight `:lang(th)` rules removed
- [x] Thai copy removed from `settings_schema.json` and `settings_data.json`
- [x] All four JSON files re-parsed after edit — valid
- [x] `kramsakon.css` brace balance checked after the scripted rule removal
      (131 open / 131 close)
- [x] Re-ran every grep — zero remaining references

### Unverified Items

> No Shopify store or CLI is available in this environment, so nothing was
> rendered. Please run:

- [ ] `shopify theme check` — expect no new offences. Specifically confirm no
      `MissingTemplate` / `UndefinedObject` from the removed snippet.
- [ ] `shopify theme dev` → load home, a product page, and open the mobile
      drawer. Looking for: no language picker anywhere, country/currency picker
      still present and functional, footer bottom row still aligned.
- [ ] Confirm English renders in Fraunces (headings) and Inter (body) — the
      Google Fonts request changed, so a typo there would silently fall back to
      Georgia/system-ui.

### Root Cause / Outcome

Not a defect — a scope change. The theme is now single-language by
construction: there is no language-picker UI to enable, and no non-English
locale that the theme itself supplies for Thai.

### Fix / Implementation Detail

The language picker was removed rather than merely defaulted to `false`. A
default-off checkbox is one click in the theme editor from putting the picker
back; removing the setting makes English-only a property of the theme instead
of a property of the current configuration. Trade-off: this diverges from
upstream Dawn, so a future Dawn merge will conflict in `header.liquid`,
`footer.liquid`, `announcement-bar.liquid` and `header-drawer.liquid`. Judged
worth it — the theme is already heavily customised (custom `kram-*` sections,
a replaced font system, a bespoke header logo snippet).

The country/currency selector was deliberately left intact. It is a separate
Shopify Markets feature, and the request concerned language.

The stock Dawn locale files for the other ~25 languages were left in place.
They are inert: a theme locale file only takes effect for a language the
merchant has published in Shopify admin. Deleting them would be a large,
purely cosmetic diff. If the intent is to make the repo literally English-only
on disk, that is a one-line follow-up (`git rm locales/*.json` except
`en.default*`), deliberately not bundled here.

### Assumptions Made

- **The country/currency selector should stay.** The request named language.
  If the store also sells in one market only, that picker is dead weight and
  should be turned off in the theme editor — not assumed here.
- **English is, or will be, the store's primary language in Shopify admin.**
  The theme cannot set this; see Unverified Items.
- The `:lang(th)` typographic corrections were removed rather than kept as
  dead CSS. They are recoverable from git history if Thai ever returns.
