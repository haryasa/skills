---
name: article-reader
description: Read and analyze articles from URLs or pasted content. Produces a structured brief with TL;DR, key takeaways, stats/data, actionable advice, notable quotes, recommended references, and an honest read recommendation (SKIP / SKIM / READ FULLY). Use this skill whenever a user shares a link, pastes article text, or asks anything like "what's this about?", "is this worth reading?", "summarize this for me", "what are the main points here?" — even if they don't explicitly ask for a summary. Also trigger when a user shares a URL and asks a follow-up question about its content.
---

# Article Reader

When a user shares a URL or pastes article content, analyze it and produce a structured brief that captures what matters, rendered as a styled HTML page opened in the browser.

## Step 1: Get the content

- **URL given**: Use WebFetch to retrieve the article. Focus on the main body text; discard navigation, ads, related-article links, and footers.
- **Content pasted**: Use it directly — don't fetch anything.
- **Partial or paywalled content**: Note it briefly ("Note: only a preview was available") then proceed with what's accessible.
- **Very short article (< 300 words)**: Note that it's a quick read before the verdict.

## Step 2: Compose the brief

Gather all the content for the brief (hold it in context — don't output it to the terminal yet):

- **Title** — the article's actual title
- **Publication/domain and estimated read time** (e.g., "The Atlantic · ~8 min read")
- **TL;DR** — 2–3 sentences: the core argument or finding, and why it matters
- **Key Takeaways** — 3–6 specific bullet points (prefer concrete facts over vague descriptions)
- **Important Info** — include only the subsections that genuinely apply:
  - *Stats & Data*: specific numbers, percentages, studies cited
  - *Actionable Advice*: concrete things the reader can do or apply
  - *Notable Quotes*: only sharp, memorable lines — don't pad with filler
  - *Recommended References*: books, papers, tools, people mentioned worth exploring
- **Read Recommendation** — one of SKIP / SKIM / READ FULLY, plus 1–2 honest sentences:
  - **SKIP** — the summary captures everything; the full article adds nothing new
  - **SKIM** — specific sections are worth reading; name them
  - **READ FULLY** — depth, nuance, prose, or narrative structure genuinely adds value a summary can't

## Step 3: Render to styled HTML

Write the brief as a self-contained HTML file to `$HOME/tmp/article-<slug>.html` (where `<slug>` is the first 4–5 words of the title, lowercased, hyphenated). Create the directory first if it doesn't exist (`mkdir -p $HOME/tmp`). Use the Write tool for the file.

Use this exact HTML structure and CSS — fill in the content, replace placeholder comments, and adapt the verdict class (`skip`, `skim`, or `read`):

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><!-- Article title --></title>
  <style>
    :root {
      --bg: #fafaf8;
      --text: #1a1a1a;
      --muted: #6b7280;
      --border: #e5e7eb;
      --accent: #2563eb;
      --skip-bg: #fee2e2; --skip-fg: #991b1b;
      --skim-bg: #fef3c7; --skim-fg: #92400e;
      --read-bg: #dcfce7; --read-fg: #166534;
    }
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: Georgia, 'Times New Roman', serif;
      background: var(--bg);
      color: var(--text);
      line-height: 1.75;
      padding: 3rem 1.25rem;
    }
    article { max-width: 740px; margin: 0 auto; }
    h1 { font-size: 1.9rem; line-height: 1.25; margin-bottom: 0.5rem; }
    .meta {
      color: var(--muted);
      font-size: 0.9rem;
      font-style: italic;
      margin-bottom: 2.25rem;
      padding-bottom: 1.5rem;
      border-bottom: 1px solid var(--border);
    }
    .section-label {
      font-family: system-ui, -apple-system, sans-serif;
      font-size: 0.75rem;
      font-weight: 700;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--muted);
      margin: 2rem 0 0.75rem;
    }
    .tldr {
      background: #eff6ff;
      border-left: 3px solid var(--accent);
      padding: 0.9rem 1.2rem;
      border-radius: 0 6px 6px 0;
      font-size: 1.02rem;
    }
    ul { padding-left: 1.35rem; }
    ul li { margin-bottom: 0.45rem; }
    .subsection-label {
      font-family: system-ui, -apple-system, sans-serif;
      font-size: 0.85rem;
      font-weight: 600;
      color: var(--muted);
      margin: 1.25rem 0 0.4rem;
    }
    blockquote {
      border-left: 3px solid var(--border);
      padding: 0.4rem 1rem;
      color: var(--muted);
      font-style: italic;
      margin: 0.4rem 0;
    }
    hr { border: none; border-top: 1px solid var(--border); margin: 2.25rem 0; }
    .verdict-badge {
      display: inline-block;
      padding: 0.35rem 1rem;
      border-radius: 999px;
      font-family: system-ui, -apple-system, sans-serif;
      font-size: 0.85rem;
      font-weight: 700;
      letter-spacing: 0.03em;
      margin-bottom: 0.6rem;
    }
    .verdict-badge.skip { background: var(--skip-bg); color: var(--skip-fg); }
    .verdict-badge.skim { background: var(--skim-bg); color: var(--skim-fg); }
    .verdict-badge.read { background: var(--read-bg); color: var(--read-fg); }
    .verdict-note { color: var(--muted); font-size: 0.97rem; }
  </style>
</head>
<body>
<article>
  <h1><!-- Title --></h1>
  <p class="meta"><!-- Publication · ~N min read --></p>

  <p class="section-label">TL;DR</p>
  <div class="tldr"><!-- 2–3 sentence summary --></div>

  <p class="section-label">Key Takeaways</p>
  <ul>
    <!-- <li>Each takeaway</li> -->
  </ul>

  <!-- Repeat the block below for each subsection that has content.
       Omit subsections with nothing to say. -->

  <p class="section-label">Important Info</p>

  <!-- Stats & Data (omit if none) -->
  <p class="subsection-label">Stats &amp; Data</p>
  <ul>
    <!-- <li>stat</li> -->
  </ul>

  <!-- Actionable Advice (omit if none) -->
  <p class="subsection-label">Actionable Advice</p>
  <ul>
    <!-- <li>advice</li> -->
  </ul>

  <!-- Notable Quotes (omit if none) -->
  <p class="subsection-label">Notable Quotes</p>
  <!-- <blockquote>"Quote text" — Attribution</blockquote> -->

  <!-- Recommended References (omit if none) -->
  <p class="subsection-label">Recommended References</p>
  <ul>
    <!-- <li>reference</li> -->
  </ul>

  <hr>

  <p class="section-label">Read Recommendation</p>
  <!-- Set class to: skip | skim | read -->
  <div class="verdict-badge skip">SKIP</div>
  <p class="verdict-note"><!-- 1–2 honest sentences --></p>

</article>
</body>
</html>
```

## Step 4: Open in the browser

After writing the file, open it using Bash:

```bash
xdg-open "$HOME/tmp/article-<slug>.html" 2>/dev/null || open "$HOME/tmp/article-<slug>.html" 2>/dev/null
```

Then tell the user the file path in one short line, e.g.: `Opened: ~/tmp/article-why-sleep-matters.html`

---

## Quality bar

- Be concrete and specific throughout. Vague bullets like "it discusses productivity" are useless.
- Only include Notable Quotes that are genuinely sharp — don't pad with filler lines.
- The read recommendation should be honest. Most well-written summaries make the original skippable. That's fine — say so.
- If the article is opinion/narrative (memoir, essay, literary journalism), lean toward SKIM or READ FULLY since the prose itself is part of the value.
- Keep the terminal output minimal — the HTML is the deliverable.
