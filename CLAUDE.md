# Backend Coding Standards (General)

These are the company coding standards for all our NestJS + PostgreSQL backend projects.
Follow them for **every** code change. Project-specific rules (layout, paths, tools) live in the project's own `CLAUDE.md`.

When existing code breaks these rules, follow the rules for **new or changed code**. Don't refactor unrelated code unless asked.

---

## 1. General

- Keep the project in well-structured directories. Use meaningful file and folder names, and group related files together.
- **Order inside classes and services:** public methods first, private methods at the bottom.
- Write self-explanatory code first. Use comments only where they add value (see §4).

---

## 2. Variables

- **camelCase** for variable names, e.g. `myVariableName`.
- **Descriptive, intention-revealing names.** Use `counter`, not `c`. No single-letter or cryptic names.
- **Short but clear:** prefer `email` over `userEmailAddress` when the context makes it clear.
- **Don't use reserved or ambiguous words** as variable names, e.g. `order`, `key`, `index`.
- **Avoid abbreviations** unless they're widely known, e.g. `min`, `max`.
- **Singular vs plural:** use `user` for one item and `users` or `customers` for collections.
- **No special characters** like `_` or `$` in variable names unless necessary.
- **Booleans** get clear, positive names: `isActive`, not `isNotInactive`.
- **Don't reuse one name** for different purposes in different parts of the code.
- **No magic numbers or strings.** Replace them with named constants (see §3).
- If an unusual name is unavoidable, add a comment explaining it.

### Interfaces / types
- Use interfaces to type variables, props and return values.
- **Interface names start with `I`**, e.g. `IUser`, `IAllBookingRowView`.

---

## 3. Constants

- **UPPER_SNAKE_CASE**, e.g. `MAX_VALUE`, `API_KEY`, `HTTP_STATUS_OK`.
- **Descriptive names** that explain the purpose without needing a comment. No cryptic or single-letter names.
- Optionally **prefix** with a module or domain name, e.g. `HTTP_STATUS_OK`.
- **Immutable.** Never change a constant at runtime.
- **Avoid names that are easy to confuse** with each other.
- **No reserved words** as constant names.
- Use constants to replace magic numbers and strings, and for configuration values and defaults.

```ts
// Bad
return 3.14159 * 2 * radius;

// Good
const PI = 3.14159;
return PI * 2 * radius;
```

### Where to put constants
- **Constants used by only one module:** put them in a `constant.ts` file inside that module.
- **Constants shared across the app:** put them in a global constants file.
- **Group related constants** in an object or array, e.g. `COLORS.RED`, `COLORS.BLUE`.
- Add a short comment for complex or non-obvious constants.
- Remove constants that are no longer used.

---

## 4. Comments

### Principles
- **Clear and concise.** No long-winded comments.
- **Use them sparingly.** Not every function needs one. Add comments when the logic is complex, so other developers can follow it.
- **Explain *why*, not *what*.** Don't repeat what the code already says.
- **Keep comments up to date** whenever you change the code they describe.
- **Use `TODO` comments** to mark code that needs attention or improvement.
- **No initials or dates in comments.** `git blame` already shows who wrote what.

### Format
```ts
// Single-line comment

/*
 * Multi-line comment.
 * It can span multiple lines.
 */
```

### Function doc comments
For functions that need documentation, use JSDoc with:
- **Description:** what the function does and *why*, including any business rule
- **`@param`:** each parameter with its type and meaning. Mark optional ones.
- **`@returns`:** the return type and meaning

```ts
/**
 * Validates the pricing rule for a booking row article.
 *
 * Ensures the cost price (priceIn) does not exceed the selling price (priceOut).
 * This is a business rule to prevent selling at a loss.
 *
 * @param {number} priceOut - The selling price of the article in the booking row.
 * @param {number} priceIn - The cost price of the article in the booking row.
 * @returns {boolean} - True if priceOut >= priceIn, otherwise false.
 */
checkPriceOutAndPriceIn(priceOut: number, priceIn: number): boolean {
  // ...
}
```

```ts
/**
 * Retrieves the default SMS confirmation text for a booking in the global web view.
 *
 * @param {number} id - The ID of the booking.
 * @param {number} [companyHasUserId] - Optional. The ID of the user.
 * @returns {Promise<defaultSMSConfirmation>} - Resolves to the default SMS confirmation text.
 */
```

### Inline comments
- Use inline comments on non-obvious config options, e.g. `allowedTags: [], // No tags are allowed`.

---
## 5. Database (PostgreSQL)

### Every table must have
- `id`
- `created`, a timestamp set by a Postgres function (DB default), not by application code

### Column order in a table
1. `id`
2. `created` timestamp
3. `updated` timestamp
4. user / company relations
5. other relation ids
6. values
7. type / state values

### Table naming
- **Table names** are singular snake_case, e.g. `company`.
- **Pivot tables** (the many-to-many link between two tables) use `_has_`, e.g. `company_has_user`, `role_has_permission`. Normal tables and one-to-many child tables don't use `_has_`.

### Rules
- **Migrations:** never change an existing migration. Create a new migration for every schema change.

---
## 6. NestJS

### Naming
- **PascalCase** for classes: modules, controllers, services, resolvers, repositories, entities and DTOs, e.g. `AuthModule`, `AuthController`, `AuthService`.
- **kebab-case** for files and folders, e.g. `auth.module.ts`, `auth.controller.ts`, `auth.service.ts`.

| Type | File | Class |
|---|---|---|
| Module | `company.module.ts` | `CompanyModule` |
| Controller | `company.controller.ts` | `CompanyController` |
| Service | `company.service.ts` | `CompanyService` |
| Resolver | `company.resolver.ts` | `CompanyResolver` |
| Repository | `repository/company.repository.ts` | `CompanyRepository` |
| Entity | `entities/company.entity.ts` | `Company` |
| GraphQL input DTO | `dto/create-company.input.ts` | `CreateCompanyInput` |
| GraphQL output DTO | `dto/login-response.object.ts` | `LoginResponseObject` |
| Other DTO (REST / provider payload) | `dto/create-shipment-label.dto.ts` | `ShipAddressDto` |
| Interface | `interface/company.interface.ts` | `ICompany` |
| Constants | `company.constant.ts` | `COMPANY_TYPE` |

### Structure
- Organize code by feature module, with modules, controllers, services, repositories, resolvers, DTOs, entities and middleware.
- Use meaningful file and folder names, and group related files together.

```
src/modules/company/
├── company.module.ts
├── company.resolver.ts
├── company.service.ts
├── company.constant.ts
├── dto/
├── entities/
├── interface/
└── repository/
```

### Modules
- Declare **all** dependencies of the module in the module file.
- If you **import** a module, **export** it as well.
- Every service in `providers` must also be in `exports`.

### Controllers
- **Keep them thin.** Business logic belongs in services.
- Use `@Controller`, `@Get`, `@Post`, `@Put` and `@Delete` for routes.
- Give each route a single, focused responsibility.
- Validate request data with **DTOs + validation pipes**.
- Use `@Req()` and `@Res()` sparingly. Prefer Nest's abstractions.

### Services
- All **business logic** lives in services.
- Follow the **Single Responsibility Principle**: each service has a well-defined role.
- Services do **not** query the database directly. They go through repositories.

### Repositories
- We use the **repository pattern**. **All database logic** lives in repositories.

### Resolvers
- **One resolver per module**, named after its entity: `@Resolver(() => Company)` → `CompanyResolver` in `company.resolver.ts`.
- **Keep them thin:** read the request, call the service, return the result. Business logic goes in the service.
- **Always set the query name** explicitly: `@Query(() => Company, { name: 'getCompany' })`.
  - `getAll<Entity>` for lists, e.g. `getAllCompany`
  - `get<Entity>` for one record by id, e.g. `getCompany`
  - `getMy<Entity>` for data scoped to the logged-in user, e.g. `getMyCompany`
- **Mutation names start with a verb:** `create<Entity>`, `update<Entity>`, `remove<Entity>`, or a specific action like `closeOutMasterShipment`.
- **The `@Args` name matches the parameter name**, in camelCase: `@Args('updateCompanyInput') updateCompanyInput: UpdateCompanyInput`.
- **Inputs come from DTOs** (`Create<Entity>Input` / `Update<Entity>Input`), not loose arguments, except simple ones like `id`.
- Add **`@Permissions(...)`** to every query and mutation that needs access control.

### DTOs
- **Folder:** DTOs live in the module's `dto/` folder, one file per DTO.
- **File and class names:** the file is kebab-case and the class is PascalCase, e.g. `create-company.input.ts` → `CreateCompanyInput`.
- **GraphQL inputs:** use `.input.ts` with `@InputType()`, and the class name ends with `Input`.
- **GraphQL outputs that aren't entities:** use `.object.ts` with `@ObjectType()`, and the class name ends with `Object`.
- **Everything else** (REST bodies, provider payloads): use `.dto.ts`, and the class name ends with `Dto`.
- **Create / update pairs** are named `Create<Entity>Input` and `Update<Entity>Input`.
- **Build the update DTO from the create DTO** with `PartialType(...)` rather than repeating the fields: `class UpdateCompanyInput extends PartialType(CreateCompanyInput)`.
- **Validate every field** with class-validator decorators (`@IsNotEmpty`, `@IsInt`, `@MaxLength`, etc.).
- **Add a clear error message** wherever the default message isn't clear: `@IsNotEmpty({ message: 'Company name is required' })`.
- **Give every field a GraphQL type** with `@Field(() => Type)`, and mark optional fields with `{ nullable: true }`.
- **Field names match the entity columns** (snake_case, e.g. `company_id`), so a DTO maps straight onto its entity.
- **No logic in DTOs:** they only describe the shape of the data and how to validate it.
- **Shared DTOs** used by more than one module go in the project's shared DTO folder (see the project's `CLAUDE.md`).

### Own-scope vs any-scope operations
When a normal user acts on their own record (their own company, account, profile…) and an admin acts on any record of the same kind, split the API into two sets that share one service. The frontend follows the same split.

- **`My` methods** (`getMy<Entity>`, `createMy<Entity>`, `updateMy<Entity>`, `removeMy<Entity>`) read the owner ID from the logged-in user's token and never accept it as an argument. Guard them with the feature's own permission.
- **General methods** (`get<Entity>`, `create<Entity>`, `update<Entity>`, `remove<Entity>`) accept the owner ID as an argument. Guard them with an admin-only permission, never the same one as the `My` methods. Otherwise any user with that permission can send someone else's ID.
- **Service:** one service for both. It takes the owner ID as its own parameter, so both resolvers pass it in.
- **DTOs:** the `My` input has no owner ID. The general input extends it and adds the ID.

### Entities
- **Folder:** entities live in the module's `entities/` folder, one file per entity, e.g. `entities/company.entity.ts`.
- **Class name:** the singular PascalCase table name, with no `Entity` suffix, e.g. table `company_has_log` → `CompanyHasLog`.
- **Class decorators:** every entity has `@Entity('table_name', { schema: 'public' })` and `@ObjectType()`.
- **Property names:** snake_case to match the database column, e.g. `company_id`, `is_success`. This is the one exception to the camelCase rule in §2.
- **Relation names:** use the singular for `@ManyToOne` / `@OneToOne` (`company`) and the plural for `@OneToMany` (`company_has_log_has_requests`).
- **Decorators on each column:** `@Field()` for GraphQL, `@Column()` for TypeORM, and a class-validator decorator such as `@IsInt()`.
- **Nullable columns:** set `nullable: true` in both `@Column` and `@Field`, and type the property as `type | null`, e.g. `message: string | null`.
- **Source of truth:** the entity is the source of truth for each column's type, nullable flag and default. Never change a column in the database without updating the entity.

### Providers
A provider is self-contained code that talks to one outside service (an API or tool). It is not tied to this project: the folder can be copied into another project, configured, and it works there too.

- **Location:** each provider lives in its own kebab-case folder under the `providers/` directory, named after the outside service it connects to.
- **Module file:** `<provider>.module.ts` registers the provider's service and exports it, so any module can import and use it.
- **Service file:** `<provider>.service.ts` holds all calls to the outside service, with one public method per action and the helpers (auth, headers, tokens) as private methods at the bottom.
- **Supporting folders:** `interface/` for the request, response and credential types, `dto/` for validated request classes when needed, and `constant.ts` for URLs, timeouts and other fixed values.
- **Independent:** a provider never imports anything from the project's modules, entities or repositories, and never touches the database.
- **Configurable:** credentials and settings are passed in from outside, so the same provider works in any project once it is configured.
- **Portable:** you can copy the provider folder into another project and it works as-is, which is how we reuse providers across projects.

### Dependency injection
- Use Nest's built-in DI with **constructor injection**.

### Middleware and interceptors
- **Middleware** handles cross-cutting concerns like authentication, logging and error handling. Create custom middleware when needed.
- **Interceptors** modify requests and responses or transform data. **No business logic** in interceptors.

### Validation
- Use **class-validator** on DTOs.
- Write custom validation pipes when needed.
- Return **clear error messages** when validation fails.

### Testing
- Write unit tests with **Jest**.
- Mock services and dependencies.
- Aim for good coverage of critical logic.

---
