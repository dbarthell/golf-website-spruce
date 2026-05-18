---
searchQuery:
  customFields:
    status:
      type: in
      values:
      - Backlog
searchOptions:
  groupBy: priority
  sortBy:
    field: modified
    direction: desc
  visibleFields:
  - status
  - size
  - assignee
  - templateType
  - modified
  showEmptyGroups: true
  showNoValueGroup: true
display:
  icon: flag-filled
---
# Backlog

Items in Backlog status, grouped by priority for planning and grooming.