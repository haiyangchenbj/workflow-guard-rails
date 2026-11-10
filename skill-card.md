## Description:

Wrap multi-step agent workflows with pre-execution checks, side-effect queues, result validation, retry budgets, checkpointing, audit logs, and failure-rule accumulation.

This skill is ready for commercial/non-commercial use.

## Publisher:

[haiyangchenbj](https://clawhub.ai/user/haiyangchenbj)

### License/Terms of Use:

MIT-0

## Use Case:

Developers and engineers use this skill to add preflight checks, checkpoint recovery, validation, and controlled handling of side effects to recurring or unattended agent workflows. It helps prevent false success and duplicate external actions while retaining an audit trail.

### Deployment Geography for Use:

Global

## Known Risks and Mitigations:

Risk: Setup may overwrite shared guidance.

Mitigation: Review the pinned companion before use and preserve any existing playbook.

Risk: Automatically accumulated failure rules may affect future workflows.

Mitigation: Require human approval before generated rules become shared guidance.

## Reference(s):

- [Guardian Patterns](references/guardian-patterns.md)
- [Pinned ERROR-PLAYBOOK companion](https://raw.githubusercontent.com/haiyangchenbj/error-playbook/d945aee01384c6163beb287446147ce768d50661/ERROR-PLAYBOOK.md)

## Skill Output:

**Output Type(s):** [Guidance, Markdown, Files]

**Output Format:** [Markdown Guardian Report with checkpoints, side-effect status, validation results, retry status, audit logs, and staged failure rules]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [The side-effect queue only protects actions routed through it. Human confirmation is required for high-privilege or irreversible actions, low-confidence results, exhausted retry budgets, and shared rule promotion.]

## Skill Version(s):

1.0.7 (source: frontmatter)

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
