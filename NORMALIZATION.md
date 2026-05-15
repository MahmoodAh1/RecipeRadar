### Schema Normalization (1NF → 3NF)

### Overview
The RecipeRadar database was designed from the ground up following relational normalization principles. Starting from a conceptual flat structure, the schema was progressively normalized through 1NF, 2NF, and finally 3NF to eliminate redundancy, ensure data integrity, and support efficient querying across 14,000+ recipes.
### 1NF — First Normal Form
## Definition
A table is in 1NF if: each column contains atomic (indivisible) values, each column contains values of a single type, each row is unique, and there are no repeating groups.
### What the Un-normalized Data Looked Like
Before normalization, a single flat record would look like:

| recipe_name | cuisine | ingredients (repeating) | user_rating |
|-------------|---------|------------------------|-------------|
| Miso Chicken | Japanese | chicken, garlic, miso, butter, salt | 4 |
| Carbonara | Italian | pasta, eggs, bacon, parmesan, pepper | 5 |


## Problems: ingredients column is a multi-valued attribute (not atomic). Multiple values crammed into one cell violate 1NF.

### 1NF Solution
Each ingredient gets its own row. No more comma-separated lists in a single cell.

| recipe_name | cuisine | ingredient_name | ingredient_amount |
|-------------|---------|-----------------|-------------------|
| Miso Chicken | Japanese | chicken | 1 whole |
| Miso Chicken | Japanese | garlic | 3 cloves |
| Miso Chicken | Japanese | miso | 3 tbsp |

Now each row is atomic and there are no repeating groups. Table is in 1NF.

### 2NF Second Normal Form
## Definition
A table is in 2NF if it is in 1NF AND every non-key attribute is fully functionally dependent on the entire primary key (no partial dependencies).

Partial Dependencies Found
In the 1NF table, the composite key is (recipe_name, ingredient_name). But:
•	cuisine depends only on recipe_name (not on ingredient_name) — partial dependency
•	user_rating depends only on recipe_name — partial dependency
•	ingredient_amount depends on the full (recipe_name, ingredient_name) — OK

### 2NF Solution — Split into Separate Tables
Separate tables for recipes, ingredients, users, favorites:

| Table | Primary Key | Attributes |
|-------|-------------|------------|
| recipes | id | cuisine_id, name, meal_type, instructions, servings |
| ingredients | id | recipe_id, name, amount, unit, quantity_text |
| users | id | username, created_at |
| favorites | id | user_id, recipe_id, added_at |


### 3NF — Third Normal Form
## Definition
A table is in 3NF if it is in 2NF AND there are no transitive dependencies (non-key attributes must not depend on other non-key attributes).
## Transitive Dependencies Found in 2NF
•	In the ingredients table: ingredient category depended on ingredient name, not on the ingredient row's primary key — transitive dependency
•	ingredient alias (alternate names like 'cilantro' → 'coriander') depended on ingredient name — transitive dependency
•	In favorites: rating depended on (user_id, recipe_id), not on the favorites row ID — should be its own table

## 3NF Solution — Final Schema
All transitive dependencies removed by creating dedicated tables:








### 3NF Solution — Final Table Justifications

| Table | Justification | Key Columns |
|-------|---------------|-------------|
| cuisines | Eliminates cuisine name repetition in recipes | id PK, name |
| recipes | Core entity. Linked to cuisine via FK | id PK, cuisine_id FK, name, meal_type, instructions, servings |
| ingredients | Atomic ingredient rows per recipe | id PK, recipe_id FK, name, name_lc (generated), amount, unit, quantity_text, importance |
| ingredient_catalog | Canonical ingredient names with category. Eliminates category repetition | id PK, canonical_name, canonical_lc, category |
| ingredient_aliases | Maps alternate names to canonical. Separate from catalog to avoid partial dep | id PK, alias_lc, canonical_name FK |
| users | Stores user accounts | id PK, username, created_at |
| pantry_items | User pantry. References canonical ingredient names | id PK, user_id FK, canonical_name FK, added_at |
| favorites | Many-to-many: users and recipes | id PK, user_id FK, recipe_id FK, added_at |
| ratings | Separated from favorites to avoid transitive dep on (user,recipe) | id PK, user_id FK, recipe_id FK, rating, rated_at |
| history | Tracks meal plan suggestions per user | id PK, user_id FK, recipe_id FK, suggested_at |
 
