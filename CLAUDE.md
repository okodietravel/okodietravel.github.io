# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The marketing site for Okodie Travel Consult (scholarships abroad, visa help, flight ticketing, tours), served by GitHub Pages from the `main` branch of `okodietravel/okodietravel.github.io` at the custom domain in `CNAME` (`okodietravel.com`). Pushing to `main` publishes to the live site.

There is no build, no package manager, no framework and no tests. The whole site is one file: `index.html`, with inline CSS, inline JS and images embedded as base64 data URIs.

## Previewing

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000. Opening `index.html` directly in a browser also works; nothing depends on a server. (Inside the Claude desktop app a background `http.server` can't read this Downloads folder, so preview the file directly there.)

## Working with index.html

- **The file is ~200 KB, almost all of it base64 image data** (the favicon, the nav logo, the footer logo and the header cover photo, set through the `--cover` custom property). The actual markup is only ~240 lines. To read the structure without the blobs, strip them into a scratch copy, e.g. `sed -E 's/data:[a-z]+\/[a-z+.-]+;base64,[A-Za-z0-9+\/=]{40,}/DATA_URI/g' index.html`, but always make edits against the real `index.html`.
- **The CSS is layered, not deduplicated.** Later rules in the single `<style>` block deliberately override earlier ones (e.g. `header` background is set three times, ending with the cover-photo gradient; `.logo`, `.hero` and `.card` are each extended further down). Check for later overrides before changing a rule, or add the change at the end.
- **Theming** uses CSS custom properties on `:root`, with a dark palette applied via `prefers-color-scheme: dark` (guarded by `:root:not([data-theme="light"])`) and again under `:root[data-theme="dark"]`. New colours should be added as tokens in all three places.
- **Layout breakpoints**: 560px (enquiry form goes to one column), 600px (nav hides everything but the "Enquire" CTA), 700px (cover photo repositions). The layout uses `env(safe-area-inset-*)` for notched phones.

## Enquiry form (the only JS)

The `#q-form` in `#enquiry` submits to the Google Form "Website enquiries (okodietravel.com)", owned by support@okodietravel.com. This repo is public and every file is served on the site, so the Form's edit ID and the Sheet ID are deliberately left out of it. To look them up, run `gws drive files list --params '{"q":"name contains '\''Website enquiries'\''"}'`. Its `action` is the Form's public `.../forms/d/e/<published ID>/formResponse` URL. Each question field's `name` is that question's `entry.<n>` ID, where `n` is the Forms API `questionId` converted from hex to decimal. The email field is named `emailAddress` because the Form uses Google's built-in "Collect email addresses: Responder input" setting rather than an email question. The Form's responders must stay set to "Anyone with the link", or anonymous submissions are rejected. Responses also flow into the linked Sheet "Website enquiries (okodietravel.com) (Responses)" (tab "Form responses 1"). The link was made in the Forms UI, because the Forms API can't set a response destination.

The Form can be managed with the `gws` CLI, which is signed in as support@okodietravel.com: `gws forms forms get|batchUpdate --params '{"formId":"<edit ID>"}'`, and `gws forms forms responses list` to confirm a test submission arrived.

The inline script intercepts submit, checks every `[required]` field is filled in and the email is valid, then POSTs the fields with `fetch(..., {mode:'no-cors'})` and shows `#q-done`. Because the response is opaque, the page can't tell whether Google accepted the entry: if the `name`s or the Service option wording stop matching the Form, submissions vanish silently. Test a real submission against the Form's responses after any change. If `action` doesn't contain `/formResponse`, the script shows a "not connected" message instead of sending.

WhatsApp and email are deliberately separate from the form: plain links in `.alt-contact` under the form, the contact list and the floating button.

Each service card's "Enquire →" link (`a.more`) carries a `data-s` attribute that is written into the `#q-service` select on click. **The `data-s` values must match the `<option>` text exactly, and the options must match the Google Form's Service choices exactly** (Google rejects unknown choices), so when renaming or adding a service, update the card, its `data-s`, the select option and the Google Form together.

## Contact details repeated across the page

The WhatsApp number (`46762623488`) and `support@okodietravel.com` appear in several places: the hero button, the contact list, the floating WhatsApp button (`.wafab`), the "Prefer to chat?" line under the enquiry form and, for the email, the FAQ text and the JSON-LD block in `<head>`. Change all of them together. The JSON-LD `TravelAgency` block and the `<title>`/description/Open Graph meta tags should also stay in step with the page content.

## CNAME

The git history shows `CNAME` being deleted and re-created several times. Don't remove or rename it; GitHub Pages needs it to keep the custom domain.
