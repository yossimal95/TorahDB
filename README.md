# TorahDB
The Torah Characters & Genealogy Database

## About
This project is a relational database for Torah characters, their family trees, and aliases, implemented using **SQLite**.

### Naming Conventions & Best Practices
- **Tables and Columns**: We use `snake_case` (e.g., `characters`, `main_name`), which is the universally accepted standard and best practice for SQL databases. It improves readability and prevents syntax issues across different database engines.
- **Hebrew Support**: SQLite natively stores text in UTF-8 encoding. Unlike SQL Server, there is no need for `NVARCHAR` or special collations (like `Hebrew_CI_AS`). You can store Hebrew characters directly in standard `TEXT` columns seamlessly.

---

## Getting Started: Creating the Database

To create the SQLite database and enable foreign key constraints, run the following in your terminal:

```bash
# Create a new SQLite database file
sqlite3 torah.db
```

Once inside the SQLite prompt, ensure foreign keys are enabled (SQLite disables them by default for backward compatibility):
```sql
PRAGMA foreign_keys = ON;
```

---

## Tables Creation

```sql
CREATE TABLE characters (
    character_id INTEGER PRIMARY KEY AUTOINCREMENT,
    main_name TEXT NOT NULL,
    gender INTEGER,
    father_id INTEGER,
    mother_id INTEGER,
    age_at_death INTEGER,

    CONSTRAINT fk_father FOREIGN KEY (father_id) REFERENCES characters(character_id),
    CONSTRAINT fk_mother FOREIGN KEY (mother_id) REFERENCES characters(character_id)
);

CREATE TABLE character_aliases (
    alias_id INTEGER PRIMARY KEY AUTOINCREMENT,
    character_id INTEGER NOT NULL,
    alias_name TEXT NOT NULL, 
     
    CONSTRAINT fk_character_alias FOREIGN KEY (character_id) 
    REFERENCES characters(character_id) ON DELETE CASCADE
);

CREATE TABLE marriages (
    marriage_id INTEGER PRIMARY KEY AUTOINCREMENT,
    husband_id INTEGER NOT NULL, 
    wife_id INTEGER NOT NULL,    

    CONSTRAINT fk_husband FOREIGN KEY (husband_id) 
    REFERENCES characters(character_id) ON DELETE NO ACTION,
    
    CONSTRAINT fk_wife FOREIGN KEY (wife_id) 
    REFERENCES characters(character_id) ON DELETE NO ACTION
);
```

---

## Examples

### ⭐ Returns character (אדם) children count and names
*Note: We use `GROUP_CONCAT` in SQLite instead of SQL Server's `STRING_AGG`.*

```sql
SELECT 
    f.main_name,
    COUNT(c.character_id) AS children_count,
    GROUP_CONCAT(c.main_name, ', ') AS children_names
FROM characters AS c
INNER JOIN characters AS f 
    ON f.character_id = c.father_id
WHERE f.main_name = 'אדם'
GROUP BY f.main_name;
```
<img width="291" height="66" alt="image" src="https://github.com/user-attachments/assets/cc5b0ac7-a674-469c-945d-3765b135a9b5" />

### ⭐ Returns characters with parents’ names

```sql
SELECT 
    c.character_id,
    c.main_name AS child_name,
    f.main_name AS father_name,
    m.main_name AS mother_name
FROM characters AS c
LEFT JOIN characters AS f
    ON c.father_id = f.character_id
LEFT JOIN characters AS m
    ON c.mother_id = m.character_id;
```
<img width="330" height="450" alt="image" src="https://github.com/user-attachments/assets/c898bd9b-f177-4685-ba73-ba5ff9765c70" />

### ⭐ Returns husbands and wives names

```sql
SELECT 
    h.main_name AS husband_name,
    w.main_name AS wife_name
FROM marriages AS m
JOIN characters AS h 
    ON m.husband_id = h.character_id 
JOIN characters AS w 
    ON m.wife_id = w.character_id;
```
<img width="181" height="368" alt="image" src="https://github.com/user-attachments/assets/22c02307-8dc8-40bd-ab80-8b0b1ff099d8" />
