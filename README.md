
# Sales Feature with Trial Key Codes

## Objective
The sales feature is designed to allow salespeople within the organization to experience the product through a trial period activated using unique key codes. These codes are manually assigned by admins specifically to salespeople, who will use them to activate their own trial periods. This system uses a combination of email addresses and device fingerprints to prevent multiple trial activations from the same device or user. The registration and activation process is limited to desktop computers to ensure controlled and secure access.

## System Components and Their Roles

### 1. Angular Frontend
The Angular frontend serves as the primary interface for both admins and salespeople.

- **Salesperson Interface for Trial Activation:**
  - Salespeople can enter their email and the key code they have received from the admin in the trial activation form.
  - The frontend generates a unique device fingerprint using the FingerprintJS library to uniquely identify the salesperson’s device.
  - This information, including the email, key code, and device fingerprint, is then sent to the backend for validation and activation of the trial.

- **Admin Interface for Key Code Management:**
  - Admins use this interface to generate and manage key codes.
  - Admins can specify details such as the trial duration and relevant metadata.
  - Admins can view and manage the status of all key codes (e.g., active, used, or expired) and assign these codes to salespeople.

- **Connecting to Backend Services:**
  - The frontend communicates with backend services through an API Gateway.
  - All actions, such as registration and trial activation, are authenticated using JWT tokens to ensure secure communication.

### 2. Trial Management Microservice
This microservice handles all the logic related to key codes and trial activations.

- **Generating and Storing Key Codes:**
  - Admins generate new key codes, which are stored in the database with details such as status, trial duration, and metadata.

- **Assigning Key Codes to Salespeople:**
  - Admins assign the generated key codes to specific salespeople who are responsible for using them to activate their own trials.

- **Handling Trial Activation Requests:**
  - When a salesperson submits a trial activation request, the service validates:
    - **Key Code Validation:** Checks if the key code exists, is active, and has not expired.
    - **Email Validation:** Ensures that the email has not been used for a trial before. If it has, the request is denied.
    - **Device Fingerprint Validation:** Verifies if the device fingerprint has been used for a trial before. If it has, the request is also denied.

- **Storing Activation Information:**
  - If the key code, email, and device fingerprint are all valid and unique, the service activates the trial, storing the activation information in the database with the trial start and end dates.
  - The key code status is updated to “used” to prevent it from being reused.

- **Sending Notifications:**
  - Once the trial is successfully activated, the service triggers the Email Service to send a confirmation email to the salesperson with details about the trial period.

### 3. User Service
The User Service manages user accounts, authentication, and verification.

- **Salesperson Registration and Authentication:**
  - This service handles the collection of salesperson email addresses and basic profile information during registration.
  - It sends a verification email to salespeople to confirm their email address before they can activate a trial.

- **Managing User Data:**
  - Stores salesperson data in a database and provides API endpoints to access this information.
  - Shares user data, such as email and verification status, with the Trial Management Microservice during trial activation requests.

- **Supporting Authentication:**
  - Issues JWT tokens to ensure secure communication between the frontend and backend.
  - Manages user sessions and supports login and logout functionalities.

### 4. Email Service
The Email Service manages all email communications related to salesperson registration, trial activations, and reminders.

- **Verifying Emails:**
  - Sends a verification email to new salespeople containing a link that they must click to verify their email address.
  - The User Service is notified once the email is successfully verified, allowing the salesperson to proceed with the trial activation process.

- **Confirming Trial Activations:**
  - Sends a confirmation email to the salesperson after a successful trial activation, including details such as the trial start and end dates.

- **Sending Expiration Reminders:**
  - Sends reminder emails to salespeople as their trial period is nearing its end, encouraging them to subscribe to the service.

- **API Integration:**
  - Provides API endpoints that can be called by other services, such as the Trial Management Microservice, to send emails based on specific events (e.g., trial activation, email verification).

### 5. API Gateway
The API Gateway serves as the entry point for all client requests, ensuring secure and efficient communication between the frontend and backend services.

- **Routing Requests:**
  - Receives all client requests and forwards them to the appropriate microservice based on the request URL and method.
  - Ensures that the request is authenticated and authorized before forwarding it to the corresponding microservice.

- **Managing Authentication and Authorization:**
  - Verifies JWT tokens to ensure that the salesperson is authenticated.
  - Implements role-based access control (RBAC) to restrict certain actions, such as key code management, to authorized users only.

- **Rate Limiting and Throttling:**
  - Implements rate limiting on endpoints to prevent abuse, such as repeated trial activation attempts or brute-force attacks.
  - Ensures that no single IP address can attempt too many trial activations in a short period.

- **Load Balancing:**
  - Distributes incoming requests across multiple instances of microservices to ensure high availability and scalability.

### 6. Database
The database is the central storage point for all data related to key codes, trial activations, and salesperson information.

- **Storing Key Codes:**
  - Saves each key code with details such as trial duration, status, and any associated metadata.
  - Tracks the status of each key code (e.g., active, used, expired) to prevent misuse.

- **Recording Trial Activations:**
  - Stores each trial activation along with the associated email and device fingerprint.
  - Records information such as activation date, expiration date, and status (active or expired).
  - Ensures that no two trials can be activated using the same email and device fingerprint combination.

- **Managing User Data:**
  - Stores salesperson information, including email, profile details, and verification status.
  - Allows the system to verify whether a salesperson is eligible for a trial activation.

- **Ensuring Security and Data Integrity:**
  - Encrypts sensitive data such as emails and fingerprints to protect salesperson information.
  - Uses constraints and foreign key relationships to maintain data integrity and consistency.

### 7. Security and Compliance
Security and compliance are integral to the sales feature, ensuring that salesperson data is protected and that the system adheres to regulatory standards.

- **Authentication and Authorization:**
  - All actions are authenticated using JWT tokens to ensure secure communication between the frontend and backend.
  - Admin actions, such as key code management, are restricted based on roles and permissions, preventing unauthorized access.

- **Data Encryption:**
  - Data such as email addresses, fingerprints, and key codes are encrypted both in transit and at rest.
  - HTTPS is enforced for all API communication to protect data integrity.

- **Rate Limiting:**
  - Rate limiting on API endpoints helps prevent abuse of the trial activation feature and brute-force attacks.
  - Monitors for suspicious activities and flags potential abuse cases for review.

- **Privacy Compliance:**
  - Salespeople are informed about the use of device fingerprinting and email verification in the privacy policy.
  - Salespeople have the right to request data deletion or opt-out as required by regulations like GDPR.

## System Flow

1. **Admin Key Code Management:**
   - Admins create key codes using the admin interface and store them in the database.
   - Admins assign these key codes to salespeople, who will use them to activate their own trial periods.

2. **Salesperson Registration and Activation:**
   - Salespeople register on the Angular frontend and verify their email through the verification link sent by the Email Service.
   - A unique device fingerprint is generated and sent along with the email to the backend for trial activation.

3. **Trial Activation Process:**
   - Salespeople enter the key code they have received from the admin.
   - The Trial Management Microservice validates the key code, email, and device fingerprint combination.
   - If valid, the trial is activated for the salesperson, and the details are stored in the database. A confirmation email is sent to the salesperson.
   - If invalid, the activation is denied, and an error message is displayed.

4. **Monitoring and Notifications:**
   - The system monitors the trial period and updates the trial status accordingly.
   - Reminder emails are sent to salespeople as their trial period nears its end, encouraging them to subscribe.
   - User access is adjusted based on the trial status.
  
   ![diagram-export-21-09-2024-17_54_38](https://github.com/user-attachments/assets/aec68af5-233b-47d2-bfe9-dd3b4e284fd5)
