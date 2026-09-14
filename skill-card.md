## Description: <br>
OpenClaw context check / context optimization (Context Budget). Detects the models currently enabled in the installation, reads each model's published context length from the vendor's official source, and proposes a window at 60% of that value. Nothing is written until the user confirms; then it patches only the window field and reports the result. It never runs automatically and creates no scheduled jobs. <br>

This skill is ready for commercial/non-commercial use. <br>

## Publisher: <br>
[dtsola](https://clawhub.ai/user/dtsola) <br>

### License/Terms of Use: <br>
MIT. <br>

## Use Case: <br>
OpenClaw users who run long agent sessions and want each model's context window set from the vendor's published specification instead of a hand-entered or default value. Typical use: run a context check, review the decision card, and confirm one adjustment; repeat when a model is added or switched. <br>

### Deployment Geography for Use: <br>
Global <br>

## Known Risks and Mitigations: <br>
Risk: The skill writes to model configuration (the context window field) through config.patch. <br>
Mitigation: Only that single field is written; every write requires explicit user confirmation on a decision card showing old and new values; nothing is written without confirmation. <br>
Risk: Values are read from vendor documentation over the network, which can be unavailable or non-canonical. <br>
Mitigation: Sources are labelled on the card with fetch time; if the source is unofficial or unreachable the value is marked as untraced and the skill refuses to write it. <br>
Risk: Changing the window can affect context compaction behaviour in long sessions. <br>
Mitigation: Compaction thresholds are left at system defaults; the suggested value keeps 40% headroom against attention dilution; changes can be undone in the same session using the recorded previous values. <br>
Risk: The skill reads local configuration and the runtime status of the installation. <br>
Mitigation: Reads are limited to configuration and status; no local data is uploaded, and no files are written to disk by this skill. <br>
