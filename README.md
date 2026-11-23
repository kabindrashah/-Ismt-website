# 🎓 ISMT College Website (Demo Project)

### A college website taken all the way from plain HTML to an automated, monitored, production-style deployment.

`HTML5` `Docker` `Kubernetes` `ArgoCD` `Grafana`

> This is my own demo project. It is not the official ISMT website.

---

## 📖 What is this project?

This started as a simple, responsive website for a college: plain
HTML, CSS, and JavaScript, built by me (jQuery is the only library). That part is done and it
works well on its own.

But the real goal of this project was bigger: **take an ordinary
website and run it through the same process real companies use
to ship and run software in production** — build it, test it
automatically, package it, deploy it to a live cluster, and watch over
it while it runs.

So this repo tells two stories at once:
1. **A website** — pages, styling, a working contact form, a
   responsive layout.
2. **A pipeline** — everything that happens *around* that website to
   turn it into something reliably running on a cluster.

---

## 🌐 The Website

| Page | File | What it's for |
|---|---|---|
| Home | `index.html` | Hero banner, highlights, featured courses |
| About | `about.html` | Institute background and mission |
| Courses | `courses.html` | Full list of available courses |
| Course Detail | `course-inner.html` | Deep-dive on one course |
| Blog | `blog.html` | List of articles |
| Post | `post.html` | A single blog post |
| Contact | `contact.html` | Contact form and details |

**Highlights:**
- Fully responsive, mobile-friendly navigation
- Smooth fade-in animations as you scroll
- Sticky navbar, back-to-top button, live countdown timer
- Client-side form validation with helpful inline feedback
- Alt text on images and SEO-friendly page titles

**Screenshots:**

![Homepage](screenshots/Homepage.png)
*Homepage*

![About](screenshots/About.png)
*About*

![Courses](screenshots/Courses.png)
*Courses*

![Blog](screenshots/Blog.png)
*Blog*

![Contact](screenshots/Contact.png)
*Contact*

**Run it the simple way** (just the website, no DevOps needed):
```bash
git clone https://github.com/kabindrashah/-Ismt-website.git
# Open index.html with VS Code's "Live Server" extension
```

---

## ⚙️ The Pipeline: How This Site Gets Deployed

```
 You push code to GitHub
          │
          ▼
 GitHub Actions runs automatically:
   ✔ checks the HTML/CSS
   ✔ scans for accidentally leaked secrets
   ✔ builds a Docker image and scans it for vulnerabilities
          │
          ▼
 ArgoCD notices the change in GitHub
   and deploys it — automatically, no one
   has to run a deploy command by hand
          │
          ▼
 The website runs inside Kubernetes
   with 2 copies running side by side
   (if one crashes, the other keeps serving traffic)
          │
          ▼
 Prometheus + Grafana watch it,
   tracking traffic and confirming it is healthy
```

In plain terms: **push code → it is automatically checked, built,
deployed, and monitored, with no manual steps in between.**

### 🐳 1. Docker — packaging the site

The website is served by **Nginx**, a small, fast web server, all
bundled into one portable container:

```bash
docker build -t ismt-website .
docker run -d -p 8081:80 ismt-website
```

### 🔄 2. CI/CD — automatic checks on every push

A GitHub Actions pipeline (`.github/workflows/ci.yml`) runs three
checks every time code is pushed:
- **HTML validation** — reports markup problems (currently informational)
- **Secret scanning** (Gitleaks) — catches accidentally committed
  passwords or keys
- **Vulnerability scanning** (Trivy) — scans the built Docker image
  for known security issues

**A practical decision:** the HTML validator reports some markup issues in my pages. I treat validation as informational for now so it does not block deployments, and fixing those issues is on my list.

### ☸️ 3. Kubernetes — running it reliably

The site runs as **2 replicas** in Kubernetes, with health checks that
automatically restart it if something goes wrong.

### 📦 4. Helm — packaging the deployment config

Instead of hand-editing YAML files, the Kubernetes setup is packaged
as a **Helm chart** (`helm/ismt-website/`) — one command deploys it,
and settings like replica count or image version can be changed
without touching the underlying templates.

```bash
helm install ismt-website helm/ismt-website
```

### 🔁 5. GitOps with ArgoCD — deployments that happen by themselves

**ArgoCD** watches this GitHub repo. When something changes, it
updates the live deployment automatically — no one needs to run
`kubectl` or `helm` by hand. Git *is* the source of truth.

### 📊 6. Monitoring — watching it while it runs

A static website doesn't normally expose any data about its own
health. To fix that, each pod runs a small **exporter** sidecar
alongside Nginx, which reads Nginx's internal stats and turns them
into data Prometheus can understand. Prometheus then finds these pods
automatically (no manual setup per pod), and Grafana turns the numbers
into a live dashboard:

![Website Dashboard](screenshots/website-dashboard.png)

- **Request Rate** — how much traffic each pod is handling right now
- **Uptime** — whether each pod is healthy (green line = good)

---

## 🧠 What I learned building this

- How to take a plain static site and containerize it properly
- How CI/CD pipelines catch problems *before* they reach production
- How Helm turns repetitive YAML into something clean and reusable
- How GitOps means "Git is in charge," not a person manually deploying
- How to monitor something (like Nginx) that doesn't naturally speak
  Prometheus's language, using a sidecar exporter
- Real debugging: a Kubernetes permission error that needed a specific
  fix (RBAC), a PowerShell quoting issue that broke a command until I
  used a patch file instead, and a password reset after a tool's
  command name changed between versions
- That real engineering sometimes means making a practical trade-off,
  such as treating HTML validation as informational while I fix the
  markup issues

---

## 🛠️ Tech Stack

**Website:** HTML5 · CSS3 · JavaScript · jQuery
**DevOps:** Docker · GitHub Actions · Gitleaks · Trivy · Kubernetes ·
Helm · ArgoCD · Prometheus · Grafana

---

## 🚀 What's Next

- Fix the HTML validation errors and make validation a blocking step
- Add alerting so I get notified automatically if the site goes down
