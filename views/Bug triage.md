---
searchOptions:
  groupBy: priority
  showEmptyGroups: true
  showNoValueGroup: true
  sortBy:
    direction: desc
    field: created
  visibleFields:
  - status
  - priority
  - assignee
searchQuery:
  customFields:
    assignee:
      type: in
      values:
      - 9w4k8n
    status:
      type: in
      values:
      - Backlog
      - Todo
  template: bug
---
# Bug triage

Saved view with current search configuration.