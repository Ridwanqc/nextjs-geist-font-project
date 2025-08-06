Below is the detailed implementation plan in proper markdown. This plan outlines all dependent files (SQL, controllers, models, views, filters) along with error handling, best practices, and modern UI/UX considerations.

---

### 1. SQL Table Structure (database.sql)

- **reservations Table**  
 - Fields:  
  - id INT PRIMARY KEY AUTO_INCREMENT  
  - customer_name VARCHAR(100) NOT NULL  
  - phone_number VARCHAR(20) NOT NULL  
  - reservation_date DATE NOT NULL  
  - number_of_people INT NOT NULL  
  - down_payment DECIMAL(10,2) NOT NULL  
  - status ENUM('not yet processed','completed','canceled') DEFAULT 'not yet processed'  
  - notes TEXT  
  - created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP  
  - updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  

- **food_orders Table**  
 - Fields:  
  - id INT PRIMARY KEY AUTO_INCREMENT  
  - reservation_id INT NOT NULL (foreign key to reservations.id)  
  - menu_name VARCHAR(100) NOT NULL  
  - quantity INT NOT NULL  
  - notes TEXT  
  - created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP  
  - updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  
 - Add a foreign key constraint to ensure referential integrity.

---

### 2. Models

- **app/Models/ReservationModel.php**  
 - Extend CodeIgniter\Model  
 - Set $table = 'reservations', define $allowedFields, and enable timestamps.  
 - Include standard error handling (try/catch if needed) in methods that are overridden.

- **app/Models/FoodOrderModel.php**  
 - Similar structure as ReservationModel with $table set to 'food_orders' and appropriate $allowedFields.

---

### 3. Controllers

#### a. Admin Authentication  
- **app/Controllers/Auth.php**  
 - Create methods:  
  - index() to load the login form  
  - loginPost() to validate posted username and password (hardcode: username “admin”, password “admin123”). On success, set session data; on failure, redirect back with an error message.  
  - logout() to destroy the session.  
 - Use CodeIgniter’s built-in validation library to sanitize input.

#### b. Reservation Handling  
- **app/Controllers/Reservation.php**  
 - Methods:  
  - index(): List all reservations with pagination (if necessary).  
  - create(): Load reservation input form view.  
  - store(): Validate inputs then save to database using ReservationModel; redirect with success or error messages.  
  - printReceipt($id): Retrieve reservation and related food orders, load the receipt view, and then generate PDF via DomPDF. Wrap PDF generation in try/catch for error handling.  
  - exportExcel(): Retrieve filtered reservations then use PhpSpreadsheet to generate and export an Excel file.

#### c. Food Orders  
- **app/Controllers/FoodOrder.php**  
 - Methods:  
  - create($reservationId): Load a food order input form view (reservation_id provided via hidden input).  
  - store(): Validate and store food order using FoodOrderModel. Provide proper redirection and error feedback.

#### d. Reports  
- **app/Controllers/Report.php**  
 - Methods:  
  - index(): Load a filter form (date range and status) along with a table of reservations.  
  - filter(): Process filter inputs and show results.  
  - exportPDF() and exportExcel(): Use DomPDF and PhpSpreadsheet respectively to export the filtered data.  
 - Ensure that export methods validate parameters and report errors gracefully.

---

### 4. Views (All views should use a clean Bootstrap-based layout)

- **app/Views/login.php**  
 - A modern, clean login form with username and password fields inside a Bootstrap container and rows. Use clear typography and spacing.

- **app/Views/reservation_form.php**  
 - Form elements for customer name, phone number, reservation date (use HTML5 date input), number of people, down payment, status (select/dropdown with options “not yet processed”, “completed”, “canceled”), and notes (textarea).  
 - Use Bootstrap classes (container, row, col-md, form-group, etc.) ensuring proper margins and padding.

- **app/Views/food_order_form.php**  
 - A similar Bootstrap form including a hidden input for reservation_id, text input for menu_name, numeric input for quantity, and a textarea for notes.

- **app/Views/receipt.php**  
 - A simple HTML receipt displaying customer name, reservation date, and an order list formatted as a table.  
 - Also include a “Print PDF” button that calls the same receipt functionality on the server.  
 - Ensure the view layout remains clean and legible.

- **app/Views/report.php**  
 - Include a filter form (date range and status dropdown) at the top.  
 - Below, list reservations in a responsive table.  
 - Add buttons for “Export PDF” and “Export Excel”.  

---

### 5. Login Middleware / Filter

- **app/Filters/AuthFilter.php**  
 - Create a custom filter that checks the admin session. If the session is not set or invalid, redirect to the login page.  
 - In the filter, use CodeIgniter’s redirect helper and session management.

- **app/Config/Filters.php**  
 - Add the AuthFilter in the globals or route-specific filters to secure all pages (except the Auth controller’s login pages).

---

### 6. Dependencies & Libraries

- **Composer Dependencies**  
 - Require “dompdf/dompdf” for PDF printing and “phpoffice/phpspreadsheet” for Excel export.  
 - Ensure Composer autoload is correctly configured in CodeIgniter.

- **Error Handling & Best Practices**  
 - Validate every form submission using CodeIgniter’s Validation service.  
 - Use try/catch blocks in PDF/Excel export functions to catch and report errors.  
 - Redirect with session flashdata for both success and error messages.  
 - Sanitize all inputs and use CodeIgniter’s built-in helper functions to avoid SQL injection.

---

### 7. UI/UX Considerations

- Modern, clean Bootstrap forms with ample whitespace, legible typography, and a responsive layout.  
- Navigation should include a secured admin dashboard with a logout button.  
- Reservation reports include export options visibly placed with intuitive text.  
- All pages (login, forms, receipts, reports) follow the same visual grid and vertical rhythm to ensure consistency.

---

### Summary

- Created two SQL tables: reservations and food_orders with proper foreign key constraints.  
- Developed models for both tables with CodeIgniter’s Model class and enabled timestamps.  
- Implemented controllers (Auth, Reservation, FoodOrder, Report) with error handling and validation.  
- Designed Bootstrap-styled views for login, reservation, food order forms, receipt, and reports.  
- Added a custom AuthFilter to protect secured pages and configured it via Filters.php.  
- Integrated DomPDF for PDF receipts and PhpSpreadsheet for Excel exports with try/catch error handling.  
- Adopted best practices for input validation, session management, and responsive UI design.  
- This plan ensures a real-world restaurant reservation application with clear separation of concerns and modern UX.
