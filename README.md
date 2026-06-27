# VLN Learning

Personal learning vault for Vision-Language Navigation (VLN), Vision-Language-Action (VLA), and embodied navigation papers.

This repository is designed to be opened directly as an Obsidian vault while staying easy to read on GitHub.

## Start Here

- [Obsidian Home](00-Home.md)
- [Paper Notes](notes/papers/README.md)
- [Concept Notes](notes/concepts/README.md)
- [Topic Maps](notes/topics/README.md)
- [Projects](notes/projects/README.md)

## Current Notes

- [VLN 学习笔记：Awesome Visual-Language-Navigation](notes/papers/VLN-awesome-survey-notes.md)

## Repository Layout

```text
.
├── 00-Home.md              # Obsidian vault entry page
├── notes/                  # Main learning notes
│   ├── papers/             # Paper reading notes
│   ├── concepts/           # Concepts, methods, metrics, datasets
│   ├── topics/             # Topic maps and reading roadmaps
│   └── projects/           # Project-level notes and plans
├── references/             # Bibliography, links, metadata
├── assets/                 # Images and small attachments used by notes
├── templates/              # Obsidian note templates
├── experiments/            # Code snippets, notebooks, small experiments
└── outputs/                # Generated outputs; ignored by Git by default
```

## Obsidian + GitHub Workflow

1. Open this repository folder as an Obsidian vault.
2. Create new notes under `notes/`.
3. Put images and small attachments under `assets/`.
4. Use Obsidian Git or command line Git to commit and push.

Useful commands:

```powershell
git status
git add .
git commit -m "docs: update learning notes"
git push
```

Large local files such as PDFs, archives, datasets, model weights, and generated outputs are ignored by default. Prefer linking to papers or storing lightweight citation metadata in `references/`.
