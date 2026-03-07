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
  - size
  - assignee
  - templateType
searchQuery:
  customFields:
    status:
      type: in
      values:
      - Backlog
---
# Prioritization

Saved view with current search configuration.