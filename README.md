# Convert to 4NF

| assignment_id | student_id | due_date | professor | assignment_topic                | classroom | grade | relevant_reading    | professor_email   |
| :------------ | :--------- | :------- | :-------- | :------------------------------ | :-------- | :---- | :------------------ | :---------------- |
| 1             | 1          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 80    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 2             | 7          | 18.11.21 | Logston   | Single table queries            | 60FA 314  | 25    | Dümmlers Chapter 11 | e.logston@foo.edu |
| 1             | 4          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 75    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 5             | 2          | 05.05.21 | Logston   | Python and pandas               | 60FA 314  | 92    | Dümmlers Chapter 14 | e.logston@foo.edu |
| 4             | 2          | 04.07.21 | Nevarez   | Spreadsheet aggregate functions | WWH 201   | 65    | Zehnder Page 87     | i.nevarez@foo.edu |
| ...           | ...        | ...      | ...       | ...                             | ...       | ...   | ...                 | ...               |

The original dataset is not compliant with 4NF mainly because it contains non-trivial multi-valued dependencies which occurs when one attribute uniquely determines another attribute or set of attributes, independently of any other attributes. For instance, a professor could give multiple assignments, and each assignment may have different due dates for different sections, which implies that there is a multi-valued dependency between `professor` and `due_date` that does not involve a candidate key. 

To convert the original dataset to 4NF, we want to have no table with two or more independent multi-valued facts about an entity and to identify potential candidate keys. Therefore, following is my proposed 4NF structure:

## Tables

### 1 Courses

| course_id | course_name |
| :-------- | :---------- |
| 1         | CSCI-UA 2   |
| 2         | CSCI-UA 60  |
| 3         | CSCI-UA 101 |
| ...       | ...         |

PK: `course_id`

### 2 Professors

| professor_id | professor_name | professor_email    |
| :----------- | :------------- | :----------------- |
| 1            | Melvin         | l.melvin@foo.edu   |
| 2            | Logston        | e.logston@foo.edu  |
| 3            | Nevarez        | i.nevarez@foo.edu  |
| ...          | ...            | ...                |

PK: `professor_id`

### 3 Sections

| section_id | course_id | professor_id | classroom |
| :--------- | :-------- | :----------- | :-------- |
| 1          | 1         | 1            | WWH 101   | 
| 2          | 2         | 2            | 60FA 314  |
| 3          | 3         | 3            | WWH 201   |
| ...        | ...       | ...          | ...       |

PK: `section_id`

FK: `course_id`, `professor_id`

### 4 Assignments

| assignment_id | course_id | section_id | assignment_topic      | due_date | 
| :------------ | :-------- | :--------- | :-------------------- | :------- |
| 1             | 1         | 1          | Data normalization    | 23.02.21 |
| 2             | 2         | 2          | Single table queries  | 18.11.21 |
| 3             | 2         | 2          | Python and pandas     | 05.05.21 |
| ...           | ...       | ...        | ...                   | ...      |

PK: `assignment_id`

FK: `course_id`, `section_id`

### 5 Readings

| reading_id | assignment_id | relevant_reading     |
| :--------- | :------------ | :------------------- | 
| 1          | 1             | Deumlich Chapter 3   |
| 2          | 2             | Dümmlers Chapter 11  |
| 3          | 3             | Dümmlers Chapter 14  |
| ...        | ...           | ...                  |

PK: `reading_id`

FK: `assignment_id`

### 6 Students

| student_id | student_name  |
| :--------- | :------------ |
| 1          | Lenore Day    |
| 2          | Lonny Miranda |
| 4          | Ezekiel Sloan |
| 7          | Janell Dalton |
| ...        | ...           |

PK: `student_id`

### 7 Enrollments 

| enrollment_id | student_id | section_id |
| :------------ | :--------- | :--------- |
| 1             | 1          | 1          |
| 2             | 2          | 2          |
| 3             | 4          | 1          |
| ...           | ...        | ...        |

PK: `enrollment_id`

FK: `student_id`, `section_id`

### 8 Grades

| student_id | assignment_id | grade |
| :--------- | :------------ | :---- |
| 1          | 1             | 80    |
| 2          | 3             | 92    |
| 4          | 1             | 75    |
| 7          | 2             | 25    |
| ...        | ...           | ...   |

FK: `student_id`, `assignment_id`

## Notes

- PK: Primary Keys. 
- FK: Foreign Keys. 
- The `Enrollments` table associates students with their enrolled sections.
- The `Grades` table keeps track of the grades students received for assignments, and it doesn't have a PK. 
- A separate `Assignments` table is needed to accommodate cases where the same assignment might be given to different sections with different due dates.
- The `Readings` table links specific readings to assignments since one assignment may have multiple readings.
- The `Sections` table records different sections of a course. 

This structure above ensures that each fact is stored only once and that all relationships are properly represented without multi-valued dependencies. The tables are linked through FKs that refer to the PKs in their respective tables. This design allows the database to be efficient, flexible, and free from update anomalies, and therefore is compliant with 4NF. 

# Entity-Relationship Diagram

The Entity-Relationship Diagram of the previous structure would look like this:

![ERD](./images/ERD.svg)

For simplicity, I only include primary keys for entities but not foreign keys and only show the relations graphically but not define it in a literal way. 

## Notes

### Entities

- **Courses** Table: Contains `course_id` (PK) and `course_name`.
- **Professors** Table: Contains `professor_id` (PK), `professor_name`, and `professor_email`.
- **Sections** Table: Contains `section_id` (PK), has `course_id` and `professor_id` linking to the **Courses** and **Professors** tables respectively, and `classroom`.
- **Assignments** Table: Contains `assignment_id` (PK), has `course_id` and `section_id` linking to the **Courses** and **Sections** tables respectively, `assignment_topic`, and `due_date`.
- **Readings** Table: Contains `reading_id` (PK) and `relevant_reading`, with `assignment_id` linking to the **Assignments** table.
- **Students** Table: Contains `student_id` (PK) and `student_name`.
- **Enrollments** Table: Contains `enrollment_id`, with `student_id` and `section_id` linking to the **Students** and **Sections** tables respectively.
- **Grades** Table: A link table between the **Students** and **Assignments** tables, containing `student_id` and `assignment_id` as foreign keys, and `grade`.


### Relationships

- **Courses** would have a 1-to-M relationship with **Sections** because a course can have multiple sections.
- **Professors** would have a 1-to-M relationship with **Sections** because a professor can teach multiple sections.
- **Sections** would have a 1-to-M relationship with **Assignments** because each section can have multiple assignments.
- **Assignments** would have a 1-to-M relationship with **Readings** because an assignment can require multiple readings.
- **Students** would have an M-to-M relationship with **Sections**, represented through the **Enrollments** link table.
- **Students** would have an M-to-M relationship with **Assignments**, represented through the **Grades** link table.
