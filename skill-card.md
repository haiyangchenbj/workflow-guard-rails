## Description:

Wraps multi-step agent workflows with pre-execution checks, checkpoints, side-effect queues, bounded retries, result validation, audit logs, and staged failure rules.

This skill is ready for commercial/non-commercial use.

## Publisher:

[haiyangchenbj](https://clawhub.ai/user/haiyangchenbj)

### License/Terms of Use:

MIT-0

## Use Case:

Developers and operators use this skill to guard unattended or multi-step agent workflows that send, publish, pay, delete, or write to external systems, and to make failures safer to retry and review.

### Deployment Geography for Use:

Global

## Known Risks and Mitigations:

Risk: Persistent failure rules outside the skill can influence later unattended runs.

Mitigation: Inspect staged rules before promotion, limit permissions on ~/.workbuddy, and review the authoritative playbook regularly.

Risk: Audit and failure entries may retain sensitive workflow data.

Mitigation: Redact secrets before recording audit or failure entries.

Risk: The external companion playbook may introduce unreviewed guidance.

Mitigation: Review the pinned download and verify its contents before using it in a workflow.

## Reference(s):

- [Workflow Guard Rails release](https://clawhub.ai/haiyangchenbj/skills/workflow-guard-rails)
- [Guardian patterns](references/guardian-patterns.md)
- [Pinned companion error playbook](https://raw.githubusercontent.com/haiyangchenbj/error-playbook/d945aee01384c6163beb287446147ce768d50661/ERROR-PLAYBOOK.md)

## Skill Output:

**Output Type(s):** [Markdown, Guidance, Configuration instructions]

**Output Format:** [Markdown guardian report and workflow guidance]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [Reports checks, queued and executed side effects, validation, retries, audit status, staged rules, and release or hold verdict.]

## Skill Version(s):

1.0.6 (source: SKILL.md frontmatter and server release)

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
