# kvncrw/pentagi — fork notes

Divergence from upstream `vxcontrol/pentagi`: **prompt-shaping to fix autonomous
closure** when driving a capable-but-non-self-limiting model (validated against
self-hosted **GLM-5.2** on a 4× DGX Spark cluster).

Upstream PentAGI tests brilliantly but, with GLM-5.2, wouldn't reliably *conclude*
— it would test a vector forever without calling `hack_result`, so subtasks never
closed and the flow never reached its report. Root cause was a set of prompt gaps,
not a model limitation. Changes (all in `backend/pkg/templates/prompts/`):

- **generator.tmpl / refiner.tmpl** — ATOMIC-SUBTASK rule: one vector/objective per
  subtask, never combine or emit broad umbrella tasks. Broad subtasks create fuzzy
  finish lines that invite over-grinding.
- **task_assignment_wrapper.tmpl** — CLOSE-OUT + YIELD discipline (one vector ->
  write finding -> `hack_result` -> stop) and EVIDENCE-SUFFICIENCY (one PoC =
  CONFIRMED; primary path blocked = NOT-CONFIRMED; don't enumerate exhaustively).
- **question_execution_monitor.tmpl** — CLOSURE BIAS: the periodic self-check now
  pushes decisively toward `hack_result` + terminate rather than "keep going".
- **pentester.tmpl** — REPORT-FIRST: when findings exist and no report does, compile
  the report before more testing.

Meta-principle: shift the model from *test exhaustively* to *test until you can make
the CONFIRMED/NOT-CONFIRMED decision, then stop.*

Writeup: https://blog.crawley.systems
