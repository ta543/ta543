## Sales Feature with Trial Key Codes - Overview

### Objective
The sales feature is designed to allow sales people within the organization to experience the product through a trial period activated using unique key codes. These codes are manually assigned by admins specifically to sales people, who will use them to activate their own trial periods. The system utilizes a combination of email addresses and device fingerprints to prevent multiple trial activations from the same device or user. The registration and activation process is limited to desktop computers to ensure controlled and secure access.

## System Components and Their Roles

### 1. Angular Frontend
The Angular frontend serves as the primary interface for both admins and sales people, providing secure and user-friendly interactions for trial activation and key code management.

#### Salesperson Interface for Trial Activation:
- **Trial Activation Form:** Sales people enter their email and the key code they received from the admin. The frontend generates a unique device fingerprint using an enhanced FingerprintJS library integration, which collects additional data points such as browser details, OS information, and installed fonts. This robust identifier is hashed for security.
- **Data Submission:** The email, key code, and hashed device fingerprint are sent to the backend for validation and trial activation. A short-lived JWT token is used for authentication, and all communication is secured via HTTPS.
- **Enhanced Device Fingerprinting:** The use of additional data points in the fingerprint generation reduces false positives and negatives, ensuring reliable device identification and preventing multiple trial activations from the same device.

#### Admin Interface for Key Code Management:
- **Key Code Generation and Assignment:** Admins generate key codes with specific parameters like trial duration and metadata. These key codes are assigned to sales people who will use them for their own trials.
- **Key Code Tracking and Expiry Management:** The system tracks all key code assignments, and unused key codes have an expiry date. Admins are notified before a key code expires, ensuring efficient management and reducing the risk of unused key codes.
- **Security and Logging:** All actions, including key code generation, assignment, and management, are logged to ensure accountability and prevent misuse.

#### Connecting to Backend Services:
- The frontend communicates with backend services through an API Gateway. All requests, such as registration and trial activation, are authenticated using JWT tokens. The tokens are short-lived to enhance security, and refresh tokens are used to maintain sessions.

### 2. API Gateway
The API Gateway serves as the central entry point for all client requests, ensuring secure and efficient communication between the frontend and backend services.

#### Routing and Security:
- **Request Routing:** Routes requests to the appropriate microservice based on the URL and method.
- **Authentication and Authorization:** Verifies JWT tokens for authentication and implements role-based access control (RBAC) to restrict certain actions, such as key code management, to authorized users only.
- **Rate Limiting and Load Balancing:** Implements rate limiting to prevent abuse of trial activation attempts and distributes incoming requests across multiple microservices for high availability and scalability.

### 3. Trial Management Microservice
This microservice handles all logic related to key codes, trial activations, and user validation.

#### Key Code Management:
- **Generating and Assigning Key Codes:** Admins generate unique key codes and assign them to sales people. These key codes are logged but not stored in the database until used for trial activation.
- **Expiry and Validation:** Unused key codes have an expiry date. If a key code is about to expire, the admin is notified, allowing for proactive management.

#### Trial Activation Process:
- **Validation Checks:**
  - **Key Code Validation:** Confirms that the key code is valid, active, and not expired.
  - **Email Validation:** Ensures that the email has not been used for a previous trial. If the email is flagged as already used, the activation request is denied.
  - **Device Fingerprint Validation:** Verifies that the hashed device fingerprint has not been used for a previous trial activation. This is done using an enhanced fingerprinting method that integrates multiple data points.

- **Rate Limiting and Abuse Prevention:**
  - Limits the number of activation attempts per email and device fingerprint combination to 5 per hour. Exceeding this limit triggers a cooldown period and sends an alert to the admin.
  - Repeated invalid activation attempts result in a temporary block of the email/device combination and an alert to the admin, protecting against brute-force attacks.

#### Storing Activation Information:
- **Post-Activation Storage:** If the key code, email, and device fingerprint are all valid and unique, the trial is activated. The activation information, including the key code used, is stored in the database.
- **Atomic Operations:** A transactional locking mechanism is used during the activation process to ensure that key codes are marked as used only once, preventing concurrent usage and maintaining data integrity.

#### Sending Notifications:
- **Confirmation Emails:** The Email Service sends a confirmation email to the sales person upon successful trial activation, providing details about the trial period.
- **Expiration Reminders:** Reminder emails are sent as the trial period nears its end, encouraging the sales person to subscribe.

### 4. User Service
The User Service handles user account management, authentication, and verification.

#### Salesperson Registration and Authentication:
- **Registration Handling:** Collects the sales person's email address and basic profile information during registration. The system limits the number of verification emails sent to an email address within a 15-minute window to prevent spamming and abuse.
- **Email Verification:** Sends a verification email to sales people. The sales person must click the link in the email to verify their address before they can activate a trial.

#### User Data Management:
- **Data Storage and Sharing:** Stores sales person data, including email and verification status. This information is shared with the Trial Management Microservice during trial activation requests.
- **Authentication Support:** Issues short-lived JWT tokens (valid for 15 minutes) and manages user sessions. Refresh tokens are used to maintain sessions, and all communications are secured using HTTPS.

### 5. Email Service
The Email Service manages all email communications for registration, trial activations, and reminders.

#### Email Verification:
- Sends a verification email containing a unique link that the sales person must click to verify their email address. Verification attempts are limited to prevent abuse.

#### Trial Activation and Reminder Emails:
- **Confirmation Emails:** Sends a confirmation email after a successful trial activation, detailing the start and end dates of the trial period.
- **Expiration Reminders:** Sends reminder emails as the trial period approaches its end, encouraging the sales person to subscribe to the service.

#### API Integration:
- Provides API endpoints for sending emails based on specific triggers such as trial activation, verification, and expiration reminders.

### 6. Database
The database is the central repository for all data related to trial activations and sales person information.

#### Data Storage and Management:
- **Trial Activation Records:** Stores each trial activation, including the associated email, hashed device fingerprint, and key code used. Ensures that no two trials can be activated using the same email and device fingerprint combination.
- **User Data:** Stores sales person information, including email, profile details, and verification status. This data is encrypted in transit and at rest to ensure security.

#### Optimization and Integrity:
- **Data Archival:** Archives old data to improve query performance and reduce storage costs.
- **Transactional Integrity:** Ensures that key code usage is atomic and prevents concurrent usage conflicts.

### 7. Security and Compliance
Security and compliance are integral to the sales feature, ensuring that sales person data is protected and that the system adheres to regulatory standards.

#### Authentication and Authorization:
- All actions are authenticated using JWT tokens to ensure secure communication between the frontend and backend.
- Admin actions, such as key code management, are restricted based on roles and permissions. Regular audits are conducted to ensure compliance and security.

#### Data Encryption:
- All sensitive data, such as email addresses, fingerprints, and key codes, is encrypted both in transit and at rest. HTTPS is enforced for all API communication to protect data integrity.

#### Privacy Compliance:
- Sales people are informed about the use of device fingerprinting and email verification in the privacy policy.
- The system provides options for data deletion and opt-out as required by GDPR. Data retention policies ensure that data is stored only for the necessary duration and securely deleted thereafter.

## System Flow

1. **Admin Key Code Management:**
   - Admins create key codes using the admin interface and assign them to sales people. Key codes are logged and tracked for accountability, and expiry dates are set.
   - Key codes are only stored in the database after being used for activation by a sales person.

2. **Salesperson Registration and Activation:**
   - Sales people register on the Angular frontend and verify their email through the verification link sent by the Email Service. Verification attempts are limited to prevent abuse.
   - A unique, hashed device fingerprint is generated and sent along with the email to the backend for trial activation.

3. **Trial Activation Process:**
   - Sales people enter the key code they received from the admin. The Trial Management Microservice validates the key code, email, and hashed device fingerprint combination.
   - If valid, the trial is activated, and the details are stored in the database. A confirmation email is sent to the sales person. If invalid, the activation is denied, and repeated invalid attempts trigger a temporary block and an admin alert.

4. **Monitoring and Notifications:**
   - The system monitors the trial period and updates the trial status. Reminder emails are sent as the trial period nears its end, encouraging the sales person to subscribe.
   - User access is adjusted based on the trial status.

5. **Security and Compliance:**
   - Regular audits, data encryption, and role-based access controls ensure the security and integrity of the system.
   - GDPR compliance is maintained with data deletion and opt-out options.


  ![diagram-export-22-09-2024-08_12_03](https://github.com/user-attachments/assets/23670e1e-709b-4d9b-9afe-8b3c0ff28b24)


