# ✅ **Coding Standard + Architecture Guide for Junior Developers**

### *A practical version you can teach in 1–2 sessions*

---

# **1. Coding Standards (Universal)**

## ✅ 1.1 Naming Conventions

**General rule:** Names must be **clear**, **descriptive**, and **consistent**.

| Type            | Style            | Examples                             |
| --------------- | ---------------- | ------------------------------------ |
| Variables       | camelCase        | `userName`, `totalAmount`            |
| Functions       | camelCase        | `calculateTotal()`, `loadInvoice()`  |
| Classes         | PascalCase       | `InvoiceService`, `UserController`   |
| Files (backend) | PascalCase       | `InvoiceService.ts`, `UserModel.php` |
| Constants       | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`                    |

### ❌ Avoid

* `x`, `tmp`, `data1`, `test3`, `obj`
* Overly short or cryptic names

### ✔ Prefer

* `invoiceItems`
* `mappedResults`
* `startDateFormatted`

---

## ✅ 1.2 Code Formatting Rules

* Use **4 spaces** for indentation (not tabs)
* Max line length **<= 120 chars**
* Add a blank line between logical sections
* Always use **strict typing**

  * TypeScript: `string`, `number`, `Record<string,string>`
  * PHP: `string $name`, `int $count`

---

## ✅ 1.3 Comments & Documentation

### When to comment?

* Explain **why**, not **what**.

### Good example:

```ts
// Prevent double submission by checking if the invoice already exists
if (invoiceRepository.exists(code)) {
```

### Bad example:

```ts
// check invoice exists
if (invoiceRepository.exists(code)) {
```


* Add JSDoc/PHPDoc for all public functions:

```ts
/**
 * Calculate the total price including tax.
 */
```

---

## ✅ 1.4 Error Handling

### ❌ Avoid

```ts
try {
  //...
} catch (e) {}
```

### ✔ Use proper logging

```ts
catch (error) {
  this.logger.error('Fail to create invoice', error);
  throw new InternalServerErrorException('Unable to create invoice');
}
```

---

## ✅ 1.5 Unit & Integration Tests (Basic)

3-act structure:

```ts
// Arrange
const service = new InvoiceService();

// Act
const total = service.calculateTotal(100, 6);

// Assert
expect(total).toBe(106);
```

---

# ---------------------------------

# **2. Architecture & Design Patterns **

Introduce only patterns they will **really use** in your NestJS + Laravel + Vue environment.

---

# ⭐ 2.1 Layered Architecture (Simple)

### **Controller → Service → Repository → Model**

```
[Controller]
  - validate inputs
  - accept HTTP requests
  - send response DTOs

[Service]
  - business logic
  - calculations
  - rules

[Repository]
  - database queries
  - ORM logic

[Model/Entity]
  - structure of data
  - relations
```

---

## Expected Flow Example

**Controller**
→ validate request
→ call service

**Service**
→ main logic
→ call repo

**Repository**
→ fetch DB
→ return plain object

**Controller**
→ return success/failure response

---

# ⭐ 2.2 DTO Pattern (Very Important)

* Prevent random JSON from going everywhere.
* Enforce structure.

## ⭐ Example 1 — DTO Pattern in NestJS (Most Common)

### ❌ Bad Example — No DTO (Dangerous!)

```ts
@Post("create")
create(@Body() body: any) {
  // body = anything the client wants to send
  return this.invoiceService.create(body);
}
```

Problems:

* No validation
* Wrong data types
* Missing required fields
* Security risk

---

## ✔ Good Example — Using DTO

### **Step 1: Create DTO**

```ts
export class CreateInvoiceDto {
  @IsString()
  code: string;

  @IsNumber()
  amount: number;

  @IsOptional()
  @IsString()
  remark?: string;
}
```

### **Step 2: Use DTO in Controller**

```ts
@Post("create")
create(@Body() dto: CreateInvoiceDto) {
  return this.invoiceService.create(dto);
}
```

### **Step 3: Service receives the validated DTO**

```ts
create(dto: CreateInvoiceDto) {
  return this.repo.save(dto);
}
```

Benefits:

* Autocomplete
* Validation
* Type safety
* Clear structure

---

# -------------------------------------------

## ⭐ Example 2 — DTO with Transformation (Mapping form input to model)

### ❌ Without DTO

```ts
create(body) {
  const invoice = new Invoice();
  invoice.a = body.c;
  invoice.b = body.vvv;
  invoice.c = body.xx; // confusing
  return repo.save(invoice);
}
```

### ✔ With DTO

```ts
export class CreateInvoiceDto {
  @IsString()
  invoiceNo: string;

  @IsNumber()
  totalAmount: number;

  @IsString()
  customerId: string;
}
```

Clean mapping:

```ts
create(dto: CreateInvoiceDto) {
  return this.invoiceRepo.save({
    invoiceNo: dto.invoiceNo,
    totalAmount: dto.totalAmount,
    customerId: dto.customerId,
  });
}
```

Meaningful fields → Clear reading.

---

# -------------------------------------------

## ⭐ Example 3 — Nested DTOs (Complex form, order with items)

### DTO for each line item

```ts
export class ItemDto {
  @IsString()
  name: string;

  @IsNumber()
  price: number;

  @IsNumber()
  qty: number;
}
```

### Parent DTO lists items

```ts
export class CreateOrderDto {
  @IsString()
  customerId: string;

  @ValidateNested({ each: true })
  @Type(() => ItemDto)
  items: ItemDto[];
}
```

Controller:

```ts
@Post()
create(@Body() dto: CreateOrderDto) {
  return this.ordersService.create(dto);
}
```

Now the client must send:

```json
{
  "customerId": "C1001",
  "items": [
    { "name": "Apple", "price": 3, "qty": 2 },
    { "name": "Orange", "price": 5, "qty": 1 }
  ]
}
```

---


## ⭐ Example 4 — DTO for Response Objects (API Output)

### Response DTO

### ❌ Without DTO

```ts
return {
  code: 'code';
  total: 123;
  createdAt: 'cretedAt';
}
```

### ✔ With DTO

```ts
export class InvoiceResponseDto {
  code: string;
  total: number;
  createdAt: string;
}
```

Controller:

```ts
return new InvoiceResponseDto(invoice);
```

Purpose:

* Prevent leaking internal database fields (`id`, `deletedAt`, etc.)
* Shape API output clearly

---

# -------------------------------------------

## ⭐ Example 6 — DTO for Filtering / Query Parameters

```ts
export class InvoiceFilterDto {
  @IsOptional()
  @IsString()
  customerId?: string;

  @IsOptional()
  @IsDateString()
  startDate?: string;

  @IsOptional()
  @IsDateString()
  endDate?: string;
}
```

Controller:

```ts
@Get()
getAll(@Query() filters: InvoiceFilterDto) {
  return this.invoiceService.filter(filters);
}
```

---

# -------------------------------------------

## ⭐ Example 7 — DTO Improves Autocomplete & Refactoring

### Instead of this:

```ts
createInvoice(payload) {
  console.log(payload.a, payload.b1, payload.c3);
}
```

### You get:

```ts
createInvoice(dto: CreateInvoiceDto) {
  console.log(dto.code, dto.amount, dto.remark);
}
```

Editor auto-suggest works:

* `dto.code`
* `dto.amount`
* `dto.remark`



---

# ⭐ 2.3 Repository Pattern

Avoid query inside controller/service.

**Bad**

```ts
const invoices = await prisma.invoice.findMany();
```

**Good**

```ts
return this.invoiceRepository.findAll();
```

---

# ⭐ 2.4 Factory Pattern (Useful for dynamic logic)

Introduce for cases like:

* Promotion strategies
* Invoice type handling
* Payment gateway selection

**Example**

```ts
const strategy = PromotionFactory.create(type);
return strategy.apply();
```

---

# ⭐ 2.5 Dependency Injection (DI)

Teach them:

* Do not `new` inside service
* Use constructor injection

```ts
constructor(private invoiceRepo: InvoiceRepository) {}
```

---

# ⭐ 2.6 Clean Code Patterns

## Avoid Magic Numbers

### ❌ Bad

```ts
const total = amount * 1.06;
```

### ✔ Good

```ts
const TAX_RATE = 0.06;
const total = amount * TAX_RATE;
```

**Why?**
Future developers instantly know what 1.06 means.

## No huge functions

Limit to **20–30 lines maximum**.

### ❌ Bad — function doing too many things

```ts
function processInvoice(invoice) {
  invoice.total = invoice.items.reduce((a, b) => a + b.price, 0);
  console.log("invoice total", invoice.total);
  saveInvoice(invoice);
}
```

### ✔ Good — split into smaller responsibilities

```ts
function calculateInvoiceTotal(invoice) {
  return invoice.items.reduce((a, b) => a + b.price, 0);
}

function saveInvoice(invoice) {
  // save...
}

function processInvoice(invoice) {
  invoice.total = calculateInvoiceTotal(invoice);
  saveInvoice(invoice);
}
```

## No long parameter list

Use objects or DTO.

### ❌ Bad

```ts
createUser("Victor", "Khoo", "victor@gmail.com", true, "admin", "MY");
```

### ✔ Good (using an object or DTO)

```ts
createUser({
  firstName: "Victor",
  lastName: "Khoo",
  email: "victor@gmail.com",
  isActive: true,
  role: "admin",
  country: "MY"
});
```

## Early Return (Reduce Nesting)**

### ❌ Bad

```ts
function login(user) {
  if (user) {
    if (user.isActive) {
      return "Logged in";
    } else {
      return "User inactive";
    }
  } else {
    return "User not found";
  }
}
```

### ✔ Good

```ts
function login(user) {
  if (!user) return "User not found";
  if (!user.isActive) return "User inactive";
  return "Logged in";
}
```

### ❌ Bad — Class does too many things

```ts
class UserController {
  getUsers() {}
  saveUsers() {}
  validateEmail() {}
  sendWelcomeEmail() {}
}
```

### ✔ Good — Separate responsibilities

```ts
class UserController {
  constructor(private userService: UserService) {}
}

class UserService {
  validateEmail() {}
  createUser() {}
}

class EmailService {
  sendWelcomeEmail() {}
}
```

## Single Responsibility Principle (SRP)

### ❌ Bad — Class does too many things

```ts
class UserController {
  getUsers() {}
  saveUsers() {}
  validateEmail() {}
  sendWelcomeEmail() {}
}
```

### ✔ Good — Separate responsibilities

```ts
class UserController {
  constructor(private userService: UserService) {}
}

class UserService {
  validateEmail() {}
  createUser() {}
}

class EmailService {
  sendWelcomeEmail() {}
}
```

---

# --------------------------------------

# **3. Folder Structure Standards (Simple & Scalable)**

## **NestJS**

```
src/
  modules/
    invoice/
      invoice.controller.ts
      invoice.service.ts
      invoice.repository.ts
      invoice.entity.ts
      dto/
      tests/
```

## **Laravel**

```
app/
  Http/
    Controllers/
  Services/
  Repositories/
  Models/
  DTO/
```

## **Vue.js**

```
src/
  api/
  components/
  composables/
  pages/
  stores/
  utils/
```

---

# --------------------------------------

# **4. Git & Code Review Rules**

## ⭐ 4.1 Commit Rules

```
feat: add invoice creation API
fix: correct tax calculation
refactor: extract validation logic
chore: update dependencies
```

## ⭐ 4.2 Pull Request Rules

* Maximum **400 lines**
* Must include:

  * Summary
  * What changed
  * Screenshot (if UI)
  * Test cases (if logic)

---
