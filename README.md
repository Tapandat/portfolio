# Tapan Datta — Portfolio (Streamlit)

Same custom-designed portfolio as the standalone HTML version, packaged to
run on [Streamlit Community Cloud](https://streamlit.io/cloud). The design
itself (dark ink/amber/teal theme, typed terminal hero, signal-trace
decoration) is rendered as a self-contained HTML/CSS/JS block inside the
app via `streamlit.components.v1.html`.

## Repo structure

```
.
├── streamlit_app.py    # entry point — the ENTIRE app, page design included
├── requirements.txt    # just streamlit
└── .streamlit/
    └── config.toml     # dark theme
```

Only two files matter: `streamlit_app.py` and `.streamlit/config.toml`.
There is deliberately no separate HTML/assets file — see below.

### Why everything lives in one file

An earlier version kept the page design in `assets/portfolio.html` and had
`streamlit_app.py` read it from disk at runtime. On Streamlit Community
Cloud that file didn't make it into the deployed repo (lost during a
GitHub upload), which crashed the app with `FileNotFoundError` before it
could render anything.

To make that class of failure impossible, the whole page — HTML, CSS, JS,
and all 8 certificate PDFs (embedded as base64 `data:` URIs) — now lives
inline as one large string inside `streamlit_app.py`. There's nothing
else to forget to push. The file is large (~4.5MB, mostly the embedded
certificates) but that's normal for a single Python file and causes no
problems for GitHub or Streamlit Cloud.

### Navigation model

The page works as four simple "pages" toggled with plain JS `display`
swaps, rather than one long scroll with anchor links (anchors don't
reliably move the outer Streamlit page from inside the embedded iframe):

- **Home** — hero, about, stack, publications, education. Shown by default.
- **Projects**, **Experience**, **Contact** — each its own page, opened via
  the Home page's three buttons (View builds / Experience / Contact).

Each non-Home page shows a **"← Back to home"** button at the top — it
only exists inside those three page containers, so it's never visible on
Home.

### Certificates are embedded, not statically served

Every certificate PDF is embedded directly as a base64 `data:` URI rather
than served from a `static/` folder. This was a deliberate fix for a
separate, unresolved Streamlit bug where relative
`app/static/...` links don't reliably resolve inside
`st.components.v1.html` specifically (streamlit/streamlit#12432). Each
"View certificate ↗" link now opens the PDF straight from the page, with
no dependency on how or where the app is hosted.

### Why the iframe height is a fixed number

`st.components.v1.html` can't auto-fit its iframe to the embedded page's
real content height — that ability (`Streamlit.setFrameHeight()`) only
exists for fully declared custom components, not this plain HTML
renderer. `streamlit_app.py` uses a fixed `height=2500` (tall enough for
the Home page, the tallest of the four) plus `scrolling=True`, so you
always get a normal, working scrollbar. The tradeoff: shorter pages
(Projects/Experience/Contact) show some empty space below them. If you
add a lot more content to Home later, bump that number up.

## Deploy it

1. **Create a GitHub repo** and push everything in this folder to it —
   just `streamlit_app.py`, `requirements.txt`, and `.streamlit/config.toml`
   at the repo root. Since it's only those three files/folders now, a
   drag-and-drop upload on github.com works fine too (no nested asset
   folder to lose).

   ```bash
   git init
   git add .
   git commit -m "Portfolio site"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```

2. **Go to** [share.streamlit.io](https://share.streamlit.io) and sign in
   with GitHub.
3. Click **New app**, pick the repo/branch, and set the main file path to
   `streamlit_app.py`.
4. Deploy. Streamlit Cloud installs `requirements.txt` automatically and
   picks up `.streamlit/config.toml` on its own.

Your app will be live at `https://<something>.streamlit.app`.

## Local test (optional)

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

## Updating content later

- **Text/projects/links**: open `streamlit_app.py` and edit the
  `PORTFOLIO_HTML` string directly — it's plain HTML/CSS/JS between the
  `r"""` and `"""` markers.
- **New certificate**: base64-encode the PDF and add it as a
  `data:application/pdf;base64,...` href wherever it belongs:

  ```bash
  base64 -w0 your-file.pdf   # macOS: base64 your-file.pdf | tr -d '\n'
  ```

- **Page height**: adjust `height=2500` at the bottom of
  `streamlit_app.py` if Home's content grows or shrinks a lot.
