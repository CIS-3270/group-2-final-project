# Airline Reservation System

A desktop flight booking app written in Java for CIS 3270 at Georgia State University. Customers can search for flights and book them, and admins can also manage the flight schedule. It works like a small Expedia or CheapTickets.

## Project overview

The app starts with a splash screen and then opens the main menu, where users register or log in. After login, the app opens a dashboard that depends on the account type:

- **Customer**: search flights, book flights, view or delete their bookings, log out.
- **Admin**: everything a customer can do, plus add, update and delete flights.

All data is stored in a relational database and accessed through JDBC. Every screen has a button that returns to the main menu.

## Features

**Accounts**
- Register with first name, last name, address, zip, state, username, password, email, SSN, and a security question and answer
- Log in with a username and password
- Recover a password with your username and the answer to your security question

**Customers**
- Search flights by origin, destination, date and time
- Book a flight, view your bookings and delete a booking
- Booking rules (the app shows an error message when one is broken, and keeps running):
  - You cannot book the same flight twice
  - You cannot book a flight whose time overlaps a flight you already booked
  - You cannot book a full flight

**Admins**
- Everything customers can do
- Add, update and delete flights

## Tech stack

| Layer | Choice |
| --- | --- |
| Language | Java [17 / 21] |
| GUI | [Swing / JavaFX] |
| Database | [MySQL / PostgreSQL / Oracle / SQL Server] |
| Data access | JDBC with the DAO pattern |
| IDE (optional) | [IntelliJ IDEA / Eclipse] |

## Prerequisites

- **JDK [17 / 21]**. Check your version with `java -version`.
- **[Database] [version]**, running locally or on a server you can reach.
- **JDBC driver jar** in `lib/`:

  | Database | Driver jar | Example JDBC URL |
  | --- | --- | --- |
  | MySQL | `mysql-connector-j-[x.y.z].jar` | `jdbc:mysql://localhost:3306/airline` |
  | PostgreSQL | `postgresql-[x.y.z].jar` | `jdbc:postgresql://localhost:5432/airline` |
  | Oracle | `ojdbc11.jar` | `jdbc:oracle:thin:@localhost:1521/XEPDB1` |
  | SQL Server | `mssql-jdbc-[x.y.z].jre11.jar` | `jdbc:sqlserver://localhost:1433;databaseName=airline;encrypt=false` |

- **JavaFX only:** the JavaFX SDK [version], downloaded from [openjfx.io](https://openjfx.io).

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/CIS-3270/group-2-final-project.git
cd group-2-final-project
```

### 2. Create the database

`sql/schema.sql` creates the tables. `sql/seed.sql` adds sample flights and a default admin account.

```bash
# MySQL
mysql -u root -p -e "CREATE DATABASE airline"
mysql -u root -p airline < sql/schema.sql
mysql -u root -p airline < sql/seed.sql

# PostgreSQL
createdb airline
psql -d airline -f sql/schema.sql
psql -d airline -f sql/seed.sql
```

For Oracle or SQL Server, run both scripts in SQL Developer or SSMS.

### 3. Configure the database connection

```bash
cp config/db.properties.example config/db.properties
```

Then edit `config/db.properties`:

```properties
db.url=jdbc:mysql://localhost:3306/airline
db.user=[your_db_user]
db.password=[your_db_password]
```

`config/db.properties` is listed in `.gitignore`, so each teammate keeps their own copy. Never commit real credentials.

### 4. Compile and run

Run these commands from the project root.

**macOS or Linux (Swing):**

```bash
mkdir -p out
javac -d out -cp "lib/*" $(find src -name "*.java")
java -cp "out:lib/*" airline.Main
```

**Windows:** use `;` instead of `:` in the classpath, or run the app from your IDE.

**JavaFX:** add `--module-path [path/to/javafx-sdk]/lib --add-modules javafx.controls` to both the `javac` and `java` commands.

**IntelliJ or Eclipse:** add the jars in `lib/` as a library, set the working directory to the project root, and run `airline.Main`.

## How to use the app

1. **Start the app.** The splash screen shows for a few seconds, then the main menu opens.
2. **Register.** Click **Register**, fill in every field, choose a security question and click **Submit**. If a field is invalid (such as a bad zip, email or SSN format, or a username that is already taken), you will see a message.
3. **Log in.** Enter your username and password. If you forgot your password, click **Forgot password**, enter your username and answer your security question.
4. **Search for flights.** On the dashboard, enter an origin, a destination and a date, plus a time if you want one, then click **Search**.
5. **Book a flight.** Select a flight and click **Book**. The app refuses the booking if it is a duplicate, overlaps another of your bookings, or the flight is full.
6. **Manage your bookings.** Open **My Flights** to see your bookings. Select one and click **Delete** to cancel it.
7. **Admin tasks.** Log in as an admin to see the **Manage Flights** tab, where you can add, edit or delete flights. The default admin account is `admin` with the password `[set in sql/seed.sql]`.
8. **Leave.** Use **Main Menu** on any screen to go back, or **Log Out** to end your session.

## Project structure

```
airline-reservation-system/
├── README.md                         # This file
├── .gitignore                        # Ignores build output, IDE files, local config
├── config/
│   └── db.properties.example         # Template for DB settings; copy to db.properties
├── lib/
│   └── [jdbc-driver].jar             # JDBC driver (committed so every teammate has the same one)
├── sql/
│   ├── schema.sql                    # CREATE TABLE statements for users, flights, bookings
│   └── seed.sql                      # Sample flights and the default admin account
├── docs/
│   ├── uml-class-diagram.png         # UML class diagram (export from draw.io / Lucidchart)
│   ├── data-model.md                 # ER diagram and description of each table and column
│   └── screenshots/                  # Screenshots of each screen for the final report
└── src/
    └── airline/
        ├── Main.java                 # Entry point: shows the splash screen, then the main menu
        ├── db/
        │   └── DatabaseConnection.java   # Reads db.properties and opens JDBC connections
        ├── model/
        │   ├── User.java             # Abstract base class for every account
        │   ├── Customer.java         # Account that can search and book flights
        │   ├── Admin.java            # Customer that can also manage flights
        │   ├── Address.java          # Street, city, state and zip (part of User)
        │   ├── Flight.java           # Flight number, route, times, capacity, seats booked
        │   └── Booking.java          # Links a Customer to a Flight
        ├── dao/
        │   ├── CrudDao.java          # Generic interface: find, findAll, insert, update, delete
        │   ├── UserDao.java          # SQL for the users table
        │   ├── FlightDao.java        # SQL for the flights table, including search
        │   └── BookingDao.java       # SQL for the bookings table
        ├── service/
        │   ├── AuthService.java      # Registration, login and password recovery
        │   └── BookingService.java   # Enforces the duplicate, conflict and full-flight rules
        ├── exception/
        │   ├── AppException.java             # Base class for all app exceptions
        │   ├── AuthenticationException.java  # Bad login or a wrong security answer
        │   ├── DataAccessException.java      # Wraps SQLException so the GUI can show a message
        │   ├── BookingException.java         # Base class for booking rule violations
        │   ├── DuplicateBookingException.java
        │   ├── ScheduleConflictException.java
        │   └── FlightFullException.java
        ├── util/
        │   ├── Validator.java        # Checks zip, email, SSN and required fields
        │   └── PasswordHasher.java   # Hashes passwords and security answers before they are stored
        └── gui/
            ├── ScreenManager.java    # Switches between screens in one window
            ├── BaseScreen.java       # Abstract screen with a shared "Main Menu" button
            ├── SplashScreen.java     # Startup splash
            ├── MainMenuScreen.java   # Register and Login buttons
            ├── RegisterScreen.java   # Registration form
            ├── LoginScreen.java      # Username and password form
            ├── PasswordRecoveryScreen.java # Username, then the security question
            ├── CustomerDashboard.java  # Tabs for Search Flights and My Flights
            ├── AdminDashboard.java     # CustomerDashboard plus a Manage Flights tab
            ├── FlightSearchPanel.java  # Search form and results table with a Book button
            ├── MyFlightsPanel.java     # The user's bookings with a Delete button
            └── FlightFormDialog.java   # Add or edit a flight (admins only)
```

## Class design

### Hierarchy

```
User (abstract)                         CrudDao<T, K> (interface)
 └── Customer                            ├── UserDao
      └── Admin                          ├── FlightDao
                                         └── BookingDao
BaseScreen (abstract)
 ├── MainMenuScreen, LoginScreen,       AppException
 │   RegisterScreen, PasswordRecoveryScreen ├── AuthenticationException
 └── CustomerDashboard                   ├── DataAccessException
      └── AdminDashboard                 └── BookingException
                                              ├── DuplicateBookingException
                                              ├── ScheduleConflictException
                                              └── FlightFullException
```

### How the design meets the OOP requirements

| Requirement | Where it is used |
| --- | --- |
| Abstract class | `User`, which declares the abstract method `getRole()`. `BaseScreen`, which declares the abstract method `buildContent()`. |
| Interface | `CrudDao<T, K>`, which all three DAOs implement. |
| Inheritance | `Customer` extends `User` and `Admin` extends `Customer`. `AdminDashboard` extends `CustomerDashboard`. Each screen extends `BaseScreen`. The booking exceptions extend `BookingException`. |
| Polymorphism | `getRole()` and `canManageFlights()` are overridden in `Customer` and `Admin`, and the login code uses them to open the right dashboard. `ScreenManager` shows any `BaseScreen` through the same method. |
| Encapsulation | Every model class has private fields with getters and setters, and setters validate their input (for example, `Flight` rejects negative capacity). |
| Composition | `User` has an `Address`. `Booking` has a `Customer` and a `Flight`. `BookingService` uses a `FlightDao` and a `BookingDao`. |

### Error handling

- The DAOs catch `SQLException` and throw `DataAccessException` instead.
- The services throw `AuthenticationException` or one of the `BookingException` subclasses.
- The GUI catches `AppException` and shows a dialog, so nothing crashes the app.
- `Main` sets a default uncaught-exception handler as a last line of defense.

## Team members and roles

Repo owner: Natasha Narine

| Name | Role | Main areas |
| --- | --- | --- |
| Natasha Narine | [Role] | [Areas] |
| Justin Okoronta | [Role] | [Areas] |
| Davon Deloatch | [Role] | [Areas] |
| Jonathan Lim | [Role] | [Areas] |

Suggested split: database (`sql/`, `db/`, `dao/`), backend logic (`model/`, `service/`, `exception/`), GUI for login, registration and recovery, and GUI for the dashboards plus `docs/`.

## Course note and license

This project is a final project for CIS 3270 at Georgia State University, Fall 2026, taught by Prof. Andrea Aria. It is an academic project and is not meant for production use.

- Use only made-up test data. Never enter real SSNs or real passwords.
- License: [None, all rights reserved / MIT]
