# Architecture

KMP-IMPACT is structured as a **5-phase pipeline**. Each phase reads a typed JSON contract from the previous one and writes its own. There is no implicit shared state: every artifact is on disk under `output/phaseN/`, which makes the pipeline reproducible, debuggable and trivially parallelizable.

<div class="kmp-callout" markdown>
**Mental model.** Static and dynamic are *complementary*, not redundant: static is complete-but-conservative; dynamic is small-but-observed. The consolidator combines them rather than trying to resolve disagreements.
</div>

## In this section

<div class="kmp-grid" markdown>

<div class="card" markdown>
### Pipeline overview
The five phases at a glance, with the data flow and design principles.

[Open →](pipeline-overview.md)
</div>

<div class="card" markdown>
### Phase 1 — Shadow build
Materialises before/ and after/ copies of the project. Applies the catalog edit on AFTER.

[Open →](phase-1-shadow-build.md)
</div>

<div class="card" markdown>
### Phase 2 — Static analysis
Tree-sitter Kotlin parser, symbol graph, BFS, source-set tagging, `expect`/`actual` scan.

[Open →](phase-2-static-analysis.md)
</div>

<div class="card" markdown>
### Phase 3 — Dynamic analysis
Builds APKs, runs DroidBot, diffs the resulting UI Transition Graphs.

[Open →](phase-3-dynamic-analysis.md)
</div>

<div class="card" markdown>
### Phase 4 — Consolidation
Merges static and dynamic evidence into the canonical artifact and computes the risk label.

[Open →](phase-4-consolidation.md)
</div>

<div class="card" markdown>
### Phase 5 — Visualization
CodeCharta JSON, HTML report, PR-comment payload.

[Open →](phase-5-visualization.md)
</div>

</div>
