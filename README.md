# Detection engineering life cycle

A single-page tool for taking detections from first requirement to production, and keeping them healthy once they are live. It runs entirely in the browser: there is no server, no account and no build step. The whole tool is one file, `index.html`.

**Open the tool:** `https://<your-username>.github.io/<repository-name>/`

## Try it with example data

Open the tool and select **try it with example data** in the empty pipeline, or choose **Share and export → Load example data**. It loads ten made-up detections that show every part of the tool: one in each stage, a rule with a data gap and partial coverage, a test that came out noisier than predicted, a live rule drowning in false positives, an overdue review, a threat parked in triage, a noisy idea held back by a gate, and a deprecated rule with its replacement.

Example detections are marked **Example**, stay in your browser, and are never included in the shared file. Remove them at any time with **Remove example data**.

## What it does

Each detection moves through six stages, with an optional seventh at the end:

| Stage | Question it answers | How it works |
|---|---|---|
| 1. Requirements Discovery | How good would the detection be? | Requirement details, then the prioritization dashboard |
| 2. Triage | How urgent is the threat? | Triage dashboard |
| 3. Investigation | What does bad look like? | Indicator type, log source review, technique and baseline checklist |
| 4. Development | Write the rule | Link to the rule, development checklist |
| 5. Testing | Does it work? | Testing checklist, predicted vs measured scores |
| 6. Deployment | Ship it | Deployment checklist, then regular reviews while the rule is live |
| 7. Deprecation (optional) | Turn it off cleanly | Reason, replacement and retirement checklist |

The overview has four views. **Pipeline** lists every detection with its next action, ranked so the most urgent work comes first. **ATT&CK coverage** shows which tactics and techniques your detections cover. **Data gaps** collects every missing or partial log source across all detections, ranked by how much they hold back. **Prediction check** compares the scores given at the start with what testing and production actually showed.

Every detection also has a change history and a one-page summary that can be copied as Markdown or printed to PDF.

## The two scores

The two scored stages answer different questions, and both are needed.

**Prioritization** scores the detection itself: coverage, durability, complexity, confidence, noisiness, actionable content and attacker cost. It answers how valuable the detection is to ship. Seven criteria, scored out of 28.

**Triage** scores the threat and the work: organizational impact, organizational relevance, whether an exploit is in the wild, how easy it is to execute, and the time needed to build. It answers when to build. Five criteria, scored out of 20.

Each criterion is scored 1 to 4, and higher is always better. That is why complexity and build time are inverted: simpler and faster score higher. Both scores map to a tier:

| Tier | Prioritization (ship) | Triage (build) | Cut-off |
|---|---|---|---|
| P1 | Deploy immediately | Build now | 85% of the maximum |
| P2 | Deploy next sprint | Build next sprint | 70% |
| P3 | Deploy when capacity allows | Backlog | 50% |
| P4 | Do not deploy | Park it | Below 50% |

The **next action** for a detection combines both tiers, and the slower of the two sets the pace. A great detection for a threat that does not affect you can wait, and an urgent threat with a weak detection idea needs a better idea first.

**Scoring gates** are hard rules that cap a tier whatever the total says. A threat that is not relevant is parked, and a detection that is very noisy or has low confidence cannot be higher than P3. Gates can be switched off from any scored stage if you want the raw dashboard result.

Testing re-scores the four criteria it can actually measure (coverage, confidence, noisiness and attacker cost). The measured values replace the predictions, so the prioritization can go up or down after testing. The original prediction is kept, and the Prediction check view shows whether your team tends to score high or low.

## Thresholds

The flags in the tool use these limits. They are defined in one place at the top of the script (`LIMITS` and `NOISE_BANDS`) if your team wants different numbers.

| Flag | Rule |
|---|---|
| Tune false positives | More than 5 false positives per week at the latest review |
| Production noise bands | Under 1 per week is silent precision, 1 to 5 low noise, 5 to 20 noisy, over 20 very noisy |
| Review due soon | Next review within 14 days |
| Review overdue | Past the review date (default every 90 days, adjustable per rule) |
| Fix broken rule | The test failed at the latest review |
| Check it still fires | No alerts at the latest review and the test was not re-run |
| Share reminder turns amber | Changes not shared for more than 2 days |

## Where the data lives

The page itself contains no detections. All work is saved automatically in the browser you use, under this site's address. It is not sent anywhere.

That has two consequences. Work done in one browser is not visible in another browser or on another device until it is shared. And clearing the browser's site data deletes it, so save a shared file regularly.

> **Do not commit your data to this repository.** If the site is public, anything in the repository is public too. The coverage map and the data gaps list together describe exactly what your organization cannot see. Keep the shared file in a private repository or an internal file share.

## Working as a team

The tool supports a shared file workflow without a server.

1. Everyone enters their name in the top bar once, so changes show who made them.
2. Before working, open **Share and export → Merge a shared file** and choose the latest `detection-pipeline.json`.
3. Do your work.
4. Choose **Share and export → Save shared file** and put the new `detection-pipeline.json` back in the shared location, or commit it to your private data repository.

Merging works per detection, and the newest change wins. After a merge, a report lists what was added, updated and deleted. It also warns about detections that both sides changed since the last sync, and about two detections with the same name. Deleted detections stay deleted across merges, unless someone changed them after the delete.

If you use Git for the shared file, pull before merging and commit right after saving. Detections are always written in the same order, so diffs stay small and readable.

## Importing existing rules

**Share and export → Import detections (CSV)** adds detections in bulk. It never overwrites, and rows whose name already exists are skipped. **Download CSV import template** gives a ready example.

Headers are matched loosely. These are recognised:

| Field | Accepted headers |
|---|---|
| Name (required) | Name, Detection, Rule name, Title |
| Owner | Owner |
| Stage | Stage, Status (stage names, or Deployed, Live, Enabled, Deprecated, Retired) |
| Techniques | Techniques, Technique, Technique IDs |
| Tactics | Tactics, Tactic (names or TA IDs) |
| Data sources | Data sources, Log sources |
| Trigger | Trigger, Request source, Source |
| Rule link | Rule link, Rule ID, Link, URL |
| Dates | Created, Deployed, Last reviewed, Deprecated |

Comma and semicolon separated files both work. Scores are not imported, so imported detections that are still in progress ask to be scored when you pick them up. Imported deployed rules go straight into the review cycle.

## Publishing on GitHub Pages

1. Create a repository and upload `index.html` (and this README) to the root of the `main` branch.
2. Go to **Settings → Pages**, set the source to **Deploy from a branch**, and choose `main` and `/ (root)`.
3. After a minute or two the tool is available at `https://<your-username>.github.io/<repository-name>/`.

To update the tool, upload a new `index.html` over the old one. Saved work in people's browsers is not affected.

On GitHub Free, Pages requires a public repository. Pro and Team can publish from a private repository, but the site itself is still reachable by anyone with the link. Restricting the site to members of your organization requires GitHub Enterprise Cloud. Since the page contains no data, a public site is acceptable as long as the shared data file is kept elsewhere.

## Browser support

The tool needs JavaScript. It works in current versions of Chrome, Edge, Firefox and Safari, on desktop and on phones. Opening the downloaded file in a preview that does not run scripts, such as the iPhone Files preview, shows a notice instead of the tool. Open it from the GitHub Pages address in a browser instead.

The page loads its fonts from Google Fonts. Without network access to Google it falls back to the system font and works the same.

## Credits

The life cycle stages and the two scoring dashboards follow the detection engineering life cycle taught in SANS SEC555, Detection Engineering and SIEM Analytics. Indicator types are ordered by David Bianco's Pyramid of Pain. Tactics and techniques refer to MITRE ATT&CK®.
