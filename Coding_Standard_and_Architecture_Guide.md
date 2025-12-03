# Coding Standard & Architecture Guide for Junior Developers

## 1. Coding Standards (Universal)

### 1.1 Naming Conventions

-   Variables: camelCase
-   Functions: camelCase
-   Classes: PascalCase
-   Constants: UPPER_SNAKE_CASE

### 1.2 Code Formatting Rules

-   4 spaces indentation
-   Max 120 chars per line
-   Strict typing required

### 1.3 Comments

-   Comment WHY, not WHAT

### 1.4 Error Handling

-   Always log errors
-   No empty catch blocks

### 1.5 Testing

-   Arrange / Act / Assert pattern

------------------------------------------------------------------------

## 2. Architecture & Design Patterns

### 2.1 Layered Architecture

Controller → Service → Repository → Model

### 2.2 DTO Pattern

-   Enforce strict data structure

### 2.3 Repository Pattern

-   No DB queries in controller

### 2.4 Factory Pattern

-   Useful for promotions, invoice types

### 2.5 Dependency Injection

-   Constructor injection only

------------------------------------------------------------------------

## 3. Folder Structure Standards

### NestJS

modules/invoice/\*

### Laravel

Services, Repositories, DTO

### Vue.js

api, components, stores

------------------------------------------------------------------------

## 4. Git & Review Guidelines

-   Commit types: feat, fix, refactor, chore
-   PR \< 400 lines

------------------------------------------------------------------------

## 5. Teaching Plan

-   Week 1: Clean code
-   Week 2: Architecture
-   Week 3: Patterns
-   Week 4: Project + Review
