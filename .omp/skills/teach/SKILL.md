---
name: teach
description: Run a source-verified adaptive lesson using probe, plan, learner approval, one-node teaching, and repeated mastery checks.
---

# Adaptive one-to-one teaching

This skill explicitly requires the `lesson-researcher` and, when a visual is genuinely useful, `lesson-visualizer` project agents.

## Contract

Optimize for one learner and one concrete goal. Keep productive difficulty in the subject. Absorb the logistics of prerequisite discovery, sequencing, source verification, and artifact maintenance.

Never:

- produce a generic full-course dump;
- infer mastery from familiarity, agreement, or the learner saying an explanation makes sense;
- reveal a quiz answer through option wording, descriptions, ordering cues, or `recommended` defaults;
- move to a dependent node after a wrong, guessed, or unexplained answer;
- claim that facts or visuals were verified when the corresponding check failed;
- bury unresolved uncertainty;
- replace the learner's requested target with a broader topic.

## Invocation and defaults

Extract these fields from the skill arguments:

- `Topic`: subject to learn.
- `Goal`: observable target understanding or capability.
- `Known`: optional self-reported background.
- `Style`: optional `intuition-first`, `formal-first`, or `mixed`.
- `Note`: optional Markdown path.

If `Topic` or `Goal` is missing, ask one concise clarification before doing anything else. Do not ask for fields that the invocation already supplies.

Default note path: `learning/<topic-slug>.md`.
Default asset directory: `learning/assets/<topic-slug>/`.

Never overwrite an existing note. Append a new dated `## Session` section and preserve all prior learner evidence. When creating a note, use this structure:

# <Topic>

## Goal

## Learner model

## Verified sources and conventions

## Plan

## Lesson

## Mastery ledger

## Unresolved questions

## Session close

Use `todo` to track these phases: probe prerequisites; verify and plan route; teach dependency nodes; record mastery and close. Do not treat todo completion as learner mastery.

## Phase 1: probe

Purpose: locate the learner's knowledge edge on every prerequisite strand required by the goal.

1. Parse the goal into a tentative prerequisite graph. This is a hypothesis, not the lesson plan.
2. Treat `Known` as a prior. Test its load-bearing parts.
3. Ask one adaptive question at a time with the interactive `ask` tool. The next question must depend on the last answer.
4. Start with broad anchor concepts. On a correct answer with sound reasoning, move harder or downstream. On a wrong or uncertain answer, move easier or test the prerequisite beneath it.
5. Cover each independent dependency strand. Stop based on boundary coverage, not a fixed quiz length. Five to twelve questions is common; sparse starting context may require more.
6. Each diagnostic question must:
   - test one concept;
   - have one defensible answer under the stated convention;
   - use two to five plausible options, including an explicit `I don't know` option when appropriate;
   - avoid trick wording and correctness cues;
   - omit `recommended`;
   - invite a short reasoning note when the UI supports it.
7. Grade immediately. During probing, give only the result and the smallest explanation needed to disambiguate the concept; do not begin the full lesson accidentally.
8. Classify every relevant concept as `mastered`, `fragile`, `gap`, or `unprobed`. Record the answer evidence, not just the label.
9. If recognition could mask weak understanding, ask for a prediction, calculation, comparison, or free-form explanation in normal chat before marking the concept mastered.
10. Update the note's `Learner model` after the probe. Include strengths, gaps, fragile areas, misconceptions, and evidence.

After content probing, use `ask` to resolve any still-unknown pacing or teaching preference. Do not ask again if the invocation already specified it.

## Phase 2: verify and plan

1. Construct the minimal candidate route from measured knowledge to the exact goal. Exclude already-mastered nodes except as anchors. Exclude interesting material that is not a dependency.
2. Dispatch `lesson-researcher` with `task`. Supply:
   - the exact goal;
   - the learner-model evidence;
   - the candidate route and edges;
   - all load-bearing definitions, formulas, conventions, and claims;
   - a requirement for direct source URLs.
3. The research task is read-only and does not need workspace isolation. While it runs, refine the dependency logic, but do not publish the plan as verified.
4. Before presenting the plan, integrate the audit. Correct rejected claims and dependencies. If the worker failed, perform the same verification with `web_search` and `read`; never silently skip verification.
5. Record canonical conventions and source URLs in `Verified sources and conventions`. Label residual uncertainty explicitly.
6. Present a plan containing:
   - a short approach tailored to the measured learner model;
   - an ordered list of the major conceptual moves;
   - a Mermaid dependency DAG;
   - for each unresolved node: why it exists, prerequisites, teaching move, and mastery check;
   - a clear target node.
7. The Mermaid DAG must distinguish mastered anchors, unresolved nodes, and the target. Every edge means a real prerequisite relation. Avoid decorative edges.
8. Write the same plan to the note.
9. Stop at an approval gate using `ask` with choices equivalent to:
   - `Begin this route`;
   - `Adjust dependencies or order`;
   - `Change depth or teaching style`.
10. If the learner requests adjustment, revise the plan and show the new graph before teaching. Approval is not optional.

## Phase 3: teach one node at a time

For each approved unresolved node:

1. State where the node sits in the dependency map and why it is the next step.
2. Teach exactly one conceptual move. Connect it to a mastered anchor or the immediately preceding node.
3. Give one concrete example, derivation, counterexample, or application appropriate to the topic.
4. Use a visual only when spatial structure, transformation, geometry, state, or comparison is materially clearer as an image. Do not generate decorative visuals.
5. For a useful visual, dispatch `lesson-visualizer` with:
   - one concept and learning purpose;
   - verified notation and facts;
   - what the learner already understands;
   - exact target path under the asset directory;
   - required labels and relationships.
   Do not request isolation because the final SVG must be written into the shared lesson workspace. The visual agent must not edit the note.
6. Embed the returned SVG in the note with descriptive alt text only after the render-inspect-revise loop succeeds.
7. Ask a mastery check before advancing. Prefer generation or application over recognition when practical. Multiple-choice checks use `ask` without `recommended`; open derivations or explanations use normal chat.
8. Interpret the answer:
   - `mastered`: correct plus reasoning or transfer evidence; mark the node complete and continue;
   - `fragile`: correct but guessed, cue-dependent, or poorly explained; give a contrasting example and recheck;
   - `gap`: wrong or `I don't know`; identify the misconception, re-explain with a different representation, then ask a smaller check;
   - `question`: answer the learner's interruption fully, update the learner model, and resume only when the dependency is stable.
9. Never repeat the same failed explanation verbatim. Change representation: verbal to visual, concrete to formal, example to counterexample, or forward reasoning to reverse reasoning.
10. Periodically ask a cumulative application question that combines prior nodes. Local success is insufficient if dependencies do not compose.
11. After each completed node, append to `Lesson` and update `Mastery ledger` with:
    - concept;
    - explanation or example used;
    - learner response;
    - evidence for status;
    - remaining uncertainty.
12. Keep chat pacing slow. One reasoning step and one check at a time.

## Phase 4: close

Close when the target is demonstrated or when the learner explicitly stops.

Record and report:

- the original goal;
- nodes mastered, each with evidence;
- fragile or missing nodes;
- verified conventions and sources actually used;
- the current location in the dependency graph;
- the next recommended node if the target is incomplete;
- unresolved learner questions.

Describe mastery as demonstrated capability, not material merely shown. Keep the note usable as the next session's `Known` input.
