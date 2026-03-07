---
display:
  color: '#f97316'
  icon: bug
schema:
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
  assignee:
    description: Person responsible for this work
    hints:
    - pinned
    label: Assignee
    multiple: false
    required: false
    type: user
---
A defect, regression, or unexpected behavior that needs investigation and resolution.