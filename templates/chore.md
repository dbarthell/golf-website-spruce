---
display:
  color: '#64748b'
  icon: tool
schema:
  assignee:
    description: Person responsible for this work
    hints:
    - pinned
    label: Assignee
    multiple: false
    required: false
    type: user
  priority:
    description: How urgent this work is
    hints:
    - pinned
    label: Priority
    multiple: false
    options:
    - description: Needs immediate attention
      display:
        color: '#ef4444'
        icon: alert-triangle
      label: Urgent
      value: Urgent
    - description: Important, prioritize soon
      display:
        color: '#f97316'
        icon: flag-filled
      label: High
      value: High
    - description: Normal priority
      display:
        color: '#eab308'
        icon: flag-filled
      label: Medium
      value: Medium
    - description: Can wait
      display:
        color: '#22c55e'
        icon: flag-filled
      label: Low
      value: Low
    required: false
    type: text
  size:
    description: Effort estimation
    hints:
    - pinned
    label: Size
    multiple: false
    options:
    - description: 1-2h (1 pt)
      display:
        color: '#22c55e'
        icon: play-card-1-filled
      label: S
      value: S
    - description: 2-4h (2 pts)
      display:
        color: '#3b82f6'
        icon: play-card-2-filled
      label: M
      value: M
    - description: 1-2d (4 pts)
      display:
        color: '#f97316'
        icon: play-card-4-filled
      label: L
      value: L
    - description: 3-5d (8 pts)
      display:
        color: '#ef4444'
        icon: play-card-8-filled
      label: XL
      value: XL
    required: false
    type: text
  status:
    defaultValue: Backlog
    description: Current state of the work
    hints:
    - pinned
    label: Status
    multiple: false
    options:
    - display:
        color: '#64748b'
        icon: circle-dashed
      label: Backlog
      value: Backlog
    - display:
        color: '#f97316'
        icon: circle-dot
      label: Todo
      value: Todo
    - display:
        color: '#3b82f6'
        icon: progress
      label: In Progress
      value: In Progress
    - display:
        color: '#8b5cf6'
        icon: eye-check
      label: In Review
      value: In Review
    - display:
        color: '#22c55e'
        icon: circle-check
      label: Done
      value: Done
    - display:
        color: '#64748b'
        icon: circle-x
      label: Cancelled
      value: Cancelled
    required: true
    type: text
---
Internal work with no direct user impact — refactoring, tech debt cleanup, dependency upgrades, OE, CI/CD, documentation.