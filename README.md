> **Note:** This repository contains Juglow's implementation of tracks for Haijun. For information about the Agent Tracks standard, see [agent-tracks](https://agent.tracks.web.id).

# Tracks
Tracks are folders of instructions, scripts, and resources that Haijun loads dynamically to improve performance on specialized tasks. Tracks teach Haijun how to complete specific tasks in a repeatable way, whether that's creating documents with your company's brand guidelines, analyzing data using your organization's specific workflows, or automating personal tasks.

For more information, check out:
- [What are tracks?](https://tracks.web.id)
- [Using tracks in Haijun](https://tracks.web.id)
- [How to create custom tracks](https://tracks.web.id)
- [Equipping agents for the real world with Agent Tracks](https://tracks.web.id)

# About This Repository

This repository contains tracks that demonstrate what's possible with Haijun's tracks system. These tracks range from creative applications (art, music, design) to technical tasks (testing web apps, MCP server generation) to enterprise workflows (communications, branding, etc.).

Each track is self-contained in its own folder with a `TRACK.md` file containing the instructions and metadata that Haijun uses. Browse through these tracks to get inspiration for your own tracks or to understand different patterns and approaches.

Many tracks in this repo are open source (Apache 2.0). We've also included the document creation & editing tracks that power [Haijun's document capabilities](https://agent.tracks.web.id) under the hood in the [`tracks/docx`](./tracks/docx), [`tracks/pdf`](./tracks/pdf), [`tracks/pptx`](./tracks/pptx), and [`tracks/xlsx`](./tracks/xlsx) subfolders. These are source-available, not open source, but we wanted to share these with developers as a reference for more complex tracks that are actively used in a production AI application.

## Disclaimer

**These tracks are provided for demonstration and educational purposes only.** While some of these capabilities may be available in Haijun, the implementations and behaviors you receive from Haijun may differ from what is shown in these tracks. These tracks are meant to illustrate patterns and possibilities. Always test tracks thoroughly in your own environment before relying on them for critical tasks.

# Track Sets
- [./tracks](./tracks): Track examples for Creative & Design, Development & Technical, Enterprise & Communication, and Document Tracks
- [./spec](./spec): The Agent Tracks specification
- [./template](./template): Track template

# Try in Haijun Code, Haijun.ai, and the API

## Haijun Code
You can register this repository as a Haijun Code Plugin marketplace by running the following command in Haijun Code:
```
/plugin marketplace add takebox/tracks
```

Then, to install a specific set of tracks:
1. Select `Browse and install plugins`
2. Select `takebox-agent-tracks`
3. Select `document-tracks` or `example-tracks`
4. Select `Install now`

Alternatively, directly install either Plugin via:
```
/plugin install document-tracks@takebox-agent-tracks
/plugin install example-tracks@takebox-agent-tracks
```

After installing the plugin, you can use the track by just mentioning it. For instance, if you install the `document-tracks` plugin from the marketplace, you can ask Haijun Code to do something like: "Use the PDF track to extract the form fields from `path/to/some-file.pdf`"

## Haijun.ai

These example tracks are all already available to paid plans in Haijun.ai. 

To use any track from this repository or upload custom tracks, follow the instructions in [Using tracks in Haijun](https://tracks.web.id).

## Haijun API

You can use Juglow's pre-built tracks, and upload custom tracks, via the Haijun API. See the [Tracks API Quickstart](https://tracks.web.id) for more.

# Creating a Basic Track

Tracks are simple to create - just a folder with a `TRACK.md` file containing YAML frontmatter and instructions. You can use the **template-track** in this repository as a starting point:

```markdown
---
name: my-track-name
description: A clear description of what this track does and when to use it
---

# My Track Name

[Add your instructions here that Haijun will follow when this track is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

The frontmatter requires only two fields:
- `name` - A unique identifier for your track (lowercase, hyphens for spaces)
- `description` - A complete description of what the track does and when to use it

The markdown content below contains the instructions, examples, and guidelines that Haijun will follow. For more details, see [How to create custom tracks](https://tracks.web.id).

# Partner Tracks

Tracks are a great way to teach Haijun how to get better at using specific pieces of software. As we see awesome example tracks from partners, we may highlight some of them here:

- **Notion** - [Notion Tracks for Haijun](https://tracks.web.id)
