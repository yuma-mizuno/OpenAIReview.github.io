# Mathematics review with the OpenAIReview PDF viewer

Review a mathematics manuscript using [skill.md](skill.md), then open its findings
beside the matching PDF. The review instructions use the
[OpenAIReview mathematics workflow](https://github.com/OpenAIReview/OpenAIReview.github.io/blob/1e205f15448208e4538b52ef54aa4c3774137e84/skill.md).
The viewer reads the resulting `issues` JSON directly and locates quoted passages
in the PDF.

## 1. Set up

You need a coding agent that can read files and run the review instructions, Git,
Python 3, the complete TeX source and a PDF of the same manuscript version.

```sh
git clone --single-branch --branch codex/review-viewer https://github.com/yuma-mizuno/OpenAIReview.github.io.git pdf-review-viewer
```

Keep manuscripts and results in a separate private folder outside this checkout.
Give the coding agent the absolute path to `pdf-review-viewer/skill.md` and ask:

> Follow this skill to review my manuscript at [source path], using the matching
> PDF at [PDF path]. Use paper ID `paper` and save the review and viewer in the
> private folder [work path].

The agent follows the skill's reading, parallel review and consolidation steps.
It records an exact source version, verifies findings against that source, and
writes `openverification_review.json`. Model use requires the agent's configured
account or provider; manuscript content is processed under that provider's terms.

## 2. Save the review and source files

The skill instructs the agent to prepare the following folder. These files can
also be prepared manually. Copy `pdf-review-viewer/math/paper.html` to `index.html`
and save the review JSON unchanged as `data/reviews/paper.json`.

```text
private-review/
  index.html
  data/
    index.json
    catalog.json
    reviews/
      paper.json
  source/
    <SOURCE_COMMIT>/
      paper/
        main.tex
        paper.pdf
        ... included source files in their original relative paths
```

`<SOURCE_COMMIT>` is the actual full 40-character Git commit identifying the
reviewed manuscript snapshot. For a manuscript outside Git, the agent records
the supplied files in an independent private snapshot repository first. The PDF
must correspond to that source version.

Write `data/index.json`, replacing the placeholders with the actual values:

```json
{
  "source": {"commit": "<SOURCE_COMMIT>"},
  "papers": [{"id": "paper", "title": "Paper title", "result_id": "1"}]
}
```

Write `data/catalog.json`:

```json
[
  {
    "id": "paper",
    "title": "Paper title",
    "result_id": "1",
    "subject": "Subject",
    "abstract": "",
    "pdf": "paper/paper.pdf",
    "tex": "paper/main.tex"
  }
]
```

The `pdf` and `tex` paths are relative to `source/<SOURCE_COMMIT>/`. A review has
the following shape; `skill.md` specifies the review criteria and field limits:

```json
{
  "collection": "openai-math",
  "paper_id": "paper",
  "commit": "<SOURCE_COMMIT>",
  "agent": "<actual coding agent>",
  "model": "<actual model>",
  "created_at": "<actual ISO 8601 review timestamp>",
  "issues": [
    {
      "title": "Short description",
      "quote": "Exact passage from the TeX source",
      "explanation": "Checked mathematical reasoning",
      "comment_type": "methodology",
      "severity": "major"
    }
  ]
}
```

Use a distinct paper ID for each manuscript. Add each paper to both index and
catalog, give it its own review JSON, and retain the source snapshot named by
that review's `commit`. The paper selector switches between them.

## 3. Open and verify

```sh
python -m http.server 8080 --bind 127.0.0.1 --directory /absolute/path/to/private-review
```

Open [http://127.0.0.1:8080/?paper=paper&contributed=1](http://127.0.0.1:8080/?paper=paper&contributed=1).
Keep `contributed=1` in the URL to use the viewer's `issues` loader. Stop the
server with Ctrl+C.

The viewer uses PDF.js text extraction and the OpenAIReview quote locator in the
browser. Findings appear in PDF page order, then top to bottom; unmatched quotes
follow in their input order. Check each issue's PDF link and each highlight's
return link. Equations, repeated text, scans and TeX commands can prevent correct
matching. An automatic highlight requires visual verification.

The macro reader checks the main TeX file and up to twelve directly included
files for supported macro definitions. Check the rendered mathematics when a
project uses deeper includes, custom style files or complex macros.

## 4. Share

Deploy the complete `private-review` folder to a static host, keeping the same
file paths and `?paper=paper&contributed=1` URL parameters. The folder contains the
manuscript PDF, TeX source and review. Use access controls appropriate for the
intended readers. A private Git repository alone does not make a hosted website
private. Add `.nojekyll` at the published root when using GitHub Pages.

This fork disables the website's manuscript uploads, review submissions and
feedback, and removes its Analytics scripts. Review JSON, source and PDF are read
from the hosted folder. The viewer loads its existing MathJax, PDF.js and fonts
from their configured CDNs.
