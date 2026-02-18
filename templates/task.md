---
description: Work item needed to deliver a feature
schema:
  priority:
    type: text
    options:
    - value: Urgent
      label: Urgent
      description: Needs immediate attention
      display:
        color: '#ef4444'
        icon: IconAlertTriangle
    - value: High
      label: High
      description: Important, prioritize soon
      display:
        color: '#f97316'
        icon: IconFlagFilled
    - value: Medium
      label: Medium
      description: Normal priority
      display:
        color: '#eab308'
        icon: IconFlagFilled
    - value: Low
      label: Low
      description: Can wait
      display:
        color: '#22c55e'
        icon: IconFlagFilled
    multiple: false
    required: false
    description: Task urgency
    label: Priority
    hints:
    - pinned
  assignee:
    type: user
    multiple: false
    required: false
    description: Person responsible for this work
    label: Assignee
    hints:
    - pinned
  status:
    type: text
    options:
    - value: Todo
      label: Todo
      display:
        color: '#64748b'
        icon: IconCircleDashed
    - value: In progress
      label: In progress
      display:
        color: '#3b82f6'
        icon: IconProgress
    - value: Done
      label: Done
      display:
        color: '#22c55e'
        icon: IconCircleCheck
    multiple: false
    required: false
    description: Current state of the work
    label: Status
    hints:
    - pinned
---
# {{title}}

Describe the task and what "done" looks like.

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Notes

Implementation details or blockers.
