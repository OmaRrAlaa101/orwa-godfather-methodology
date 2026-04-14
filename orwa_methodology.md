# 🎯 Orwa Godfather — Full Bug Bounty Methodology, Mindset & GemeniFlow Integration
> **Sources**: ZomaCon 2026 Podcast (literal translation) · Bugcrowd Hacker Spotlight (2023) · SecurityStories Interview · Medium writeups · AMA session · bSides Odisha 2023 talk · LinkedIn methodology posts · Twitter/X tips

---

## 🧠 PART 1 — Mindset: How Orwa Thinks

### 1.1 The 70/10/20 Formula
This is Orwa's core framework for how he allocates mental energy on any engagement:

| Phase | Weight | Why |
|-------|--------|-----|
| **Recon** | 70% | The foundation of everything — without it, you find nothing |
| **Report writing** | 20% | Quality matters in bug bounty; a bad report = wasted bug |
| **Exploit** | 10% | Once you've found the right place, exploitation is often the easy part |

> *"The moment you are sure it's vulnerable to SQL Injection, here the Recon stage ends. The exploit is what comes after."*

Most hunters have this inverted — they obsess over exploitation techniques but spend almost no time on what actually leads them to the vulnerable endpoint. The exploit is 10% of the problem. Getting there is 70%.

**Bug Bounty reports ≠ Pentest reports**: Keep reports brief and concise. A pentest-style report submitted to a bug bounty program will likely be rejected — it's annoying for triage teams. "The best talk is that which is brief and concise."

---

### 1.2 What Recon Actually Means (Orwa's Expanded Definition)
Most people define Recon as: collect subdomains → filter live → run Nuclei. Orwa calls this the copy-paste approach — it doesn't even require hacker thinking, just watching a YouTube tutorial.

**Orwa's actual Recon scope**:
- Reading about the company (employees, services, tech stack, business logic)
- Subdomain enumeration + live filtering
- **Fuzzing** — Orwa explicitly puts fuzzing *inside* Recon, not exploitation
- Full port scanning + origin IP discovery
- Endpoint collection (URLScan, VirusTotal, Wayback Machine, Archive.org)
- GitHub/GitLab/Gist leak hunting
- Dorking (Google, Bing, DuckDuckGo, URLScan)
- Finding password reset tokens, activation keys, session tokens in indexed URLs
- Understanding the application by clicking every feature (help, support, purchase, settings)
- Monitoring for changes and updates over time

> *"Even clicking every tab — help, support, or purchase section — to understand the application, that's Recon."*

**The key insight**: For bugs like LFI, File Upload, or SQLi — getting to the vulnerable endpoint is Recon. The exploit is the small part at the end. Specialists in XSS or SSRF say the same thing: their main challenge is **reaching the location where they can test** — not the exploit itself.

---

### 1.3 Sustainable Intensity, Not Sprinting
Orwa hunts **5–6 hours per day**, full-time, without burning out. Fresh eyes find more bugs than exhausted ones.

**Critical habit**: After finding a bug, do **not** report immediately. Rest first, then write the report. This produces better reports and fewer mistakes.

---

### 1.4 Depth Over Breadth — Specialize and Monitor
The biggest mistake hunters make is feeling distracted by the huge number of programs available. Orwa's solution: deliberate specialization + long-term familiarity.

- **Pick 2–3 favorite targets** to "play with" deeply. Try new platforms only when bored.
- **Work the same company for years**: when you know an app deeply for a year, any tiny change is immediately obvious to you. Orwa has worked one company continuously since 2021.
- **Like your target**: If you love PlayStation, work PlayStation's program. The highest earners on that program are PlayStation fans — personal interest gives background knowledge that tools can't replicate. Same logic for car programs: Ford or Porsche hunters who are car enthusiasts find things others miss.
- **Collaborate**: Divide programs with trusted friends to go deep rather than wide.

> *"When you know an app for a year, any tiny change is obvious to you."*

---

### 1.5 The Monitoring Mindset — Where Big-Company Bugs Actually Come From
Large companies (Yahoo, PayPal, AT&T, Dell, Google, Meta, Tesla) have had bug bounty programs for 10–15+ years. Yet people still find High and Critical bugs in them constantly. How?

**The answer is: UPDATES. Not clever exploits — monitoring.**

Every time a large company deploys an update, something new is exposed. The hunter who finds bugs in big companies is the one who monitors daily and is in the right place at the right time.

> *"Bugs in large companies are found because something was updated. I focus on monitoring VirusTotal, URLScan, GitHub — you never know when an employee will commit code with credentials."*

**Real example — Orwa at Ubisoft**: He found AWS credentials on a Ubisoft main domain. It wasn't strategic genius — a JS file update caused an AWS bucket and its credentials to surface. He had tools and Burp extensions watching for new endpoints. After he reported it, 100 duplicates followed. He was first because he was monitoring.

**Real example — Ashraf Bassiouni at Epic Games**: Watched Orwa's Shodan monitoring videos, applied the technique to Epic Games, found an open RDP port that wasn't there the day before → got full access → Critical bug.

**Tactic: Re-test fixed bugs in adjacent assets**: If a bug was reported and fixed on `admin.corp.com`, immediately test `dev-admin.corp.com`, `staging-admin.corp.com`, `test-admin.corp.com`. The same vulnerability often exists in parallel environments.

---

### 1.6 Never Fear AI Agents
> *"AI agents are currently stupid. One agent I tested tried to run subdomain enumeration on a local GitHub repository."*

- AI follows training on old reports and templates — no "out of the box" thinking
- Success stories like "Axiom" are built by huge research teams with private exploits and templates — the AI is not the magic, the private data is
- What AI **cannot** do well: Broken Access Control, IDOR, Price Manipulation, response manipulation, blind vulnerabilities, timing attacks, OOB callbacks
- **Price manipulation example**: changing a price from Dollars to Rubles to pay less — AI doesn't understand currency semantics or business logic
- AI agents are doing the job of copy-paste hunters, just faster — human creativity still wins

> *"People feared WAFs and scanners, but the number of CVEs increases every year."*

Keep evolving alongside AI. Move with modern attack surfaces (GraphQL, AI agent apps, new frameworks) — don't just study old books.

---

### 1.7 Scope Expansion — Leaked Data Is Always In Scope
```
Program says in-scope:  www.target.com, blog.target.com
Orwa's mental scope:    *.target.* (any asset belonging to the company)
```
If a leak is found on an out-of-scope subdomain but the data belongs to the company, it is reportable. The data is always in scope even when the asset isn't explicitly listed.

---

### 1.8 The N/A and Duplicate Mindset
Duplicates are routing signals, not failures. They tell you which vulnerabilities are already known — route your next session toward less-tested attack surfaces. Accept N/As. Every one is a step closer.

---

## 🔍 PART 2 — Recon Methodology (7 Phases, Complete)

---

### Phase 1 — Subdomain Enumeration

**Tools**: `amass`, `httpx`, `naabu`, `ReconFTW`

```bash
# Passive subdomain enumeration
amass enum -passive -norecursive -noalts -df list-domains.txt -o subs.txt

# Probe for live hosts + tech detection
cat subs.txt | httpx -silent -status-code -title -tech-detect -o live_hosts.txt

# Port scan — exclude ports already covered by httpx
naabu -list live_hosts.txt -exclude-ports 80,443 -o ports.txt
```

**CRITICAL — Never filter out 4xx subdomains**: 95% of hunters exclude subdomains returning 4xx status codes. This is a massive wasted opportunity. A subdomain returning 403 or 404 often has hidden endpoints that only appear when fuzzed. Keep everything. Fuzz everything.

**Habit**: Check targets **2x per day** — new subdomains appear with every company update.

---

### Phase 2 — Full Port Scanning (All 65,535 Ports)

Most hunters scan only the top 1,000 ports. Developers know this, so they deliberately open services on high port numbers thinking no one will find them.

```bash
# Full port scan — all 65,535 ports
naabu -host {target} -p - -o {target}_allports.txt

# Or with nmap:
nmap -p- --open -T4 -iL live_hosts.txt -oN full_ports.txt
```

**Why this matters**:
- High ports often expose internal services with no WAF
- Developers open RDP, database ports, admin panels, API servers on non-standard ports thinking they're invisible
- Orwa found **4 Critical bugs at the same company** purely from full port scanning

**What to look for on non-standard ports**:
- Internal login portals (no authentication required — assumes "internal = trusted")
- Default credentials on exposed services
- Unprotected APIs
- Database interfaces (MongoDB, Elasticsearch, Redis exposed raw)
- RDP, VNC, SSH with weak/default credentials

---

### Phase 3 — Endpoint Collection + Multi-Source Monitoring (Daily)

**Sources**: `gau`, `waybackurls`, `urlscan.io`, `VirusTotal`, `Wayback Machine`, `Archive.org`

These sources are not for one-time recon — they must be **monitored daily**. Every day, new endpoints get indexed, new files get uploaded, new URLs get scanned. The hunter who bookmarks and checks daily finds bugs that one-time scanners miss entirely.

**urlscan.io workflow** — Orwa found a critical ATO using *nothing but urlscan*:
1. Search `page.domain:target.com`
2. Filter: **"Recently indexed"**
3. **Bookmark the tab**
4. Open every morning — new scans appear at the top
5. Flag: `token=`, `reset=`, `activate=`, `key=` parameters; admin paths; backup files

**VirusTotal workflow**:
1. Search for target domain in VirusTotal → check "Relations" tab
2. Monitor for newly submitted files or URLs — companies accidentally upload backups to VirusTotal
3. Watch for `.zip`, `.sql`, `.bak`, `.exe` files appearing — these are goldmines
4. New subdomains appearing in VirusTotal before active enumeration picks them up

**Wayback Machine / Archive.org**:
- Search for old endpoints that may still be live
- Password reset links, activation tokens, session tokens in archived URLs
- Open an archived token URL → if still valid → account takeover without any exploit

---

### Phase 4 — Multi-Engine Dorking

Orwa emphasizes **Bing and DuckDuckGo dorking** as systematically underused. Most hunters only use Google.

**Google**:
```
site:target.com
site:target.com ext:php OR ext:json OR ext:xml
site:target.com inurl:admin
```

**Bing** (underused edge):
```
site:https://www.target.com/
```

**URLScan**: Search for the target domain directly on urlscan.io — it indexes sensitive admin paths, API endpoints, and token URLs that don't appear in Google.

**GitHub — credential leak dork set**:
```
"target.com" password or secret
"target.atlassian" password
"target.okta" password
"corp.target" password
"jira.target" password
"target.onelogin" password
"target.service-now" password
"target" NOT www.target.com      ← reduces noise
```

**GitHub org dorks**:
```
org:targetorgname password NOT www.target.com
org:targetorgname secret
org:targetorgname https://
org:targetorgname host:
org:targetorgname internal
```

**GitHub user dorks** (after identifying an employee via LinkedIn):
```
user:employee_handle linkedin
user:employee_handle full name
user:employee_handle https://
user:employee_handle Ldap
```

**Pro tip**: Cross-reference GitHub users with LinkedIn. Many employees hide their employer on GitHub but not on LinkedIn. This confirms whether leaked data actually belongs to your target.

**Recommended tool**: GitDorker (400+ automated dorks)

---

### Phase 5 — Shodan/Fofa Infrastructure Monitoring (Change Detection)

**Don't just scan Shodan once — use it as a daily change-detection tool.**

```
# Shodan dorks for target
org:"Target Company Name"
ssl:"target.com"
hostname:"target.com"
```

**The monitoring workflow**:
1. Run Shodan dork on day 1 → note the IP count (e.g., 320 IPs)
2. Run the same dork next day → if 350 IPs appear → **investigate the 30 new ones immediately**
3. New IPs = new servers = likely no WAF = no hardening = high probability of bugs

> *"If you dork a company on Shodan and see 320 IPs, then 350 the next day — the skill is in following up. That's where 80–90% of bugs in big companies come from."*

**What to test on newly discovered IPs**:
- Default credentials on all exposed services
- Authentication bypass (services that assume "internal IP = trusted")
- Full port scan the new IP specifically
- Check for RDP, VNC, SSH, admin panels, databases

**Also use**: Fofa (indexes different assets than Shodan — often finds things Shodan misses entirely)

---

### Phase 6 — Sensitive Data Leak Hunting (Multi-Platform, Daily)

**gist.github.com** — systematically overlooked:
```
site:gist.github.com "target.com"
site:gist.github.com "target" password
```

**GitLab**:
```
search: "target.com" password
search: "target" api_key
```

**GitHub continuous monitoring**:
1. `github.com/search?q="target.com"&type=code`
2. Filter: **"Recently indexed"**
3. Bookmark → open every morning
4. Use NOT to filter noise: `"target.com" password NOT example NOT test`

**Tools**:
```bash
# Extract from leaked .git directory
./gitdumper.sh https://target.com/.git/ output/
./extractor.sh output/ extracted/

# Scan org repos for verified secrets
trufflehog github --org=targetorgname --only-verified
```

---

### Phase 7 — Fuzzing (Orwa's Full Framework)

Orwa treats fuzzing as **part of Recon**, not a separate exploitation phase. Fuzzing is how you reach locations that only a small number of hunters have ever accessed — directly reducing your duplicate rate.

**Who to fuzz**: Do NOT waste fuzzing on standard apps with open registration or well-documented APIs. Focus on:
- Subdomains returning **403** — something is there, you're just not allowed in yet
- Subdomains returning **404** — hidden routes may exist behind the surface
- Subdomains returning **301/302** — follow the redirect + fuzz the destination
- Subdomains returning **blank 200 OK** — highest priority: content exists but is hidden

> *"I once fuzzed a blank 200 OK page and found `/upload.php`."*

**What to look for**:
- Internal login portals
- Backup files: `.exe`, `.dll`, `.zip`, `.tar`, `.sql`, `.bak`
- `readme.md` / `README.md` — **AI-built apps frequently expose these with credentials and full endpoint documentation**
- `.env`, `.env.local`, `.env.production`

**The AI-app readme exploit**: Many companies now build internal tools with AI coding assistants. These tools leave `readme.md` files with endpoint documentation, API keys, and credentials. A researcher found Bugcrowd CTF flags purely by fuzzing for `readme.md` on an AI-generated app. This attack surface is growing every month.

**Wordlist customization by tech stack**:
```
PHP targets:    .php, .php3, .php5, .sql, .tar, .bak, .old
ASP.NET:        .aspx, .asmx, .ashx, .config, .cs, .bak
Java:           .jsp, .jspx, .do, .action, .xml, .properties
Generic paths:  /admin, /api, /app, /manage, /portal, /internal
Backup files:   .zip, .tar.gz, .sql, .dump, .backup, .exe, .dll
AI app files:   readme.md, README.md, .env, .env.local, docs/
```

**Customize Nuclei templates**: Add common directories like `/api`, `/admin`, `/app` to your existing Nuclei templates. A template checking `/admin` might miss `/app/admin` — always add path variations.

**Wordlists**:
- Random Robbie bruteforce lists — [github.com/random-robbie/bruteforce-lists](https://github.com/random-robbie/bruteforce-lists)
- fuzzuli for backup patterns — [github.com/musana/fuzzuli](https://github.com/musana/fuzzuli)
- Orwa's own lists — [github.com/orwagodfather/WordList](https://github.com/orwagodfather/WordList)

---

## 🔥 PART 3 — Exploitation Methodology

### 3.1 Bug Class Priority (Orwa's Ranked Model)

| Rank | Bug Class | Why |
|------|-----------|-----|
| 1 | **Information Disclosure** | Easiest to find, easiest to exploit, huge impact, 90% of Orwa's P1/P2s |
| 2 | **Zero-Days in Third-Party Software** | One zero-day affects every company using that software — massive ROI |
| 3 | **Authentication Bypass** | Critical severity — found via port scanning, IP testing, fuzzing |
| 4 | **IDOR** | High impact (P1/P2), logic-based so scanners miss it |
| 5 | **SQL Injection** | Server-side, high bounty, Orwa's signature class |
| 6 | **SSTI → RCE** | Highest reward, requires framework knowledge |
| 7 | **XSS** | Lower bounty but useful in chains |

**90% of Orwa's P1/P2 findings start with Information Disclosure.** It is his #1 because: low barrier to find, no exploit code needed, impact is often Critical (credentials, keys, PII, internal infrastructure), and it chains into every other bug class.

---

### 3.2 Information Disclosure — Primary Hunting Surface

Orwa's information disclosure hunting targets:
- **GitHub/Gist/GitLab**: Employee commits with credentials, API keys, internal URLs
- **Shodan/Fofa**: IPs with no WAF, exposed admin services, open ports with default credentials
- **VirusTotal**: Uploaded backup files, SQL dumps, configuration files  
- **URLScan**: Indexed admin paths, token URLs, session cookies in query strings
- **Default credentials**: `admin/admin`, `admin/password`, `admin/123456` on every admin panel
- **Full port scanning**: Services on non-standard ports with no authentication
- **AI-app artifacts**: `readme.md`, `.env` files left by AI code generators

---

### 3.3 SQL Injection — Orwa's Full Technique

**Step 1 — Surface collection in Burp**:
- Spider the full host
- Fuzz all directories for `.php` endpoints using a PHP wordlist
- Run Active Scan on **all POST requests**
- Setting: `Keep Maximum insertion points per base request: 10`

**Step 2 — Initial probe (error-based)**:
```sql
1'
```
MySQL error returns → injectable. Log it, move to time-based.

**Step 3 — Time-based payloads by framework**:
```sql
-- PHP (4 variants for different quote contexts):
(select(0)from(select(sleep(6)))v)
/*'+(select(0)from(select(sleep(6)))v)+'"+(select(0)from(select(sleep(6)))v)+"*/

-- ASPX / SQL Server:
orwa'; waitfor delay '0:0:6' --

-- GraphQL:
orwa') OR 11=(SELECT 11 FROM PG_SLEEP(6))--
```

**Step 4 — Non-standard surfaces** (where others don't look):
```
/sitemap.xml?offset=1;SELECT IF((8303>8302),SLEEP(9),2356)#
/robots.txt?{param}=1'
/feed?{param}=1'
/api/v*/search?q=1'
```

**Step 5 — SQLMap automation** (after manual confirmation):
```bash
sqlmap -r request.txt --level=5 --risk=3 --batch --dbs
```

**Step 6 — Chain it**: SQLi → credential extraction → admin panel access → ATO or RCE.

---

### 3.4 Authentication Bypass — Three Techniques

**Technique 1 — The 302 Body Trick**:
1. In Burp Suite: turn off "Follow Redirects"
2. Check `Content-Length` of the 302 response
3. If large (e.g. 6443 bytes) → full protected page content is in the redirect body
4. The app rendered the page then redirected — browser hides it, Burp shows the truth

**Technique 2 — IP-Based Bypass**:
- Internal-only services on non-standard ports often have no authentication
- Services on raw IPs (not behind a domain) skip WAF and often skip auth entirely
- Test discovered IPs directly at all open ports for admin panels that assume "internal = trusted"

**Technique 3 — Cross-Client Registration Bypass**:
- If a client/app has registration disabled, capture the registration POST request from a different client where registration IS enabled
- Replay that request against the client where registration is disabled
- The backend often doesn't enforce the restriction per-client
- Applies to any disabled feature — not just registration

---

### 3.5 Zero-Days in Third-Party Software — Orwa's #2 Bug Class

Find a vulnerability in a third-party product (CMS, framework, library, SaaS tool). Report it as a CVE. Then test every company using that software — all are potentially vulnerable.

**How to find companies using specific software**:
```bash
# Shodan
http.title:"Powered by X"
http.html:"generator=X" org:"Target Company"

# Google dorking
site:target.com intext:"Powered by X"
site:target.com "/x-admin/"
```

This is why Orwa has 10+ CVEs — one research effort scales to hundreds of companies.

---

### 3.6 Symfony/PHP Debug Panel — The $35,000 Chain

**Step 1**: Find `app_dev.php` in production:
```
https://target.com/app_dev.php/_profiler/empty/search/results?limit=100
```
→ Returns all session tokens, profiler logs, request history

**Step 2**: Read profiler logs → find leaked cookies, usernames, passwords

**Step 3**: Test LFI:
```
https://target.com/app_dev.php/_profiler/open?file=app/config/parameters.yml
```

**Step 4**: Chain leaked credentials + LFI → Full account takeover → $35,000

**Always check in production**:
```
app_dev.php          phpinfo.php
/.git/HEAD           /.env / .env.local
/debug/              /_profiler/
/server-status       /.DS_Store
/WEB-INF/web.xml     readme.md / README.md
```

---

### 3.7 Info Disclosure → Chain Building (Core Loop)

```
Info Disclosure → Credentials / Tokens / Endpoints / IPs
       ↓
Authentication Bypass / IDOR / SQLi / Default Creds
       ↓
Account Takeover / RCE / Privilege Escalation
       ↓
P1 Bounty
```

Information disclosure is never just informational. It is always the first link.

---

### 3.8 Nuclei Template Factory — One Bug, Many Bounties

> *"Create a special template for each vulnerability you discover and run it over nuclei on all programs."*

Every manually discovered bug → write a Nuclei template → run across entire program portfolio → one bug becomes many bounties.

**Customization tip**: Don't just use default templates. Add path variations (`/api/`, `/admin/`, `/app/`, `/v1/`, `/v2/`) to every template. Cover the same vulnerability at all common path prefixes.

---

## 🛠️ PART 4 — Complete Tool Stack

| Category | Tool | Purpose |
|----------|------|---------|
| Recon | `amass` | Subdomain enumeration |
| Recon | `httpx` | Live host probing + tech stack |
| Recon | `naabu` | Port scanning (use `-p -` for all 65,535) |
| Recon | ReconFTW | Multi-tool automation wrapper |
| Recon | Shodan | Infrastructure monitoring + daily change detection |
| Recon | Fofa | Alternative to Shodan, different index |
| Recon | URLScan.io | Daily endpoint monitoring (bookmark "recently indexed") |
| Recon | VirusTotal | Newly uploaded files + subdomain discovery |
| Recon | Wayback Machine | Historical endpoints, old tokens |
| Fuzzing | `ffuf` / `dirsearch` | Directory + endpoint discovery |
| Fuzzing | `fuzzuli` | Backup file fuzzing |
| Git Recon | `GitTools` | Git repo dump + extraction |
| Git Recon | GitDorker | 400+ automated GitHub dorks |
| Scanning | `nuclei` | Custom templates per bug class (add path variations) |
| Exploitation | Burp Suite | Interception, spidering, active scanning, JS secret detection via extensions |
| Exploitation | `sqlmap` | SQLi automation (after manual confirmation) |

**Wordlists** (all published by Orwa):
- [github.com/orwagodfather/WordList](https://github.com/orwagodfather/WordList)
- [github.com/orwagodfather/XSS-Payloads](https://github.com/orwagodfather/XSS-Payloads)
- [github.com/orwagodfather/SQL-Wordlist](https://github.com/orwagodfather/SQL-Wordlist)
- [github.com/orwagodfather/My-WordLISTs](https://github.com/orwagodfather/My-WordLISTs)

**Daily intelligence monitoring schedule**:

| Source | Frequency | What to flag |
|--------|-----------|-------------|
| urlscan.io | Daily (bookmarked, recently indexed) | New endpoints, token URLs, admin paths |
| GitHub | 2x daily (recently indexed) | New commits with credentials |
| VirusTotal | Daily | Newly uploaded backup files, new subdomains |
| gist.github.com | Daily | Employee pastes with secrets |
| Shodan/Fofa | Daily (IP count delta) | New IPs appearing for target org |
| GitLab | Weekly | Source code, configs |
| Twitter/X | Continuous | Community tips, new techniques |
| Bugcrowd LevelUp | Weekly | Program updates |

---


---

## 📋 PART 6 — Orwa's Rules: Quick Reference

```
MINDSET
✅ 70% Recon · 10% Exploit · 20% Report — this is your allocation
✅ Recon = everything before you are certain a bug exists
✅ Fuzzing IS Recon — it's how you reach the vulnerable endpoint
✅ 5-6 hours/day max — sustainable beats heroic
✅ Rest before reporting — never submit tired
✅ Keep 2-3 "favorite" targets, know them deeply for years
✅ Like your target — passion gives background knowledge tools can't replicate
✅ N/A and duplicates are routing signals, not failures
✅ Don't fear AI agents — they can't think out of the box
✅ Keep evolving: GraphQL, AI-app attack surfaces, modern frameworks

RECON
✅ NEVER filter out 4xx subdomains — fuzz every single one
✅ Blank 200 OK pages = highest fuzzing priority
✅ Scan ALL 65,535 ports — not just top 1,000
✅ Monitor URLScan + VirusTotal + GitHub 2x daily (bookmark "recently indexed")
✅ Monitor Shodan/Fofa for new IPs appearing for target org (track IP count delta)
✅ Use Bing + DuckDuckGo + URLScan dorking — not just Google
✅ Check gist.github.com daily — most hunters skip it entirely
✅ AI-built apps expose readme.md, .env, swagger.json — always fuzz for these
✅ Cross-reference GitHub users with LinkedIn for target confirmation
✅ Write a Nuclei template for every bug you find manually + add path variations
✅ Re-test fixed bugs in parallel environments (dev-, staging-, test-)

EXPLOITATION
✅ Information disclosure is #1 — 90% of P1/P2s start here
✅ Full port scan → test default creds on every non-standard port
✅ Never auto-follow redirects in Burp — check the 302 response body
✅ Test non-standard SQLi surfaces: sitemap.xml, robots.txt, feeds
✅ Try default credentials on every admin panel and every exposed service
✅ Check for app_dev.php, debug endpoints, .env in production
✅ Cross-client request replay to bypass disabled features
✅ Third-party zero-days scale to hundreds of companies — think multiply
✅ Leaked data is always in scope even when the asset isn't
```

---

## 🔗 References

| Resource | Link |
|----------|------|
| ZomaCon 2026 Podcast | ZomaCon 2026 — "Recon Mastering: Orwa Internals and recon hunting mindset in bugbounty" |
| Orwa's GitHub | [github.com/orwagodfather](https://github.com/orwagodfather) |
| Orwa's Medium | [orwaatyat.medium.com](https://orwaatyat.medium.com) |
| Orwa's Twitter/X | [x.com/GodfatherOrwa](https://x.com/GodfatherOrwa) |
| Bugcrowd Hacker Spotlight | [bugcrowd.com/blog/researcher-spotlight-orwagodfather](https://www.bugcrowd.com/blog/researcher-spotlight-orwagodfather/) |
| bSides Odisha 2023 Slides | [Google Slides](https://docs.google.com/presentation/d/1j8OWdRoN4xL2oikd7k2vdKhhLISSa9sz/edit) |
| Jhaddix methodology | [github.com/jhaddix](https://github.com/jhaddix) |
| zseano methodology | [zseano.com](https://zseano.com) |
| offsec.tools | [offsec.tools](https://offsec.tools/) |
| Random Robbie wordlists | [github.com/random-robbie/bruteforce-lists](https://github.com/random-robbie/bruteforce-lists) |
| fuzzuli | [github.com/musana/fuzzuli](https://github.com/musana/fuzzuli) |
