# Smart Library Management System (SLMS)

A **Core Java–based Library Management System** developed as a **Build Your Own Project** for the **Java Programming** course under the **VITyarthi flipped-course evaluation**. The application provides a practical implementation of Java concepts such as **OOP principles, abstraction, inheritance, polymorphism, interfaces, collections, exception handling, and file handling**. It also incorporates **Singleton, Factory, and DAO design patterns** to organize the system into a structured, maintainable, and real-world application.


## Overview

The **Library Management System** provides separate access levels for **Administrators, Librarians, and Members**, with each role receiving a dedicated set of operations. The application supports **book inventory management, borrowing and returning, overdue fine tracking, user administration, and library reporting**. All records are maintained in **readable CSV-based storage**, allowing the system to operate as a completely **self-contained Core Java application without requiring any third-party libraries or external services**.


## Features
* **Role-specific functionality** — The system supports three user categories: **Admin, Librarian, and Member**. Each role is provided with its own set of operations through an abstract `User` hierarchy and runtime polymorphism.

* **Library inventory operations** — Librarians and authorized users can maintain the book collection by **adding new books, modifying details, deleting records, viewing the catalog, and performing searches** based on title, author, or category.

* **Complete borrowing cycle** — Members can issue and return books subject to defined borrowing rules. The application prevents multiple active issues of the same book, assigns a **14-day return deadline**, and calculates overdue charges automatically according to the number of delayed days.

* **Account administration** — New members can create their own accounts, while administrators have higher-level privileges to **register staff accounts, create administrative users, and remove existing user records**.

* **Library insights and reports** — Built-in reports provide information such as **current inventory, frequently issued books, pending/overdue returns, and individual member borrowing history with applicable fines**.

* **Robust error handling** — Input and business-rule checks are performed throughout the application using dedicated custom exceptions, including cases for **invalid data, duplicate records, missing books/users, failed authentication, and unavailable books**.

* **Activity tracking** — Important system events are automatically recorded with timestamps in `logs/app.log`. A **Singleton-based logging mechanism** ensures that logging is handled through a centralized instance.

* **Secure credential storage** — User passwords are converted into **SHA-256 hashes** before being stored, ensuring that the original plaintext credentials are not saved in the application's data files.

* **Built-in verification tests** — The project includes **20 test cases** covering authentication, input validation, user/book operations, and the complete **issue → return → fine calculation** process. Tests use in-memory implementations, keeping them independent of external testing libraries.


## Technologies / Tools Used

| Concern            | Choice                                            |
|--------------------|----------------------------------------------------|
| Language           | Java 17+ (developed & tested on OpenJDK 21)        |
| Persistence        | Plain CSV files under `data/` (no DB engine needed)|
| Build              | `javac`/`java` directly, or the provided `build.sh`|
| Testing            | Hand-rolled `Assert` harness + in-memory fake DAOs |
| Design patterns    | Singleton, Factory, DAO                            |
| Diagrams           | Mermaid (renders natively on GitHub) + Graphviz PNGs|

No external libraries, JDBC drivers, or Maven/Gradle setup are required —
this keeps the project trivial to clone and run anywhere a JDK is
installed.

## Project Structure

```
LibraryManagementSystem/
├── README.md
├── statement.md
├── build.sh                     # compile + run helper script
├── .gitignore
├── docs/
│   ├── diagrams/                 # Mermaid diagrams (render on GitHub) + PNG links
│   │   ├── architecture.md
│   │   ├── usecase.md
│   │   ├── class-diagram.md
│   │   ├── sequence-diagram.md
│   │   ├── er-diagram.md
│   │   └── workflow.md
│   ├── images/                   # Rendered PNGs used by the diagrams and the PDF report
│   └── ProjectReport.pdf         # Full design & evaluation report
├── src/
│   ├── main/java/com/slms/
│   │   ├── Main.java
│   │   ├── model/                # Role, User, Admin, Librarian, Member, Book, Transaction, TransactionStatus
│   │   ├── exception/             # 6 custom checked exceptions
│   │   ├── util/                  # AppLogger, FileManager (Singletons), ValidationUtil, IdGenerator
│   │   ├── dao/                   # BookDAO/UserDAO/TransactionDAO + CSV-backed impls
│   │   ├── service/                # UserService, BookService, TransactionService, ReportService, UserFactory, PasswordUtil
│   │   └── ui/                     # ConsoleUI (menu-driven front end)
│   └── test/java/com/slms/test/    # TestRunner + service tests + in-memory fake DAOs
├── data/                          # Generated at runtime (books.csv, users.csv, transactions.csv)
└── logs/                          # Generated at runtime (app.log)
```

## Design Diagrams

All diagrams live in [`docs/diagrams/`](docs/diagrams) as Mermaid code
(renders automatically on GitHub) with a matching PNG in
[`docs/images/`](docs/images):

- [System Architecture](docs/diagrams/architecture.md)
- [Use Case Diagram](docs/diagrams/usecase.md)
- [Class / Component Diagram](docs/diagrams/class-diagram.md)
- [Sequence Diagram — Issue Book](docs/diagrams/sequence-diagram.md)
- [ER / Storage Design](docs/diagrams/er-diagram.md)
- [Process Flow / Workflow](docs/diagrams/workflow.md)

## Non-Functional Requirements

| Requirement       | How it's addressed                                                                 |
|--------------------|-------------------------------------------------------------------------------------|
| **Performance**    | In-memory list operations over small CSV datasets; O(n) lookups are acceptable at the target scale (a single library branch) and isolated behind the DAO interfaces so storage can later be swapped for a database with no service-layer changes. |
| **Security**       | Passwords are SHA-256 hashed before storage; role-based menus prevent Members from reaching Admin/Librarian operations. |
| **Reliability**    | Every service method validates input and fails predictably via typed exceptions instead of crashing; file writes are atomic per-operation (read-modify-write-all). |
| **Scalability**    | Layered architecture (UI → Service → DAO → Storage) means the CSV storage engine can be replaced by JDBC/a real database by only rewriting the DAO implementations. |
| **Maintainability**| Clear package-by-layer structure, Javadoc-style comments, and consistent naming make the codebase easy to extend (e.g. adding a new report or role). |
| **Error handling** | Six custom checked exceptions communicate exactly what went wrong; the UI layer catches and reports each one with a human-readable message instead of a stack trace. |
| **Logging/monitoring** | A Singleton `AppLogger` timestamps every significant action and error to `logs/app.log`. |
| **Resource efficiency** | Files are opened, read/written, and closed per operation (try-with-resources); no persistent open handles or connection pools are needed for this scale. |

## Installation & Running

**Prerequisites:** JDK 17 or later (`javac`/`java` on your `PATH`).

```bash
# 1. Clone the repository
git clone https://github.com/eshant01/Library-Management-System-SLMS-.git
cd LibraryManagementSystem

# 2. Compile the main application
javac -d out $(find src/main -name "*.java")

# 3. Run it
java -cp out com.slms.Main
```

Or simply run the provided helper script:

```bash
chmod +x build.sh
./build.sh run
```

On first run, SLMS automatically creates a default Admin account:

```
username: admin
password: admin123
```

(Change this password immediately after first login in a real deployment —
`UserService` has no built-in password-change flow yet; see
**Future Enhancements** in the report.)

## Testing

The project ships with a dependency-free unit test suite (no JUnit/Maven
required — the build has no internet access to Maven Central, so tests run
against small in-memory fake DAOs instead):

```bash
# Compile main + test sources
javac -d out $(find src/main -name "*.java")
javac -d testout -cp out $(find src/test -name "*.java")

# Run all tests
java -cp out:testout com.slms.test.TestRunner
```

Or: `./build.sh test`

Expected output ends with a summary line such as:

```
Passed: 20  Failed: 0
```

Tests cover: input validation (empty fields, invalid ISBNs, non-positive
counts), duplicate detection (usernames, ISBNs), authentication
(correct/incorrect credentials), password hashing, book issue/return
lifecycle, availability tracking, and fine calculation.

## Screenshots

Run `./build.sh run` and try the flow: **Login as admin → Manage Catalog →
Add Book → Manage Transactions → Issue Book → Reports → Inventory
Summary** to see the full module set in action. (Add your own terminal
screenshots here before submission if required by your instructor.)

## License

Academic project submitted for course evaluation. No specific license
applied; reuse for educational purposes.

## Author
Name: Eshant Baranwal
Registration Number: 24BCY10212
