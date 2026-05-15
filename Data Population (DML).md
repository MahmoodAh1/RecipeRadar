### Database Population & Validation

### Population Method
The database was populated using a combination of Python scripts and direct SQL INSERT statements. The primary data source was the combined.csv file containing 14,882 real recipe records.
### Step 1 — Load Cuisines
```sql
INSERT IGNORE INTO cuisines (name) VALUES
('American'), ('Italian'), ('Japanese'), ('Indian'),
('Mexican'), ('Chinese'), ('French'), ('Greek'), ...;
```

### Step 2 — Load Recipes from CSV
Recipes were loaded using a Python import script that read `combined.csv` and executed parameterized INSERT statements:

```sql
INSERT INTO recipes (cuisine_id, name, instructions, servings, meal_type, difficulty)
VALUES (%s, %s, %s, %s, %s, %s);
```

### Step 3 — Load Ingredients (fix_ingredients.py)
A dedicated Python script (`fix_ingredients.py`) was written to re-import missing ingredients:

- Queried DB for all recipes with 0 ingredients (found 13,276)
- Read `combined.csv` and matched recipes by (cuisine, name) pair
- Parsed `ingredients_json` column containing structured ingredient objects
- Inserted each ingredient with: name, amount, unit, quantity_text, importance
- Committed in batches of 500 recipes for performance
- Result: 133,000+ ingredient rows inserted

```sql
INSERT INTO ingredients (recipe_id, name, amount, unit, quantity_text, importance)
VALUES (%s, %s, %s, %s, %s, %s);
```
### Step 4 — Data Cleanup

```sql
-- Remove empty cuisines
SET SQL_SAFE_UPDATES = 0;
DELETE FROM cuisines WHERE id NOT IN
    (SELECT DISTINCT cuisine_id FROM recipes);

-- Remove invalid cuisine entry
DELETE FROM recipes WHERE cuisine_id =
    (SELECT id FROM cuisines WHERE name = 'In a medium saucepan');
DELETE FROM cuisines WHERE name = 'In a medium saucepan';
```

### Validation Queries
#### Row Count Validation

| Validation Query | Expected Result |
|------------------|-----------------|
| `SELECT COUNT(*) FROM cuisines;` | 40 |
| `SELECT COUNT(*) FROM recipes;` | 14,349 |
| `SELECT COUNT(*) FROM ingredients;` | ~133,000+ |
| `SELECT COUNT(*) FROM recipes WHERE meal_type = 'Breakfast';` | 957 |
| `SELECT COUNT(*) FROM recipes WHERE meal_type = 'Lunch';` | 1,720 |
| `SELECT COUNT(*) FROM recipes WHERE meal_type = 'Dinner';` | 8,841 |
| `SELECT COUNT(*) FROM recipes WHERE meal_type = 'Dessert';` | 3,109 |

### NULL Check Validation

```sql
-- Check for NULL recipe names
SELECT COUNT(*) FROM recipes WHERE name IS NULL;  -- Expected: 0

-- Check for NULL cuisine_id
SELECT COUNT(*) FROM recipes WHERE cuisine_id IS NULL;  -- Expected: 0

-- Check for NULL ingredient names
SELECT COUNT(*) FROM ingredients WHERE name IS NULL;  -- Expected: 0

-- Check for NULL user IDs in pantry
SELECT COUNT(*) FROM pantry_items WHERE user_id IS NULL;  -- Expected: 0
```

### Foreign Key Integrity Validation

```sql
-- Orphan ingredients (no matching recipe)
SELECT COUNT(*) FROM ingredients i
WHERE NOT EXISTS (SELECT 1 FROM recipes r WHERE r.id = i.recipe_id);
-- Expected: 0

-- Orphan recipes (no matching cuisine)
SELECT COUNT(*) FROM recipes r
WHERE NOT EXISTS (SELECT 1 FROM cuisines c WHERE c.id = r.cuisine_id);
-- Expected: 0

-- Orphan favorites (no matching user)
SELECT COUNT(*) FROM favorites f
WHERE NOT EXISTS (SELECT 1 FROM users u WHERE u.id = f.user_id);
-- Expected: 0

-- Ratings out of range
SELECT COUNT(*) FROM ratings WHERE rating < 1 OR rating > 5;
-- Expected: 0
```

### Recipes with Ingredients Validation

```sql
-- Confirm all recipes now have at least one ingredient
SELECT COUNT(*) AS recipes_with_ingredients
FROM recipes r
WHERE EXISTS (SELECT 1 FROM ingredients i WHERE i.recipe_id = r.id);
-- Expected: 14,349 (all recipes)

-- Top recipes by ingredient count
SELECT r.name, COUNT(i.id) AS ing_count
FROM recipes r
JOIN ingredients i ON i.recipe_id = r.id
GROUP BY r.id ORDER BY ing_count DESC LIMIT 5;
```

### Cuisine Distribution Validation

```sql
SELECT c.name, r.meal_type, COUNT(*) AS cnt
FROM recipes r
JOIN cuisines c ON c.id = r.cuisine_id
GROUP BY c.name, r.meal_type
ORDER BY c.name, r.meal_type;
```

### Final Database Statistics

| Metric | Value |
|--------|-------|
| Total Recipes | 14,349 |
| Total Ingredients | 133,000+ |
| Total Cuisines | 40 |
| Avg Ingredients per Recipe | ~9 |
| Recipes with Breakfast | 957 |
| Recipes with Lunch | 1,720 |
| Recipes with Dinner | 8,841 |
| Recipes with Dessert | 3,109 |
| Total Tables | 10 |
| Normalization Level | 3NF |
