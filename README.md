Below is a complete markdown document that outlines the user stories with associated technologies, the database schema, and the concrete project structure for your Flask mobile web application with 3D visualization.

---

# Mobile Web App for Body Measurements with 3D Visualization

This document describes the complete requirements and design for the mobile web app. The application allows users to register, log in, input personal data, capture body measurements via real-time pose detection, verify and export measurements, and finally visualize the measurements on an interactive 3D human model.

---

## User Stories & Associated Technologies

### 1. User Registration & Login

**User Story:**  
_As a new user, I want to register an account so that I can securely log in and use the app. As an existing user, I want to log in using my credentials so that I can access my profile and data._

**Technologies:**
- **Backend:** Flask, SQLAlchemy (ORM), MySQL
- **Security:** Bcrypt (or Argon2) for password hashing
- **Front-End:** HTML5, CSS3, Bootstrap (or Tailwind CSS for responsiveness)
- **Templates:** Jinja2 (for server-side rendered pages) or RESTful API endpoints if using a separate front-end framework

**Acceptance Criteria:**
- Registration form collects username, email, and password.
- Input validation on both client-side and server-side.
- Passwords are securely hashed before storage.
- Login page validates credentials and creates a secure session or returns a JWT.

---

### 2. User Profile & Personal Data Input

**User Story:**  
_As a registered user, I want to provide my basic personal data (e.g., height, weight, gender) so that the app can accurately calculate measurements and tailor subsequent processes._

**Technologies:**
- **Backend:** Flask for profile data management, SQLAlchemy, MySQL
- **Front-End:** Responsive HTML/CSS forms (Bootstrap or similar frameworks)
- **Client-Side Validation:** JavaScript (vanilla JS or libraries like jQuery)

**Acceptance Criteria:**
- A secure profile page accessible only after authentication.
- A form for adding/editing personal details that updates the database.
- Real-time validation feedback on input fields.

---

### 3. Real-Time Pose Detection & Measurement Capture

**User Story:**  
_As a logged-in user, I want to capture real-time body data via my mobile camera so that the app can use computer vision to determine my body measurements accurately._

**Technologies:**
- **Client-Side Detection:**
  - **Camera & Pose Detection:** JavaScript with TensorFlow.js or MediaPipe.
  - **UI/UX:** Mobile-friendly JavaScript and CSS (using Bootstrap or a specialized mobile UI kit).
- **Alternate/Server-Side Option:**
  - **Backend Processing:** Flask with OpenCV and MediaPipe (if processing on the server).
  - **Data Transfer:** HTTP(s) endpoints for image/frame uploads.
- **Data Format:** JSON payloads containing key pose points and computed measurements.

**Acceptance Criteria:**
- The app accesses the phone’s camera with proper permission.
- A live feed displays with overlaid pose detection markers (key points).
- Body measurements (e.g., limb lengths, distances) are calculated in real time.
- Processed measurement data is sent as JSON to a Flask endpoint.

---

### 4. Measurement Verification & Re-Capture

**User Story:**  
_As a user who just captured my measurements, I want to review them for accuracy and be prompted to re-capture if necessary so that only correct data is saved._

**Technologies:**
- **Backend:** Flask to process and save measurement data and update the accuracy status.
- **Front-End:** JavaScript (or frameworks like React/Vue for enhanced interactivity) for dynamic feedback.
- **Design:** Clear UI elements (alerts, confirmation dialogs) using CSS frameworks.

**Acceptance Criteria:**
- A summary screen displays the captured measurement data.
- Users can either "Accept" the measurements or choose "Retake" them.
- Accepted measurements are saved to the database with a flag (e.g., `is_accurate`).

---

### 5. Export Measurements to Excel

**User Story:**  
_As a user, I want to export my measurement data to an Excel file so that I can save, share, or use it for further analysis._

**Technologies:**
- **Backend:** Flask endpoint using Pandas or XlsxWriter to generate Excel files.
- **Database:** MySQL (accessed via SQLAlchemy) to retrieve measurement records.
- **Front-End:** A simple "Export" button on the measurement review page that triggers file download.

**Acceptance Criteria:**
- Clicking the export button triggers a download of an Excel (.xlsx) file containing the user’s measurement history.
- The Excel file is formatted and clearly structured for ease of use.

---

### 6. 3D Model Visualization & Interaction

**User Story:**  
_As a user, I want to visualize my body measurements on a 3D human model so that I can see a dynamic representation of my physique and interact with it (rotate, zoom, etc.)._

**Technologies:**
- **Front-End:** 
  - **3D Rendering:** Three.js (or Babylon.js) for interactive 3D rendering in the browser.
  - **Template:** A dedicated HTML template (e.g., `model.html`) for the 3D view.
- **Backend:** Flask to serve static 3D model files and calculation parameters.
- **Data Transfer:** JSON for sending measurement parameters to adjust the model.
- **UI/UX:** Mobile-friendly controls for interaction (rotate, zoom, pan).

**Acceptance Criteria:**
- The system loads a default 3D human mesh.
- The model dimensions are dynamically adjusted based on the captured measurements.
- Users can interact with the 3D model seamlessly on mobile devices.

---

## Database Schema

### Users Table

```sql
CREATE TABLE Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Profiles Table

```sql
CREATE TABLE Profiles (
    profile_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    height FLOAT,
    weight FLOAT,
    gender VARCHAR(50),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);
```

### Measurements Table

```sql
CREATE TABLE Measurements (
    measurement_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    measurement_data JSON,  -- Stores computed measurements (e.g., chest, waist, hips, etc.)
    is_accurate BOOLEAN DEFAULT FALSE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);
```

### Models Table (Optional for storing 3D model file references)

```sql
CREATE TABLE Models (
    model_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    model_path VARCHAR(255),  -- Reference to model file location if needed
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);
```

---

## Project Structure

Below is an example project directory structure for your Flask-based mobile web app that includes a dedicated `model.html` for the 3D view:

```
my_measurement_app/
├── app.py                        # Main Flask application entry point
├── requirements.txt              # Python dependencies (Flask, SQLAlchemy, etc.)
├── config.py                     # Configuration settings (database URI, secret keys, etc.)
├── run.py                        # Script to run the application (for development/production)
├── migrations/                   # Database migration scripts (if using Flask-Migrate)
├── models/
│   ├── __init__.py               # Initialization file for all models
│   ├── user.py                   # User model (for registration and login)
│   ├── profile.py                # Profile model (for personal data like height, weight, etc.)
│   └── measurement.py            # Measurement model (for body measurement data)
├── routes/
│   ├── __init__.py               # Blueprint initialization for routes
│   ├── auth.py                   # Endpoints for registration, login, and logout
│   ├── profile.py                # Endpoints for profile viewing and updating
│   ├── measurement.py            # Endpoints for capturing, verifying, and saving measurements
│   ├── export.py                 # Endpoint for exporting measurements to Excel
│   └── model.py                  # Endpoint to serve the 3D model view
├── static/
│   ├── css/
│   │   └── main.css              # Custom stylesheets (including Bootstrap overrides)
│   ├── js/
│   │   ├── model-viewer.js       # JavaScript for initializing and interacting with the 3D model (e.g., Three.js code)
│   │   └── main.js               # Additional client-side scripts (e.g., for camera handling and form validations)
│   └── images/                   # Directory for static images or icons
├── templates/
│   ├── layout.html               # Base template with common header, footer, and navigation
│   ├── login.html                # Template for the login page
│   ├── register.html             # Template for the registration page
│   ├── profile.html              # Template for the user profile and personal data input
│   ├── capture.html              # Template for the real-time measurement capture page
│   ├── verify.html               # Template for measurement verification and re-capture
│   ├── export.html               # Template for export confirmation/download
│   └── model.html                # Dedicated template for the 3D model visualization view
└── docs/
    └── README.md                 # Project documentation (features, API endpoints, setup instructions, etc.)
```

---

## Additional Notes

- **Security & Data Integrity:**  
  - Deploy using HTTPS and secure your endpoints with proper session or token management.  
  - Store passwords using strong hashing (e.g., Bcrypt or Argon2) and always validate inputs on both client and server.

- **Responsiveness & User Experience:**  
  - Use frameworks like Bootstrap or Tailwind CSS to ensure the application is mobile-friendly.  
  - Implement progressive enhancement for cases with limited JavaScript support.

- **Performance:**  
  - For real-time pose detection, consider client-side processing with TensorFlow.js/MediaPipe if device performance allows, reducing server load.  
  - Cache static assets and consider CDN distribution for faster load times on mobile devices.

- **Scalability & Maintainability:**  
  - Use Flask Blueprints to modularize your code (authentication, profile, measurement, export, and model features).  
  - Containerize your app using Docker for easier deployment and scalability.

- **Testing & Documentation:**  
  - Write unit and integration tests using a framework like pytest to ensure each endpoint works as intended.  
  - Maintain detailed developer documentation in the `/docs` folder for easier onboarding and project handover.

---

This complete markdown file should serve as a blueprint for creating your Flask mobile web application, guiding development from user registration through to interactive 3D visualization.
