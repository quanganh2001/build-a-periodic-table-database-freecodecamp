# Part 1: Fixing the database
- Let's connect database:
```txt
codeally@7a95d0719683:~/project$ psql --username=freecodecamp --dbname=postgres
psql (12.9 (Ubuntu 12.9-2.pgdg20.04+1))
Type "help" for help.

postgres=> \c periodic_table 
You are now connected to database "periodic_table" as user "freecodecamp".
```
- You should rename the `weight` column to `atomic_mass`
```sql
ALTER TABLE properties RENAME COLUMN weight TO atomic_mass;
```
- You should rename the `melting_point` column to `melting_point_celsius` and the `boiling_point` column to `boiling_point_celsius`
```sql
ALTER TABLE properties RENAME COLUMN melting_point TO melting_point_celsius;
ALTER TABLE properties RENAME COLUMN boiling_point TO boiling_point_celsius;
```
- Your `melting_point_celsius` and `boiling_point_celsius` columns should not accept null values
```sql
ALTER TABLE properties ALTER COLUMN melting_point_celsius SET NOT NULL;
ALTER TABLE properties ALTER COLUMN boiling_point_celsius SET NOT NULL;
```
- You should add the `UNIQUE` constraint to the `symbol` and `name` columns from the elements table
```sql
ALTER TABLE elements ADD UNIQUE(symbol);
ALTER TABLE elements ADD UNIQUE(name);
```
Let's check `elements` table: `\d elements`
```txt
                        Table "public.elements"
    Column     |         Type          | Collation | Nullable | Default 
---------------+-----------------------+-----------+----------+---------
 atomic_number | integer               |           | not null | 
 symbol        | character varying(2)  |           |          | 
 name          | character varying(40) |           |          | 
Indexes:
    "elements_pkey" PRIMARY KEY, btree (atomic_number)
    "elements_atomic_number_key" UNIQUE CONSTRAINT, btree (atomic_number)
    "elements_name_key" UNIQUE CONSTRAINT, btree (name)
    "elements_symbol_key" UNIQUE CONSTRAINT, btree (symbol)
```
- Your `symbol` and `name` columns should have the `NOT NULL` constraint
```sql
ALTER TABLE elements ALTER COLUMN name SET NOT NULL;
ALTER TABLE elements ALTER COLUMN symbol SET NOT NULL;
```
- You should set the `atomic_number` column from the `properties` table as a foreign key that references the column of the same name in the `elements` table
```sql
ALTER TABLE properties ADD FOREIGN KEY(atomic_number) REFERENCES elements(atomic_number);
```
- You should create a `types` table that will store the three types of elements
- Your `types` table should have a `type_id` column that is an integer and the primary key
- Your `types` table should have a `type` column that's a `VARCHAR` and cannot be null. It will store the different types from the type column in the properties table
```sql
CREATE TABLE types(type_id SERIAL NOT NULL);
ALTER TABLE types ADD COLUMN type VARCHAR(20) NOT NULL;
ALTER TABLE types ADD PRIMARY KEY(type_id);
```
- You should add three rows to your `types` table whose values are the three different types from the `properties` table
```sql
INSERT INTO types(type) VALUES('nonmetal'), ('metal'), ('metalloid');
```
- Your `properties` table should have a `type_id` foreign key column that references the `type_id` column from the `types` table. It should be an `INT` with the `NOT NULL` constraint
- Each row in your `properties` table should have a `type_id` value that links to the correct type from the types table
```sql
ALTER TABLE properties ADD COLUMN type_id INT REFERENCES types(type_id);
UPDATE properties SET type_id=1 WHERE type='nonmetal';
UPDATE properties SET type_id=2 WHERE type='metal';
UPDATE properties SET type_id=3 WHERE type='metalloid';
ALTER TABLE properties ALTER COLUMN type_id SET NOT NULL;
```
- You should capitalize the first letter of all the `symbol` values in the `elements` table. Be careful to only capitalize the letter and not change any others
```sql
UPDATE elements SET symbol = 'He' WHERE symbol = 'he';
UPDATE elements SET symbol = 'Li' WHERE symbol = 'li';
UPDATE elements SET symbol = 'Mt' WHERE symbol = 'mT';
```
- You should remove all the trailing zeros after the decimals from each row of the `atomic_mass` column. You may need to adjust a data type to `DECIMAL` for this. The final values they should be are in the `atomic_mass.txt` file
```sql
ALTER TABLE properties ALTER COLUMN atomic_mass SET DATA TYPE DECIMAL(9,0);
ALTER TABLE properties ALTER COLUMN atomic_mass SET DATA TYPE DECIMAL;
```
- You should add the element with atomic number 9 to your database. Its name is Fluorine, symbol is F, mass is 18.998, melting point is -220, boiling point is -188.1, and it's a nonmetal
```sql
UPDATE properties SET atomic_mass = 15 WHERE atomic_number = 8;
INSERT INTO elements(atomic_number, symbol, name) VALUES(9, 'F', 'Fluorine');
INSERT INTO elements(atomic_number, symbol, name) VALUES(10, 'Ne', 'Neon');
INSERT INTO properties(atomic_number, type, atomic_mass, melting_point_celsius, boiling_point_celsius, type_id) VALUES(9, 'nonmetal', 18.998, -220, -188.1, 1);
```
- You should add the element with atomic number 10 to your database. Its name is Neon, symbol is Ne, mass is 20.18, melting point is -248.6, boiling point is -246.1, and it's a nonmetal
```sql
INSERT INTO properties(atomic_number, type, atomic_mass, melting_point_celsius, boiling_point_celsius, type_id) VALUES(10, 'nonmetal', 20.18, -248.6, -246.1, 1);
```
# Part 2: Create your git repository
First, I create folder `periodic_table`. Then change directory

Second, I create `element.sh` then give permissions this sh file.

Let's turn it into a git repository with `git init`

Let add all files with `git add`. Then commit: `git commit -m "Initial commit"`. Change branch to main: `git branch main`

- Your properties table should not have a type column
```sql
ALTER TABLE properties DROP COLUMN type;
```
I add database connection:
```sh
#!/bin/bash

PSQL="psql -X --username=freecodecamp --dbname=periodic_table --tuples-only -c"
```
Then, I add all then commit: `git commit -m "feat: add database connection"`

- If you run `./element.sh`, it should output only `Please provide an element as an argument.` and finish running.
```sh
#!/bin/bash

PSQL="psql -X --username=freecodecamp --dbname=periodic_table --tuples-only -c"

if [[ $1 ]]
then
  # if [[ ! $1 =~ ^[0-9]+$ ]]
  # then
  echo $1
  # fi
  else
  echo "Please provide an element as an argument."
fi
```
Update bring element:
```sh
#!/bin/bash

PSQL="psql -X --username=freecodecamp --dbname=periodic_table --tuples-only -c"

if [[ $1 ]]
then
  if [[ ! $1 =~ ^[0-9]+$ ]]
  then
  ELEMENT=$($PSQL "SELECT atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, symbol, name, type FROM properties JOIN elements USING(atomic_number) JOIN types USING(type_id) WHERE elements.name LIKE '$1%' ORDER BY atomic_number LIMIT 1")
  else
  ELEMENT=$($PSQL "SELECT atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, symbol, name, type FROM properties JOIN elements USING(atomic_number) JOIN types USING(type_id) WHERE elements.atomic_number=$1")
  fi
    if [[ -z $ELEMENT ]]
    then
      echo "I could not find that element in the database."
    else
      echo $ELEMENT | while read ATOMIC_NUMBER
      do
        echo $ATOMIC_NUMBER
      done

    fi
  else
  echo "Please provide an element as an argument."
fi
```
Update commit: `git commit -m "feat: bring element"`

Fix: bring separated data
```sh
#!/bin/bash

PSQL="psql -X --username=freecodecamp --dbname=periodic_table --tuples-only -c"

if [[ $1 ]]
then
  if [[ ! $1 =~ ^[0-9]+$ ]]
  then
  ELEMENT=$($PSQL "SELECT atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, symbol, name, type FROM properties JOIN elements USING(atomic_number) JOIN types USING(type_id) WHERE elements.name LIKE '$1%' ORDER BY atomic_number LIMIT 1")
  else
  ELEMENT=$($PSQL "SELECT atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, symbol, name, type FROM properties JOIN elements USING(atomic_number) JOIN types USING(type_id) WHERE elements.atomic_number=$1")
  fi
    if [[ -z $ELEMENT ]]
    then
      echo "I could not find that element in the database."
    else
      echo $ELEMENT | while IFS=\| read ATOMIC_NUMBER ATOMIC_MAS
      do
        echo "$ATOMIC_NUMBER - $ATOMIC_MAS"
      done

    fi
  else
  echo "Please provide an element as an argument."
fi
```
Update commit: `git add .`, `git commit -m "fix: bring separated data"`

Let's checkout main: `git checkout main`

Update: bring data:
```sh
#! /bin/bash

PSQL="psql --username=freecodecamp --dbname=periodic_table --no-align --tuples-only -c"

if [[ $1 ]]
  then
  if [[ ! $1 =~ ^[0-9]+$ ]]
  then
  ELEMENT=$($PSQL "SELECT atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, symbol, name, type FROM properties JOIN elements USING(atomic_number) JOIN types USING(type_id) WHERE elements.name LIKE '$1%' ORDER BY atomic_number LIMIT 1")
  else
  ELEMENT=$($PSQL "SELECT atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, symbol, name, type FROM properties JOIN elements USING(atomic_number) JOIN types USING(type_id) WHERE elements.atomic_number=$1")
  fi
    if [[ -z $ELEMENT ]]
    then
      echo "I could not find that element in the database."
    else
      echo $ELEMENT | while IFS=\| read ATOMIC_NUMBER ATOMIC_MASS MPC BPC SY NAME TYPE
      do
        echo "The element with atomic number $ATOMIC_NUMBER is $NAME ($SY). It's a $TYPE, with a mass of $ATOMIC_MASS amu. $NAME has a melting point of $MPC celsius and a boiling point of $BPC celsius." 
      done

    fi
  else
  echo  "Please provide an element as an argument."
fi
```
When run `element.sh` with argument is 2, the output is:
```txt
The element with atomic number 1 is Hydrogen (H). It's a nonmetal, with a mass of 1.008 amu. Hydrogen has a melting point of -259.1 celsius and a boiling point of -252.9 celsius.
```
- You should delete the non existent element, whose `atomic_number` is 1000, from the two tables
```sql
UPDATE properties SET atomic_mass = 1.008 WHERE atomic_number = 1;
DELETE FROM elements WHERE atomic_number = 1000;
```
Let's check `SELECT * FROM elements;`:
```txt
 atomic_number | symbol |   name    
---------------+--------+-----------
             1 | H      | Hydrogen
             4 | Be     | Beryllium
             5 | B      | Boron
             6 | C      | Carbon
             7 | N      | Nitrogen
             8 | O      | Oxygen
             2 | He     | Helium
             3 | Li     | Lithium
             9 | F      | Fluorine
            10 | Ne     | Neon
(10 rows)
```
