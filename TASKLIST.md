# Online Reservation System — Project Roadmap & Task List

An organized, step-by-step implementation guide to build your Java Swing + JDBC transport reservation system.

---

## 📁 Recommended Project Structure
```text
Reservation_System/
├── pom.xml
├── TASKLIST.md
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/reservation/
    │   │       ├── Main.java              # Entry point (initializes FlatLaf & DB, opens LoginForm)
    │   │       ├── db/
    │   │       │   └── Database.java      # DB connection & table auto-creation
    │   │       ├── model/
    │   │       │   ├── User.java          # User entity
    │   │       │   ├── Train.java         # Train entity
    │   │       │   └── Reservation.java   # Reservation entity
    │   │       ├── dao/
    │   │       │   ├── UserDAO.java       # User authentication queries
    │   │       │   ├── TrainDAO.java      # Train lookup queries
    │   │       │   └── ReservationDAO.java# Booking & cancellation queries
    │   │       └── ui/
    │   │           ├── LoginForm.java     # Login window
    │   │           ├── DashboardFrame.java# Main tabbed dashboard
    │   │           ├── BookingPanel.java  # Ticket booking form
    │   │           └── CancelPanel.java   # Ticket cancellation form
    │   └── resources/                     # Icons, local SQLite database, assets
    └── test/
        └── java/
```

---

## 📋 Task Checklist

### Phase 1: Database Architecture & Setup
- [ ] **1.1 Design Database Tables**
  - **`users` table**: `id INTEGER PRIMARY KEY AUTOINCREMENT`, `username TEXT UNIQUE NOT NULL`, `password TEXT NOT NULL`
  - **`trains` table**: `train_number INTEGER PRIMARY KEY`, `train_name TEXT NOT NULL`, `source TEXT NOT NULL`, `destination TEXT NOT NULL`
  - **`reservations` table**:
    - `pnr_number TEXT PRIMARY KEY` (Unique 6 to 8-digit identifier)
    - `passenger_name TEXT NOT NULL`
    - `train_number INTEGER NOT NULL`
    - `train_name TEXT NOT NULL`
    - `class_type TEXT NOT NULL` (e.g. Sleeper, AC 3-Tier, AC 2-Tier, AC First Class)
    - `journey_date TEXT NOT NULL` (Format: `YYYY-MM-DD`)
    - `source TEXT NOT NULL`
    - `destination TEXT NOT NULL`
    - `booking_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP`
- [ ] **1.2 Create `Database.java` Helper**
  - Define `getConnection()` returning a `java.sql.Connection`.
  - Add an `initDatabase()` method that executes `CREATE TABLE IF NOT EXISTS` for all tables.
  - Insert initial seed records:
    - Default user: `admin` / `admin123`
    - Sample trains:
      - `12001` - *Shatabdi Express* (New Delhi -> Bhopal)
      - `12951` - *Rajdhani Express* (Mumbai Central -> New Delhi)
      - `12301` - *Howrah Rajdhani* (Howrah -> New Delhi)
      - `12626` - *Kerala Express* (New Delhi -> Trivandrum)

---

### Phase 2: Authentication Module (Login)
- [ ] **2.1 Build `LoginForm.java` (Swing GUI)**
  - Window title: *"Reservation System - Login"*
  - Center on screen using `setLocationRelativeTo(null)`
  - Input components:
    - Username field (`JTextField`)
    - Password field (`JPasswordField`)
    - "Login" button (`JButton`)
    - "Reset" button (`JButton`) to clear inputs
- [ ] **2.2 Implement `UserDAO.java` Authentication Logic**
  - Method: `public boolean authenticate(String username, String password)`
  - Use `PreparedStatement`: `SELECT * FROM users WHERE username = ? AND password = ?`
  - On Success: Open `DashboardFrame.java` and dispose `LoginForm`.
  - On Failure: Display `JOptionPane.showMessageDialog` (*"Invalid username or password"*).

---

### Phase 3: Main Dashboard
- [ ] **3.1 Build `DashboardFrame.java`**
  - Main window container containing:
    - Header with greeting and current user
    - `JTabbedPane` with two main tabs:
      1. **Book Ticket** (`BookingPanel`)
      2. **Cancel Ticket** (`CancelPanel`)
    - Logout button in menu or header to return to `LoginForm`.

---

### Phase 4: Reservation & Booking Module
- [ ] **4.1 Build `BookingPanel.java` (Swing Form)**
  - Form Fields:
    - **Passenger Name**: `JTextField`
    - **Train Number**: `JTextField` (numeric only)
    - **Train Name**: `JTextField` (read-only: `setEditable(false)`)
    - **Class Type**: `JComboBox<String>` (Options: `Sleeper`, `AC 3-Tier`, `AC 2-Tier`, `AC First Class`)
    - **Date of Journey**: `JTextField` (placeholder: `YYYY-MM-DD`)
    - **Source Station**: `JTextField`
    - **Destination Station**: `JTextField`
  - Action Buttons:
    - **"Book Ticket"**: Submits reservation
    - **"Clear"**: Resets all fields
- [ ] **4.2 Implement Train Auto-Population**
  - Add `FocusListener` or search button on the **Train Number** field.
  - Query `trains` table via `TrainDAO.java` for entered train number.
  - Automatically fill **Train Name**, **Source**, and **Destination** if train exists.
- [ ] **4.3 Input Validation**
  - Ensure all mandatory fields are filled.
  - Validate that train number is numeric.
  - Validate date matches format `YYYY-MM-DD` and is not in the past.
- [ ] **4.4 PNR Generation & Database Insertion**
  - Create a method in `ReservationDAO.java` to generate a random unique PNR (e.g., 6–8 digits).
  - Insert record into `reservations` table using parameterized `PreparedStatement`.
- [ ] **4.5 Confirmation Dialog**
  - Display a styled `JOptionPane.showMessageDialog` showing summary:
    - PNR Number
    - Passenger Name
    - Train details
    - Journey Date & Class
  - Clear the form fields upon successful confirmation.

---

### Phase 5: Cancellation Module
- [ ] **5.1 Build `CancelPanel.java`**
  - Input field for **PNR Number**.
  - **"Fetch Details"** button.
  - Card/Display area (read-only `JTextArea` or labeled panel) showing:
    - Passenger Name
    - Train Number & Name
    - Source $\rightarrow$ Destination
    - Date & Class
  - **"Confirm Cancellation"** button (disabled by default until valid PNR is fetched).
- [ ] **5.2 Fetch Booking by PNR**
  - In `ReservationDAO.java`: `public Reservation getReservationByPNR(String pnr)`
  - Execute `SELECT * FROM reservations WHERE pnr_number = ?`.
  - If found: Populate the details card and enable the "Confirm Cancellation" button.
  - If not found: Show `JOptionPane.showMessageDialog` (*"No booking found with this PNR"*).
- [ ] **5.3 Confirm Cancellation & Deletion**
  - On clicking "Confirm Cancellation", prompt user:
    - `JOptionPane.showConfirmDialog(..., "Are you sure you want to cancel this ticket?", ...)`
  - If user selects **YES**:
    - Call `deleteReservation(String pnr)` in `ReservationDAO.java`.
    - Run `DELETE FROM reservations WHERE pnr_number = ?`.
    - Show *"Ticket cancelled successfully!"*.
    - Reset fields and re-disable the cancellation button.

---

### Phase 6: Polish, UI Theme & Security
- [ ] **6.1 Enable FlatLaf Look and Feel**
  - In `Main.java`, initialize before launching UI:
    ```java
    FlatLightLaf.setup(); // or FlatDarkLaf.setup();
    ```
- [ ] **6.2 Database Resource Management**
  - Ensure all `Connection`, `PreparedStatement`, and `ResultSet` use `try-with-resources` blocks to avoid memory leaks.
- [ ] **6.3 Packaging & Run Testing**
  - Test building the project: `mvn clean package`
  - Verify all validations, edge cases, and user workflows.
