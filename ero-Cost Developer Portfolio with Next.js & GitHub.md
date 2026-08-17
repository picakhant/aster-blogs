---
title: "Zero-Cost Developer Portfolio with Next.js & GitHub: A Student's Guide"
date: 2026-08-17
description: "How a CS student built a portfolio and blog at zero cost using Next.js, GitHub as a free CMS, and Vercel's free tier — including the full architecture and a step-by-step guide."
tags: [nextjs, github, portfolio, vercel, typescript, tutorial]
---

# Zero-Cost Developer Portfolio with Next.js & GitHub: A Student's Guide

> **Note for readers:** This post is available in both English and Myanmar (Burmese). Scroll down to the [Myanmar version](#မြန်မာဘာသာ) section at the end.

As a Computer Science student, one of the first things you'll hear from seniors and recruiters is: *"You need a portfolio."* And if you're like me, your next thought is: *"But I don't have money for a domain, a VPS, or a CMS."*

Good news — you don't need any of that.

This is the story of how I built [my personal portfolio](https://aster-dev.vercel.app) completely **at zero cost**, using tools I already had: Next.js, GitHub, and Vercel's free tier. No database. No CMS. No backend server. Just markdown files and a bit of smart architecture.

Let me break down the purpose, the cost breakdown, the architecture, and — most importantly — how **you** can use this exact project for yourself.

---

## 1. What This Project Is

This is a single-page application portfolio built with **Next.js (App Router)** and TypeScript. It has four main pages:

- **Home** — profile image, name, role, bio, tech stack cards, and a fake terminal window (because every developer loves a terminal aesthetic).
- **Projects** — automatically loads your public GitHub repositories and shows them as cards.
- **Blog** — renders posts straight from markdown files in a GitHub repository. Write → commit → live.
- **Contact** — a "Say Hello" email button, social links, and your location/timezone.

The whole thing is dark-themed (Dracula via daisyUI) with a theme toggle, framer-motion animations, and proper SEO metadata.

---

## 2. The Purpose: Why GitHub as a "Backend"?

The core idea is simple: **your GitHub account is your content management system.**

- **Projects page** → searches your repos with the `portfolio` topic.
- **Blog page** → reads every `*.md` file at the root of a blog repository (default: `aster-blogs`).

This means:

- **No database** — no MongoDB, no PostgreSQL, no free-tier limits to worry about.
- **No CMS admin panel** — editing content is a `git commit` away.
- **No rebuilds** — the site fetches fresh data at runtime with ISR revalidation, so pushing a new post goes live automatically after a short delay.
- **Version control for free** — your content history is your Git history.

For a student, this is huge. Your "website stack" collapses into tools you already use daily.

---

## 3. The Zero-Cost Breakdown for Students

Let's walk through every line item where a portfolio normally costs money, and what I use instead:

| Cost Item | Typical Cost | This Project |
|---|---|---|
| Hosting | $5–20/mo VPS | **Free** — Vercel Hobby plan |
| Domain | $10–15/yr | **Free** — `yourname.vercel.app` subdomain |
| Database | $10+/mo | **Free** — none needed, GitHub API is the data source |
| CMS | $10+/mo | **Free** — markdown files in a repo |
| Image hosting | varies | **Free** — `public/` folder or GitHub raw URLs |
| Auth/security | varies | **Free** — no user accounts; public data only |

### The only "secret" you might add (and it's free)

The `.env` file has one optional value: `GITHUB_TOKEN`. It's a GitHub personal access token used to raise your API rate limit from 60 to 5,000 requests per hour. It needs **no special permissions** because your repos are public — and the site works fine without it.

> Tip: Keep `.env.local` out of Git (the `.gitignore` already does this). Never commit tokens.

---

## 4. Architecture: What's Actually Going On

This is where the real learning value lives. The project is a clean example of modern Next.js patterns.

### Tech Stack

- **Next.js 16 (App Router)** — server components, dynamic routes, ISR
- **TypeScript** — types for every data shape
- **Tailwind CSS + daisyUI** — styling with the Dracula theme
- **Framer Motion** — scroll animations
- **react-markdown + remark-gfm + rehype-raw** — markdown rendering
- **gray-matter** — parses YAML frontmatter from blog posts

### The Layering

```
site.config.ts        ← ONE config file: your name, links, github settings
       ↓
lib/github.ts         ← data layer: fetch repos, check READMEs
lib/blog.ts           ← data layer: list + parse markdown posts
       ↓
app/                  ← pages (server components)
       ↓
components/           ← UI (cards, grid, markdown renderer)
```

Every piece of personal data lives in **one file**: `site.config.ts`. You never need to hunt through components to change your name, links, or GitHub username.

### How the data flows

1. **Projects**: `getPortfolioRepos()` calls the GitHub search API (`user:YOU+topic:portfolio`), then checks each repo for a README. Cards appear for repos with the `portfolio` topic.
2. **Project detail**: `[slug]/page.tsx` fetches the repo README and renders it with the markdown renderer.
3. **Blog list**: `getBlogPosts()` lists the contents of the blog repo, fetches each `.md` file from `raw.githubusercontent.com`, and parses YAML frontmatter (`title`, `date`, `description`, `tags`).
4. **Blog post**: `[slug]/page.tsx` fetches the single file and renders the content.

### The clever part: ISR (Incremental Static Regeneration)

Notice the `next: { revalidate }` option on every fetch:

```ts
const response = await fetch(API_URL, {
  headers: githubHeaders(token),
  next: { revalidate: REVALIDATE }, // 180s for projects, 600s for blog
});
```

This is the magic that makes a "static" site feel live:

- Pages are generated at **build time** (fast, SEO-friendly, cached).
- After `revalidate` seconds, the cache is revalidated in the **background** with fresh data.
- Detail pages use `generateStaticParams()` so `/blog/my-post` and `/projects/my-repo` are pre-built.

The result: **build-time speed with live-content freshness — and no server costs.** This is the pattern to learn and steal for your own projects.

---

## 5. How You Can Use This Project (Step by Step)

The project is intentionally built to be forked and customized. Here's exactly how to make it yours:

### Step 1 — Clone and install

```bash
git clone <your-fork-url>
cd profile
npm install
cp .env.example .env.local
```

### Step 2 — Edit ONE file: `site.config.ts`

Every field has a short comment. Change:

- Your name, role, bio, and profile image (drop the image into `public/`)
- Contact email, Telegram, location, timezone
- `github.username` → your GitHub username
- `blog.repoName` → the name of your blog repo (default `aster-blogs`)

That's it. You should not need to touch any other code.

### Step 3 — Show your projects

On GitHub, open each repo you want to display → **Settings → Topics** → add the `portfolio` topic. The site picks them up automatically.

- Add the `featured` topic to pin a project to the top.
- Remove the topic to hide a repo.

### Step 4 — Write your first blog post

Create a repo called `aster-blogs` (or whatever you set in config) and add a markdown file at its root:

```markdown
---
title: "My first post"
date: 2026-08-17
description: "A short summary shown on the blog card."
tags: [nextjs, github]
---

Write your post in **markdown** here.
```

- `title` and `date` are required; the rest is optional.
- Push the file → it appears on `/blog` after the refresh time (600s by default).
- Images must use absolute URLs (`![alt](https://example.com/img.png)`) since relative paths would point at your own domain.
- Delete the file to remove the post.

### Step 5 — Deploy for free

Push the code to your GitHub repo, then:

1. Go to [vercel.com](https://vercel.com) and sign in with GitHub.
2. **Add New → Project** → select this repo.
3. Vercel detects Next.js automatically and builds it.
4. Add `GITHUB_TOKEN` in **Settings → Environment Variables** (optional).
5. Deploy. Every push to `main` re-deploys automatically.

You now have a live portfolio at `yourname.vercel.app` — free forever on the Hobby plan.

---

## 6. Things I Learned Worth Stealing

A few patterns from this project that will level up your own work:

1. **Graceful empty states.** If you have no repos or posts, the page shows a helpful "how to add content" message instead of breaking.
2. **Fallbacks everywhere.** Every fetch is wrapped in try/catch and returns `[]` or `null` on failure — the site never crashes from an API hiccup.
3. **SEO out of the box.** Metadata, Open Graph, Twitter cards, `robots.ts`, and `sitemap.ts` are all wired up.
4. **A single source of truth.** `site.config.ts` centralizes all config, making the project beginner-friendly.
5. **Revalidate, don't rebuild.** ISR means you get static speed and dynamic content without paying for a server.

---

## 7. Conclusion

You don't need money to look professional online. With **Next.js + GitHub + Vercel**, a student can ship a real, maintainable, SEO-ready portfolio — and learn modern React patterns while doing it.

If you want the exact code, it's on GitHub, MIT-free to fork and customize. Go clone it, change the config, and ship your own site this week.

---

<a id="မြန်မာဘာသာ"></a>

## 🇲🇲 မြန်မာဘာသာ (Myanmar Version)

> **Note :** ဤဆောင်းပါးသည် Next.js၊ GitHub နှင့် Vercel တို့ကို အသုံးပြုပြီး ကုန်ကျစရိတ် တစ်ပြားမှ မကုန်ဘဲ ($0 Cost) Portfolio နှင့် Blog တည်ဆောက်နည်း နည်းပညာဆိုင်ရာ လမ်းညွှန် ဖြစ်ပါသည်။

ကွန်ပျူတာတက္ကသိုလ် ကျောင်းသားတစ်ယောက်အနေနဲ့ စီနီယာတွေနဲ့ အလုပ်ခေါ်သူတွေဆီက အမြဲကြားရလေ့ရှိတဲ့ စကားတစ်ခုကတော့ *"မင်းမှာ Portfolio ရှိဖို့ လိုတယ်"* ဆိုတာပါပဲ။ ဒါပေမဲ့ ငါတို့ ကျောင်းသားတွေ စဉ်းစားမိကြတာက *"ငါ့မှာ Domain ဝယ်ဖို့၊ VPS Hosting ဒါမှမဟုတ် CMS ဝယ်ဖို့ ပိုက်ဆံ မရှိဘူးလေ"* ဆိုတာပါပဲ။

သတင်းကောင်းကတော့ အဲဒါတွေ ဘာမှ ဝယ်စရာ မလိုပါဘူး။

ဒါဟာ ငါ့ရဲ့ [Personal Portfolio](https://aster-dev.vercel.app) ကို လက်ရှိ ရှိပြီးသား Tools တွေဖြစ်တဲ့ **Next.js, GitHub နဲ့ Vercel Free Tier** တို့ကိုပဲ သုံးပြီး **လုံးဝ $0 စရိတ်**နဲ့ ဘယ်လို တည်ဆောက်ခဲ့လဲဆိုတဲ့ အတွေ့အကြုံပဲ ဖြစ်ပါတယ်။ Database မလို၊ CMS မလို၊ Backend Server လည်း ဝယ်စရာ မလိုပါဘူး။ Markdown ဖိုင်လေးတွေနဲ့ စနစ်ကျတဲ့ Architecture အနည်းငယ်ပဲ လိုတာပါ။

ဒီ project ရဲ့ ရည်ရွယ်ချက်၊ စရိတ်စက၊ Architecture နဲ့ မင်းပါ ကိုယ်တိုင် ပြန်သုံးနိုင်မယ့် အဆင့်တွေကို အသေးစိတ် ရှင်းပြပေးသွားပါမယ်။

---

### ၁။ ဒီ Project က ဘာလဲ?

ဒါဟာ **Next.js (App Router)** နဲ့ TypeScript ကို သုံးပြီး ရေးသားထားတဲ့ Single-page Application Portfolio တစ်ခု ဖြစ်ပါတယ်။ အဓိက စာမျက်နှာ (၄) ခု ပါဝင်ပါတယ် -

- **Home** — Profile ဓာတ်ပုံ၊ နာမည်၊ Bio၊ Tech Stack Card များနှင့် Developer များ နှစ်သက်သည့် Fake Terminal Window။
- **Projects** — GitHub က Public Repositories တွေကို အလိုအလျောက် ဆွဲထုတ်ပြီး Card လေးများအဖြစ် ပြသပေးခြင်း။
- **Blog** — GitHub Repository ထဲက Markdown ဖိုင်များကို တိုက်ရိုက် Render လုပ်ပေးခြင်း (Write → Commit → Live)။
- **Contact** — "Say Hello" Email Button၊ Social Links များ၊ တည်နေရာနှင့် Timezone။

UI တစ်ခုလုံးကို daisyUI (Dracula theme) နဲ့ Framer Motion animation တွေ၊ SEO Metadata တွေ သေချာ ထည့်သွင်းထားပါတယ်။

---

### ၂။ ဘာကြောင့် GitHub ကို "Backend" အဖြစ် သုံးတာလဲ?

အဓိက အိုင်ဒီယာက ရိုးရှင်းပါတယ် - **မင်းရဲ့ GitHub အကောင့်က မင်းရဲ့ Content Management System (CMS) ဖြစ်ပါတယ်။**

- **Projects Page** → Repo တွေထဲမှာ `portfolio` ဆိုတဲ့ Topic ပါတဲ့ Repo တွေကို လိုက်ရှာပေးတယ်။
- **Blog Page** → Blog Repo (Default: `aster-blogs`) ရဲ့ Root ထဲမှာရှိတဲ့ `*.md` ဖိုင်တိုင်းကို လာဖတ်ပေးတယ်။

ဒါကြောင့် -

- **Database မလိုပါ** — MongoDB, PostgreSQL စတဲ့ Free-tier ကန့်သတ်ချက်တွေ စိတ်ပူစရာ မလိုတော့ဘူး။
- **CMS Admin Panel မလိုပါ** — Content တွေကို `git commit` လုပ်လိုက်ရုံနဲ့ ပြင်ပြီးသား ဖြစ်သွားမယ်။
- **Rebuild လုပ်စရာ မလိုပါ** — ISR (Incremental Static Regeneration) ကို သုံးထားလို့ Post အသစ် တင်လိုက်ရင် ခဏအကြာမှာ အလိုအလျောက် Live ဖြစ်သွားမယ်။
- **Version Control အခမဲ့ရမယ်** — Git History ကပဲ ကိုယ့်ရဲ့ Content History ဖြစ်သွားမယ်။

---

### ၃။ ကျောင်းသားများအတွက် $0 စရိတ် တွက်ချက်မှု

ပုံမှန် Portfolio တစ်ခုဆောက်ရင် ကုန်ကျနိုင်တဲ့ စရိတ်နဲ့ ငါတို့ အစားထိုး သုံးထားတာတွေကို ယှဉ်ပြထားပါတယ် -

| ကုန်ကျစရိတ် ခေါင်းစဉ် | ပုံမှန် ကုန်ကျစရိတ် | ဒီ Project မှာ သုံးထားသည်များ |
|---|---|---|
| Hosting | $5–20/လ (VPS) | **အခမဲ့** — Vercel Hobby plan |
| Domain | $10–15/နှစ် | **အခမဲ့** — `yourname.vercel.app` subdomain |
| Database | $10+/လ | **အခမဲ့** — မလိုပါ (GitHub API ကို Data source အဖြစ် သုံးသည်) |
| CMS | $10+/လ | **အခမဲ့** — Repo ထဲက Markdown ဖိုင်များ |
| Image Hosting | အပြောင်းအလဲရှိ | **အခမဲ့** — `public/` folder သို့မဟုတ် GitHub raw URLs |

> **Tip:** `.env.local` ထဲမှာ `GITHUB_TOKEN` ထည့်ထားရင် GitHub API ရဲ့ Request Limit ကို ၁ နာရီမှာ အကြိမ် ၆၀ ကနေ ၅,၀၀၀ အထိ တိုးမြှင့်ပေးနိုင်ပါတယ်။ (Public Data ဖြစ်လို့ Token မရှိလည်း အလုပ်လုပ်ပါတယ်)။

---

### ၄။ Architecture နှင့် အလုပ်လုပ်ပုံ

ဒီ Project ရဲ့ Architecture ကို အောက်ပါအတိုင်း အလွှာများ (Layers) ခွဲခြားထားပါတယ် -

```text
site.config.ts        ← တစ်ခုတည်းသော Config ဖိုင် (နာမည်၊ links၊ github settings)
↓
lib/github.ts         ← Data Layer (Repos များနှင့် README များကို ဆွဲထုတ်ခြင်း)
lib/blog.ts           ← Data Layer (Markdown post များကို ဖတ်ယူခြင်း)
↓
app/                  ← Pages (Server Components)
↓
components/           ← UI Components

```

#### Data Flow အလုပ်လုပ်ပုံ
1. **Projects:** `getPortfolioRepos()` က GitHub Search API ကို ခေါ်ယူပြီး `portfolio` topic ပါတဲ့ Repo များကို ဆွဲထုတ်ပေးတယ်။
2. **Blog List:** `getBlogPosts()` က Blog Repo ထဲက `.md` ဖိုင်များ၏ YAML Frontmatter (`title`, `date`, `description`) များကို ဖတ်ယူပေးတယ်။
3. **ISR (Incremental Static Regeneration):** Next.js ရဲ့ `next: { revalidate }` ကို သုံးထားလို့ Website က Static Page လို မြန်ဆန်နေပြီး နောက်ကွယ်မှာတော့ Data တွေကို အလိုအလျောက် Fresh ဖြစ်အောင် Update လုပ်ပေးနေပါတယ်။

---

### ၅။ ဒီ Project ကို မင်းကိုယ်တိုင် ဘယ်လို သုံးမလဲ? (Step by Step)

#### Step 1 — Clone နှင့် Install လုပ်ခြင်း
```bash
git clone <your-fork-url>
cd profile
npm install
cp .env.example .env.local
```

#### Step 2 — `site.config.ts` ဖိုင်တစ်ခုတည်း ပြင်ရန်

ဖိုင်ထဲမှာ ကိုယ့်ရဲ့ နာမည်၊ Bio၊ Telegram၊ GitHub Username နဲ့ Blog Repo Name တွေကို ပြောင်းပေးလိုက်ရုံပါပဲ။

#### Step 3 — Projects များ ပြသခြင်း

GitHub Repo ရဲ့ **Settings → Topics** မှာ `portfolio` ဆိုတဲ့ topic လေး ထည့်လိုက်ရုံနဲ့ Website မှာ အလိုအလျောက် လာပေါ်ပါလိမ့်မယ်။

#### Step 4 — Blog Post စရေးခြင်း

`aster-blogs` ဆိုတဲ့ Repo ဆောက်ပြီး Root ထဲမှာ `.md` ဖိုင် ပြုလုပ်ပါ:

```markdown
---
title: "ငါ့ရဲ့ ပထမဆုံး Post"
date: 2026-08-17
description: "Blog card မှာ ပြသပေးမယ့် အတိုချုပ် စာသား"
tags: [nextjs, github]
---

ဒီနေရာမှာ **Markdown** ဖြင့် စာစရေးပါ။
```

#### Step 5 — Vercel တွင် အခမဲ့ Deploy လုပ်ခြင်း

Code များကို GitHub သို့ Push တင်ပြီး [Vercel.com](https://vercel.com) တွင် Import လုပ်ကာ Deploy ပြုလုပ်ပါ။ `yourname.vercel.app` ဖြင့် အခမဲ့ Live အသုံးပြုနိုင်ပါပြီ။

---

### ၆။ သင်ယူရရှိခဲ့သော အဓိက အချက်များ

1. **Graceful Empty States:** Data မရှိသေးချိန်မှာ Error မတက်ဘဲ လမ်းညွှန်စာသားလေးတွေ ပြပေးထားခြင်း။
2. **Fallbacks Everywhere:** Fetch မရခဲ့ရင်လည်း `[]` သို့မဟုတ် `null` ပြန်ပေးလို့ Website လုံးဝ Crash မဖြစ်ခြင်း။
3. **A Single Source of Truth:** `site.config.ts` တစ်ခုတည်းနဲ့ Project တစ်ခုလုံးကို ထိန်းချုပ်နိုင်ခြင်း။
4. **ISR Pattern:** Server ဖိုး ကုန်စရာမလိုဘဲ Speed နဲ့ Dynamic Content နှစ်ခုလုံး ရရှိခြင်း။

---

### ၇။ နိဂုံး

ကျောင်းသားတစ်ယောက်အနေနဲ့ Professional ဆန်တဲ့ Portfolio ရဖို့ ပိုက်ဆံအများကြီး ကုန်စရာ မလိုပါဘူး။ **Next.js + GitHub + Vercel** အကူအညီနဲ့ Modern React Patterns တွေကိုလည်း သင်ယူရင်း ကိုယ်ပိုင် Website တစ်ခုကို ချက်ချင်း တည်ဆောက်နိုင်ပါတယ်။

Code တွေအကုန်လုံးကို GitHub မှာ MIT License ဖြင့် Open-source တင်ပေးထားတာမို့လို့ Fork လုပ်ပြီး ဒီတစ်ပတ်အတွင်း မင်းရဲ့ Portfolio ကို အကောင်အထည်ဖော်လိုက်ပါ!

---

*Written by Aster Julian Ray — CS Student at UCSPyay & Full-Stack Web Developer.*
