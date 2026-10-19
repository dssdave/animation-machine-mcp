<p align="center"><img src="icon.png" width="96" alt="Animation Machine"></p>

<h1 align="center">Animation Machine MCP Server</h1>

<p align="center"><b>Give Claude, ChatGPT, Cursor and other AI apps a render button.</b><br>
Your AI writes an HTML animation; Animation Machine renders it to MP4 or GIF with music, voiceover and captions, in every aspect ratio.</p>

<p align="center">
<a href="https://animation-machine.com/mcp-server">Website</a> ·
<a href="https://animation-machine.com/docs/mcp">Full docs</a> ·
<a href="#setup">Setup</a> ·
<a href="#tools">Tools</a> ·
<a href="#example-prompts">Example prompts</a> ·
<a href="https://animation-machine.com/make-videos-with-claude">Guide: videos with Claude</a>
</p>

---

**Server URL:** `https://animation-machine.com/mcp`
**Transport:** Streamable HTTP (remote, hosted, nothing to install)
**Auth:** OAuth 2.1 (dynamic client registration or client ID metadata documents, PKCE) or a personal API key
**Pricing:** included in the [$5/month Animation Machine plan](https://animation-machine.com/pricing)

This repository holds documentation, examples and the MCP Registry manifest. The server itself is hosted by Animation Machine.

**Full reference** (every tool and parameter, OAuth endpoints, limits, errors): [animation-machine.com/docs/mcp](https://animation-machine.com/docs/mcp) · as Markdown for agents: [docs/mcp.md](https://animation-machine.com/docs/mcp.md) · server card: [/.well-known/mcp/server-card.json](https://animation-machine.com/.well-known/mcp/server-card.json)

## What it does

AI assistants are great at writing animation code (GSAP, CSS keyframes, Web Animations, Lottie, Three.js, canvas) but can't play it back and record it. This server lets them:

- **Render** an HTML animation (inline HTML, a multi-file project, or an uploaded zip) to **MP4 or GIF**, in **16:9, 9:16 and 1:1** from one request
- **Finish a video** you upload: royalty-free **music**, AI **voiceover**, burned-in **captions** (16 languages), title and end cards, a corner handle, your **logo**
- **Preview** a single frame first, so the AI can check its own work
- Use **motion-graphics templates** (title card, lower third, quote, stat counter) with no code
- Write a social **Post Kit** (caption, hooks, hashtags) for the finished video

It renders *code*, frame by frame in a real browser, so text, numbers, colors and timing come out exactly as written. It is not a text-to-video model.

## Setup

### claude.ai, Claude Desktop, Claude mobile
1. **Settings → Connectors → Add custom connector**
2. Name: `Animation Machine`, URL: `https://animation-machine.com/mcp`
3. **Connect**, sign in to Animation Machine, **Allow**

### Claude Code
```bash
claude mcp add --transport http animation-machine https://animation-machine.com/mcp
```
Then run `/mcp` → **Authenticate**.

### ChatGPT
Settings → Apps & Connectors → Advanced → enable **Developer mode** → **Create** → URL `https://animation-machine.com/mcp`, authentication **OAuth** → sign in and Allow.

### Cursor (`~/.cursor/mcp.json`)
```json
{
  "mcpServers": {
    "animation-machine": {
      "url": "https://animation-machine.com/mcp",
      "headers": { "Authorization": "Bearer am_live_…" }
    }
  }
}
```

### VS Code (`.vscode/mcp.json`)
```json
{
  "servers": {
    "animation-machine": {
      "type": "http",
      "url": "https://animation-machine.com/mcp",
      "headers": { "Authorization": "Bearer am_live_…" }
    }
  }
}
```

### Anything else
Any client that speaks remote MCP over Streamable HTTP works. stdio-only clients can bridge with:
```bash
npx mcp-remote https://animation-machine.com/mcp --header "Authorization: Bearer am_live_…"
```

**API keys:** sign in at [animation-machine.com](https://animation-machine.com/app.html) → account menu → **AI agents (MCP)** → Create API key. Keys can be revoked there at any time, and connected OAuth apps can be disconnected.

## Tools

| Tool | What it does | Hints |
|---|---|---|
| `render` | Render HTML / a files map / an uploaded zip, or finish an uploaded video. Options: `aspects`, `resolution`, `format`, `fps`, `duration_seconds`, `loop_count`, `crop_to_fill`, `music_url`, `music_volume`, `voiceover_text`, `voiceover_voice`, `captions`, `caption_style`, `caption_language`, `text_overlay`, `intro_title`, `outro_title`, `logo_upload_id`. Returns `job_ids`. | write |
| `render_template` | `title_card`, `lower_third`, `quote` or `stat` from a few fields, plus the same output options. | write |
| `preview_frame` | One still frame (image) at `at_seconds` plus the detected animation length. | read-only |
| `get_job` | Status, progress, then `download_url` (24 h), `watch_url` (share page), `studio_url` (open in the editor). | read-only |
| `cancel_job` | Cancel a queued or running render. | destructive |
| `create_upload` | One-time `PUT` URL for a local zip / video / logo. | write |
| `search_music` | Search 500,000 royalty-free CC0 tracks. | read-only |
| `list_voices` | Voices for `voiceover_text`. | read-only |
| `post_kit` | Caption, hooks, hashtags and description for a finished render. | read-only |
| `list_renders` | Your 10 most recent renders. | read-only |
| `get_account` | Plan, usage, job in progress, limits. | read-only |

Typical flow: write the animation → `preview_frame` → `render` → poll `get_job` (it returns `retry_after_sec`) → share `download_url`.

## Example prompts

- *"Make a 5-second animated intro for my YouTube channel 'Build Notes': dark navy, the title slides in letter by letter. Render it as an MP4."*
- *"Turn this launch announcement into a 10-second animated video and render it 16:9, 9:16 and 1:1 with upbeat royalty-free music."*
- *"Here are our Q3 numbers. Make a 15-second animated bar chart and render it for LinkedIn (1:1)."*
- *"I'll upload demo.mp4. Add calm music, captions in Spanish, my @handle in the corner and an end card that says 'Try it free'."*
- *"Use the stat template counting up to 10,000 with the label 'happy customers', then write a Post Kit for Instagram."*

More in [`examples/`](examples/).

## Limits

Built to stay fast for everyone on a shared render server:

- One job (render or preview) at a time per account; agents queue behind people using the website
- HTML animations ≤ 60 s, GIFs ≤ 15 s, uploaded videos ≤ 3 min (loops included)
- Up to 1080p, 24 or 30 fps, up to 3 aspect ratios per request
- Inline HTML ≤ 2 MB, multi-file projects ≤ 5 MB, uploads ≤ 95 MB
- 200 renders a month (shared with the web app), 5 AI voiceovers and 20 Post Kits a day via agents

Need 4K or longer videos? Every render has a `studio_url` to keep editing in the browser.

## Security & privacy

- OAuth 2.1 with PKCE (S256); tokens are stored hashed and can be revoked from the account menu
- Connected apps can render, read their own renders and search music, nothing else: no billing, no account changes, no other users' data
- Animations render in a sandboxed headless browser behind an egress filter: no access to private networks or cloud metadata
- Download links are HMAC-signed and expire after 24 hours; renders are deleted after 32 days
- Uploaded files and generated media are used only to produce your video. Music comes from Freesound (CC0); AI voiceover and Post Kit use Google Gemini; captions use a self-hosted Whisper model.
- Full policy: [animation-machine.com/privacy](https://animation-machine.com/privacy) · Terms: [animation-machine.com/terms](https://animation-machine.com/terms)

## Support

support@getcashflow.co · [animation-machine.com](https://animation-machine.com)

© Cash Flow Crescendo Inc. Documentation in this repository is MIT-licensed. The hosted service is subject to the Animation Machine terms.
