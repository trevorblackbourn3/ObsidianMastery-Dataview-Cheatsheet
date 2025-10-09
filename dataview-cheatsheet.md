# Dataview Cheatsheet

## Query Types

```dataview
LIST                  # Bulleted list of notes
LIST field            # List with additional info
TABLE field1, field2  # Table with columns
TASK                  # Show checklist items
CALENDAR date-field   # Calendar view
```

---

## Query Structure

```dataview
QUERY_TYPE field1, field2
FROM "folder" OR #tag
WHERE condition
SORT field ASC/DESC
LIMIT number
```

---

## FROM Sources

```dataview
FROM "Projects"                    # Single folder
FROM "Projects" OR "Archive"       # Multiple folders
FROM #tag                          # By tag
FROM "Projects" AND !"Archive"     # Exclude subfolder
FROM [[Specific Note]]             # Single note
```

---

## WHERE Filters

| Operator | Example | Description |
|----------|---------|-------------|
| `=` | `status = "active"` | Equals |
| `!=` | `status != "done"` | Not equals |
| `>` | `priority > 3` | Greater than |
| `<` | `deadline < date(today)` | Less than |
| `>=` | `rating >= 4` | Greater or equal |
| `<=` | `pages_read <= 100` | Less or equal |
| `AND` | `status = "active" AND priority = "high"` | Both conditions |
| `OR` | `status = "active" OR status = "review"` | Either condition |
| `contains()` | `contains(tags, "urgent")` | Contains value |
| `!field` | `!deadline` | Field doesn't exist |
| `field` | `deadline` | Field exists |

---

## File Metadata (Implicit Fields)

| Field | Description |
|-------|-------------|
| `file.name` | Note title |
| `file.folder` | Folder path |
| `file.path` | Full file path |
| `file.size` | File size (bytes) |
| `file.ctime` | Creation time |
| `file.mtime` | Modification time |
| `file.day` | Date (for YYYY-MM-DD daily notes) |
| `file.tags` | All tags in note |
| `file.outlinks` | Links going out |
| `file.inlinks` | Backlinks to this note |

---

## Date Functions

```dataview
date(today)                 # Today's date
date(2026-01-15)           # Specific date
dur(7 days)                # Duration
date(today) + dur(1 week)  # Date arithmetic
date(today) - dur(30 days) # 30 days ago
```

**Date arithmetic:**
```dataview
WHERE deadline < date(today)              # Overdue
WHERE deadline = date(today)              # Due today
WHERE deadline > date(today)              # Future
WHERE deadline <= date(today) + dur(7d)   # Due this week
```

---

## Math Functions

```dataview
round(3.7)           # Round to integer (4)
sum(rows.field)      # Sum all values in group
length(list)         # Count items in list
min(list)            # Minimum value
max(list)            # Maximum value
```

---

## String Functions

```dataview
contains(string, "text")    # Check if contains substring
contains(file.name, "2025") # Files with "2025" in name
split(string, "-")          # Split string into array
```

---

## Sorting

```dataview
SORT deadline ASC             # Ascending (earliest first)
SORT deadline DESC            # Descending (latest first)
SORT priority DESC, name ASC  # Multiple fields
SORT file.mtime DESC          # Most recently modified
```

---

## Grouping

```dataview
GROUP BY status                        # Group by field
GROUP BY owner                         # Group by owner

# Display grouped results
TABLE rows.file.link AS "Items", count(rows) AS "Count"
FROM "Projects"
GROUP BY status
```

**Accessing grouped data:**
- `rows` = all items in group
- `rows.file.link` = links to all files
- `count(rows)` = number of items
- `rows.field` = specific field from all items

---

## Inline Fields (In Note Body)

```markdown
Status:: active
Priority:: high
Deadline:: 2026-01-15
Owner:: Sarah
Tags:: project, urgent
```

**Use `Key:: Value` syntax anywhere in note body**

---

## Practical Examples

### All active projects
```dataview
TABLE status, deadline, owner
FROM "Projects"
WHERE status = "active"
SORT deadline ASC
```

### Overdue tasks
```dataview
TASK
FROM "Projects"
WHERE !completed AND due < date(today)
SORT due ASC
```

### Recently modified notes
```dataview
LIST file.mtime AS "Last Modified"
FROM "Notes"
SORT file.mtime DESC
LIMIT 10
```

### Books finished this year
```dataview
TABLE finished, rating
FROM "Books"
WHERE status = "finished" AND finished >= date(2025-01-01)
SORT finished DESC
```

### Projects by status
```dataview
TABLE rows.file.link AS "Projects", count(rows) AS "Count"
FROM "Projects"
GROUP BY status
```

### High priority items due this week
```dataview
TABLE deadline, priority
FROM "Projects" OR "Tasks"
WHERE priority = "high"
  AND deadline >= date(today)
  AND deadline <= date(today) + dur(7 days)
SORT deadline ASC
```

---

## Common Mistakes

| ❌ Wrong | ✅ Right |
|---------|----------|
| `FROM Projects` | `FROM "Projects"` (needs quotes) |
| `WHERE Status = "active"` | `WHERE status = "active"` (case-sensitive) |
| `WHERE deadline < 2026-01-15` | `WHERE deadline < date(2026-01-15)` (use date()) |
| Forgetting to check field exists | `WHERE deadline AND deadline < date(today)` |

---

## DataviewJS Quick Reference

```dataviewjs
// Get pages
let pages = dv.pages('"Projects"');

// Filter
let active = pages.where(p => p.status === "active");

// Sort
let sorted = active.sort(p => p.deadline, 'asc');

// Display as list
dv.list(pages.file.link);

// Display as table
dv.table(
  ["Name", "Status", "Deadline"],
  pages.map(p => [p.file.link, p.status, p.deadline])
);

// Display paragraph
dv.paragraph("Some text");

// Loop through pages
for (let page of pages) {
  dv.paragraph(page.file.name);
}

// Count tasks
let taskCount = page.file.tasks.where(t => !t.completed).length;
```

---

## Tips

1. **Always use quotes** around folder names in FROM clause
2. **Use lowercase** for field names (consistency matters)
3. **Check field existence** before comparing values
4. **Use date() function** for all date comparisons
5. **Test queries incrementally** - add one filter at a time
6. **Use LIMIT** while testing to avoid overwhelming output

---

## Quick Test Query

Want to see all notes with a specific field? Try:

```dataview
TABLE status
FROM ""
WHERE status
```

This shows every note in your vault that has a `status` field.
