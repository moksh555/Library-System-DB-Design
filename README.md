# Library Management & Membership System — ERD Overview

## 1. Project Overview

This Entity-Relationship Diagram (ERD) models a comprehensive **library management and membership system**. It captures:
- People (employees, members, guests)
- Memberships (gold / silver)
- Library cards and promotions
- Books, authors, publishers, and multi-category classification
- Comments/reviews on books
- Borrowing workflow including borrowing details and payments
- Employee roles (e.g., trainer, receptionist, cataloging manager)
- Training/certification relationships
- Guest visit logging

The design emphasizes normalization, role specialization, traceability of activity (borrowing, training), and support for flexible membership/promotion rules.

## 2. Core Entities & Attributes

### Person
Base entity representing any human in the system. Attributes (examples):
- `PersonID` (PK)
- `FullName`, `FirstName`, `MiddleName`, `LastName`
- `Address`
- `Gender`
- `DateOfBirth`
- `PhoneNumber`

### Employee (specialization of Person)
- `EmployeeID` (PK, FK to Person)
- `JobTitle`
- `StartDate`
- `Employee_Shift`
- Role specialization: Cataloging Manager, Library Supervisor, Receptionist (disjoint/overlapping as per diagram)

### Member (specialization of Person)
- Members are issued library cards.
- Related to `LibraryCard` via an `Issued` relationship.

### LibraryCard
- `LibraryCardID` (PK)
- `DateOfIssue`
- `MembershipLevel` (e.g., Gold, Silver)
- Relationship to Promotions (a card can be associated with promotions it “provides”)
- Issued to a Member (1:1 or 1:N depending on policy)

### GoldMembership / SilverMembership
Subtype details (if separate entities) indicating different levels, contained in a `Guest_Log` or attached to library cards.

### Guest_Log
Tracks guest visits; contains:
- `Guest_ID`
- `Guest_Name`
- `Guest_Address`
- `Guest_Phone`
- `Gold_Member_ID` (if applicable)
- Relationship indicating containment of membership types.

### Promotion
- `P_Code` (PK)
- `P_Description`
- Provided to cards and/or members.

### Trainer
- Trainers have certifications (`Trainer_certificate`, `Certificate_number`)
- Trains employees (likely Receptionists or others), with training events tracked including issue date of certificates.

### Books
- `Book_ID` (PK)
- `Book_Category`, `Book_Title`, `Book_Description`, `Main_Content`
- Relationships:
  - Written by Authors (N:1)
  - Published by Publishers (N:1)
  - Belongs to multiple categories (e.g., `Cate1`, `Cate2`, `Cate3`) — likely a many-to-many resolved via a category linking structure.
  - Has Comments (N:N) or Comments are about Books.

### Author
- `AuthorID` (PK)
- `AuthorName`
- Possibly other books attributed (`Author_Other_Books`)

### Publisher
- `Pub_ID` (PK)
- `Pub_Name`
- `Publisher_Address`
- `Publisher_Other_Books`
- Maintains catalog(s) of books.

### Comments
- `Comment_ID` (implied)
- `Rating`
- `Comment_Time`
- `Main_Content`
- Relationship to Books (About), possibly to Persons (who commented).

### Borrowing_Details
Tracks the borrowing events:
- `Borrowing_ID` (PK)
- `Date_Of_Issue`
- `Return_Date`
- `Return_Policy`
- Relationship “When Borrowed” to books and/or members.

### Payment_Details
- `Payment_ID` (PK)
- `Payment_Type`
- `Payment_Time`
- `Amount`
- Tied to borrowing events (fines, membership payments, etc.)

## 3. Relationships & Cardinalities (selected)

- **Person → Employee / Member:** Specialization (ISA). A person may be an employee or a member (or both if allowed).  
- **Member → LibraryCard:** Issued relationship (1:1 or 1:N).  
- **LibraryCard → Promotion:** Provides (N:N implied via associative entity).  
- **Books → Author:** Many books can be written by one author; a book can have multiple authors if extended (currently shown as N:1).  
- **Books → Publisher:** Many books published by one publisher (N:1).  
- **Books → Comments:** One book can have many comments; comments are about books (N:N or 1:N depending on implementation).  
- **Books → Borrowing_Details:** When borrowed relationship; a book can appear in many borrowing records.  
- **Borrowing_Details → Payment_Details:** One-to-one or one-to-many (a borrowing may generate payments).  
- **Trainer → Employee:** Trainer trains employees; training events issue certificates.  
- **Guest_Log → Gold/Silver Membership:** A guest log “contains” membership type (1:1 containment).  

## 4. Assumptions

- Employee roles (Cataloging Manager, Supervisor, Receptionist) may be disjoint or overlapping—diagram hints at a specialization hierarchy.  
- Promotions can be applied to cards or members and are reusable (hence many-to-many).  
- Categories for books are multi-valued (multiple category attributes) and likely implemented via a join table in a normalized schema.  
- Borrowing details are timestamped and trigger payment obligations (late fee, etc.).
- Comments include a rating and timestamp, allowing reviews to be audited.

## 5. Normalization Notes

- The design separates concerns: People, Roles, Content (Books), and Transactions (Borrowing, Payments).  
- Multi-valued attributes (like book categories) should be implemented with linking/junction tables (e.g., `BookCategories(book_id, category_id, description)`).  
- Specialization (Person → Employee/Member) avoids redundancy of shared personal info.  
- Promotions, training, and memberships are modular to support easy extension.

## 6. Example Logical Schema Snippets (SQL-style)

```sql
CREATE TABLE Person (
  PersonID SERIAL PRIMARY KEY,
  FullName VARCHAR(255),
  MiddleName VARCHAR(100),
  LastName VARCHAR(100),
  Address TEXT,
  Gender VARCHAR(10),
  DateOfBirth DATE,
  PhoneNumber VARCHAR(20)
);

CREATE TABLE Employee (
  EmployeeID INT PRIMARY KEY REFERENCES Person(PersonID),
  JobTitle VARCHAR(100),
  StartDate DATE,
  Employee_Shift VARCHAR(50)
);

CREATE TABLE Member (
  MemberID INT PRIMARY KEY REFERENCES Person(PersonID)
);

CREATE TABLE LibraryCard (
  LibraryCardID SERIAL PRIMARY KEY,
  MemberID INT REFERENCES Member(MemberID),
  DateOfIssue DATE,
  MembershipLevel VARCHAR(50)
);

CREATE TABLE Promotion (
  P_Code VARCHAR(50) PRIMARY KEY,
  P_Description TEXT
);

CREATE TABLE Book (
  Book_ID SERIAL PRIMARY KEY,
  Book_Title TEXT,
  Book_Description TEXT,
  Main_Content TEXT
);

CREATE TABLE Author (
  AuthorID SERIAL PRIMARY KEY,
  AuthorName VARCHAR(255)
);

CREATE TABLE Publisher (
  Pub_ID SERIAL PRIMARY KEY,
  Pub_Name VARCHAR(255),
  Publisher_Address TEXT
);

-- Junction example for multi-category
CREATE TABLE BookCategory (
  BookID INT REFERENCES Book(Book_ID),
  CategoryName VARCHAR(100),
  Description TEXT,
  PRIMARY KEY (BookID, CategoryName)
);
