# ex-bbc.net: assessment and migration plan

Assessed 22 Nov 2026 from the outside only (public pages, HTTP headers, DNS, whois). No server access yet. Everything below marked "observed" was seen directly; anything else is labelled as inference or as needing confirmation.

## 1. Summary

1. The forum is small and still in daily use. Members posted obituaries on 16 and 20 Nov 2026.
2. Only the admin side is broken: `AdminIndex.pl` returns HTTP 500 (observed). Reading, posting and login pages work.
3. The software is YaBB 2.3.1, released January 2009 and never patched. It has no maintained upstream.
4. The data is tiny: 3,560 topics, 7,597 posts, 1,450 members. Sizing is not a constraint on any option.
5. The domain registration expires on 2 Jan 2027, six weeks from now. This is the most urgent item and has nothing to do with the software.
6. Recommendation: back up first, try to get admin working again on the current install (likely a small fix), then migrate to SMF 2.1.8 at a calm pace. Don't move to the 2016 YaBB release.

## 2. Current site

| Item | Observed value |
|---|---|
| Forum software | YaBB 2.3.1 (`$Revision: 1.18.2.4.2.13`), Perl CGI, flat-file storage |
| Web server | Apache/2.4.71 (Debian) |
| Host | Positive Internet, 185.27.20.100 |
| TLS | Let's Encrypt, valid to 27 Jan 2027 |
| Domain | Created 2002-01-02, registrar Tucows, expires 2027-01-02 |
| DNS | dns0/dns1.positive-internet.com |
| Charset | ISO-8859-1 |
| Membership | Closed; join by emailing a request to the administrators |

### Boards

| Board | Topics | Posts |
|---|---:|---:|
| News and Comment | 1,806 | 3,799 |
| Notices, obituaries and tributes | 1,051 | 2,064 |
| History | 231 | 496 |
| Pensions | 148 | 453 |
| Where are they now? | 155 | 283 |
| For Those Of Sound Mind | 45 | 194 |
| Ex-pats | 22 | 117 |
| Reunions | 46 | 74 |
| Help! | 22 | 56 |
| Nations, Regions, and Local Radio | 19 | 26 |
| Tech OPs | 7 | 19 |
| Endboard | 8 | 16 |
| **Total** | **3,560** | **7,597** |

Board totals add up to the forum's own statistics, so the counts are consistent.

### Other observations

1. `AdminIndex.pl` returns HTTP 500 with Apache's bare error page (654 bytes). This is not YaBB refusing a guest: YaBB's own permission errors render a full forum page (for example `YaBB.pl?action=myprofile` returns 403 "An Error Has Occurred! ... only for members"), and a non-existent script returns 404. So the admin script exists but crashes before YaBB can check who is asking. The owner's report that admin was lost after the restore fits this. Not confirmed for a logged-in admin; the Apache error log will say why.
2. The site serves every page and form over both HTTP and HTTPS with no redirect, and all internal links and the login form post to `http://`. Passwords therefore travel in clear text for anyone who follows the site's own links.
3. `/yabbfiles/Attachments/` has directory listing switched on. It is currently empty, so there are no attachments to migrate.
4. The primary MX record points at `mial.ex-bbc.net`, which does not resolve (typo for `mail`). Mail falls through to the secondary, `bludger.positive-internet.com`. No SPF record.
5. The forum's "most users ever online" figure is 2,213 on 12 Nov 2026, of which 2,204 were guests. The previous members record is 10. That spike is bot traffic, not people. It falls in the same weeks as the outage the owner describes, but nothing here shows the two are connected.
6. The owner reports the site being taken down during a "PHP vulnerability issue". YaBB is Perl, not PHP, so whatever that issue was, it was at the host level, not in the forum code. Worth asking Positive Internet what they changed when they restored it.

## 3. Software versions and vulnerabilities

### YaBB 2.3.1 (current)

1. Released 4 Jan 2009. Nearly 18 years without patches.
2. Known YaBB CVEs ([CVE-2007-3208](https://nvd.nist.gov/vuln/detail/CVE-2007-3208), CVSS 10, admin takeover via register/profile; [CVE-2007-3295](https://nvd.nist.gov/vuln/detail/CVE-2007-3295), authenticated Perl code execution) are listed against 2.1 and earlier. 2.3.1 is not listed as affected, and I could not confirm whether it is.
3. A later XSS report against 2.5.2 ([SourceForge bug 69](https://sourceforge.net/p/yabb/bugs/69)) was never fixed. Code shared across 2.x versions is plausibly affected; not verified.
4. Practical risk is lower than the CVE list suggests because registration is closed. The bigger exposure is passwords sent over plain HTTP and old Perl CGI running on a current Apache.
5. Old YaBB 2.3 stores passwords as unsalted MD5 hashes (from knowledge of the codebase; confirm on the files). Any migration will likely mean members reset their passwords.

### YaBB "2016" release (option offered by the host)

1. The last YaBB release on the [SourceForge project](https://www.sourceforge.net/p/yabb/news) news page is 2.5.2 (June 2013). The "2016" version mentioned to the owner is presumably a later release from yabbforum.com; I could not find its release notes.
2. Either way it is a dead project. Upgrading buys a few years at best and leaves the forum on Perl CGI with nobody maintaining it. Not recommended.

### Simple Machines Forum (proposed)

1. Current release is SMF 2.1.8, which fixes [CVE-2026-39903](https://nvd.nist.gov/vuln/detail/cve-2026-39903) (attachment approval permission bypass in earlier 2.1.x). Actively maintained; 3.0 is in alpha.
2. Needs PHP and MySQL/MariaDB. Third-party package metadata says PHP 7.1+ and MySQL 5.6+; confirm against the [official requirements](https://wiki.simplemachines.org/smf/Which_version_of_SMF_should_I_use).
3. SMF began as a fork of YaBB SE, the PHP rewrite of YaBB. It is a reasonable "descendant", but it shares no code or data format with the Perl YaBB 2.x currently running.
4. The converter is the real problem. The official YaBB 2 converter targets SMF 1.x, was listed as beta/partial, and users reported it missing categories ([SMF forum thread](https://www.simplemachines.org/community/index.php?msg=748648)). There is no maintained YaBB 2 to SMF 2.1 converter that I could find. That matches what the owner read on the SMF forums.

### Alternatives considered

1. **phpBB, MyBB, Discourse, Flarum.** None has a maintained YaBB 2 importer either. Discourse is heavy to host for 1,450 members. No reason to prefer them over SMF, which is what Positive Internet and the owner are already expecting.
2. **Freeze as a static archive plus a new forum.** Viable fallback if the migration stalls: render every topic to HTML so nothing is lost, and start fresh on SMF.

## 4. Sizing

1. Text: 7,597 posts. Allowing 1 KB per post with markup, that is under 10 MB. The whole YaBB data directory (messages, members, board index files) is likely under 50 MB. Inference; confirm with `du` once we have access.
2. Attachments: none.
3. Avatars and templates: small; the default YaBB theme is in use.
4. Traffic: "not a busy site". Peak real members online was 10 (2012). Bot traffic is the only load worth planning for.
5. Hosting: any basic shared PHP/MySQL plan is enough. Staying on Positive Internet is simplest, since they already host DNS, mail and the site.
6. Converter effort: because the data is small and in plain text files, writing our own YaBB 2.3 to SMF 2.1 import script is a few days' work rather than weeks. That is probably less risky than nursing the old SMF 1.x converter through two upgrade hops.

## 5. Plan

### Phase 0: now (this week)

1. **Renew the domain** for several years. Expires 2 Jan 2027. Confirm who holds the Tucows/registrar account.
2. **Full backup** of `cgi-bin/yabb/` and `yabbfiles/` (or the whole home directory) to somewhere off Positive Internet. Nothing else happens before this.
3. Get SFTP/SSH access (or ask Positive Internet to provide the tarball and the Apache error log).

### Phase 1: get admin working on the current install

1. Read the Apache error log for the `AdminIndex.pl` 500. Typical causes after a restore: lost execute bits on `.pl` files, wrong file ownership, a Perl module missing on the newer Debian, or a CGI config change on the host.
2. If it is one of those, fix it. The owner gets admin back within days, and the migration becomes a choice rather than an emergency.
3. Quick hardening while there: redirect HTTP to HTTPS, set the forum URL to `https://`, turn off directory listing, fix the `mial` MX typo.

### Phase 2: migrate to SMF 2.1.8

1. Copy the YaBB data locally into this repo's working area (data stays out of git).
2. Write and test an import script: categories, boards, members (with email and join date), topics, posts with original timestamps and authors, and a redirect map from `YaBB.pl?num=` and `board=` URLs to the new SMF URLs.
3. Convert YaBB markup (YaBB-code and smilies) to SMF BBCode. Re-encode ISO-8859-1 to UTF-8.
4. Stand up SMF 2.1.8 on a staging subdomain at Positive Internet, import, and get the owner and a couple of regulars to check the obituaries board in particular.
5. Decide the password approach: email every member a reset link at cutover. Write the email for a non-technical audience, since many members are retired.
6. Cutover: put YaBB in read-only, final import, switch the forum URL, keep old URLs redirecting so links in emails and search results still work.

### Phase 3: afterwards

1. Hand the owner a one-page admin guide and turn on SMF's update notifications.
2. Keep the final YaBB backup and a static HTML export as a permanent archive.

## 6. Open questions for the owner

1. Who controls the domain registrar account and the Positive Internet account?
2. Can we get SSH/SFTP access, or will Positive Internet supply files and logs?
3. What exactly did Positive Internet change when they restored the site?
4. Who else moderates, and do they need accounts on staging?
5. Is the membership-by-email process staying?
