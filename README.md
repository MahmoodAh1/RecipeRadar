# RecipeRadar

🍽️ RecipeRadar
A Pantry-Powered Recipe Finder & Meal Planner

Python  |  Flask  |  MySQL  |  HTML / CSS / JS
 
🍽️ RecipeRadar
A pantry-powered recipe finder and meal planner — search 14,000+ recipes, plan your week, and cook with what you already have.

About the Project

RecipeRadar was built as a DBMS semester project. The goal was to design a fully normalized (3NF) relational database and connect it to a real working web application.
The app solves a simple everyday problem — "What can I cook with what I have?" — by matching your available ingredients against a database of 14,000+ real-world recipes across 40+ world cuisines.

Features

Features

| Feature | Description |
|---------|-------------|
| 🔍 Search by Name | Search recipes with cuisine and meal type filters. Paginated results. |
| 📅 Daily Meal Plan | Generate Breakfast, Lunch, Dinner + optional dessert from any cuisine. |
| 🗓️ Weekly Meal Plan | Plan all 7 days at once. One cuisine or random per day. View full day detail. |
| 🧠 Ingredient Matching | Enter what you have — get the best matching recipe with match percentage. |
| 🫙 My Pantry | Save staple ingredients. Auto-included in ingredient matching. |
| 🛒 Shopping List | Categorized list of what you need to buy vs what you already have. |
| ⭐ Favorites & Ratings | Save recipes and rate them 1–5 stars. |
| 🌍 Browse Cuisines | Explore 40+ world cuisines. Click any to browse its recipes. |
Tech Stack

Backend
•	Python 3.10+
•	Flask (REST API)
•	mysql-connector-python

Database
•	MySQL 8.0
•	Fully normalized to 3NF
•	10 relational tables

Frontend
•	Vanilla HTML, CSS, JavaScript
•	Single-page application (no frameworks)
•	Modal-based navigation
•	Responsive design

Database Schema

The database is normalized to Third Normal Form (3NF). All transitive and partial dependencies have been eliminated.

Table Relationships
```
cuisines
└── recipes                (cuisine_id → cuisines.id)
        └── ingredients         (recipe_id → recipes.id)
                └── ingredient_aliases
                        └── ingredient_catalog

users
├── pantry_items           (user_id → users.id)
├── favorites              (user_id → users.id, recipe_id → recipes.id)
├── ratings                (user_id → users.id, recipe_id → recipes.id)
└── history                (user_id → users.id, recipe_id → recipes.id)
```
Tables
| Table | Description |
|-------|-------------|
| `cuisines` | All cuisine names (Italian, Japanese, etc.) |
| `recipes` | Core recipe data linked to a cuisine |
| `ingredients` | Atomic ingredient rows per recipe |
| `ingredient_catalog` | Canonical ingredient names with shopping categories |
| `ingredient_aliases` | Maps alternate names → canonical (e.g. cilantro → coriander) |
| `users` | User accounts (username-based login) |
| `pantry_items` | User's saved pantry ingredients |
| `favorites` | User's saved favorite recipes |
| `ratings` | User ratings (1–5 stars) per recipe |
| `history` | Recently suggested recipes to avoid repetition |

 
Project Structure

```
RecipeRadar/
│
├── flask_app.py          # Flask backend — all API endpoints
├── app2.py               # Original CLI version of the app
├── index(1).html         # Frontend — single-page application
├── fix_ingredients.py    # Data repair script (re-imports missing ingredients)
├── combined.csv          # Source dataset (14,882 real recipes)
└── README.md
```

Getting Started

Prerequisites
•	Python 3.10+
•	MySQL 8.0
•	pip

Step 1 — Clone the Repository
git clone https://github.com/MahmoodAh1/reciperadar.git
cd reciperadar

Step 2 — Install Dependencies
pip install flask mysql-connector-python

Step 3 — Set Up the Database
Open MySQL Workbench or your MySQL terminal and create the database:
CREATE DATABASE recipe_app;
USE recipe_app;

Then create all tables using the DDL scripts in Milestone 4 documentation, or run the CREATE TABLE statements for all 10 tables in order:
•	cuisines
•	recipes
•	ingredients
•	ingredient_catalog
•	ingredient_aliases
•	users
•	pantry_items
•	favorites
•	ratings
•	history

Step 4 — Configure Database Credentials
Open flask_app.py and update the database config at the top:
DB_CONFIG = {
    "host": "127.0.0.1",
    "port": 3306,
    "user": "root",
    "password": "your_password_here",
    "database": "recipe_app"
}

Step 5 — Import the Dataset
Place combined.csv in the project folder, then run:
python fix_ingredients.py
This will import all 14,000+ recipes and their ingredients into the database. It takes a few minutes.

Step 6 — Run the App
python flask_app.py

Open your browser and go to:
http://127.0.0.1:5000

 
API Endpoints

| Method   | Endpoint | Description |
|--------|----------|-------------|
| `POST`   | `/api/login`              | Login or register a user |
| `POST`   | `/api/logout`             | Logout current user |
| `GET`    | `/api/cuisines`           | Get all cuisines with recipes |
| `GET`    | `/api/recipes/search`     | Search recipes (q, cuisine, meal_type, page) |
| `GET`    | `/api/recipes/<id>`       | Get full recipe detail with ingredients |
| `GET`    | `/api/daily-plan`         | Generate daily meal plan |
| `GET`    | `/api/weekly-plan`        | Generate weekly meal plan |
| `GET`    | `/api/weekly-dessert`     | Get a dessert suggestion |
| `POST`   | `/api/match-ingredients`  | Find best matching recipe for ingredients |
| `GET`    | `/api/shopping-list/<id>` | Get categorized shopping list for a recipe |
| `GET`    | `/api/pantry`             | Get user's pantry items |
| `POST`   | `/api/pantry`             | Add a pantry item |
| `DELETE` | `/api/pantry/<name>`      | Remove a pantry item |
| `GET`    | `/api/favorites`          | Get user's saved favorites |
| `POST`   | `/api/favorites/<id>`     | Toggle favorite (save/unsave) |
| `POST`   | `/api/rate/<id>`          | Rate a recipe 1–5 stars |

 
How It Works

Ingredient Matching Algorithm
1.	User enters ingredients and selects a cuisine
2.	Each ingredient is normalized via the ingredient_aliases table
3.	All recipes in the cuisine are fetched with their ingredients
4.	Python loops through every recipe counting how many user ingredients appear using word-boundary matching
5.	Recipe with the highest match count is returned along with match %, provided items, and missing items

Meal Planning
•	Random cuisine selection only picks from cuisines that actually have recipes (JOIN-filtered)
•	Dessert detection uses MySQL REGEXP to find sweet-named recipes (cake, pie, ice cream, etc.)
•	History table prevents the same recipe from being suggested repeatedly

Shopping List
•	Fetches all recipe ingredients via load_recipe_full()
•	Compares against user's pantry items from pantry_items table
•	Splits into need[] (grouped by ingredient category) and have[] lists
•	Returns categorized JSON with checkboxes for each item

Session Management
•	Flask uses server-side sessions to track logged-in users
•	Every API call reads session user_id for personalized data
•	Covers pantry, favorites, ratings, and history per user

Dataset

|         Metric         |    Value     |            Notes              |
|--------|-------|-------|
| Source                 | combined.csv | Real-world recipe data        |
| Total CSV rows         | 14,882       | Before cleaning               |
| Recipes imported       | 14,349       | After removing empty cuisines |
| Total ingredients      | 133,000+     | Across all recipes            |
| Cuisines               | 40           | After removing empty ones     |
| Avg ingredients/recipe | ~9           | Calculated from DB            |
| Breakfast recipes      | 957          | meal_type = Breakfast         |
| Lunch recipes          | 1,720        | meal_type = Lunch             |
| Dinner recipes         | 8,841        | meal_type = Dinner            |
| Dessert recipes        | 3,109        | meal_type = Dessert           |
Team

Built by:

Name	                      GitHub	           Role
[Mahmood Ahmad Sajjad]	@MahmoodAh1	Backend, Database, Frontend
   [Hoorain Saud]	      @Hoorain630	Backend, Database, Frontend



RecipeRadar — because great meals start with what's already in your kitchen.
