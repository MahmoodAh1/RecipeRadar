Recipe Radar
Relational Database Schema Description
1. Database Overview
The recipe app database powers the Recipe Radar application. It consists of 10 interrelated tables organized into four functional groups:
• Core Data: cuisines, recipes, ingredients “ the foundational recipe content”.
• Ingredient Management: ingredient_catalog, ingredient_aliases.
• User Interaction: users, pantry_items, favorites, ratings, history,  “personalization features including pantry tracking, favorites, ratings  and usage history”.
2. Table Definitions


### users
Stores registered user accounts for the application.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| id | INT AUTO_INCREMENT | PK | NOT NULL | Unique user identifier |
| username | VARCHAR(60) | UK | NOT NULL | Unique display name |


Relationships:
• One-to-Many with pantry_items, favorites, ratings, history

### cuisines
Lookup table for world cuisines (66 entries including Pakistani, Italian, etc.).

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| id | INT AUTO_INCREMENT | PK | NOT NULL | Unique cuisine identifier |
| name | VARCHAR(80) | UK | NOT NULL | Cuisine name (unique) |


Relationships:
• One-to-Many with recipes (each recipe belongs to one cuisine)


### recipes
Core table storing all recipe data including instructions, servings, meal type, and difficulty.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| id | INT AUTO_INCREMENT | PK | NOT NULL | Unique recipe identifier |
| cuisine_id | INT | FK | NOT NULL | References cuisines(id) |
| name | VARCHAR(120) | | NOT NULL | Recipe name |
| instructions | TEXT | | NULL | Step-by-step cooking instructions |
| servings | INT | | NOT NULL | Number of servings (default 1) |
| meal_type | VARCHAR(50) | | NULL | Breakfast, Lunch, Dinner, Snack, Dessert |
| difficulty | VARCHAR(50) | | NULL | Easy, Medium, Hard |


Relationships:
• Many-to-One with cuisines (FK: cuisine_id → cuisines.id, ON DELETE CASCADE)
• One-to-Many with ingredients, favorites, ratings, history.
• Unique constraint on (cuisine_id, name)

### ingredients
Stores ingredients for each recipe with quantity details and importance ranking.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| id | INT AUTO_INCREMENT | PK | NOT NULL | Unique ingredient entry identifier |
| recipe_id | INT | FK | NOT NULL | References recipes(id) |
| name | VARCHAR(120) | | NOT NULL | Ingredient name |
| name_lc | VARCHAR(120) | | Generated | Lowercase version (stored/computed) |
| amount | DECIMAL(10,2) | | NULL | Numeric quantity |
| unit | VARCHAR(20) | | NULL | Measurement unit (g, ml, etc.) |
| quantity_text | VARCHAR(80) | | NULL | Display text (e.g. '2 tbsp') |
| importance | TINYINT | | NOT NULL | Priority ranking 1–5 (default 3) |


Relationships:
• Many-to-One with recipes (FK: recipe_id → recipes.id, ON DELETE CASCADE)


### ingredient_catalog
Master catalog of canonical ingredient names with category classification.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| canonical_name | VARCHAR(120) | PK | NOT NULL | Standard ingredient name |
| canonical_lc | VARCHAR(120) | | Generated | Lowercase version (stored/computed) |
| category | ENUM | | NOT NULL | Produce, Meat, Dairy, Grains, Spices, Canned, Frozen, Other |


Relationships:
• One-to-Many with ingredient_aliases, pantry_items.

### ingredient_aliases
Maps alternate ingredient names to their canonical form (e.g. 'tomatoes' → 'tomato').

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| alias | VARCHAR(120) | PK | NOT NULL | Alternate ingredient name |
| alias_lc | VARCHAR(120) | | Generated | Lowercase alias (stored/computed) |
| canonical_name | VARCHAR(120) | FK | NOT NULL | References ingredient_catalog |


Relationships:
• Many-to-One with ingredient_catalog (FK: canonical_name, ON DELETE CASCADE)


### pantry_items
Tracks which ingredients a user currently has available at home.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| user_id | INT | PK, FK | NOT NULL | References users(id) |
| canonical_name | VARCHAR(120) | PK, FK | NOT NULL | References ingredient_catalog |


Relationships:
• Many-to-One with users (ON DELETE CASCADE)
• Many-to-One with ingredient_catalog (ON DELETE CASCADE)


### favorites
Junction table for user's favorite recipes.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| user_id | INT | PK, FK | NOT NULL | References users(id) |
| recipe_id | INT | PK, FK | NOT NULL | References recipes(id) |


Relationships:
• Many-to-One with users (ON DELETE CASCADE)
• Many-to-One with recipes (ON DELETE CASCADE)

### ratings
Stores user ratings for recipes with automatic timestamp tracking.

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| user_id | INT | PK, FK | NOT NULL | References users(id) |
| recipe_id | INT | PK, FK | NOT NULL | References recipes(id) |
| rating | TINYINT | | NOT NULL | Numeric rating value |
| updated_at | TIMESTAMP | | NOT NULL | Auto-updated on change |


Relationships:
• Many-to-One with users (ON DELETE CASCADE)
• Many-to-One with recipes (ON DELETE CASCADE)

### history
Logs each time a user views or cooks a recipe (allows multiple entries per user-recipe pair).

| Attribute | Data Type | Key | Nullable | Description |
|-----------|-----------|-----|----------|-------------|
| user_id | INT | FK | NOT NULL | References users(id) |
| recipe_id | INT | FK | NOT NULL | References recipes(id) |
| used_at | TIMESTAMP | | NOT NULL | Timestamp of usage |


Relationships:
• Many-to-One with users (ON DELETE CASCADE)
• Many-to-One with recipes (ON DELETE CASCADE)



3. Key Conventions
• PK = Primary Key: Uniquely identifies each row in a table.
• FK = Foreign Key: References a primary key in another table, enforcing referential integrity.
• UK = Unique Key: Ensures no duplicate values in the column.
• ON DELETE CASCADE: When a parent row is deleted, all dependent child rows are automatically removed.
• AUTO_INCREMENT: The database engine automatically generates sequential integer values for new rows.
• Generated (Stored): Computed column whose value is derived from another column and physically stored on disk.
