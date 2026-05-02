# GridFix (CSCI 440 - Group 2)

A civic engagement and infrastructure issue-reporting web application built for College Station, TX. Citizens and city employees can submit, track, and resolve infrastructure reports (potholes, damaged roads, etc.) through an interactive map interface with a full role-based workflow.

---

## Features

### Interactive Map & Report Submission
- Leaflet.js map centered on College Station with OpenStreetMap tiles
- Click anywhere on the map to place a report at that location
- Reverse geocoding via Nominatim automatically fills in the address
- "Find Me" button filters visible reports to a 0.5-mile radius around the user
- Marker clustering keeps the map clean when many reports exist
- Reports reload dynamically after any action — no page refresh needed

### Report Lifecycle
Reports follow a structured workflow managed by role:

| Status | Description |
|---|---|
| `pending` | Newly submitted, awaiting an employee |
| `accepted` | Claimed by an employee |
| `pending_completion` | Employee marked done, awaiting supervisor sign-off |
| `completed` | Supervisor confirmed resolution |
| `rejected` | Rejected by a supervisor |

**Employee actions:** Accept pending reports, mark accepted reports as complete, reset to pending.  
**Supervisor/Admin actions:** Assign reports to employees, confirm or reject completions, change status manually, archive reports.

### Photo Uploads
- Attach multiple photos per report (JPG, JPEG, PNG, GIF — 5 MB max each)
- Photos stored in `/uploads/reports/{reportId}/` with unique filenames
- Viewable in a modal with thumbnail navigation

### Messaging System
- Private one-to-one conversations between any two users
- Conversation list shows last message preview and timestamp
- New conversations started from a searchable user list
- Unread message tracking with background polling every 10 seconds
- Floating widget accessible from any page

### User Profiles & Skills
- View any user's profile including their role, assigned reports, and listed skills
- Users can select from ~50 predefined skills (plumbing, electrical, carpentry, etc.)
- Password change form with current-password verification

### Authentication & Security
- **Registration** with email verification (24-hour token, sent via Mailgun)
- **Google reCAPTCHA v3** on registration (score threshold: 0.5)
- **Argon2ID** password hashing (64 MB memory, 4 iterations)
- **Password reset** via email link (1-hour token expiry)
- Secure session cookies: HTTPOnly, SameSite=Strict, HTTPS flag
- Prepared statements throughout — no raw SQL interpolation
- Server-side `htmlspecialchars()` and client-side `escapeHtml()` for XSS prevention

### Archiving
- Supervisors/Admins can archive completed or rejected reports
- Archived data preserved in `archived_reports` with a full JSON backup of the original report and photos
- Atomic transaction: archive insert and active-record delete happen together or not at all

---

## Role System

| Role | Capabilities |
|---|---|
| `user` | Submit reports, view map, use messaging |
| `employee` | All above + accept/complete reports |
| `supervisor` | All above + assign, confirm, reject, archive reports |
| `admin` | Full access including manual status changes |

---

## Tech Stack

**Backend**
- PHP (procedural) with MySQLi
- Argon2ID password hashing (PHP 7.2+)
- Mailgun API via Composer for transactional email

**Frontend**
- Bootstrap 5.3.0
- Leaflet.js 1.9.4 + leaflet.markercluster 1.4.1
- Bootstrap Icons 1.10.0
- Vanilla JS (ES6) with Fetch API

**Database**
- MySQL 5.7+

**Composer Dependencies**
- `mailgun/mailgun-php` ^4.3
- `kriswallsmith/buzz` ^1.3
- `nyholm/psr7` ^1.8

---

## Database Schema

### Core Tables
- **`users`** — accounts with role, verification token, reset token, Argon2ID hash
- **`reports`** — title, description, address, lat/lng, status, assignment/completion tracking
- **`report_photos`** — file paths linked to reports, supports active and archived types
- **`archived_reports`** — full JSON backup of deleted reports for audit purposes

### Messaging Tables
- **`conversations`** — message threads
- **`conversation_participants`** — junction table linking users to conversations
- **`messages`** — content, sender, read status, timestamp

### Auxiliary Tables
- **`skills`** / **`user_skills`** — predefined skill list and user assignments
- **`permissions`** / **`user_permissions`** — permission framework (for future use)

---

## API Endpoints

All endpoints return JSON.

| Endpoint | Method | Purpose |
|---|---|---|
| `userAuth.php` | POST | Register, login, request/submit password reset |
| `verify.php` | GET | Email verification via token |
| `Save_Reports.php` | POST | Submit a new report with optional photos |
| `Get_Reports` | GET | Fetch reports for map (supports geolocation filter) |
| `ReportActions.php` | POST | Accept, complete, assign, confirm, archive reports |
| `View_Reports.php` | GET | Paginated report list with search and tab filtering |
| `messaging.php` | POST | Get conversations, get/send messages, start conversation |
| `get_users` | GET | List all users (for new message recipient search) |

---

## Setup & Deployment

**Requirements:** PHP 7.2+, MySQL 5.7+, Composer, HTTPS in production

1. Clone the repository and run `composer install`
2. Import the schema from `SQL Create Statements.txt` into your MySQL database
3. Configure database credentials, Mailgun API key/domain, and reCAPTCHA v3 keys in the relevant PHP files
4. Ensure `/uploads/reports/` is writable by the web server
5. Set up HTTPS — session cookies require the secure flag in production

---

## Project Structure

```
├── index.html              # Landing / login redirect page
├── login.html              # Login form
├── registration.html       # Registration form with reCAPTCHA
├── index.php               # Main map page (authenticated)
├── profile.php             # User profile view
├── logout.php              # Session destroy + redirect
├── session.php             # Secure session configuration
├── userAuth.php            # Auth handler (login, register, reset)
├── authFunctions           # Role-check helpers (isEmployee, isSupervisor, etc.)
├── verify.php              # Email verification handler
├── reset_password.php      # Password reset form + handler
├── mailer.php              # Mailgun email sending wrapper
├── Save_Reports.php        # Report creation + photo upload
├── Get_Reports             # Report fetch for map (JSON)
├── ReportActions.php       # Report status actions
├── View_Reports.php        # Paginated report list
├── messaging.php           # Messaging API
├── messagingWidget.js      # Floating messaging UI widget
├── messaging-widget.css    # Messaging widget styles
├── get_users               # User list for messaging
├── api.js                  # Frontend API helpers
├── scripts.js              # General frontend utilities
├── composer                # Composer autoload configuration
├── composer.lock           # Locked dependency versions
└── SQL Create Statements.txt  # Full database schema
```
