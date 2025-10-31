# SQL Cheat Sheet - Monty Lookup Chat

Quick reference for common database operations. Copy and paste these queries into Supabase SQL Editor.

## 📋 Table of Contents
- [View Data](#view-data)
- [Add Developer](#add-developer)
- [Add Project](#add-project)
- [Add Items](#add-items)
- [Delete Operations](#delete-operations)
- [Update Operations](#update-operations)
- [Bulk Operations](#bulk-operations)

---

## View Data

### View All Developers
```sql
SELECT name
FROM developers_new
ORDER BY name;
```

### View All Projects by Developer
```sql
SELECT d.name as developer, p.name as project
FROM projects p
JOIN developers_new d ON d.id = p.developer_id
WHERE p.name != 'Company Information'
ORDER BY d.name, p.name;
```

### View All Projects (Including Company Information)
```sql
SELECT d.name as developer, p.name as project
FROM projects p
JOIN developers_new d ON d.id = p.developer_id
ORDER BY d.name, p.name;
```

### View Items for a Specific Project
```sql
SELECT name, type
FROM items
WHERE project_id = (
  SELECT id FROM projects WHERE name = 'Gateway'
)
ORDER BY type, name;
```

### Count Items by Type for a Project
```sql
SELECT
  i.type,
  COUNT(*) as count
FROM items i
JOIN projects p ON p.id = i.project_id
WHERE p.name = 'Gateway'
GROUP BY i.type
ORDER BY i.type;
```

### View Everything (Complete Database State)
```sql
SELECT
  d.name as developer,
  p.name as project,
  COUNT(i.id) as item_count
FROM developers_new d
LEFT JOIN projects p ON p.developer_id = d.id
LEFT JOIN items i ON i.project_id = p.id
GROUP BY d.name, p.name
ORDER BY d.name, p.name;
```

---

## Add Developer

### Add Single Developer
```sql
INSERT INTO developers_new (name)
VALUES ('Damac Properties');
```

### Add Multiple Developers
```sql
INSERT INTO developers_new (name)
VALUES
  ('Damac Properties'),
  ('Dubai Properties'),
  ('Nakheel');
```

### Verify Developer Was Added
```sql
SELECT name FROM developers_new WHERE name = 'Damac Properties';
```

---

## Add Project

### Add Project (Easiest Method)
```sql
INSERT INTO projects (developer_id, name)
SELECT id, 'DAMAC Hills 2'
FROM developers_new
WHERE name = 'Damac Properties';
```

### Add Project to SCC Vertex
```sql
INSERT INTO projects (developer_id, name)
SELECT id, 'Building 309'
FROM developers_new
WHERE name = 'SCC Vertex';
```

### Add Multiple Projects to Same Developer
```sql
INSERT INTO projects (developer_id, name)
SELECT id, project_name
FROM developers_new,
(VALUES
  ('Project Alpha'),
  ('Project Beta'),
  ('Project Gamma')
) AS v(project_name)
WHERE name = 'Premier Choice';
```

### Verify Project Was Added
```sql
SELECT d.name as developer, p.name as project
FROM projects p
JOIN developers_new d ON d.id = p.developer_id
WHERE p.name = 'Building 309';
```

---

## Add Items

### Add Single Floor Plan
```sql
INSERT INTO items (project_id, name, type)
SELECT p.id, 'Floor Plan - A101 - Gateway', 'floor_plan'
FROM projects p
WHERE p.name = 'Gateway';
```

### Add Single Sales Offer
```sql
INSERT INTO items (project_id, name, type)
SELECT p.id, 'Sales Offer - 101 - PHPP - 60/40', 'sales_offer'
FROM projects p
WHERE p.name = 'Gateway';
```

### Add Single Brochure Item
```sql
INSERT INTO items (project_id, name, type)
SELECT p.id, 'Gateway - Brochure', 'brochure'
FROM projects p
WHERE p.name = 'Gateway';
```

### Add Multiple Items at Once
```sql
INSERT INTO items (project_id, name, type)
SELECT p.id, v.name, v.type
FROM projects p,
(VALUES
  ('Floor Plan - A101 - Gateway', 'floor_plan'),
  ('Floor Plan - A102 - Gateway', 'floor_plan'),
  ('Sales Offer - 101 - PHPP', 'sales_offer'),
  ('Gateway - Brochure', 'brochure')
) AS v(name, type)
WHERE p.name = 'Gateway';
```

---

## Delete Operations

### Delete a Developer (WARNING: Deletes all projects and items too!)
```sql
-- First, check what will be deleted:
SELECT
  d.name as developer,
  COUNT(DISTINCT p.id) as project_count,
  COUNT(i.id) as item_count
FROM developers_new d
LEFT JOIN projects p ON p.developer_id = d.id
LEFT JOIN items i ON i.project_id = p.id
WHERE d.name = 'Damac Properties'
GROUP BY d.name;

-- Then delete:
DELETE FROM developers_new WHERE name = 'Damac Properties';
```

### Delete a Project (WARNING: Deletes all items too!)
```sql
-- First, check what will be deleted:
SELECT
  p.name as project,
  COUNT(i.id) as item_count
FROM projects p
LEFT JOIN items i ON i.project_id = p.id
WHERE p.name = 'Building 309'
GROUP BY p.name;

-- Then delete:
DELETE FROM projects WHERE name = 'Building 309';
```

### Delete a Project from Specific Developer
```sql
DELETE FROM projects p
USING developers_new d
WHERE p.developer_id = d.id
  AND d.name = 'SCC Vertex'
  AND p.name = 'Building 309';
```

### Delete Specific Items
```sql
-- Delete all floor plans for a project
DELETE FROM items i
USING projects p
WHERE i.project_id = p.id
  AND p.name = 'Gateway'
  AND i.type = 'floor_plan';

-- Delete a specific item by name
DELETE FROM items
WHERE name = 'Floor Plan - A101 - Gateway';
```

---

## Update Operations

### Rename a Developer
```sql
UPDATE developers_new
SET name = 'Damac Properties LLC'
WHERE name = 'Damac Properties';
```

### Rename a Project
```sql
UPDATE projects
SET name = 'Building 310'
WHERE name = 'Building 309';
```

### Rename an Item
```sql
UPDATE items
SET name = 'Floor Plan - B101 - Gateway'
WHERE name = 'Floor Plan - A101 - Gateway';
```

### Change Item Type
```sql
UPDATE items
SET type = 'sales_offer'
WHERE name = 'Floor Plan - A101 - Gateway';
```

### Move Project to Different Developer
```sql
UPDATE projects
SET developer_id = (
  SELECT id FROM developers_new WHERE name = 'New Developer'
)
WHERE name = 'Gateway';
```

---

## Bulk Operations

### Generate 200 Sales Offers (Pattern: 101-310, Two Types Each)
```sql
INSERT INTO items (project_id, name, type)
SELECT
  p.id,
  'Sales Offer - ' || unit || ' - ' || offer_type,
  'sales_offer'
FROM projects p,
LATERAL (
  SELECT * FROM generate_series(101, 310) AS unit
) units,
LATERAL (
  SELECT * FROM unnest(ARRAY['PHPP - 60/40', '40/60 Standard']) AS offer_type
) types
WHERE p.name = 'Gateway';
```

### Generate 100 Floor Plans (Pattern: A101-A200)
```sql
INSERT INTO items (project_id, name, type)
SELECT
  p.id,
  'Floor Plan - A' || unit || ' - Gateway',
  'floor_plan'
FROM projects p,
generate_series(101, 200) AS unit
WHERE p.name = 'Gateway';
```

### Generate Floor Plans with Multiple Formats
```sql
INSERT INTO items (project_id, name, type)
SELECT
  p.id,
  'Floor Plan - ' || unit_letter || unit_number || ' - Gateway',
  'floor_plan'
FROM projects p,
unnest(ARRAY['A', 'B', 'C']) AS unit_letter,
generate_series(101, 110) AS unit_number
WHERE p.name = 'Gateway';
```

### Delete All Items of a Type from a Project
```sql
DELETE FROM items i
USING projects p
WHERE i.project_id = p.id
  AND p.name = 'Gateway'
  AND i.type = 'floor_plan';
```

### Copy All Items from One Project to Another
```sql
INSERT INTO items (project_id, name, type)
SELECT
  (SELECT id FROM projects WHERE name = 'New Project'),
  name,
  type
FROM items
WHERE project_id = (SELECT id FROM projects WHERE name = 'Gateway');
```

---

## Utility Queries

### Find Developer ID
```sql
SELECT id, name FROM developers_new WHERE name = 'Premier Choice';
```

### Find Project ID
```sql
SELECT id, name FROM projects WHERE name = 'Gateway';
```

### Find All Projects Without Items
```sql
SELECT p.name as project
FROM projects p
LEFT JOIN items i ON i.project_id = p.id
WHERE p.name != 'Company Information'
GROUP BY p.name
HAVING COUNT(i.id) = 0;
```

### Count Total Records
```sql
SELECT
  (SELECT COUNT(*) FROM developers_new) as developers,
  (SELECT COUNT(*) FROM projects) as projects,
  (SELECT COUNT(*) FROM items) as items;
```

### Find Duplicate Items
```sql
SELECT name, COUNT(*) as count
FROM items
GROUP BY name
HAVING COUNT(*) > 1;
```

---

## Common Patterns

### Pattern 1: Add Developer → Add Project → Add Items
```sql
-- Step 1: Add developer
INSERT INTO developers_new (name) VALUES ('New Developer');

-- Step 2: Add project
INSERT INTO projects (developer_id, name)
SELECT id, 'New Project'
FROM developers_new
WHERE name = 'New Developer';

-- Step 3: Add items
INSERT INTO items (project_id, name, type)
SELECT p.id, v.name, v.type
FROM projects p,
(VALUES
  ('New Project - Brochure', 'brochure'),
  ('New Project - Availability', 'availability')
) AS v(name, type)
WHERE p.name = 'New Project';
```

### Pattern 2: Verify Before Delete
```sql
-- Always check first!
SELECT * FROM projects WHERE name = 'Test Project';

-- If safe to delete:
DELETE FROM projects WHERE name = 'Test Project';

-- Verify deletion:
SELECT * FROM projects WHERE name = 'Test Project'; -- Should return nothing
```

### Pattern 3: Bulk Add with Pattern
```sql
-- For projects with predictable naming patterns
INSERT INTO items (project_id, name, type)
SELECT
  p.id,
  format('Floor Plan - %s%s - %s', letter, number, p.name),
  'floor_plan'
FROM projects p,
unnest(ARRAY['A', 'B']) AS letter,
generate_series(101, 110) AS number
WHERE p.name = 'Your Project';
```

---

## Tips

1. **Always verify before deleting** - Use SELECT to check what you're about to delete
2. **Use transactions for complex operations** - Wrap multiple statements in BEGIN/COMMIT
3. **Test queries with LIMIT first** - Add `LIMIT 10` when testing bulk operations
4. **Use WHERE clauses carefully** - Double-check your WHERE conditions
5. **Keep backups** - Supabase has point-in-time recovery, but be careful

## Emergency Recovery

### Restore from Backup (Supabase Dashboard)
1. Go to Settings → Database
2. Click "Point in Time Recovery"
3. Select timestamp before mistake
4. Restore

### Export Current Data (Backup)
```sql
-- Export developers
COPY (SELECT * FROM developers_new) TO STDOUT WITH CSV HEADER;

-- Export projects
COPY (SELECT * FROM projects) TO STDOUT WITH CSV HEADER;

-- Export items
COPY (SELECT * FROM items) TO STDOUT WITH CSV HEADER;
```

---

**Need help with a specific query?** Ask Claude Code - provide the developer, project, and what you want to accomplish!
