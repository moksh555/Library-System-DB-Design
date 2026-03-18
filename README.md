# 📚 Library Management System — Database Design

A comprehensive **Entity-Relationship (ER) database design** for a library management and membership system. The design covers the full operational scope of a library: people (members, employees, guests), book catalog management, borrowing workflows, membership tiers, promotions, employee training, and payment processing.

## ✨ Features

- **Person specialization hierarchy** — a single `Person` base entity specializes into `Employee` and `Member` subtypes, avoiding attribute redundancy
- **Tiered membership system** — Gold and Silver membership levels associated with library cards and promotions
- **Multi-category book classification** — books support multiple categories via a junction table structure (`BookCategory`)
- **Full borrowing lifecycle** — `Borrowing_Details` tracks issue date, return date, and return policy; linked to `Payment_Details` for fine/fee management
- **Employee role specialization** — roles include Cataloging Manager, Library Supervisor, and Receptionist with a training/certification subsystem (`Trainer`)
- **Guest visit logging** — `Guest_Log` tracks non-member visits with optional membership association
- **Book reviews** — `Comments` entity stores ratings, timestamps, and content linked to books

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Design Tool | ERD (Entity-Relationship Diagram) |
| Diagrams | ERD Diagram (conceptual) + Physical Diagram |
| Reference SQL | PostgreSQL-compatible DDL |

## 📊 ER Diagrams

The repository includes two diagram artifacts:

| File | Description |
|------|-------------|
| `ERD_DIAGRAM.jpg` | Conceptual ER diagram showing entities, relationships, and cardinalities |
| `PHYSICAL_DIAGRAM.jpg` | Physical data model showing table structures and foreign keys |

## 🗄️ Core Entities

| Entity | Key Attributes | Notes |
|--------|---------------|-------|
| `Person` | PersonID, FullName, Address, Gender, DateOfBirth, PhoneNumber | Base type |
| `Employee` | EmployeeID (FK→Person), JobTitle, StartDate, Employee_Shift | ISA specialization |
| `Member` | MemberID (FK→Person) | ISA specialization |
| `LibraryCard` | LibraryCardID, DateOfIssue, MembershipLevel | Issued to Member |
| `Book` | Book_ID, Book_Title, Book_Description, Main_Content | Core catalog entity |
| `Author` | AuthorID, AuthorName | N:1 with Books |
| `Publisher` | Pub_ID, Pub_Name, Publisher_Address | N:1 with Books |
| `Borrowing_Details` | Borrowing_ID, Date_Of_Issue, Return_Date, Return_Policy | Transaction record |
| `Payment_Details` | Payment_ID, Payment_Type, Payment_Time, Amount | Linked to borrowing |
| `Promotion` | P_Code, P_Description | M:N with LibraryCard |
| `Trainer` | Certificate, Certificate_number | Trains Employees |
| `Guest_Log` | Guest_ID, Guest_Name, Guest_Phone | Tracks guest visits |
| `Comments` | Comment_ID, Rating, Comment_Time, Main_Content | Reviews on Books |

## 🔗 Key Relationships & Cardinalities

| Relationship | Cardinality | Notes |
|-------------|------------|-------|
| Person → Employee/Member | 1:0..1 (ISA) | Disjoint or overlapping roles |
| Member → LibraryCard | 1:1 or 1:N | Card issuance |
| LibraryCard → Promotion | N:M | Via associative entity |
| Book → Author | N:1 | One primary author per book |
| Book → Publisher | N:1 | One publisher per book |
| Book → BookCategory | 1:N | Multi-category junction |
| Book → Comments | 1:N | Reviews linked to books |
| Book → Borrowing_Details | 1:N | Borrow history per book |
| Borrowing_Details → Payment_Details | 1:1 or 1:N | Fee generation |
| Trainer → Employee | N:M | Training events with certificates |

## 🏗️ Sample DDL (PostgreSQL)

```sql
CREATE TABLE Person (
  PersonID SERIAL PRIMARY KEY,
  FullName VARCHAR(255),
  Address TEXT,
  Gender VARCHAR(10),
  DateOfBirth DATE,
  PhoneNumber VARCHAR(20)
);

CREATE TABLE Member (
  MemberID INT PRIMARY KEY REFERENCES Person(PersonID)
);

CREATE TABLE LibraryCard (
  LibraryCardID SERIAL PRIMARY KEY,
  MemberID INT REFERENCES Member(MemberID),
  DateOfIssue DATE,
  MembershipLevel VARCHAR(50) CHECK (MembershipLevel IN ('Gold', 'Silver'))
);

CREATE TABLE Book (
  Book_ID SERIAL PRIMARY KEY,
  Book_Title TEXT NOT NULL,
  Book_Description TEXT,
  Main_Content TEXT
);

CREATE TABLE BookCategory (
  BookID INT REFERENCES Book(Book_ID),
  CategoryName VARCHAR(100),
  Description TEXT,
  PRIMARY KEY (BookID, CategoryName)
);

CREATE TABLE Borrowing_Details (
  Borrowing_ID SERIAL PRIMARY KEY,
  Book_ID INT REFERENCES Book(Book_ID),
  MemberID INT REFERENCES Member(MemberID),
  Date_Of_Issue DATE,
  Return_Date DATE,
  Return_Policy TEXT
);

CREATE TABLE Payment_Details (
  Payment_ID SERIAL PRIMARY KEY,
  Borrowing_ID INT REFERENCES Borrowing_Details(Borrowing_ID),
  Payment_Type VARCHAR(50),
  Payment_Time TIMESTAMP,
  Amount DECIMAL(10, 2)
);
```

## 📐 Normalization Notes

- **3NF/BCNF compliant** — all non-key attributes depend only on the primary key
- Multi-valued book categories resolved via `BookCategory` junction table
- Shared personal attributes factored into `Person` to avoid duplication across `Employee` and `Member`
- Promotions and training are modular — adding new promotion types or certification types requires no schema changes
