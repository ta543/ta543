# Overview of the Sales Feature with Trial Key Codes

## Objective
The sales feature is designed to allow potential customers to experience the product through a trial period activated using unique key codes. These codes are manually assigned by admins to salespeople, who then distribute them to potential customers. The system uses a combination of email addresses and device fingerprints to prevent multiple trial activations from the same device or user. This ensures controlled and secure access to the trial, with the registration process limited to desktop computers.

## System Components and Their Roles

### 1. Angular Frontend
The Angular frontend serves as the primary interface for both users and admins.

- **User Interface for Trial Activation:**
  - Users can enter their email and the key code provided by the salesperson in the trial activation form.
  - The frontend generates a unique device fingerprint using the FingerprintJS library, which helps identify the user’s device uniquely.
  - This information, including the email, key code, and device fingerprint, is then sent to the backend for validation and activation of the trial.

- **Admin Interface for Key Code Management:**
  - Admins use this interface to generate and manage key codes.
  - Admins can specify details like trial duration and any relevant metadata.
  - Admins can view and manage the status of all key codes (e.g., active, used, or expired) and assign these codes to salespeople for distribution.

- **Connecting to Backend Services:**
  - The frontend communicates with backend services through an API Gateway.
  - All actions such as registration and trial activation are authenticated using JWT tokens to ensure secure communication.

### 2. Trial Management Microservice
This microservice handles all the logic related to key codes and trial activations.

- **Generating and Storing Key Codes:**
  - Admins generate new key codes, which are stored in the database with details like status, trial duration, and metadata.

- **Assigning Key Codes to Salespeople:**
  - Admins assign the generated key codes to specific salespeople who are responsible for distributing them to potential customers.

- **Handling Trial Activation Requests:**
  - When a user submits a trial activation request, the service validates:
    - **Key Code:** Checks if the key code exists, is active, and has not expired.
    - **Email:** Ensures that the email has not been used for a trial before. If it has, the request is denied.
    - **Device Fingerprint:** Verifies if the device fingerprint has been used for a trial before. If it has, the request is also denied.

- **Storing Activation Information:**
  - If the key code, email, and device fingerprint are all valid and unique, the service activates the trial, storing the activation information in the database with the trial start and end dates.
  - The key code status is updated to “used” to prevent it from being reused.

- **Sending Notifications:**
  - Once the trial is successfully activated, the service triggers the Email Service to send a confirmation email to the user with details about the trial period.

### 3. User Service
The User Service manages user accounts, authentication, and verification.

- **User Registration and Authentication:**
  - This service handles the collection of user email addresses and basic profile information during registration.
  - It sends a verification email to users to confirm their email address before they can activate a trial.

- **Managing User Data:**
  - Stores user data in a database and provides API endpoints to access this information.
  - Shares user data, such as email and verification status, with the Trial Management Microservice during trial activation requests.

- **Supporting Authentication:**
  - Issues JWT tokens to ensure secure communication between the frontend and backend.
  - Manages user sessions and supports login and logout functionalities.

### 4. Email Service
The Email Service manages all email communications related to user registration, trial activations, and reminders.

- **Verifying Emails:**
  - Sends a verification email to new users containing a link that users must click to verify their email address.
  - The User Service is notified once the email is successfully verified, allowing the user to proceed with the trial activation process.

- **Confirming Trial Activations:**
  - Sends a confirmation email to the user after a successful trial activation, including details like the trial start and end dates.

- **Sending Expiration Reminders:**
  - Sends reminder emails to users as their trial period is nearing its end, encouraging them to subscribe to the service.

- **API Integration:**
  - Provides API endpoints that can be called by other services, such as the Trial Management Microservice, to send emails based on specific events (e.g., trial activation, email verification).

### 5. API Gateway
The API Gateway serves as the entry point for all client requests, ensuring secure and efficient communication between the frontend and backend services.

- **Routing Requests:**
  - Receives all client requests and forwards them to the appropriate microservice based on the request URL and method.
  - Ensures that the request is authenticated and authorized before forwarding it to the corresponding microservice.

- **Managing Authentication and Authorization:**
  - Verifies JWT tokens to ensure that the user is authenticated.
  - Implements role-based access control (RBAC) to restrict certain actions, such as key code management, to authorized users only.

- **Rate Limiting and Throttling:**
  - Implements rate limiting on endpoints to prevent abuse, such as repeated trial activation attempts or brute-force attacks.
  - Ensures that no single IP address can attempt too many trial activations in a short period.

- **Load Balancing:**
  - Distributes incoming requests across multiple instances of microservices to ensure high availability and scalability.

### 6. Database
The database is the central storage point for all data related to key codes, trial activations, and user information.

- **Storing Key Codes:**
  - Saves each key code with details like trial duration, status, and any associated metadata.
  - Tracks the status of each key code (e.g., active, used, expired) to prevent misuse.

- **Recording Trial Activations:**
  - Stores each trial activation along with the associated email and device fingerprint.
  - Records information such as activation date, expiration date, and status (active or expired).
  - Ensures that no two trials can be activated using the same email and device fingerprint combination.

- **Managing User Data:**
  - Stores user information, including email, profile details, and verification status.
  - Allows the system to verify whether a user is eligible for a trial activation.

- **Ensuring Security and Data Integrity:**
  - Encrypts sensitive data such as emails and fingerprints to protect user information.
  - Uses constraints and foreign key relationships to maintain data integrity and consistency.

### 7. Security and Compliance
Security and compliance are integral to the sales feature, ensuring that user data is protected and that the system adheres to regulatory standards.

- **Authentication and Authorization:**
  - All user actions are authenticated using JWT tokens to ensure secure communication between the frontend and backend.
  - Admin actions such as key code management are restricted based on roles and permissions, preventing unauthorized access.

- **Data Encryption:**
  - Data such as email addresses, fingerprints, and key codes are encrypted both in transit and at rest.
  - HTTPS is enforced for all API communication to protect data integrity.

- **Rate Limiting:**
  - Rate limiting on API endpoints helps prevent abuse of the trial activation feature and brute-force attacks.
  - Monitors for suspicious activities and flags potential abuse cases for review.

- **Privacy Compliance:**
  - Users are informed about the use of device fingerprinting and email verification in the privacy policy.
  - Users have the right to request data deletion or opt-out as required by regulations like GDPR.

## System Flow

1. **Admin Key Code Management:**
   - Admins create key codes using the admin interface and store them in the database.
   - Admins assign these key codes to salespeople, who will distribute them to potential users.

2. **Salesperson Key Code Distribution:**
   - Salespeople receive the key codes from the admin and distribute them to potential customers through various communication channels (e.g., email, phone).

3. **User Registration and Activation:**
   - Users register on the Angular frontend and verify their email through the verification link sent by the Email Service.
   - A unique device fingerprint is generated and sent along with the email to the backend for trial activation.

4. **Trial Activation Process:**
   - Users enter the key code received from the salesperson.
   - The Trial Management Microservice validates the key code, email, and device fingerprint combination.
   - If valid, the trial is activated, and the details are stored in the database. A confirmation email is sent to the user.
   - If invalid, the activation is denied, and an error message is returned.

5. **Monitoring and Notifications:**
   - The system monitors the trial period and updates the trial status accordingly.
   - Reminder emails are sent to users as their trial period nears its end, encouraging them to subscribe.
   - User access is adjusted based on the trial status.

![diagram-export-21-09-2024-17_15_38](https://github.com/user-attachments/assets/1ed83852-6687-4123-a3e8-fefec4d71146)

