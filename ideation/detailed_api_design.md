
# Detailed API Design

Base Path: /api/v1

Authentication: JWT tokens

## Auth

POST /auth/login
POST /auth/register
POST /auth/logout

## Users

GET /users/me
GET /users/{id}
PUT /users/{id}

## Students

GET /students
POST /students
GET /students/{id}

## Classes

GET /classes
POST /classes
GET /classes/{id}

## Assignments

GET /assignments
POST /assignments
POST /assignments/{id}/submit

## Attendance

POST /attendance
GET /attendance/{student_id}

## Messaging

GET /messages
POST /messages

## Announcements

GET /announcements
POST /announcements
