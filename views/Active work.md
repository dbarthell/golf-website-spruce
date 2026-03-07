---
searchQuery:
  customFields:
    status:
      type: in
      values:
      - Todo
      - In Progress
      - In Review
searchOptions:
  groupBy: assignee
  sortBy:
    field: status
    direction: asc
  visibleFields:
  - status
  - priority
  - size
  - templateType
  - tasks
  showEmptyGroups: false
  showNoValueGroup: false
---
# Active Work

Everything committed or in-flight, grouped by assignee.