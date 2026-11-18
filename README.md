# WST2 · Activity 2 – Role-Based Dashboard

**Name:** Rey Anthony E. Dahunog · **Section:** BS Info Tech 4C

Extends Activity 1 (secure login) with three roles – **admin, instructor, student** – and protects every page on the server. Deny by default.

## Access matrix
| Page | Admin | Instructor | Student |
|---|:-:|:-:|:-:|
| `admin/manage_users.php` | ✓ | ✗ | ✗ |
| `instructor/encode_grades.php` | ✓ | ✓ | ✗ |
| `student/my_grades.php` | ✗ | ✗ | ✓ |

## What was built
- `auth.php` – `require_login()` (redirects to `BASE_URL . '/login.php'`, then `exit`) and `require_role(string ...$roles)` (strict `in_array`, otherwise HTTP 403 + friendly page, then `exit`).
- Each protected page calls `require_role(...)` on its first line, exactly as the matrix says.
- `includes/header.php` – menu links shown by role (usability only; the real lock is `require_role()`).
- `403.php` – short, polite message with a **Go back to dashboard** button, no technical details.
- `student/my_grades.php` – the student ID comes from `$_SESSION['user_id']`, never from the URL (IDOR fixed).
- `dashboard.php` – uses `auth.php` + `require_login()`.

## Test results (all 11 passed)
| # | Logged in as | URL typed | Expected | Actual |
|---|---|---|---|---|
| 1 | dahunog_admin | admin/manage_users.php | Page opens | Page opens (HTTP 200) |
| 2 | dahunog_admin | instructor/encode_grades.php | Page opens | Page opens (HTTP 200) |
| 3 | dahunog_admin | student/my_grades.php | 403 page | 403 page (HTTP 403) |
| 4 | dahunog_instructor | admin/manage_users.php | 403 page | 403 page (HTTP 403) |
| 5 | dahunog_instructor | instructor/encode_grades.php | Page opens | Page opens (HTTP 200) |
| 6 | dahunog_instructor | student/my_grades.php | 403 page | 403 page (HTTP 403) |
| 7 | dahunog_student1 | admin/manage_users.php | 403 page | 403 page (HTTP 403) |
| 8 | dahunog_student1 | instructor/encode_grades.php | 403 page | 403 page (HTTP 403) |
| 9 | dahunog_student1 | student/my_grades.php | Own grades only | Own grades only – dahunog_student1's 3 subjects |
| 10 | dahunog_student1 | student/my_grades.php?student_id=5 (dahunog_student2) | Still dahunog_student1's grades | Still dahunog_student1's grades – URL id ignored |
| 11 | (logged out) | admin/manage_users.php | Login page | Redirected to the login page |

## How to run (XAMPP)
1. Copy this folder into `C:\xampp\htdocs\wst2_auth` and start Apache + MySQL.
2. In phpMyAdmin run `database/01_users_table.sql`, then `database/02_grades_table.sql`.
3. Register four accounts at http://localhost/wst2_auth/register.php, then set the admin and instructor roles with the `UPDATE` lines in `02_grades_table.sql`.
4. Log in as the instructor and encode grades for both students.
