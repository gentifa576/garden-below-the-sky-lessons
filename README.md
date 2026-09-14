# Garden Below the Sky: Learn Godot By Building

A beginner-friendly course for making a 2D game in Godot, one playable milestone at a time. No programming background, paid assets, Codex, terminal, or AI subscription is required.

**[Start with the course overview](OVERVIEW.md).** It explains installation, the chapter sequence, and how to learn with or without ChatGPT.

| Chapter | What you build | Lesson |
| --- | --- | --- |
| 1 | A character you can move around a room with solid walls | [Walk Around Home](chapter_1/LESSON.md) |
| 2 | A square-board automatic duel with health, results, and restart | [An Automatic Duel](chapter_2/LESSON.md) |
| 3 | Ally placement, moving teams, range-based attacks, and reset to a chosen layout | [Placement And Moving Teams](chapter_3/LESSON.md) |

This repository contains lessons and complete code references, not a ready-to-run Godot project. You create the project and scenes yourself in the editor. Solutions are intentionally separated into each chapter's `cheat_sheet/SOLUTION.md`.

## Use With Ordinary ChatGPT

Open a normal ChatGPT conversation and send this, replacing the URL placeholder with this repository's actual GitHub URL:

```text
Be my beginner Godot tutor using this repository: <REPOSITORY_URL>

Read README.md, then OVERVIEW.md, including its "ChatGPT Tutor Protocol".
Start with chapter_1/LESSON.md unless I tell you I have completed it.
Confirm which files you actually accessed. If you cannot read GitHub, ask
me to paste or attach OVERVIEW.md and the relevant lesson instead. Do not
pretend to have read inaccessible files.

I am new to programming. I will operate Godot and write the code myself.
Guide one checkpoint at a time, explain new concepts, and offer hints
before solutions. Do not jump ahead or assume you can see my editor.
Ask for my Godot version and current checkpoint, then help me begin.
Do not open the cheat sheet unless I ask or need a targeted comparison.
```

If browsing is unavailable, use GitHub's **Raw** view to get the Markdown text, then paste it into the chat. File uploads are optional; plain text is enough. The [overview](OVERVIEW.md#when-github-access-is-unavailable) gives the fallback workflow and a progress note for future conversations.

A repository link is not a guaranteed integration. ChatGPT features vary, private repositories may be inaccessible, and a model cannot infer your local progress from these files. No coding-agent access is needed or expected.

## For ChatGPT And Other Tutors

When asked to guide a learner from this repository, use the [ChatGPT Tutor Protocol](OVERVIEW.md#chatgpt-tutor-protocol). Read the overview and the relevant lesson, not every solution. Ask for evidence of the learner's current state; repository examples are not evidence that their project works. Do not invent missing chapters or claim access to their machine.

## Publication Scope

The first three completed milestones are documented here. New chapters are added at completed-milestone boundaries, not during every development session. A walkable room and a separate repeatable team battle with placement are the current results, not a finished farming game. Chapter 3 adds no feeding, growth, tower progression, or saving; later planned work is unchanged.

See the [validation notes](OVERVIEW.md#validation) for the Godot version and the distinction between automated checks and hands-on testing. No external art or font files are included.
