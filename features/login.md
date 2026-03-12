Authentication & Login Module Specification

0. Ticket numbers:

MOBILE_TICKET_NUMBER: DEV16
WEB_TICKET_NUMBER:DEV2
BACKEND_TICKET_NUMBER:DEV13

1. Feature Overview

The Login Module provides secure authentication and authorization for the Skoolz digital school platform.

Skoolz consists of four main components:
	•	Database
	•	Backend API
	•	Web Application
	•	Mobile Application

The authentication module must support:
	•	Multi-tenant access
	•	Password authentication
	•	OTP authentication (Email + SMS)
	•	Biometric authentication (Mobile)
	•	Scoped tokens for API authorization
	•	Multiple user roles (student, staff, guardian)

The module must ensure that authentication is centralized while authorization remains tenant-aware.

⸻

2. Multi-Tenant Architecture

Skoolz is a multi-tenant platform, where each school organization represents a tenant.

Each request to the system must be associated with a tenant context.

Tenant Identification

A tenant can be identified using one of the following:
	•	organizationId
	•	organizationNumber
	•	custom domain (e.g. school.skoolz.app)
	•	request header X-Tenant-ID

Example:

X-Tenant-ID: org_98231


⸻

Tenant Data Model

Organization Table

Field	Type	Description
id	UUID	Unique organization identifier
organizationNumber	String	School registration number
name	String	Organization name
status	Enum	active / suspended
createdAt	Timestamp	Creation date


⸻

Organization User Mapping

This table connects users to organizations.

Field	Type
id	UUID
organizationId	UUID
userId	UUID
role	Enum
createdAt	Timestamp

Roles may include:

STUDENT
STAFF
GUARDIAN
ADMIN


⸻

3. User Model

Users are global identities in the platform and may belong to multiple tenants.

User Table

Field	Type	Description
id	UUID	Unique user ID
username	String	Unique login name
displayName	String	User full name
email	String	Email address
phoneNumber	String (optional)	Phone number
passwordHash	String	Hashed password
status	Enum	active / suspended
createdAt	Timestamp


⸻

4. Profile Models

Each user role has its own profile table.

Student Profile

Field	Type
id	UUID
userId	UUID
studentNumber	String
grade	String


⸻

Staff Profile

Field	Type
id	UUID
userId	UUID
employeeNumber	String
department	String


⸻

Guardian Profile

Field	Type
id	UUID
userId	UUID
relationship	String


⸻
5. Account and password Creation
- Allow for password with at least 6 characters with a minimum of a cap and a special characters
- User are not verified until an email is sent to them for verification
- The verification process sends an email with an OTP that needs to be verified
- Phone number also needs to be verified, before allowing login via OTP on SMS.

- Track the state of user (verified and not verified in the database)
- User which are not verified will be presented with a pop up on successful authentication, but they won't be logged in.
- Account for a forgot password or password reset flow

6. Phone number verification

On the first login, a settings screen need to show details and indicate that the phone number is not verified(if it was provided)
There should be a button next to it that triggers teh verification flow which will sent an SMS via the server and the OTP needs to be entered on the view and if it's correct the server will notified via API of the details and shall mark the user phone number as verified.

5. Authentication Methods

The login module supports multiple authentication mechanisms.

5.1 Password Authentication

Users authenticate with:

username/email + password

Flow:
	1.	User submits credentials
	2.	Backend verifies password hash
	3.	Tenant membership validated
	4.	OTP verification
	5.	Scoped token issued

⸻

5.2 OTP Authentication

OTP can be delivered via:
	•	Email
	•	SMS (only when the phone number is provided verified, if not this option should be hidden)

OTP Scenarios
	•	login verification
	•	password reset
	•	suspicious login attempt
	•	step-up authentication

OTP Table

Field	Type
id	UUID
userId	UUID
code	String
deliveryMethod	Enum (EMAIL, SMS)
expiresAt	Timestamp
used	Boolean


⸻

5.3 Mobile Biometric Authentication

The mobile application supports biometric authentication using:
	•	Face ID
	•	Fingerprint
	•	Device authentication

Flow:
	1.	User logs in once with password
	2.	Refresh token stored securely
	3.	Mobile enables biometric unlock
	4.	Biometric unlock retrieves stored token

Biometric verification is device-side, not server-side.

⸻

6. Token System

The authentication system uses JWT-based scoped tokens.

Two token types are issued.

⸻

Access Token

Short-lived token used for API calls.

Example lifetime:

15 minutes

Payload:

{
  "sub": "user_id",
  "tenant": "organization_id",
  "roles": ["student"],
  "scopes": ["attendance.read", "grades.read"],
  "exp": 1712345678
}


⸻

Refresh Token

Used to obtain a new access token.

Lifetime example:

30 days

Stored securely in database.

⸻

Token Scopes

Scopes control API access.

Examples:

attendance.read
attendance.write
grades.read
grades.write
student.profile
staff.profile

Scopes are derived from:
	•	role
	•	organization permissions
	•	feature flags

⸻

7. Login Flow

Standard Login Flow

User → API
      → Validate credentials
      → Verify tenant membership
      → Optional OTP challenge
      → Generate access token
      → Generate refresh token
      → Return tokens

Response example:

{
  "accessToken": "...",
  "refreshToken": "...",
  "expiresIn": 900,
  "tenantId": "org_123"
}


⸻

8. Authentication Endpoints

Login

POST /auth/login

Request

{
  "username": "john",
  "password": "password",
  "organizationId": "org_123"
}


⸻

Request OTP

POST /auth/otp/request


⸻

Verify OTP

POST /auth/otp/verify


⸻

Refresh Token

POST /auth/token/refresh


⸻

Logout

POST /auth/logout

Invalidates refresh token.

⸻

9. Security Requirements

The module must enforce the following:

Password Security
	•	bcrypt or argon2 hashing
	•	minimum password complexity
	•	brute-force protection

⸻

Rate Limiting

Apply rate limits to:
	•	login attempts
	•	OTP requests
	•	token refresh

Example:

5 login attempts / minute


⸻

Device & Session Tracking

Session table:

Field	Type
id	UUID
userId	UUID
deviceId	String
ipAddress	String
createdAt	Timestamp
expiresAt	Timestamp


⸻

Audit Logs

Every authentication event must be logged.

Examples:
	•	login success
	•	login failure
	•	OTP request
	•	OTP verification
	•	token refresh

⸻

10. Database Replication Considerations

Because Skoolz supports database replication, the login module must:
	•	ensure tenant isolation
	•	support read replicas
	•	write authentication events to primary node

Authentication data such as:

users
tokens
sessions
otp

must remain consistent across replicas.

Strategies:
	•	primary-write / replica-read
	•	token validation independent of replica lag (JWT)

⸻

11. Web vs Mobile Authentication

Web

Uses:

HTTP-only secure cookies

or

Bearer tokens


⸻

Mobile

Uses:

Secure token storage
Keychain / Keystore
Biometric unlock


⸻

12. Error Handling

Example responses.

Invalid credentials:

{
  "error": "INVALID_CREDENTIALS"
}

OTP required:

{
  "error": "OTP_REQUIRED"
}

Tenant not found:

{
  "error": "TENANT_NOT_FOUND"
}
