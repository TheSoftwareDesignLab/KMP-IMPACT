# Guides

Task-oriented walkthroughs. Each guide assumes you have already completed [Getting Started](../getting-started/index.md).

<div class="kmp-callout" markdown>
**New project? Start at the top.** The order below mirrors the typical adoption path: prepare your repo, wire CI, understand the flow, then go deep on the day-to-day tasks.
</div>

<div class="kmp-grid" markdown>

<div class="card" markdown>
### Preparing an existing KMP project
Nine-step checklist that turns an arbitrary KMP repository into one that meets every prerequisite — before you touch the install guide.

[Open →](preparing-a-kmp-project.md)
</div>

<div class="card" markdown>
### Configuring GitHub Actions
Drop the workflow into a target repo, vendor the analyzer, enable GitHub Pages, set permissions.

[Open →](github-actions.md)
</div>

<div class="card" markdown>
### Configuring Dependabot
30-second primer plus the biasing rules that keep the AFTER APK compiling.

[Open →](dependabot.md)
</div>

<div class="card" markdown>
### How everything talks to each other
End-to-end walkthrough of the Dependabot → workflow → analyzer → report flow, with a sequence diagram.

[Open →](how-it-works.md)
</div>

<div class="card" markdown>
### Analyzing a bump locally
Run the analyzer against any KMP repository on your machine — static-only or with the dynamic phase.

[Open →](analyzing-locally.md)
</div>

<div class="card" markdown>
### Writing Scenarios
Author YAML scenarios with ground truth for benchmarks and regression suites.

[Open →](scenarios.md)
</div>

<div class="card" markdown>
### Reading the Report
Navigate the six tabs of the HTML report and the PR comment payload.

[Open →](reading-the-report.md)
</div>

<div class="card" markdown>
### Diagnosing a BLOCKED phase
Triage flow for the most common failure modes — APK won't build, no UTG, Pages cancelled.

[Open →](diagnosing-blocked.md)
</div>

</div>
