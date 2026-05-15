Milestone 4 — CREATE TABLE Statements (DDL)

All tables were created with proper constraints including Primary Keys, Foreign Keys, NOT NULL, UNIQUE, CHECK constraints, and auto-generated columns where applicable.
DDL Scripts
1. cuisines
CREATE TABLE cuisines (
    id   INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uq_cuisine_name (name)
) ENGINE=InnoDB;
2. recipes
CREATE TABLE recipes (
    id           INT NOT NULL AUTO_INCREMENT,
    cuisine_id   INT NOT NULL,
    name         VARCHAR(300) NOT NULL,
    instructions LONGTEXT,
    servings     INT DEFAULT 1,
    meal_type    VARCHAR(50),
    difficulty   VARCHAR(50),
    PRIMARY KEY (id),
    CONSTRAINT fk_recipe_cuisine FOREIGN KEY (cuisine_id)
        REFERENCES cuisines(id) ON DELETE CASCADE
) ENGINE=InnoDB;
3. ingredients
CREATE TABLE ingredients (
    id            INT NOT NULL AUTO_INCREMENT,
    recipe_id     INT NOT NULL,
    name          VARCHAR(120) NOT NULL,
    name_lc       VARCHAR(120) GENERATED ALWAYS AS (LOWER(name)) STORED,
    amount        FLOAT DEFAULT 0,
    unit          VARCHAR(50),
    quantity_text VARCHAR(200),
    importance    INT DEFAULT 3,
    PRIMARY KEY (id),
    CONSTRAINT fk_ing_recipe FOREIGN KEY (recipe_id)
        REFERENCES recipes(id) ON DELETE CASCADE
) ENGINE=InnoDB;
4. ingredient_catalog
CREATE TABLE ingredient_catalog (
    id             INT NOT NULL AUTO_INCREMENT,
    canonical_name VARCHAR(120) NOT NULL,
    canonical_lc   VARCHAR(120) GENERATED ALWAYS AS (LOWER(canonical_name)) STORED,
    category       VARCHAR(80),
    PRIMARY KEY (id),
    UNIQUE KEY uq_canonical (canonical_name)
) ENGINE=InnoDB;
5. ingredient_aliases
CREATE TABLE ingredient_aliases (
    id             INT NOT NULL AUTO_INCREMENT,
    alias_lc       VARCHAR(120) NOT NULL,
    canonical_name VARCHAR(120) NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uq_alias (alias_lc),
    CONSTRAINT fk_alias_catalog FOREIGN KEY (canonical_name)
        REFERENCES ingredient_catalog(canonical_name) ON DELETE CASCADE
) ENGINE=InnoDB;
6. users
CREATE TABLE users (
    id         INT NOT NULL AUTO_INCREMENT,
    username   VARCHAR(80) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uq_username (username)
) ENGINE=InnoDB;
7. pantry_items
CREATE TABLE pantry_items (
    id             INT NOT NULL AUTO_INCREMENT,
    user_id        INT NOT NULL,
    canonical_name VARCHAR(120) NOT NULL,
    added_at       DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uq_user_ingredient (user_id, canonical_name),
    CONSTRAINT fk_pantry_user FOREIGN KEY (user_id)
        REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB;
8. favorites
CREATE TABLE favorites (
    id        INT NOT NULL AUTO_INCREMENT,
    user_id   INT NOT NULL,
    recipe_id INT NOT NULL,
    added_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uq_fav (user_id, recipe_id),
    CONSTRAINT fk_fav_user   FOREIGN KEY (user_id)   REFERENCES users(id)   ON DELETE CASCADE,
    CONSTRAINT fk_fav_recipe FOREIGN KEY (recipe_id) REFERENCES recipes(id) ON DELETE CASCADE
) ENGINE=InnoDB;
9. ratings
CREATE TABLE ratings (
    id        INT NOT NULL AUTO_INCREMENT,
    user_id   INT NOT NULL,
    recipe_id INT NOT NULL,
    rating    INT NOT NULL,
    rated_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uq_rating (user_id, recipe_id),
    CONSTRAINT chk_rating CHECK (rating BETWEEN 1 AND 5),
    CONSTRAINT fk_rat_user   FOREIGN KEY (user_id)   REFERENCES users(id)   ON DELETE CASCADE,
    CONSTRAINT fk_rat_recipe FOREIGN KEY (recipe_id) REFERENCES recipes(id) ON DELETE CASCADE
) ENGINE=InnoDB;
10. history
CREATE TABLE history (
    id           INT NOT NULL AUTO_INCREMENT,
    user_id      INT NOT NULL,
    recipe_id    INT NOT NULL,
    suggested_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT fk_hist_user   FOREIGN KEY (user_id)   REFERENCES users(id)   ON DELETE CASCADE,
    CONSTRAINT fk_hist_recipe FOREIGN KEY (recipe_id) REFERENCES recipes(id) ON DELETE CASCADE
) ENGINE=InnoDB;






Constraint Summary
Table	PK	FK	UNIQUE	CHECK
cuisines	id	—	name	—
recipes	id	cuisine_id	—	—
ingredients	id	recipe_id	—	—
ingredient_catalog	id	—	canonical_name	—
ingredient_aliases	id	canonical_name	alias_lc	—
users	id	—	username	—
pantry_items	id	user_id	(user_id, canonical_name)	—
favorites	id	user_id, recipe_id	(user_id, recipe_id)	—
ratings	id	user_id, recipe_id	(user_id, recipe_id)	rating 1-5
history	id	user_id, recipe_id	—	—
 
