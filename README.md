
# Sales Feature Implementation Overview with Trial Period Key Codes

## Objective
Implement a sales feature as a microservice that allows potential customers to activate a trial period using manually assigned key codes. The system uses a combination of email addresses and device fingerprints to prevent multiple trial activations from the same device or user, with the registration process limited to desktop computers.

## System Components

### 1. Angular Frontend
- **User Interface for Trial Activation:**
  - Users can enter their email and key code in the trial activation form.
  - Device fingerprint is generated using the FingerprintJS library to uniquely identify the device.
  - A POST request is made to the Trial Management Service with the key code, email, and device fingerprint to validate and activate the trial.

- **Admin Interface for Key Code Management:**
  - Admins log into the admin panel to generate and manage key codes.
  - Admins can generate new key codes with parameters like trial duration and metadata, which are then sent to the Trial Management Service.
  - Admins can view the status of all key codes, including active, used, or expired, and can deactivate or delete them if necessary.

- **Communication with Backend Services:**
  - The Angular frontend communicates with the backend via the API Gateway.
  - All user actions (e.g., registration, trial activation) are authenticated using JWT tokens to ensure secure communication.

### 2. Trial Management Microservice
- **Key Code Generation and Storage:**
  - When an admin generates a new key code through the Angular admin interface, the Trial Management Service creates a unique key code and stores it in the database with associated metadata and status (active).

- **Trial Activation Process:**
  - Upon receiving a trial activation request, the service performs multiple validations:
    - **Key Code Validation:** Checks if the key code exists, is active, and has not expired.
    - **Email Validation:** Checks if the email has been used for a trial before. If it has, the activation is denied.
    - **Device Fingerprint Validation:** Verifies if the fingerprint has been used for a trial before. If it has, the activation is denied.

- **Storing Activation Details:**
  - If all validations pass, the service stores the email, device fingerprint, and key code in the database with the trial start and expiration date.
  - The key code status is updated to “used” to prevent it from being reused.

- **Preventing Multiple Trials:**
  - By storing the combination of email and device fingerprint, the service ensures that the same device and email cannot activate multiple trials, even if the user registers with different email addresses.

- **Sending Notifications:**
  - Once the trial is activated, the service triggers the Email Service to send a confirmation email to the user with details about the trial period.

### 3. User Service
- **User Registration and Authentication:**
  - Handles user registration by collecting email addresses and basic profile information.
  - Sends a verification email to the user for confirming their email address before they can activate a trial.

- **User Data Management:**
  - Stores user data in the User Database and provides API endpoints to access user information.
  - Shares user information, such as email and verification status, with the Trial Management Service during trial activation requests.

- **Authentication Support:**
  - Issues JWT tokens for secure communication between the Angular frontend and backend services.
  - Manages user sessions and supports login/logout functionality.

### 4. Email Service
- **Email Verification:**
  - Sends a verification email to the user upon registration. The email contains a unique verification link that the user must click to verify their email address.
  - The User Service is notified upon successful verification, allowing the user to proceed with trial activation.

- **Trial Activation Confirmation:**
  - Sends a confirmation email to the user once the trial has been successfully activated, detailing the start and end dates of the trial period.

- **Trial Expiration Reminders:**
  - Sends reminder emails to the user before the trial period expires, encouraging them to subscribe to the service.

- **API Integration:**
  - Provides API endpoints for other services to trigger email sending based on specific events (e.g., trial activation, user verification).

### 5. API Gateway
- **Routing Requests:**
  - Receives all client requests and forwards them to the relevant microservice (e.g., Trial Management Service, User Service) based on the request URL and method.
  - Ensures that the request is authenticated and authorized before forwarding it to the microservice.

- **Authentication and Authorization:**
  - Verifies JWT tokens provided by the client to ensure the user is authenticated.
  - Enforces role-based access control (RBAC) to restrict certain actions (e.g., key code management) to authorized users only.

- **Rate Limiting and Throttling:**
  - Implements rate limiting on endpoints to prevent abuse, such as repeated trial activation attempts or brute-force attacks.
  - Limits the number of trial activation attempts from a single IP address within a given timeframe.

- **Load Balancing:**
  - Distributes incoming requests across multiple instances of microservices to ensure high availability and scalability.

### 6. Database
- **Key Code Storage:**
  - Stores each key code with details like trial duration, status, associated metadata, and the admin who created it.
  - Tracks the status of each key code (e.g., active, used, expired) to prevent reuse.

- **Trial Activations:**
  - Stores the combination of email and device fingerprint for each trial activation.
  - Includes details such as activation date, expiration date, and status (active, expired).
  - Ensures the uniqueness of the email and fingerprint combination to prevent multiple trials from the same user/device.

- **User Data:**
  - Stores user information, including email, profile details, and verification status.
  - Allows querying user data to verify eligibility for trial activation.

- **Security and Integrity:**
  - Sensitive data, such as email addresses and fingerprints, are encrypted to ensure security.
  - Implements constraints and foreign key relationships to maintain data integrity and consistency.

### 7. Security and Compliance
- **Authentication and Authorization:**
  - All user actions are authenticated using JWT tokens, ensuring secure communication between the frontend and backend.
  - Admin actions are restricted based on roles and permissions, preventing unauthorized access to key code management.

- **Data Encryption:**
  - Data such as email addresses, fingerprints, and key codes are encrypted at rest and in transit.
  - HTTPS is enforced for all API communication to protect data integrity.

- **Rate Limiting:**
  - The API Gateway enforces rate limiting to prevent abuse of the trial activation feature and brute-force attacks.
  - Monitors suspicious activities and flags potential abuse cases for review.

- **Privacy Compliance:**
  - Users are informed about the use of device fingerprinting and email verification in the privacy policy.
  - Users have the right to request data deletion or opt-out as required by regulations like GDPR.

### 8. System Interactions and Communication
- **Frontend to API Gateway:**
  - The Angular frontend sends all user requests (e.g., trial activation, registration) to the API Gateway.
  - The API Gateway authenticates and forwards the request to the appropriate microservice.

- **Trial Management Service to Database:**
  - Stores and retrieves key code and trial activation data.
  - Validates key codes, emails, and fingerprints against stored records.

- **User Service to Trial Management Service:**
  - Shares user data (e.g., email verification status) with the Trial Management Service for trial activation.
  - Receives trial activation status updates for user profiles.

- **Email Service Integration:**
  - Receives events from the Trial Management Service to send emails based on specific actions (e.g., trial activation, reminders).

## System Flow

1. **Admin Key Code Management:**
   - Admin generates key codes and assigns them to potential users.
   - Key codes are stored in the database and managed via the Trial Management Microservice.

2. **User Registration and Fingerprinting:**
   - Users register using the Angular frontend and verify their email.
   - A unique device fingerprint is generated and sent to the backend along with the email.

3. **Trial Activation:**
   - Users enter the key code and submit it with their email and fingerprint.
   - The Trial Management Service validates the data and activates the trial if the conditions are met.

4. **Email Notifications:**
   - Confirmation and reminder emails are sent to the user via the Email Service.

5. **Monitoring and Analytics:**
   - Admins monitor trial activations, key code usage, and system performance through the admin panel.
   - Anomalies and abuse are flagged and addressed.

## Future Enhancements
- **Automated Key Code Distribution:**
  - Integrate with marketing tools to automate the distribution of key codes based on user behavior or campaign triggers.

- **Advanced Fraud Detection:**
  - Use machine learning algorithms to detect patterns of abuse based on key code usage, device fingerprints, and user behavior.

- **Multi-Platform Support:**
  - Extend the trial activation feature to mobile apps, ensuring a consistent experience across web and mobile platforms.

- **Referral Program Integration:**
  - Implement a referral program where existing users can share key codes with potential customers, incentivizing conversions.

