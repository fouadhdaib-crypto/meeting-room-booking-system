# Gather — meeting room workspace

A compact, responsive PHP/MySQL application with an integrated administration dashboard. The refreshed design uses a dark green sidebar, compact metrics, dense tables, room summaries, clear status labels, and mobile navigation. No frontend build process or external fonts/CDNs are required.

## Requirements and setup

- PHP 8.0+ with PDO MySQL; MySQL 8+ or MariaDB 10.4+; InnoDB tables.
- Create a database named `meeting_room`, then import `database/meeting_room.sql` **only for a fresh installation**. This is the supplied sample dataset, including its existing accounts. Do not reimport it over an existing database.
- Set `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD`, or edit `config/database.php`. Set `APP_TIMEZONE` if needed; the default is `Asia/Amman`.
- Existing installations use the same tables and columns; no new schema is required. Ensure `room` and `booking` use InnoDB so transactions, locks and foreign keys work. The included fresh-install SQL specifies it explicitly.
- Configure your web server document root to `public/`, keeping `app/`, `config/`, `database/`, `bin/`, and `tests/` outside the public web root.
- For local development, from this directory run:

```sh
php -S 127.0.0.1:8080 -t public
```

Open `http://127.0.0.1:8080/register.php`, create an account, and promote it from the terminal:

```sh
php bin/promote-admin.php your-email@example.com
```

Sign out and sign in again. Administrators land on `admin.php`; employees land on `dashboard.php`. The sample `ahmad` administrator has a placeholder password hash, so use the account creation/promotion flow rather than expecting a demo password.

## Admin features

Every section has list/read, details, create, edit and confirmed delete operations, backed by the database:

| Section | Managed fields |
| --- | --- |
| Bookings | Title, date, times, room, organizer, expected attendees, status |
| Meeting rooms | Room name/number, capacity, location |
| People | Employee name, linked user account, department |
| Departments | Department name |
| User accounts | Username, email, employee/admin role, password |
| Attendee lists | Booking-to-employee assignments |

Create a user account and department before creating an employee. Create rooms and employees before booking. Add attendee assignments after the booking exists. Expected attendee count includes the organizer; individual attendance assignments are managed separately and may not exceed that count.

- Search, status filters, 15-row pagination, real database counts, upcoming meetings and today’s room booking counts.
- Employee pages show live workspace data. Employees can submit pending requests with their own employee identity. Admin CRUD is available only through the admin workspace and is checked against the current database role.
- Passwords are hashed. Leaving an edit password blank retains the existing password.
- Booking times must form a valid same-day interval. Pending and confirmed bookings reserve a room; cancelled bookings do not. Adjacent intervals are allowed. Room locks serialize booking writes; overlapping intervals and capacity violations are rejected.
- Deleting a booking also removes its attendee assignments. Referenced rooms, departments, employees and linked users cannot be deleted until related records are reassigned or deleted. Deleting an employee does not delete their user account.
- Administrators cannot delete or demote their own account. Deleted user sessions are rejected on the next workspace request.
- Forms use CSRF protection, prepared queries, HTML escaping, post/redirect/get and explicit server-rendered deletion confirmation. No write happens through a GET link.

## Validation

The editing environment did not include PHP or MySQL, so PHP execution and database integration could not be run there. JavaScript syntax and isolated mobile-menu/submit-handler tests passed. Browser rendering could not be verified because this environment also lacks a browser executable. Run these checks on your PHP/MySQL environment before deployment:

```sh
find app public views bin tests -name '*.php' -exec php -l {} \;
php tests/validation.php
```

The integration suite requires an **empty disposable database** whose name starts with `gather_test_`. It imports sample tables, exercises all six CRUD sections, validates search/filtering, capacity and overlap rejection, role restrictions, password retention and relationship protection. It intentionally leaves that disposable database in place for inspection.

```sh
# First create an empty database named gather_test_crud with your MySQL tools.
TEST_DB_NAME=gather_test_crud php tests/integration.php
```

For browser verification, sign in once as an admin and once as an employee. Confirm that create/edit/delete actions persist after refresh, expired CSRF tokens reject writes, employee access to `admin.php` returns 403, and narrow layouts allow navigation and horizontal table scrolling. This delivery is source code for your PHP host; it has not been deployed.
# meeting-room-booking-system
