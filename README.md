
# Overview of the Sales Feature with Trial Key Codes

## System Components

### 1. Angular Frontend
- **User Interface for Trial Activation:**
  - Users can enter their email and a key code in the trial activation form.
  - We generate a unique device fingerprint using the FingerprintJS library to identify the user’s device.
  - This data, including the email and key code, is sent to our backend to validate and activate the trial.

- **Admin Interface for Key Code Management:**
  - Admins can log into an admin panel to create and manage key codes.
  - They can specify details like how long the trial should last and any additional information. This is then sent to the backend for processing.
  - Admins can see the status of all key codes (active, used, or expired) and manage them as needed.

- **Connecting to Backend Services:**
  - The frontend communicates securely with the backend through an API Gateway.
  - User actions like registration and trial activation are authenticated using JWT tokens to ensure secure communication.

### 2. Trial Management Microservice
- **Generating and Storing Key Codes:**
  - When an admin creates a new key code, the Trial Management Service generates a unique code and stores it in the database with all necessary details, like its current status (active).

- **Handling Trial Activation Requests:**
  - When a user submits a trial activation request, the service checks:
    - If the key code is valid, active, and hasn’t expired.
    - If the email has already been used for a trial. If so, the request is denied.
    - If the device fingerprint has been used for a trial before. If it has, the request is also denied.

- **Storing Activation Information:**
  - If all checks are passed, the service saves the email, device fingerprint, and key code details in the database, along with the trial start and end dates.
  - The key code status is updated to “used” to prevent it from being reused.

- **Preventing Multiple Trials:**
  - By storing the email and device fingerprint together, we ensure that a user can’t activate multiple trials, even if they use different email addresses.

- **Sending Notifications:**
  - Once a trial is activated, the service triggers an email to be sent to the user, confirming the trial period details.

### 3. User Service
- **Managing User Registration and Authentication:**
  - This service handles user registration, collecting email addresses and basic profile info.
  - It sends a verification email to users before they can activate a trial.

- **Managing User Data:**
  - Stores user information in a database and provides APIs to access this data.
  - Shares user information, like email and verification status, with the Trial Management Service during trial activation.

- **Supporting Authentication:**
  - Issues JWT tokens for secure communication between the frontend and backend.
  - Manages user sessions and supports login and logout actions.

### 4. Email Service
- **Verifying Emails:**
  - Sends a verification email to new users. This email contains a link that users need to click to verify their email address.
  - Informs the User Service once an email is successfully verified, allowing the user to proceed with the trial.

- **Confirming Trial Activations:**
  - Sends a confirmation email once a trial is activated, including details like the start and end dates.

- **Sending Expiration Reminders:**
  - Notifies users when their trial is about to end, encouraging them to subscribe.

- **API Integration:**
  - Offers API endpoints for other services to trigger emails based on events like trial activation or user verification.

### 5. API Gateway
- **Routing Requests:**
  - Receives all client requests and directs them to the appropriate microservice, such as the Trial Management Service or User Service.
  - Verifies that each request is authenticated and authorized before forwarding it.

- **Managing Authentication and Authorization:**
  - Checks JWT tokens to make sure users are authenticated.
  - Enforces role-based access control (RBAC) to limit certain actions, like key code management, to authorized users only.

- **Rate Limiting and Throttling:**
  - Limits the number of requests to prevent abuse, like repeated trial activation attempts or brute-force attacks.
  - Ensures that no single IP can attempt too many trial activations in a short time.

- **Load Balancing:**
  - Distributes requests across multiple service instances to maintain high availability and scalability.

### 6. Database
- **Storing Key Codes:**
  - Saves each key code with its details like trial duration, status, and metadata.
  - Tracks the status of each key code (active, used, or expired) to prevent misuse.

- **Recording Trial Activations:**
  - Stores each trial activation with the associated email and device fingerprint.
  - Records details like the activation date, expiration date, and status (active or expired).
  - Ensures that no two trials can be activated using the same email and device fingerprint combination.

- **Managing User Data:**
  - Stores user information such as email, profile details, and verification status.
  - Allows the system to verify if a user is eligible for a trial activation.

- **Ensuring Security and Data Integrity:**
  - Encrypts sensitive information like emails and fingerprints to protect user data.
  - Uses constraints and relationships to maintain data consistency.

### 7. Security and Compliance
- **Authentication and Authorization:**
  - All actions are authenticated using JWT tokens to secure communications between the frontend and backend.
  - Admin activities are restricted based on roles to prevent unauthorized access to key code management.

- **Data Encryption:**
  - Encrypts data such as emails, fingerprints, and key codes both in transit and at rest.
  - Enforces HTTPS for all API communications to protect data integrity.

- **Rate Limiting:**
  - Prevents abuse of the trial activation feature through rate limiting.
  - Monitors suspicious activities and flags potential abuse cases for review.

- **Privacy Compliance:**
  - Informs users about the use of device fingerprinting and email verification in the privacy policy.
  - Allows users to request data deletion or opt-out as required by regulations like GDPR.

### 8. System Interactions and Communication
- **Frontend to API Gateway:**
  - The Angular frontend sends all user requests to the API Gateway, such as trial activations and registrations.
  - The API Gateway authenticates and forwards these requests to the appropriate microservice.

- **Trial Management Service to Database:**
  - Manages key code and trial activation data.
  - Validates key codes, emails, and fingerprints against the records stored in the database.

- **User Service to Trial Management Service:**
  - Shares user information like email verification status with the Trial Management Service to facilitate trial activation.
  - Updates user profiles with the trial activation status.

- **Email Service Integration:**
  - Sends emails based on events triggered by the Trial Management Service, such as trial activation or reminders.

## System Flow

1. **Admin Key Code Management:**
   - Admins create key codes and assign them to potential users.
   - These key codes are stored in the database and managed by the Trial Management Microservice.

2. **User Registration and Fingerprinting:**
   - Users register through the Angular frontend and verify their email.
   - A unique device fingerprint is generated and sent to the backend with the email.

3. **Trial Activation:**
   - Users enter the key code, email, and fingerprint.
   - The Trial Management Service checks this data and activates the trial if all conditions are met.

4. **Email Notifications:**
   - The Email Service sends confirmation and reminder emails to the user.

5. **Monitoring and Analytics:**
   - Admins can monitor trial activations, key code usage, and system performance through the admin panel.
   - Any anomalies or abuse are flagged and addressed.
