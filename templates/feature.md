---
description: Shippable piece of work
schema:
  tasks:
    type: relationship
    template: task
    multiple: true
    required: false
    description: Related task items
    label: Tasks
  status:
    type: text
    options:
    - value: Planned
      label: Planned
      display:
        color: '#64748b'
        icon: IconCircleDashed
    - value: In progress
      label: In progress
      display:
        color: '#3b82f6'
        icon: IconProgress
    - value: In review
      label: In review
      display:
        color: '#8b5cf6'
        icon: IconEyeCheck
    - value: Shipped
      label: Shipped
      display:
        color: '#22c55e'
        icon: IconRocket
    - value: Cancelled
      label: Cancelled
      display:
        color: '#64748b'
        icon: IconCircleX
    multiple: false
    required: true
    defaultValue: Planned
    description: Current state of the feature
    label: Status
    hints:
    - pinned
  assignee:
    type: user
    multiple: false
    required: false
    description: Person responsible for delivery
    label: Assignee
    hints:
    - pinned
  priority:
    type: text
    options:
    - value: High
      label: High
      display:
        color: '#ef4444'
        icon: IconFlagFilled
    - value: Medium
      label: Medium
      display:
        color: '#f97316'
        icon: IconFlagFilled
    - value: Low
      label: Low
      display:
        color: '#22c55e'
        icon: IconFlagFilled
    multiple: false
    required: false
    description: Feature priority
    label: Priority
    hints:
    - pinned
---
# {{title}}

Describe the feature goal and user value.

## Requirements

- Requirement 1
- Requirement 2

## Notes

Discussion, tradeoffs, or related links.
