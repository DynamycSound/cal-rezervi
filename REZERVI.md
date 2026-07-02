# Cal.diy – Termi.rs Edition (opciono / u rezervi)

> **STATUS (jul 2026):** Termi.rs (bivši Rezervi.rs) sada ima UGRAĐEN nativni
> kalendar slobodnih termina u glavnom repou (`rezervi-waas`) - mušterije biraju
> stvarne slobodne termine direktno na stranici. **Ovaj cal.diy fork zato NIJE
> potreban za osnovni proizvod** i nije deployovan. Čuva se za napredne
> slučajeve (timovi, Google Calendar sync itd.).
>
> Brend/domeni: rezervi.rs → **termi.rs**, cal.rezervi.rs → **cal.termi.rs**.
> Uputstva ispod su i dalje tačna, samo zameni domen.

# Cal.diy – Rezervi.rs Edition

> White-label booking engine za Rezervi.rs WaaS
> Fork: calcom/cal.diy → DynamycSound/cal-rezervi
> Licenca: MIT

## Šta je promenjeno za Rezervi.rs

- Branding: Rezervi.rs – boje #0f172a / #e11d48
- Domain: https://cal.rezervi.rs
- Embed dozvoljen sa: https://rezervi.rs, https://*.rezervi.rs
- Signup isključen – `NEXT_PUBLIC_DISABLE_SIGNUP=true`
  – naloge pravi Agency Admin ručno preko DB ili `/settings/admin`
- Seed korisnici (iz Rezervi.rs WaaS):
  - milan / zubar-ns@rezervi.rs
  - jelena / frizerka@rezervi.rs
  - mika / servis@rezervi.rs
  - luna / luna@rezervi.rs
  – lozinka se postavlja pri prvom login-u – ili setuj u DB
- Cal.diy API se koristi IZOLOVANO – Rezervi.rs sajt je 0kb JS Astro, samo iframe embed:
  ```
  <iframe src="https://cal.rezervi.rs/milan?embed=true&theme=light" ...>
  ```
- PayPal / klijent sajtovi / SEO – sve ostaje u Rezervi.rs WaaS repo-u:
  https://github.com/DynamycSound/rezervi-waas (private)

## Brzi deploy – Vercel (preporučeno – FREE)

Cal.diy je **zvanično podržan na Vercel-u**, NE na Cloudflare Pages bez teškog porta.

**Besplatne opcije – 0€/mesec:**
- Vercel Hobby – 0€
- Neon Postgres – Free tier – 0.5GB, 190 compute sati/mes – https://neon.tech – DOVOLJNO za start (10-20 klijenata)
- Supabase Postgres – Free – 500MB – takođe radi
- Railway – $5 – ako želiš više

**Koraci – Vercel – 15 min:**

1. Vercel → Add New Project → Import Git Repository
   → `DynamycSound/cal-rezervi`
   Framework: Next.js – auto detect
   Root Directory: `./` 
   Build Command: `cd ../.. && yarn build` – ili ostavi default – cal.diy ima turbo
   Install: `yarn install`

2. Vercel → Storage → Create Database → Postgres
   – Neon – Free – copy `DATABASE_URL`
   – ili: Supabase → copy connection string

3. Environment Variables – Vercel → Project → Settings → Environment Variables:

```
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=<openssl rand -base64 32>
CALENDSO_ENCRYPTION_KEY=<openssl rand -base64 24>
NEXTAUTH_URL=https://cal.rezervi.rs
NEXT_PUBLIC_WEBAPP_URL=https://cal.rezervi.rs
NEXT_PUBLIC_LICENSE_CONSENT=true
NEXT_PUBLIC_DISABLE_SIGNUP=true
# Email (opciono – Resend – free 3000/mes)
# EMAIL_SERVER_HOST=
# EMAIL_SERVER_USER=
# ...
```

4. Deploy – Vercel build ~8-12 min prvi put

5. DNS – Cloudflare:
```
cal  CNAME  cname.vercel-dns.com  Proxied (narandžasto)
```
   Vercel → Domains → Add `cal.rezervi.rs` – verifikuje automatski

6. Prvi login:
- idi na https://cal.rezervi.rs/auth/register – ako je `DISABLE_SIGNUP=true`, onda prvo napravi admina direktno u bazi:
```sql
-- preko Neon SQL editor
-- korisnici iz Rezervi.rs seed-a se ručno dodaju
```
  Ili privremeno stavi `NEXT_PUBLIC_DISABLE_SIGNUP=false`, registruj 4 korisnika (milan, jelena, mika, luna), pa vrati na `true`.

7. Povezivanje sa Rezervi.rs WaaS:
- Rezervi.rs Admin → klijent → polje `cal_link` = cal.diy username
  - zubar-ns → `milan`
  - frizerka-jelena → `jelena`
  - auto-servis-mika → `mika`
  - kozmeticki-luna → `luna`
- Na klijentskom sajtu se automatski pojavi:
  `<iframe src="https://cal.rezervi.rs/milan?embed=true">`

To je to – booking radi, Viber CTA ostaje primarni.

---

## Cloudflare – da li može cal.diy?

**Kratko: NE out-of-the-box. Moguće uz port – 3-5 dana.**

Zašto:
- cal.diy = Next.js 14 + Prisma + PostgreSQL
- Cloudflare Pages/Workers nema Node.js Prisma runtime
- Treba: Prisma → D1 adapter (experimental), NextAuth → Workers Edge, file uploads → R2, cron → Workers Cron
- Cal.com tim je zato zatvorio Cal.com kod – baš zbog security + infra kompleksnosti

**Ako baš hoćeš 100% Cloudflare – 0€:**
Preporuka autora Rezervi.rs WaaS-a:
Koristi **Rezervi Booking Lite** – već je skeleton u `rezervi-waas` repo-u:
- tabela `bookings` u D1
- `/api/booking-create` – 0kb JS
- Admin kalendar pregled
– mogu da ti uključim za 1 dan – bez cal.diy zavisnosti – 100% Cloudflare – 0€ zauvek

---

## Integracija: Rezervi.rs ↔ cal.diy

```
[rezervi.rs]  Astro 0kb JS – Cloudflare Pages
  |
  |-- /c/zubar-ns  →  iframe
  |                   ↓
  |            [cal.rezervi.rs] – cal.diy – Vercel + Neon Postgres (FREE)
  |                   ↑
  |-- admin dashboard → unosi cal_link = "milan"
  |
  |-- PayPal $25/mo → unlock
  |
  Viber CTA  ← primarna konverzija – 0kb JS – radi uvek
```

API veza – minimalna – namerno:
- Rezervi.rs NE poziva cal.diy API server-side – samo iframe embed
- Razlog: 0kb JS, 0 tajni na klijentskom sajtu, SEO čist
- Ako želiš API sync: cal.diy ima `/api/bookings` – može webhook → Rezervi.rs `/api/cal-webhook` – skeleton postoji

---

## Korisni linkovi

- Rezervi WaaS (glavni): https://github.com/DynamycSound/rezervi-waas (private)
- Cal fork: https://github.com/DynamycSound/cal-rezervi (upravo fork-ovano)
- Cal.diy upstream: https://github.com/calcom/cal.diy
- Demo klijenti:
  - zubar-ns.rezervi.rs
  - frizerka-jelena.rezervi.rs
  - kozmeticki-luna.rezervi.rs
- PayPal Plan: P-6S779054V07906302NJB55AA
- Vercel projekat: rezervi-waas – prj_TN6Oh5bpsPV1N5vMnUh9evUauhrT

---

*Rezervi.rs + cal.diy – by Stefan – Beograd – jun 2026*
*MIT – MIT – obe strane*
