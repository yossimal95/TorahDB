# TorahDB
מאגר דמויות התורה ואילן יוחסין (The Torah Characters & Genealogy Database)

[English](README.md)

## אודות
פרויקט זה הינו מסד נתונים יחסי (Relational Database) עבור דמויות מהתורה, עצי המשפחה שלהן וכינויים/שמות נוספים, הממומש באמצעות **SQLite**.

### מוסכמות שמות ושיטות עבודה מומלצות (Naming Conventions & Best Practices)
- **טבלאות ועמודות**: אנו משתמשים ב-`snake_case` (לדוגמה: `characters`, `main_name`), שהוא הסטנדרט המקובל והמומלץ ביותר עבור מסדי נתוני SQL. הדבר משפר את הקריאות ומונע בעיות תחביר במנועי מסדי נתונים שונים.
- **תמיכה בעברית**: SQLite שומר טקסט באופן טבעי בקידוד UTF-8. בשונה מ-SQL Server, אין צורך ב-`NVARCHAR` או ב-collations מיוחדים (כגון `Hebrew_CI_AS`). ניתן לשמור תווים בעברית ישירות בעמודות `TEXT` סטנדרטיות בצורה חלקה.

---

## תחילת עבודה: יצירת מסד הנתונים

כדי ליצור את מסד הנתונים של SQLite ולהפעיל אילוצי מפתחות זרים (Foreign Key constraints), יש להריץ את הפקודה הבאה בטרמינל:

```bash
# יצירת קובץ מסד נתונים חדש של SQLite
sqlite3 torah.db
```

לאחר הכניסה לשורת הפקודה של SQLite, יש לוודא שמפתחות זרים מופעלים (SQLite מכבה אותם כברירת מחדל לצורך תאימות לאחור):
```sql
PRAGMA foreign_keys = ON;
```

---

## יצירת הטבלאות

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

## דוגמאות

### ⭐ החזרת מספר הילדים ושמותיהם עבור הדמות "אדם"
*הערה: אנו משתמשים ב-`GROUP_CONCAT` ב-SQLite במקום ב-`STRING_AGG` של SQL Server.*

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

### ⭐ החזרת דמויות יחד עם שמות ההורים

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

### ⭐ החזרת שמות בעלים ונשותיהם

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
<img width="181" height="368" alt="image" src="https://github.com/user-attachments/assets/873176c4-f10c-4edd-9036-6ba76752e6f0" />
