# Notes to Nuke — Griptape × ShotGrid / ftrack

Run AI workflows straight from **ShotGrid** or **ftrack**. A supervisor writes review notes on a shot and sets the task to **Send to Griptape**. A service on the farm picks it up, an AI agent turns the notes into knob changes on the Nuke comp, renders a before/after review movie, and publishes a new version back to the tracker. It also replies to each note with what changed.

**Demo:** https://youtu.be/Xvyw5QE5J9U

> This repo is a project overview. The source code isn't public; I'm happy to walk through it on a call.

## How it works

1. **Trigger.** A supervisor sets the task's status to *Send to Griptape*, or right-clicks it and picks *Run with Griptape*. Artists and supervisors install nothing.
2. **Pick-up.** An always-on Python service on a farm machine polls ShotGrid and listens for ftrack events. It finds the shot's open notes and the Nuke script to work on.
3. **Agent pass.** A Griptape workflow hands the notes and the comp to an AI agent. The agent finds the nodes the notes refer to (on any node, in any comp) and changes their knobs.
4. **Render and review.** Nuke renders headless at low priority. The service builds a before/after review movie.
5. **Publish.** A new version goes back to the tracker, with a thumbnail and a playable movie. Each note gets a reply saying what changed. Anything that needs an artist becomes a Paint / Roto task.

Progress shows up as a reply on the note, with time remaining. If a job fails, the task goes back to its previous status with a note saying why.

## Design choices

- **A human approves, not the agent.** Every run publishes a *new* version for review. It never overwrites the artist's script.
- **No new app to learn.** It's triggered from the tracker the studio already uses.
- **Tracker-agnostic.** The job format is separate from ShotGrid, so adding ftrack was a small adapter rather than a rewrite.
- **Explicit over guessed.** The Nuke script comes from a published file or a version field rather than being inferred from task metadata, so it never opens the wrong comp.
- **No secrets in config.** Tracker keys live in Griptape's own settings. The service generates its own token for the right-click link.
- **Works on a locked-down network.** It uses the OS certificate store, so it runs behind corporate TLS interception.
- **Extensible.** Any Griptape workflow saved between a Start Flow and an End Flow node shows up in the tracker's picker. A second example turns notes into a concept frame with an image model.

## Built with

Python · ShotGrid and ftrack REST APIs · Griptape Nodes (agent orchestration) · Nuke (headless Python) · Windows, macOS and Linux service install

*Experimental tool, built with AI assistance. Tested on Windows with ShotGrid and ftrack.*
