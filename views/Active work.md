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
  - modified
  showEmptyGroups: true
  showNoValueGroup: true
display:
  icon: bolt
---
# Active Work

Everything committed or in-flight, grouped by assignee.