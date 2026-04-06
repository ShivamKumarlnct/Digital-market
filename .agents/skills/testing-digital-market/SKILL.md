# Testing Digital-market Frontend + Backend

## Overview
The Digital-market project is a static HTML/CSS/JS frontend connected to a FastAPI backend deployed on Fly.dev.

## Deployed URLs
- **Frontend:** Deployed via `deploy_frontend` tool (static site)
- **Backend:** Deployed on Fly.dev with persistent SQLite volume at `/data/app.db`
- **Backend API base:** Set in `script.js` as `API_BASE` constant

## Testing Approach

### Browser-based Form Testing
The site uses keyboard event handlers for navigation (keys like 'h', 'a', 'b', 'c' trigger page switches). This means **typing directly into form fields via computer use can trigger unintended page navigation**.

**Recommended approach:** Use Playwright via CDP (`http://localhost:29229`) to fill and submit forms:
```python
from playwright.async_api import async_playwright
browser = await p.chromium.connect_over_cdp("http://localhost:29229")
page = browser.contexts[0].pages[0]
await page.evaluate("showPage('contact')")
await page.fill('#contact-name', 'Test User')
await page.fill('#contact-email', 'test@example.com')
```

### Forms to Test
1. **Contact form** (Contact Us page): Fields `#contact-name`, `#contact-email`, `#contact-message`. Submit via `#contactForm button`. Success: green message in `#contact-status`.
2. **Newsletter form** (Blog page, scroll to bottom): Field `#newsletter-email`. Submit via `#newsletterForm button`. Success: green message in `#newsletter-status`. Duplicate emails show "You are already subscribed!".
3. **Blog comment form** (open a blog post first): Fields `#comment-name`, `#comment-email`, `#comment-website`, `#comment-text`. Success: green message in `#comment-status` + comment appears in `#comments-list`.

### Backend Admin Endpoints (no auth required)
- `GET /api/contacts` -- all contact form submissions
- `GET /api/subscribers` -- all newsletter subscribers
- `GET /api/comments/{post_index}` -- comments for a specific blog post
- `GET /healthz` -- health check

Use these to verify data was actually persisted after form submissions.

### Page Navigation
Use `showPage('home')`, `showPage('about')`, `showPage('blog')`, `showPage('contact')` via JS evaluate rather than clicking nav links to avoid timing issues.

To open a blog post: `openBlogPost(0)` (index-based).

## Chrome Setup
Chrome binary is at `/opt/.devin/chrome/chrome/linux-133.0.6943.126/chrome-linux64/chrome`. Start with:
```bash
DISPLAY=:0 /opt/.devin/chrome/chrome/linux-133.0.6943.126/chrome-linux64/chrome --no-first-run --disable-session-crashed-bubble --disable-infobars --start-maximized --remote-debugging-port=29229 "<URL>" &disown
```
The `google-chrome` wrapper command only works when Chrome is already running with CDP on port 29229.

## Devin Secrets Needed
None -- no authentication required for this project.
