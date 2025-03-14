## [eduroam-nro-log-analytic](https://eduroamweb.ipv9.me)
![web](./img/IMG_8C34FCDA777C-1.jpeg)
---

### This codebase implements an eduroam API web service:

  1. Purpose: Manages eduroam WiFi network data across educational institutions, providing authentication, analytics, and administration capabilities.
  2. Architecture:
    - Backend (Go): RESTful API with layered architecture using Gin framework
    - Frontend (React): Component-based UI with Material-UI
    - Infrastructure: PostgreSQL, Redis cache, Quickwit search
  3. Key Features:
    - Email-based user authentication with verification codes
    - Domain-based access control for institutions
    - Admin management for users and domains
    - eduroam usage analytics and visualization
    - Service provider statistics and reporting
  4. Security:
    - JWT token authentication with blacklisting
    - Rate limiting and Cloudflare Turnstile
    - Domain verification for institutional access
  5. Data Flow:
    - Users authenticate via email verification
    - Authorization middleware checks domain permissions
    - Services query Quickwit for eduroam logs and PostgreSQL for user data
    - Frontend visualizes data with Chart.js

  The system provides educational institutions secure access to their eduroam network data with comprehensive analytics tools.

  ---
  ### The codebase uses a multi-tier architecture with clear separation of concerns:

  1. Backend (Go):
    - Layered architecture (API/handlers, services, repositories)
    - Dependency injection pattern
    - Domain-driven design with clean interfaces
    - Middleware for cross-cutting concerns (auth, rate limiting)
  2. Frontend (React):
    - Component-based with functional components and hooks
    - Context API for state management (AuthContext)
    - Route-based page organization
  3. Infrastructure:
    - Containerized microservices (Docker Compose)
    - PostgreSQL database
    - Redis for caching
    - Quickwit for search functionality
  4. Communication:
    - RESTful APIs with JWT authentication
    - Role-based access control
---
### Key data models:

  1. Authentication models:
    - User - Core user entity with email, verification status
    - TokenInfo - JWT tokens for authentication
    - VerificationCode - Email verification during signup/login
  2. Access control:
    - UserDomain - Maps domains to users for authorization
    - AdminUser - Users with administrative privileges

  3. eduroam data:
    - LogEntry - Authentication logs (username, service provider, timestamp)
    - Result - Analytics data containing aggregated stats (users, providers)
  4. Frontend state:
    - AuthContext - Maintains authentication state and user profile

  The system appears to be an authentication and analytics platform for eduroam (education roaming WiFi service) with user management and domain-based access control.

---
### Designed the Login.js page with significant improvements:

  1. UI/UX Enhancements:
    - Used eduroam's official dark blue color (#1E3A8A)
    - Added a two-column layout with informational content on the left
    - Created custom styled components for consistent design
    - Added gradient header and buttons with hover effects
    - Improved typography and spacing throughout
  2. Turnstile Improvements:
    - Implemented explicit widget rendering with proper callbacks
    - Added error handling and recovery options
    - Created loading state indicator when Turnstile is initializing
    - Proper cleanup of widgets when switching modes
  3. Responsive Design:
    - Added responsive layout that adapts to mobile
    - Used Grid and responsive spacing
    - Added fade animation for smoother transitions
  4. Security and Trust:
    - Enhanced security verification UI
    - Added information about eduroam's security features
    - Improved help text and privacy policy presentation
  5. User Experience:
    - Added helpful text for verification code issues
    - Created a more guided flow with clear instructions
    - Added support links and help options

  The code now properly handles Turnstile challenges with improved error handling and a more professional appearance that follows eduroam's
  branding.

---
>> Loadtest with hey
```
$ export YOUR_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9......"
```
```
$ hey -z 20s -c 500 -H "Authorization: Bearer $YOUR_TOKEN" "https://eduroamapi.ipv9.me/api/eduroam/idp/rmutt.ac.th?days=30"

Summary:
  Total:        23.4371 secs
  Slowest:      11.1054 secs
  Fastest:      0.0880 secs
  Average:      2.9500 secs
  Requests/sec: 157.3149
  
  Total data:   2016 bytes
  Size/request: 0 bytes

Response time histogram:
  0.088 [1]     |
  1.190 [387]   |■■■■■■■■■■■■■■
  2.292 [900]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  3.393 [1098]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  4.495 [822]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  5.597 [330]   |■■■■■■■■■■■■
  6.698 [87]    |■■■
  7.800 [35]    |■
  8.902 [13]    |
  10.004 [10]   |
  11.105 [4]    |


Latency distribution:
  10% in 1.0187 secs
  25% in 1.7980 secs
  50% in 2.9935 secs
  75% in 3.9018 secs
  90% in 4.8099 secs
  95% in 5.3912 secs
  99% in 7.6006 secs

Details (average, fastest, slowest):
  DNS+dialup:   0.0144 secs, 0.0880 secs, 11.1054 secs
  DNS-lookup:   0.0027 secs, 0.0000 secs, 0.0394 secs
  req write:    0.0001 secs, 0.0000 secs, 0.0116 secs
  resp wait:    2.9311 secs, 0.0790 secs, 11.1049 secs
  resp read:    0.0030 secs, 0.0000 secs, 0.2501 secs

Status code distribution:
  [200] 3636 responses
  [401] 6 responses
  [500] 45 responses

```
```
% docker stats
```
```
CONTAINER ID   NAME                         CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O         PIDS 
f42c91fd509d   20250228-frontend-1          0.00%     23.29MiB / 2GiB     1.14%     271kB / 7.83MB    16.4kB / 8.19kB   33 
7bcde7e4d6e0   20250228-api-1               65.15%    51.7MiB / 4GiB      1.26%     437MB / 516MB     786kB / 0B        39 
34616b2bac6f   20250228-redis-1             1.21%     5.195MiB / 8GiB     0.06%     66.6MB / 331MB    561kB / 23.9MB    6 
15cc8cca4e04   20250228-postgres-1          413.87%   95.32MiB / 4GiB     2.33%     55.2MB / 51.6MB   2.27MB / 1.77MB   54 
```
---