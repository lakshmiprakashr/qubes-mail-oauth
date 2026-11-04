# Qubes Mail OAuth website

Public informational site for the private **Qubes Mail** OAuth application.

This repository contains only the static website used for the application's
Google OAuth branding and privacy disclosures. It does **not** contain the
application source, credentials, OAuth client secrets, tokens, email content,
internal network details, or private configuration.

**Status:** prepared locally. Not pushed, not published, and not connected to
Google OAuth settings. Publication requires explicit operator authorization.

## Planned public URLs

- Home: <https://lakshmiprakashr.github.io/qubes-mail-oauth/>
- Privacy: <https://lakshmiprakashr.github.io/qubes-mail-oauth/privacy.html>

All internal links are relative so the site works under the repository
subpath and under a future custom domain.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | Application homepage: identity, functionality, data access, privacy summary, contact. |
| `privacy.html` | Gmail OAuth privacy policy, including local storage, retention, deletion, sharing, and Google Limited Use disclosure. |
| `README.md` | Local preview, validation, GitHub Pages, Google branding, and publication instructions. |
| `.gitignore` | Ignores local tooling output and common secret-bearing files. |

The pages are self-contained: inline CSS only, no cookies, no analytics, no
tracking pixels, no external fonts or images, and no JavaScript.

## Local preview

From this directory:

```sh
python3 -m http.server 8000
```

Open <http://localhost:8000/>. Stop the server with `Ctrl+C`.

## Validation

The pages were checked before the initial commit for:

- parseable HTML with a single `html`, `head`, `body`, `title`, and `h1`;
- balanced elements and unique `id` attributes;
- no `<script>` elements or inline event handlers;
- no cookies, analytics, tracking, or external resource loads;
- a viewport meta tag and language attribute;
- working relative links and fragments;
- no credentials, tokens, real email addresses, or private configuration.

Re-run a local link check with:

```sh
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path

class Links(HTMLParser):
    def __init__(self):
        super().__init__()
        self.links = []
    def handle_starttag(self, tag, attrs):
        attrs = dict(attrs)
        for key in ("href", "src"):
            if key in attrs:
                self.links.append(attrs[key])

errors = []
for page in ("index.html", "privacy.html"):
    parser = Links()
    parser.feed(Path(page).read_text(encoding="utf-8"))
    for link in parser.links:
        if link.startswith(("http://", "https://", "mailto:", "#")):
            continue
        target = (Path(page).parent / link.split("#", 1)[0]).resolve()
        if not target.exists():
            errors.append(f"{page}: missing {link}")
if errors:
    print("\n".join(errors))
    raise SystemExit(1)
print("internal links: PASS")
PY
```

Optional HTML5 validation with Node.js:

```sh
npx --yes html-validate@latest index.html privacy.html
```

For a public-page check, use the W3C Nu validator against the deployed URLs
after publication: <https://validator.w3.org/nu/>.

## GitHub Pages setup

These are human steps. Do not run them until publication is authorized.

1. Review `index.html`, `privacy.html`, and the operator-review comments in the
   HTML. Resolve every `OPERATOR REVIEW REQUIRED` item.
2. Confirm the repository contains only public website files.
3. Create the GitHub repository `lakshmiprakashr/qubes-mail-oauth`. Do not
   initialize it with a README, license, or `.gitignore`.
4. Add the remote and push the existing local `main` branch:

   ```sh
   git remote add origin git@github.com:lakshmiprakashr/qubes-mail-oauth.git
   git push -u origin main
   ```

5. In the repository, open **Settings → Pages**.
6. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
7. Set **Branch** to `main` and **Folder** to `/ (root)`. Save.
8. Wait for the Pages deployment to finish. Check the **Actions** tab and the
   Pages URL shown in **Settings → Pages**.
9. Verify both planned URLs while signed out:
   - <https://lakshmiprakashr.github.io/qubes-mail-oauth/>
   - <https://lakshmiprakashr.github.io/qubes-mail-oauth/privacy.html>
10. Confirm the privacy link on the homepage points to the privacy policy URL
    that will be entered in Google Auth Platform.

Repository visibility note: GitHub Pages for a private repository requires a
paid GitHub plan. The simplest path for a public site is a public repository
containing only these public website files.

## Google OAuth Branding instructions

Use the **Google Auth Platform** in Google Cloud Console. The current console
screens are Branding, Audience, Clients, Data Access, and Verification Center.

### Branding fields

| Field | Recommended value |
| --- | --- |
| App name | `Qubes Mail` (must match the homepage branding) |
| User support email | An email address you monitor and are willing to show on the consent screen. Do not add it to this repository. |
| App logo | Optional. Square PNG/JPG/BMP, 120×120 px recommended, under 1 MB. Do not use Google branding. |
| Application home page | `https://lakshmiprakashr.github.io/qubes-mail-oauth/` |
| Privacy policy | `https://lakshmiprakashr.github.io/qubes-mail-oauth/privacy.html` |
| Terms of service | Optional. Leave blank unless you create a terms page. |
| Authorized domains | `lakshmiprakashr.github.io` (add before the URLs above) |
| Developer contact email | An email address you monitor. Do not add it to this repository. |

The app name, homepage, and privacy policy must accurately identify the same
application. The privacy policy must be linked from the homepage and must match
the URL entered on the OAuth consent screen.

### Authorized domain and ownership verification

Google requires the homepage and privacy policy to be hosted on a verified
domain owned by the project. For GitHub Pages:

1. Add the site to Google Search Console using the account that is an
   owner or editor on the Google Cloud project.
2. Prefer a **URL-prefix** property for the full published site path, or a
   **Domain** property if you control DNS.
3. Verify ownership with the HTML file or HTML meta tag method. Keep the
   verification file or tag after verification succeeds.
4. Enter `lakshmiprakashr.github.io` in **Authorized domains** before entering
   the homepage or privacy URL.

**Unresolved risk:** `github.io` is a shared hosting domain. Google's guidance
prefers a domain you own, and there are reports of Google Auth Platform
rejecting `*.github.io` authorized domains even after Search Console
verification. Do not assume this step is complete until the Authorized domains
field accepts the domain. If it is rejected, use a custom domain you own with
DNS verification and update:

- both pages' titles, metadata, and any absolute URLs;
- this README's planned URLs;
- the Google Branding fields;
- any bookmarks or OAuth documentation.

### Gmail scope disclosure

The current application design uses **`https://mail.google.com/`** for Gmail
IMAP and SMTP access with OAuth 2.0. Google classifies this as a **restricted**
Gmail scope. Google describes it as permission to read, compose, send, and
permanently delete email.

The homepage and privacy policy disclose this scope. If the shipped OAuth
client uses a different scope, update both pages before publishing.

### Publishing status decision

Choose one path deliberately:

1. **Testing** — limited to listed test users, but authorizations and refresh
   tokens expire after seven days. This is impractical for ongoing mail sync.
2. **In production, unverified** — avoids the seven-day Testing expiry, but
   users see an unverified-app warning and the app is capped at 100 users.
   Google's documentation describes a personal-use exception: if you are the
   only user, or only a few users known personally, you do not need to submit
   for verification and may proceed through the unverified-app screen.
3. **In production, verified** — removes the unverified warning and allows the
   app name/logo to appear. This requires brand verification and, because the
   app uses a restricted Gmail scope, restricted-scope data access verification.
   A security assessment may also apply; confirm the current requirement with
   Google for a local-only app.

For a single-user private application, path 2 is the usual minimum. Path 3 is
required only if you want verified branding and unrestricted distribution.

### Brand verification steps

Brand verification is required if you want the app name or logo displayed on
the consent screen. It is also required before data access verification.

1. Open **Google Auth Platform → Branding**.
2. Complete the fields above.
3. Click **Verify Branding**.
4. Fix any issues shown. If you disagree, request additional review.
5. When the status is **Ready to publish**, click **Publish branding**.
6. Publish within seven days; otherwise the result expires and must be
   re-verified.

### Restricted-scope verification, if pursued

1. Publish branding first.
2. Open **Verification Center → Data access**.
3. Add `https://mail.google.com/` and save the scope.
4. Provide a justification explaining why IMAP/SMTP requires the full Gmail
   scope and why narrower scopes do not work for this architecture.
5. Provide a demonstration video showing the end-to-end OAuth flow, the consent
   screen, the application's actual functionality, and the same app name and
   branding as the submitted site.
6. Respond to any Google Trust and Safety requests.
7. Determine whether a security assessment applies. The app is designed to
   process mail locally, but confirm this with Google before claiming an
   exemption.

Reference pages:

- Brand verification: <https://developers.google.com/identity/protocols/oauth2/production-readiness/brand-verification>
- Verification requirements: <https://support.google.com/cloud/answer/13464321>
- Restricted scope verification: <https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification>
- Gmail API scopes: <https://developers.google.com/workspace/gmail/api/auth/scopes>
- Google API Services User Data Policy: <https://developers.google.com/terms/api-services-user-data-policy>

## Unresolved Google requirements

These items require operator review or a Google-side decision. They are not
resolved by this local website.

- [ ] Confirm the actual OAuth scope requested by the shipped client. The site
      currently states `https://mail.google.com/`.
- [ ] Decide the publishing status: Testing, In production unverified, or
      In production verified.
- [ ] Verify `lakshmiprakashr.github.io` ownership in Google Search Console
      with a project owner/editor account.
- [ ] Confirm Google accepts `github.io` as an authorized domain. If not, use a
      custom domain and update all URLs.
- [ ] Decide whether to display an app name/logo and complete brand
      verification.
- [ ] If pursuing verified status, complete restricted-scope data access
      verification, including the scope justification and demonstration video.
- [ ] Confirm whether a security assessment applies to the local-only design.
- [ ] Choose the user support email, developer contact email, and whether to
      provide a terms-of-service page.
- [ ] Review every `OPERATOR REVIEW REQUIRED` comment in the HTML and remove
      the comments only after the claim is verified.
- [ ] Confirm whether optional mail-analysis or AI features process Google user
      data and, if so, disclose the data flow before publishing.
- [ ] Obtain explicit authorization to publish.

## Privacy claims requiring operator approval

The HTML contains hidden operator-review comments next to claims that cannot be
proven from this website repository alone. Verify each against the shipped
application before publication:

1. The OAuth client requests `https://mail.google.com/`.
2. OAuth tokens are encrypted at rest and plaintext tokens are not persisted.
3. No Qubes Mail-operated server receives or stores mail or tokens.
4. Optional analysis features run on equipment controlled by the user and do
   not send Google user data to a third party.
5. The folder defaults are Inbox, Sent, Drafts, and Trash.

Repository evidence supports items 1, 2, 4, and 5. Item 3 is supported by the
local-only architecture. The live OAuth client configuration, deployed token
store, and optional AI endpoint still require operator confirmation.

If any claim is inaccurate, update both pages and this README.

## Publication authorization gate

**STOP.** Do not create the remote repository, push, enable GitHub Pages, or
change Google OAuth settings until the operator explicitly authorizes public
publication.

When authorized, the human publishing sequence is:

1. Complete the unresolved-requirements checklist.
2. Re-run the local validation commands above.
3. Create the remote repository and push `main`.
4. Enable GitHub Pages from `main` and `/ (root)`.
5. Verify both public URLs and the homepage-to-privacy link.
6. Configure Google Auth Platform Branding with the published URLs.
7. Verify the authorized domain in Search Console.
8. Decide and set the publishing status.
9. Verify branding if the app name/logo should appear.
10. Submit restricted-scope verification if pursuing verified status.
11. Test the OAuth consent flow and confirm the consent screen shows the
    correct app name, homepage, privacy policy, and scope.

No remote, push, Pages deployment, or Google settings change was performed
during preparation.
