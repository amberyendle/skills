---
name: "science-talk-builder"
description: "Build or revise a timed, animated PowerPoint for a university or scientific talk, with speaker notes, click cues, references and Q&A prep. Use for any presentation, slides or viva task."
---

# Science talk builder

This skill captures a workflow that produced a finished 10-minute scientific presentation (a Masters assessment on a protein therapeutic) through many rounds of feedback. Use it whenever the user needs a live talk: a class presentation, a journal club, a lab meeting, a viva or a conference slot. The user is usually a working scientist or clinician, so the audience is expert and the slides must hold up to questions.

## 1. Start by pinning down the brief

Get these from the request or the attached brief before building. Ask at most one short question if something blocks the build, otherwise state your assumption and go.

- Topic and the exact required sections (for example cause, symptoms, diagnosis, therapeutic, limitations, future).
- Hard time limit and Q&A time. Size the deck at about one slide per 1 to 2 minutes and never plan past the limit.
- Audience (mixed clinicians and scientists means technical, not lay).
- The marking rubric, if any. Weightings tell you where to spend effort (for example content 30%, presentation 30%, answering questions 20%, asking questions 20%). If the audience must be asked questions, put two real discussion questions in the talk.
- Reference rules (usually references on the slides themselves) and any image rules.
- Whether the user already has a PowerPoint. If they do, edit their file. Do not rebuild from scratch.

## 2. Output format

- Deliver an editable .pptx, built with python-pptx or pptxgenjs, using the pptx skill. Presenters want to run it in PowerPoint, add their own edits and keep their own animations. A web or HTML artifact deck drifted off-screen in the user's real renderer, so do not use one for a live talk.
- When the user uploads a deck, work on a copy, keep every slide and animation they did not ask you to change, and name each output with a new version number (v2, v3). Never overwrite their upload.
- Slide size 13.333 x 7.5 in. Set an explicit font on every run (Arial is safe), hex colours without a hash, margins of at least 0.5 in, body text at least 14 pt.

## 3. Content rules

- One message per slide. Plain, descriptive headings, for example "Approved enzyme replacement therapies", with a small uppercase eyebrow label above (for example THERAPEUTIC). Never use cute or dramatic headings; the user finds them cringe.
- Use real figures and real numbers from papers. Prefer reviews and trials the user can cite. Put a short "Sources:" line on every content slide and a full references slide at the end.
- Images must be properly licensed and credited on the slide. CC BY is fine with credit. CC BY-NC-ND can be cropped but not recoloured or altered. Product pack shots and Drugs.com style images are not licensed: flag them, do not silently use them.
- Never invent coordinates, variant names, doses or citations. If a fact cannot be verified (the sandbox often blocks ClinVar, PMC, RCSB and similar), say so plainly, ask the user for a PDF or screenshot, and list it in the pre-presentation checklist.
- Prefer simple versions: if the user calls a chart chaotic, show only the most common items. Keep legends, but move them below a chart rather than leaving them crowding the top.

## 4. Visual style (the user's stated taste)

- Professional but interesting. Not AI-looking. A dark forest green with muted ochre and sand accents worked. Tone colours down when asked; no bright colours, no accent stripes, no underlines under titles.
- Fix boring slides with structure, not decoration: a numbered step flow for a mechanism, a native column chart plus small evidence cards for trial data, an icon-in-circle row for a summary, a sand-coloured callout for a single key point. Do not overdo it. When the user says a slide is "too much", remove cards and keep short bold statements with small icons and labels.
- Use native charts (python-pptx add_chart) so numbers stay editable. Group multi-shape graphics so they animate and move as one.
- Keep text short on slides and put the detail in the speaker notes. Check slides at the size they will be seen over Zoom: avoid text under about 12 pt.

## 5. Animation

Presenters like each segment to appear as they speak it, one click per idea.

- Use fade entrances of about 250 ms with a decelerate ease. Stagger objects revealed on the same click by 60 ms. Do not add slide transitions: the presenter changes slides by keyboard and motion on every change only slows them down.
- Never delete the user's existing timing. python-pptx preserves a slide's timing element when you save. To add effects, append or insert pars into the main sequence and give every new cTn a unique id (maximum existing id plus one).
- Timing structure: p:timing under p:sld, after clrMapOvr and before extLst. In the main sequence, each click is a par containing an inner par; each effect is a par whose cTn has presetID 10 (fade), presetClass entr, nodeType clickEffect for the first object and withEffect for others, with a p:set on style.visibility and a p:animEffect with transition in and filter fade (dur 250, decel 100000).
- An object with an entrance effect is hidden until its click, so tie each image to the same click as its caption.
- After the user edits the deck, recount clicks per slide from the timing XML and make the notes match (see below).

## 6. Speaker notes

- Write notes in the first person, in spoken language, with a time range at the top of each slide, for example (2:50-3:50), running continuously to the end.
- Put the cue `*click*` on its own line wherever the presenter must click, so the notes sequence matches the animation order exactly. The number of cue lines must equal the number of click animations on that slide. Slides without animation get none.
- Count words for the whole talk. At about 140 words per minute, stay under the limit with a margin. Report the estimated runtime and tell the user to rehearse with a timer.
- Give an intro that is plain and confident. Avoid roadmap-style filler and theatrical openers; offer two or three alternatives if the user dislikes the first.
- Put audience questions in the summary slide or in the closing notes, and say them aloud.

## 7. Build and QA loop

1. Read the user's deck: dump all text and notes (the pptx skill's slide_text.py), list every shape with ids and positions, and render to images to see it.
2. Plan the slide order first, then edit content.
3. Edit with python-pptx. Remember: assigning text_frame.text collapses runs, so set run.text. Text in grouped shapes and split across runs will not match a simple search, so walk the XML (a:t elements) and join runs. Scaling a group by setting width and height scales its children but not font sizes; set wrap to none on small text inside shrunken groups. Remove a textbox whose last paragraph you deleted, or the file will fail validation.
4. Validate with the pptx skill's validate.py using the original as baseline, then convert with soffice to PDF and pdftoppm to JPG and look at every changed slide. Fix overflow, overlap and unreadable contrast (for example pale text on a pale fill) before sending.
5. Send the file with a short summary of what changed and what still needs the user's check. Say plainly that animations could not be previewed here and were checked only in XML.

## 8. Companion deliverables

- A Q&A prep document, as a Word file when asked: about 15 likely questions with each question bold and large and a short answer of 20 to 40 seconds beneath, key phrases bolded so the presenter can scan while being asked. Add four questions the presenter can ask the audience and a few safe phrases for when they are unsure.
- A pre-presentation checklist: unverified facts, missing references, image credits and layout overlaps, timed rehearsal, an animation click-through in slideshow mode on the machine they will present from, and a Zoom screen-share test.

## 9. Handling the user's own edits

When the user sends back a changed file, diff it against your last version (text, shape count, animation count, notes). Do not overwrite their design choices. Report problems you find (for example overlaps, missing credits, notes out of step with clicks) and fix only what is safe, such as re-syncing the click cues. Ask before changing their layout.