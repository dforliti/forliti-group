# Forliti Research Group — website

Everything here is static. No build step, no framework, no server-side code.
Upload the folder somewhere that serves files over HTTP and it works.

```
index.html          the entire site — all pages, all styles, all photos
videos/             the project videos, two encodings of each
README.md           this file
```

This is the complete site — nothing to merge. `videos/` holds six MP4 files.

`index.html` is self-contained apart from the videos: every person photo, poster
frame and icon is embedded in it. The videos are separate so they stream on
demand instead of being downloaded with the page.

---

## Putting it online

**GitHub Pages.** Create a repository, drop these files at its root, then in
Settings → Pages choose "Deploy from a branch" and pick `main` / `/ (root)`.
The site appears at `https://<user>.github.io/<repo>/` within a minute or two.

**A St. Thomas web directory or any other host.** Copy the whole folder in,
keeping `videos/` next to `index.html`. Nothing else is required.

**Checking it locally.** Opening `index.html` by double-clicking mostly works,
but browsers block some things on `file://` URLs. To see it exactly as visitors
will, run a local server from this folder:

```
python3 -m http.server 8000
```

then open `http://localhost:8000`.

---

## Keeping it updated

The intended workflow is that you don't edit this file by hand — you ask Claude
to make the change, and it edits `index.html` in place.

For that to be efficient, Claude needs to be able to reach this folder:

1. **Keep this folder on your computer** (not only on the web host).
2. **In the Claude desktop app, create a project** for the site and **connect
   this folder** to it. Project instructions and memory persist across tasks, so
   the conventions below don't have to be re-explained each time.
3. **Paste the instructions block below** into the project's instructions.

Then a typical update is one sentence plus a drag-and-drop:

> "Add two new students — photos attached. Sarah Chen, BSME, expected 2029,
>  working on twin impinging jets. Marcus Webb, BSME + Physics, 2028, coaxial
>  jets. Bios attached."

> "Move Joel Rodich to alumni, he's at Boston Scientific now."

> "Add this video to the liquid impinging jets project."

Claude edits `index.html`, and you re-upload it (or commit it, see below).

### Why GitHub Pages is worth it

If you host on GitHub Pages and keep a local clone as the connected folder, the
loop closes: Claude edits the file, commits and pushes, and the live site
updates within a minute. No manual upload step, plus you get version history and
one-click rollback if something goes wrong. On a university web directory you
get the same edits but you upload the file yourself afterwards.

### Project instructions to paste

```
This project maintains the Forliti Research Group website. The site is
index.html in the connected folder — a single self-contained file. Videos
live in videos/ as separate files.

Content lives in labelled arrays near the top of the <script> block:
PROJECTS, RESEARCH_AREAS, PEOPLE, PUBLICATIONS, NEWS, CAPABILITIES,
DATASETS. Edit those; never hand-write markup. Layout is automatic.

Rules:
- Never alter a student bio, including apparent typos. Flag them and let
  me decide.
- If my spelling of a name differs from the site's, ask before changing
  it. The site has usually been right.
- "destination" means where a person is NOW, not their first job after
  UST. Overwrite it when they move.
- group is "pi", "current" or "alumni". Moving someone is changing that
  one word.
- Person photos are embedded as data URIs, square. Keep it that way.
- Videos are separate files in videos/, in two encodings: .webm (VP9
  CRF 31) and .mp4 (H.264 CRF 23). Flow videos are silent (-an); talk
  recordings keep their audio.
- Never trim a clip that shows a transition, onset or progression. Only
  trim footage that is statistically stationary throughout.
- After any change, verify the page renders with no console errors
  before telling me it's done.
```

---

## Editing the content

If you ever do want to edit by hand, everything is in one place.

Open `index.html` in any text editor. Near the top of the `<script>` block,
after the styles, there is a clearly marked content section. Everything you
would normally want to change lives there, in plain lists:

| List | Holds |
|---|---|
| `PROJECTS` | the research projects, their text, videos and recorded talks |
| `RESEARCH_AREAS` | the tags on the home page |
| `PEOPLE` | everyone — PI, current members, alumni |
| `PUBLICATIONS` | the papers |
| `NEWS` | the news items |
| `CAPABILITIES` | empty; fill it and the Capabilities page builds itself |
| `DATASETS` | empty; fill it and the Data page builds itself |

Layout, sizing, ordering and responsive behaviour are all automatic. You never
edit HTML markup to add content.

### Adding a person

Copy any entry in `PEOPLE` and change the fields:

```js
{ name:"Jane Doe", group:"current", role:"Undergraduate",
  photo:"data:image/jpeg;base64,...",       // or null for a generated monogram
  program:"BSME + Physics", year:"2028",
  destination:null,                          // where they are now, once they leave
  projects:["impinging"],                    // ids from PROJECTS
  bio:"..." },
```

- `group` is `"pi"`, `"current"` or `"alumni"`. Moving someone to alumni is
  changing that one word — they drop out of the card grid and into the compact
  roster automatically.
- `destination` is optional and shows as an amber arrow line meaning "where they
  are now". Overwrite it when someone changes jobs.
- `photo` can be a `data:` URI (keeps the file self-contained) or a path like
  `"photos/jane.jpg"` if you would rather keep photos as separate files.
  Photos are shown square; anything not square is centre-cropped by the browser.

### Adding a video to a project

Put the files in `videos/`, then add an entry to that project's `clips` array:

```js
clips:[{ mp4:"videos/my-clip.mp4",
         webm:null,                      // optional VP9 copy, offered first
         poster:"data:image/jpeg;base64,...",
         w:512, h:512,
         caption:"What the viewer is looking at." }]
```

`w` and `h` are the video's pixel dimensions — they are used to size the frame
so tall clips render at full height instead of sitting in a wide black box.

If a video file is missing, the page shows its poster frame with a small note
rather than a dead player, so the page never looks broken.

### Adding a recorded talk

A project can also carry a `talks` array. Each entry renders as a card that
opens a large pop-up player — useful when the video is a presentation and the
slides need to be readable:

```js
talks:[{ kind:"Recorded presentation",
         title:"Talk title",
         speaker:"Who gave it",
         meta:"2024",
         href:"https://youtu.be/...",   // link out — opens in a new tab
         poster:null }]                 // or a data: URI for a thumbnail
```

An entry can be served two ways, and the card adapts on its own:

- **`href`** — the card is a link and opens the recording in a new tab. Right for
  full-length talks: no large file on your host, and streaming services handle
  seeking and captions properly.
- **`webm` + `mp4`** (no `href`) — the card opens the built-in pop-up player,
  reading the files from `videos/`. Right for short clips you want to keep
  self-contained.

Switching between the two is swapping which fields are present; nothing else
changes.

Talk videos keep their audio; the flow videos are silent by design. Encode them
the same way but **without** the `-an` flag, adding `-c:a aac -b:a 128k` for the
MP4 and `-c:a libopus -b:a 128k` for the WebM.

---

## About the video encoding

Clips ship as `.mp4` (H.264), which every current browser plays — Chrome, Edge,
Firefox, Safari, iOS and Android. To add a clip in the same style:

```
ffmpeg -i source.mov -an -c:v libx264 -preset slow -crf 23 \
       -pix_fmt yuv420p -movflags +faststart videos/name.mp4

ffmpeg -ss 10 -i videos/name.mp4 -frames:v 1 -q:v 3 poster.jpg
```

`-movflags +faststart` matters: it puts the index at the front of the file so
playback starts immediately instead of after the whole download.

A clip entry may also carry a `webm` field pointing at a VP9 copy, and the page
will offer it first. That's optional extra coverage for browsers built without
proprietary codecs; it is not needed for any mainstream browser, and these
files ship MP4-only to keep the download small.

Lower `-crf` means higher quality and a larger file. 23 is close to visually
lossless for schlieren; the fine turbulent texture starts to smooth out above
about 28, which matters because that texture is the data.

---

## Current state

- 47 people, all with photos. One current member, 45 alumni, 32 of whom have
  a current position listed.
- Six projects, four with video. Twin impinging gaseous jets leads, and links out
  to Xijun Tan's recorded presentation on YouTube.
- Seven publications, two news items.
- The Capabilities page is filled in. The Data Repository page is live but
  empty — that content could not be recovered from the old site.
