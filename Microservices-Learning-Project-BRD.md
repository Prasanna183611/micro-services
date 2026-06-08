# Employee Leave Management System - Microservices Learning Project

**Version:** 1.0
**Document Type:** Business Requirements Document (BRD)
**Project Type:** Spring Boot Microservices Learning Project

---

# 1. Introduction

## 1.1 Purpose

The purpose of this project is to build a distributed microservices-based application that demonstrates real-world microservice architecture patterns and communication mechanisms.

The application simulates an Employee Leave Management System where employees belong to departments, apply for leave, managers approve or reject leave requests, and notifications are generated based on business events.

This project is intended to provide hands-on experience with:

* Service decomposition
* Database-per-service architecture
* Synchronous service communication
* Asynchronous event-driven communication
* API Gateway
* Service Discovery
* Fault Tolerance
* Containerized deployment

---

# 2. Objectives

The primary objectives are:

* Understand how independent services communicate
* Learn synchronous communication using REST APIs
* Learn asynchronous communication using RabbitMQ
* Implement API Gateway routing
* Implement Service Discovery using Eureka
* Implement Circuit Breaker and Retry patterns
* Deploy multiple services independently
* Follow microservice best practices

---

# 3. Business Overview

The organization requires a system that can:

* Manage employees
* Manage departments
* Allow employees to apply for leave
* Allow managers to approve or reject leave requests
* Notify employees when actions occur

The system should be developed as multiple independent services.

---

# 4. Project Scope

## In Scope

### Employee Management

* Create Employee
* Update Employee
* Delete Employee
* View Employee
* Assign Department
* Assign Manager

### Department Management

* Create Department
* Update Department
* View Department

### Leave Management

* Apply Leave
* View Leave
* Approve Leave
* Reject Leave

### Notification Management

* Employee Created Notification
* Leave Applied Notification
* Leave Approved Notification
* Leave Rejected Notification

---

## Out of Scope

* Payroll Management
* Attendance Management
* Recruitment Management
* Performance Management
* Advanced Authentication Systems
* Mobile Application

---

# 5. System Architecture

```text
Client
   |
   v
API Gateway
   |
------------------------------------------------
|               |              |               |
v               v              v               v
Employee     Department      Leave      Notification
Service      Service         Service       Service

                     |
                     v
                  RabbitMQ
```

---

# 6. Microservices Definition

## 6.1 Employee Service

### Responsibility

Maintain employee information.

### Database

#### Employee

| Field        | Type   |
| ------------ | ------ |
| id           | Long   |
| employeeCode | String |
| firstName    | String |
| lastName     | String |
| email        | String |
| designation  | String |
| departmentId | Long   |
| managerId    | Long   |
| status       | String |

### APIs

#### Create Employee

POST /employees

#### Get Employee

GET /employees/{id}

#### Update Employee

PUT /employees/{id}

#### Delete Employee

DELETE /employees/{id}

---

## 6.2 Department Service

### Responsibility

Maintain department information.

### Database

#### Department

| Field          | Type   |
| -------------- | ------ |
| id             | Long   |
| name           | String |
| location       | String |
| headEmployeeId | Long   |

### APIs

#### Create Department

POST /departments

#### Get Department

GET /departments/{id}

#### Update Department

PUT /departments/{id}

---

## 6.3 Leave Service

### Responsibility

Manage employee leave requests.

### Database

#### Leave Request

| Field       | Type   |
| ----------- | ------ |
| id          | Long   |
| employeeId  | Long   |
| leaveType   | String |
| startDate   | Date   |
| endDate     | Date   |
| reason      | String |
| status      | String |
| appliedDate | Date   |
| approvedBy  | Long   |

### APIs

#### Apply Leave

POST /leave

#### Get Leave

GET /leave/{id}

#### Approve Leave

PUT /leave/{id}/approve

#### Reject Leave

PUT /leave/{id}/reject

---

## 6.4 Notification Service

### Responsibility

Consume events and generate notifications.

### Database (Optional)

#### Notification Log

| Field     | Type      |
| --------- | --------- |
| id        | Long      |
| eventType | String    |
| message   | String    |
| sentTime  | Timestamp |

---

# 7. Entity Relationships

## Employee ↔ Department

```text
Employee
    |
    +---- departmentId ----> Department
```

## Employee Self Relationship

```text
Employee
    |
    +---- managerId ----> Employee
```

## Leave Request ↔ Employee

```text
Leave Request
       |
       +---- employeeId ----> Employee
```

## Department Head

```text
Department
      |
      +---- headEmployeeId ----> Employee
```

---

# 8. Functional Requirements

## FR-01 Create Employee

### Actor

HR Admin

### Flow

1. HR submits employee details.
2. Employee Service validates request.
3. Employee record is stored.
4. EmployeeCreatedEvent is published.

### Output

Employee successfully created.

---

## FR-02 View Employee Details

### Actor

Manager

### Flow

1. Request employee details.
2. Employee Service retrieves employee.
3. Employee Service calls Department Service.
4. Department details are retrieved.
5. Combined response is returned.

### Communication

```text
Employee Service
      |
      v
Department Service
```

---

## FR-03 Apply Leave

### Actor

Employee

### Flow

1. Employee submits leave request.
2. Leave Service validates employee.
3. Leave request is stored.
4. LeaveAppliedEvent is published.

### Communication

```text
Leave Service
      |
      v
Employee Service
```

---

## FR-04 Approve Leave

### Actor

Manager

### Flow

1. Manager approves leave.
2. Leave status updated.
3. LeaveApprovedEvent published.
4. Notification Service consumes event.

---

## FR-05 Reject Leave

### Actor

Manager

### Flow

1. Manager rejects leave.
2. Leave status updated.
3. LeaveRejectedEvent published.
4. Notification Service consumes event.

---

# 9. Communication Requirements

## 9.1 Synchronous Communication

### Purpose

Retrieve real-time information from another service.

### Technology

OpenFeign

### Communication Paths

```text
Employee Service
      |
      v
Department Service
```

```text
Leave Service
      |
      v
Employee Service
```

---

## 9.2 Asynchronous Communication

### Purpose

Publish business events.

### Technology

RabbitMQ

### Communication Paths

```text
Employee Service
      |
      v
RabbitMQ
      |
      v
Notification Service
```

```text
Leave Service
      |
      v
RabbitMQ
      |
      v
Notification Service
```

---

# 10. Event Definitions

## EmployeeCreatedEvent

```json
{
  "employeeId": 1,
  "employeeName": "John Doe"
}
```

## LeaveAppliedEvent

```json
{
  "leaveId": 101,
  "employeeId": 1
}
```

## LeaveApprovedEvent

```json
{
  "leaveId": 101,
  "employeeId": 1,
  "status": "APPROVED"
}
```

## LeaveRejectedEvent

```json
{
  "leaveId": 101,
  "employeeId": 1,
  "status": "REJECTED"
}
```

---

# 11. API Gateway Requirements

## Technology

Spring Cloud Gateway

## Responsibilities

* Request Routing
* Central Entry Point
* Logging
* Authentication (Future)

## Routes

```text
/api/employees/**
/api/departments/**
/api/leave/**
```

---

# 12. Service Discovery Requirements

## Technology

Netflix Eureka

### Responsibilities

* Register services automatically
* Discover services dynamically
* Remove hardcoded URLs

### Example

Instead of:

```text
http://localhost:8082
```

Use:

```text
http://DEPARTMENT-SERVICE
```

---

# 13. Fault Tolerance Requirements

## Technology

Resilience4j

### Retry

If Department Service fails:

* Retry 3 times
* Wait configured interval

### Circuit Breaker

If service remains unavailable:

Return fallback response.

Example:

```json
{
  "departmentName": "Unavailable"
}
```

---

# 14. Database Requirements

Each service must own its database.

## Employee Service Database

```text
employee_db
```

## Department Service Database

```text
department_db
```

## Leave Service Database

```text
leave_db
```

## Notification Service Database

```text
notification_db
```

### Rule

Services must never access another service database directly.

Correct:

```text
Leave Service
      |
      v
Employee API
```

Incorrect:

```text
Leave Service
      |
      v
Employee Database
```

---

# 15. Non-Functional Requirements

## Availability

Target availability: 99%

## Scalability

Services should be independently scalable.

## Maintainability

Each service must have:

* Separate Source Code
* Separate Configuration
* Separate Database
* Independent Deployment

## Security

Phase 1:

* Basic JWT Authentication

Future:

* OAuth2
* Role-Based Access Control

---

# 16. Deployment Requirements

Each service must run independently.

| Service              | Port |
| -------------------- | ---- |
| API Gateway          | 8080 |
| Employee Service     | 8081 |
| Department Service   | 8082 |
| Leave Service        | 8083 |
| Notification Service | 8084 |
| Eureka Server        | 8761 |
| RabbitMQ             | 5672 |

---

# 17. Implementation Roadmap

## Phase 1

* Employee Service
* Department Service
* CRUD APIs
* Separate Databases

## Phase 2

* OpenFeign Integration
* Employee + Department Aggregation

## Phase 3

* API Gateway

## Phase 4

* Eureka Service Discovery

## Phase 5

* Leave Service

## Phase 6

* RabbitMQ Integration

## Phase 7

* Notification Service

## Phase 8

* Resilience4j
* Retry Pattern
* Circuit Breaker Pattern

## Phase 9

* Docker Compose
* End-to-End Testing

---

# 18. Success Criteria

The project will be considered complete when:

* All services run independently
* Service-to-service communication works
* RabbitMQ events are processed successfully
* API Gateway routes requests correctly
* Eureka discovers all services
* Circuit Breaker handles failures gracefully
* All services can be started using Docker Compose

---

# End of Document

