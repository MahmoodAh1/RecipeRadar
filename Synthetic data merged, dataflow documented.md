### Dataset Preparation & Dataflow

### Dataset Overview
RecipeRadar uses real-world recipe data sourced and cleaned from public recipe datasets and some synthetic data combined into a single combined.csv file. The data was then processed and imported into MySQL.
### Row Counts & Data Sources

| Table | Row Count | Source |
|-------|-----------|--------|
| recipes | 14,349 | combined.csv — real recipe data |
| ingredients | ~100,000+ | combined.csv — ingredients_json column |
| cuisines | 40 | Derived from recipe data |
| ingredient_catalog | ~500 | Extracted from ingredients |
| ingredient_aliases | ~200 | Manually curated mappings |
| users | Live data | Created on login |
| favorites / ratings / history | Live data | Created by user interaction |

### Data Cleaning Steps
•	Removed 77 cuisines that had no associated recipes
•	Deleted 1 invalid cuisine entry named 'In a medium saucepan' (instruction text imported as cuisine name)
•	Removed recipes with no ingredients (orphan records from incomplete imports)
•	Re-imported ~13,000 recipes whose ingredients were missing using fix_ingredients.py script
•	ingredient name_lc column auto-generated using MySQL GENERATED ALWAYS AS (LOWER(name)) STORED
•	All ingredient names normalized to lowercase for case-insensitive matching
•	Duplicate recipe names handled by matching on (cuisine, name) pair

### CSV Files Per Table
The following CSV files can be exported from MySQL Workbench using Table Data Export:

| CSV File | Export Query |
|----------|--------------|
| cuisines.csv | `SELECT * FROM cuisines;` |
| recipes.csv | `SELECT * FROM recipes;` |
| ingredients.csv | `SELECT id, recipe_id, name, amount, unit, quantity_text, importance FROM ingredients;` |
| ingredient_catalog.csv | `SELECT * FROM ingredient_catalog;` |
| ingredient_aliases.csv | `SELECT * FROM ingredient_aliases;` |
| users.csv | `SELECT * FROM users;` |
| pantry_items.csv | `SELECT * FROM pantry_items;` |
| favorites.csv | `SELECT * FROM favorites;` |
| ratings.csv | `SELECT * FROM ratings;` |
| history.csv | `SELECT * FROM history;` |

### Dataflow Description
## How Data Moves Through the System
The system operates across three layers: Browser (Frontend), Flask Server (Backend), and MySQL (Database). Below is the complete dataflow for each major feature:
## User Registration / Login
•	User enters username in browser
•	Frontend sends POST /api/login with {username}
•	Flask queries: SELECT id FROM users WHERE username = ?
•	If not found: INSERT INTO users (username) VALUES (?)
•	User ID stored in Flask server-side session
•	Response: {user_id, username} returned to browser
## Recipe Search
•	User types keyword and selects filters
•	Frontend sends GET /api/recipes/search?q=chicken&cuisine=Italian&meal_type=Dinner&page=1
•	Flask builds dynamic SQL with LIKE and JOIN to cuisines table
•	Returns paginated JSON list of {id, name, cuisine, meal_type, servings}
•	User clicks a recipe → GET /api/recipes/{id}
•	Flask calls load_recipe_full() which JOINs recipes, cuisines, ingredients, ingredient_catalog, ingredient_aliases
•	Returns complete recipe object with full ingredients list and instructions
## Ingredient Matching
•	User enters ingredients (e.g., chicken, garlic, tomato) and selects cuisine
•	Frontend sends POST /api/match-ingredients with {ingredients[], cuisine}
•	Flask normalizes each ingredient via ingredient_aliases lookup
•	Fetches all recipes + ingredients for the selected cuisine
•	Python loop counts matching ingredients per recipe using word-boundary regex
•	Returns best match with match_count, total_ing, provided[], missing[]

## Meal Planning
•	User selects cuisine (or random) → GET /api/daily-plan
•	Flask randomly picks from cuisines that HAVE recipes (JOIN-filtered)
•	Runs 3 separate random queries: one per meal_type (Breakfast/Lunch/Dinner)
•	Each result stored in history table to avoid repetition
•	Dessert query uses MySQL REGEXP to find sweet-named recipes
•	Full plan returned as JSON, recipe names rendered as clickable links
## Shopping List
•	User clicks Shopping List on a recipe
•	Flask calls GET /api/shopping-list/{recipe_id}
•	load_recipe_full() fetches all recipe ingredients with categories
•	Flask queries pantry_items for user's saved items
•	Python compares recipe ingredients vs pantry → splits into need[] and have[]
•	need[] grouped by ingredient_catalog.category
•	Returns categorized shopping list JSON
 
