SAIT Student Enrollment Database
A comprehensive case study and database design for SAIT (Southern Alberta Institute of Technology) student enrollment, created by Hiral Pandya and team.


Table of Contents:
Introduction
Objectives & Mission Statement
Database Schema Overview
   Entity–Relationship Diagram
   Star‐Schema Design
Table Definitions
   dim_student
   dim_course
   dim_instructor
   dim_classroom
   enrollment (Fact Table)
Sample SQL Queries
Sample Result Screenshots
Repository Structure
Team Contributions
How to Get Started
License


Introduction
The SAIT Student Enrollment Database is a fully documented case study and implementation of a relational database system designed to manage student enrollments, courses, instructors, classrooms, and payments for an educational institution.

Objectives & Mission Statement
Objective
  Align students and instructors with their schedules and preferences.
  Track enrollment details for upcoming intake programs.

Mission Statement
   Design a comprehensive database system to streamline enrollment processes across SAIT programs.
   Create efficient data relationships between key entities (Students, Courses, Instructors, Classrooms).
   Enable administrative decision‐making through readily accessible, organized data.

Database Schema Overview
Entity–Relationship Diagram
   Below is the high‐level Entity–Relationship Diagram (ERD) for the SAIT Student Enrollment Database. It illustrates the tables, primary keys, foreign keys, and cardinalities in a star‐schema layout, with Enrollment as the central fact table and all other entities as dimensions.
   
erd-overview.png
Star‐Schema Design
 The overall design follows a classical star‐schema pattern:
  - Fact Table
     -> enrollment: Contains transactional enrollment records, each linking a student, course, instructor, classroom, and payment.
  - Dimension Tables
     -> dim_students
     -> dim_courses
     -> dim_instructors
     -> dim_classrooms

Table Definitions
1) dim_student

CREATE TABLE dim_student (
    student_id      INT             PRIMARY KEY,    -- Unique student identifier
    first_name      VARCHAR(50)     NOT NULL,
    last_name       VARCHAR(50)     NOT NULL,
    gender          CHAR(1)         NOT NULL,       -- e.g., 'M', 'F', 'O'
    date_of_birth   DATE            NOT NULL,
    address         VARCHAR(200),
    phone_number    VARCHAR(20),
    email_address   VARCHAR(100),
    admission_date  DATE,
    course_id       VARCHAR(10)                      -- FK → dim_course(course_id)
);
    - student_id (PK): Surrogate integer key for each student.
    - first_name / last_name: Biographical data.
    - gender: Single‐character code.
    - date_of_birth: For age calculation, birthday reports, etc.
    - address / phone_number / email_address: Contact information.
    - admission_date: Date the student was admitted.
    - course_id (FK): References the student’s primary course in dim_course.
      
2) dim_course

CREATE TABLE dim_course (
    course_id     VARCHAR(10)     PRIMARY KEY,       -- e.g., “DMKT101”
    course_code   VARCHAR(10),                       -- Can be same as course_id or a subset
    course_title  VARCHAR(100)    NOT NULL,          -- e.g., “Digital Marketing”
    credits       INT
);

   - course_id (PK): Short alphanumeric code for the course.
   - course_code: Additional code field (if different from course_id).
   - course_title: Full descriptive name.
   - credits: Number of credit hours.

3) dim_instructor

CREATE TABLE dim_instructor (
    instructor_id   INT             PRIMARY KEY,    -- Unique numeric ID for each instructor
    first_name      VARCHAR(50)     NOT NULL,
    last_name       VARCHAR(50)     NOT NULL,
    email_address   VARCHAR(100),
    phone_number    VARCHAR(20),
    course_id       VARCHAR(10),                      -- FK → dim_course(course_id)
    FOREIGN KEY (course_id) REFERENCES dim_course(course_id)
);

    - instructor_id (PK): Surrogate integer key.
    - first_name / last_name: Name of the instructor.
    - email_address / phone_number: Contact details.
    - course_id (FK): The primary course the instructor teaches.

4) dim_classroom

CREATE TABLE dim_classroom (
    classroom_id   INT             PRIMARY KEY,      -- Unique classroom identifier
    building_name  VARCHAR(100)    NOT NULL,         -- e.g., “North Wing”
    room_number    VARCHAR(10),                       -- e.g., “104”
    course_id      VARCHAR(10),                       -- FK → dim_course(course_id)
    FOREIGN KEY (course_id) REFERENCES dim_course(course_id)
);

    - classroom_id (PK): Surrogate integer for each classroom.
    - building_name: Name of the building.
    - room_number: Room number within that building.
    - course_id (FK): If a course is associated with this room by default.

5) enrollment (Fact Table)
CREATE TABLE fact_enrollment (
    enrollment_id    INT           PRIMARY KEY,      -- Unique enrollment transaction ID
    student_id       INT,                           -- FK → dim_student(student_id)
    course_id        VARCHAR(10),                   -- FK → dim_course(course_id)
    instructor_id    INT,                           -- FK → dim_instructor(instructor_id)
    classroom_id     INT,                           -- FK → dim_classroom(classroom_id)
    enrollment_date  DATE,
    payment_amount   INT,                           -- e.g., 20000 for uniform payment
    payment_status   VARCHAR(20),                   -- e.g., “PAID”, “PENDING”
    FOREIGN KEY (student_id)    REFERENCES dim_student(student_id),
    FOREIGN KEY (course_id)     REFERENCES dim_course(course_id),
    FOREIGN KEY (instructor_id) REFERENCES dim_instructor(instructor_id),
    FOREIGN KEY (classroom_id)  REFERENCES dim_classroom(classroom_id)
);

      - enrollment_id (PK): Surrogate integer key for each enrollment record.
      - student_id (FK): Links to dim_student.
      - course_id (FK): Links to dim_course
      - instructor_id (FK): Links to dim_instructor.
      - classroom_id (FK): Links to dim_classroom.
      - enrollment_date: Date the student enrolled (e.g., 2025-02-10).
      - payment_amount: Amount paid (set to 20000 for uniformity).
      - payment_status: Payment state (“PAID”)


Sample SQL Queries
 1. Get Full Enrollment Details for Each Student
 Description:
  Retrieve every enrollment record—joining all dimensions—to display student information, course title, instructor name, classroom building, enrollment date, payment amount, and payment status.
   SELECT
    s.student_id,
    s.first_name,
    s.last_name,
    c.course_title,
    i.first_name AS instructor_first_name,
    i.last_name  AS instructor_last_name,
    cl.building_name,
    e.enrollment_date,
    e.payment_amount,
    e.payment_status
FROM fact_enrollment AS e
JOIN dim_student    AS s  ON e.student_id    = s.student_id
JOIN dim_course     AS c  ON e.course_id     = c.course_id
JOIN dim_instructor AS i  ON e.instructor_id = i.instructor_id
JOIN dim_classroom  AS cl ON e.classroom_id  = cl.classroom_id;

Joins:
fact_enrollment (e) → dim_student (s) on e.student_id = s.student_id
fact_enrollment → dim_course (c) on e.course_id = c.course_id
fact_enrollment → dim_instructor (i) on e.instructor_id = i.instructor_id
fact_enrollment → dim_classroom (cl) on e.classroom_id = cl.classroom_id

Preview Screenshot: docs/results/1_Results_Screenshot.png

2. List All Students Enrolled in “Digital Marketing” Course
Description:
Filter enrollments for those where the course title is “Digital Marketing.” Display student ID, name, course title, and enrollment date.
SELECT
    s.student_id,
    s.first_name,
    s.last_name,
    c.course_title,
    e.enrollment_date
FROM fact_enrollment AS e
JOIN dim_student AS s ON e.student_id = s.student_id
JOIN dim_course  AS c ON e.course_id  = c.course_id
WHERE c.course_title = 'Digital Marketing';

joins:
fact_enrollment → dim_student
fact_enrollment → dim_course

Filter:
WHERE c.course_title = 'Digital Marketing'

Preview Screenshot: docs/results/2_Results_Screenshot.png


3. Count of Students per Instructor
Description:
Generate a report showing how many students each instructor has taught (counting distinct students per instructor).
SELECT
    i.instructor_id,
    i.first_name,
    i.last_name,
    COUNT(DISTINCT e.student_id) AS total_students
FROM fact_enrollment AS e
JOIN dim_instructor AS i ON e.instructor_id = i.instructor_id
GROUP BY
    i.instructor_id,
    i.first_name,
    i.last_name;
Join:
fact_enrollment → dim_instructo
Aggregate:
COUNT(DISTINCT e.student_id) AS total_students
Group By:
i.instructor_id, i.first_name, i.last_name
Preview Screenshot: docs/results/3_Results_Screenshot.png


4. Payment Analysis per Course by Instructor (Dec 2024 – Apr 2025)
Description:
Produce a financial summary—per instructor and per course—showing total students, total collected, average payment, and the first/last enrollment dates for paid enrollments between December 1, 2024, and April 30, 2025.
SELECT
    i.instructor_id,
    CONCAT(i.first_name, ' ', i.last_name) AS instructor_name,
    c.course_title,
    COUNT(DISTINCT e.student_id)            AS total_students,
    SUM(e.payment_amount)                   AS total_collected,
    AVG(e.payment_amount)                   AS avg_payment,
    MIN(e.enrollment_date)                  AS first_enrollment,
    MAX(e.enrollment_date)                  AS last_enrollment
FROM fact_enrollment AS e
JOIN dim_instructor AS i ON e.instructor_id = i.instructor_id
JOIN dim_course     AS c ON e.course_id     = c.course_id
JOIN dim_student    AS s ON e.student_id    = s.student_id
JOIN dim_classroom  AS cl ON e.classroom_id = cl.classroom_id
WHERE e.payment_status = 'PAID'
  AND e.enrollment_date BETWEEN '2024-12-01' AND '2025-04-30'
GROUP BY
    i.instructor_id,
    instructor_name,
    c.course_title
ORDER BY total_collected DESC;

joins:
fact_enrollment → dim_instructor, dim_course, dim_student, dim_classroom
Filter:
WHERE e.payment_status = 'PAID
AND e.enrollment_date BETWEEN '2024-12-01' AND '2025-04-30
Aggregates:
Students: COUNT(DISTINCT e.student_id)
Total Collected: SUM(e.payment_amount)
Average Payment: AVG(e.payment_amount)
Enrollment Range: MIN(e.enrollment_date) and MAX(e.enrollment_date)
Grouping & Ordering:
Grouped by instructor and course, ordered by total_collected DESC
Preview Screenshot: docs/results/4_Results_Screenshot.png




