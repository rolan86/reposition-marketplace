# Stack Detector — Reference Document

This is a supporting reference document read by the `reposition-equip` orchestrator (SKILL.md).
It defines the rules Claude follows to detect a project's technology stack and configure the toolkit accordingly.
There are two paths: **Quick Path** (manifest-based auto-detection) and **Guided Path** (interactive Q&A).

---

## Section 1 — Quick Path Detection

### Step 1: Read Manifest Files

Scan only the **PROJECT_ROOT** directory (not recursively). Look for the following files in this exact order:

1. `package.json`
2. `requirements.txt`
3. `pyproject.toml`
4. `go.mod`
5. `pom.xml`
6. `build.gradle`
7. `build.gradle.kts`
8. `Cargo.toml`

Read whichever manifest files are present. If multiple are present, note all of them — the project may be a multi-service or monorepo setup.

### Step 2: Map Manifest Content to Template Name

Apply the following rules in order:

#### `package.json` (Node.js / TypeScript / JavaScript)

- If `dependencies` or `devDependencies` contains `react`, `react-dom`, or `next` → template: **`typescript-react`**
- Else if `dependencies` or `devDependencies` contains `@angular/core` → template: **`angular`**
- Else (no React or Angular detected) → template: **`generic`** *(TypeScript/JS project)*

#### `requirements.txt` (Python)

- If file contains `fastapi` (case-insensitive line match) → template: **`python-fastapi`**
- Else → template: **`generic`** *(Python project)*

#### `pyproject.toml` (Python)

- If file contains `fastapi` under `[tool.poetry.dependencies]`, `[project.dependencies]`, or any `dependencies` key → template: **`python-fastapi`**
- Else → template: **`generic`** *(Python project)*

#### `go.mod` (Go)

- Presence of `go.mod` alone is sufficient → template: **`go-api`**

#### `pom.xml` (Java/Maven)

- If file contains `spring-boot` in any artifact ID or dependency groupId → template: **`java-spring`**
- Else → template: **`generic`** *(Java project)*

#### `build.gradle` or `build.gradle.kts` (Java/Kotlin/Gradle)

- If file contains `spring-boot` in any plugin ID, dependency, or import → template: **`java-spring`**
- Else → template: **`generic`** *(Java project)*

#### `Cargo.toml` (Rust)

- Presence of `Cargo.toml` alone is sufficient → template: **`generic`** *(Rust project)*

#### No Manifest Found

1. Check for common source file extensions in PROJECT_ROOT and one level deep:
   - `.ts`, `.tsx`, `.js`, `.jsx` → likely TypeScript/JavaScript
   - `.py` → likely Python
   - `.go` → likely Go
   - `.java` → likely Java
   - `.cs` → likely C#
   - `.rs` → likely Rust
2. If source files are found → **fall to Guided Path** (cannot auto-detect without a manifest)
3. If no source files and no manifest → inform the user:
   > "I couldn't detect a project structure here. If you're starting a new project, try running `/reposition-init` first to scaffold your repository."

### Step 3: Confirm with User

After identifying a template, present this confirmation prompt before proceeding:

> "I see a **[Language] / [Framework]** project with **[list key detected dependencies or identifiers]**.
> I'll configure your toolkit for this stack. Sound right? (yes / no)"

Replace `[Language]`, `[Framework]`, and `[key deps]` with what was actually detected. Examples:
- "I see a **TypeScript / React** project with `react`, `react-dom`, `vite`. I'll configure your toolkit for this stack. Sound right?"
- "I see a **Python / FastAPI** project with `fastapi`, `pydantic`. I'll configure your toolkit for this stack. Sound right?"
- "I see a **Go** project with module `github.com/example/myapp`. I'll configure your toolkit for this stack. Sound right?"

**If the user confirms (yes):** Proceed to Section 3 — Output using the detected values.

**If the user says no:** Fall to Guided Path (Section 2). Do not re-attempt auto-detection.

---

## Section 2 — Guided Path Q&A

When auto-detection fails or the user rejects the detected stack, ask the following questions sequentially. Wait for each answer before asking the next.

### Question 1 — Primary Language

> "What is the primary language of this project?"

Present as a numbered list:

1. TypeScript (or JavaScript)
2. Python
3. Go
4. Java
5. C#
6. Rust
7. Other

**Note:** C#, Rust, and Other all map to the **`generic`** template. Inform the user:
> "For [C# / Rust / Other] projects, I'll use a general-purpose template that covers core repository best practices without framework-specific tooling."

Store answer as `language`.

---

### Question 2 — Application Type

> "What kind of application is this?"

Present as a numbered list:

1. Web API (REST or GraphQL backend)
2. Full-stack web app (frontend + backend in same repo)
3. CLI tool
4. Library / SDK
5. Multi-service system (monorepo or multiple services)

Store answer as `app_type`.

**Special case — Multi-service:** If the user selects option 5, skip Question 3 and go directly to Question 4.

---

### Question 3 — Framework

> "Which framework are you using?"

Show only the options relevant to the language selected in Question 1:

**TypeScript / JavaScript:**
1. React / Next.js
2. Angular
3. Vue
4. Express
5. None (plain Node.js or Deno)

**Python:**
1. FastAPI
2. Django
3. Flask
4. None (plain Python)

**Go:**
1. Gin
2. Echo
3. Chi
4. Standard library (no framework)

**Java:**
1. Spring Boot
2. Quarkus
3. Micronaut
4. None (plain Java)

**C# / Rust / Other:**
> "Framework selection is not applicable for this language — skipping to next question."
(Skip to Question 5.)

Store answer as `framework`.

**Template mapping for guided path (language + framework → template):**

| Language         | Framework               | Template           |
|------------------|-------------------------|--------------------|
| TypeScript       | React / Next.js         | `typescript-react` |
| TypeScript       | Angular                 | `angular`          |
| TypeScript       | Vue / Express / None    | `generic`          |
| Python           | FastAPI                 | `python-fastapi`   |
| Python           | Django / Flask / None   | `generic`          |
| Go               | Any (Gin/Echo/Chi/Std)  | `go-api`           |
| Java             | Spring Boot             | `java-spring`      |
| Java             | Quarkus / Micronaut / None | `generic`       |
| C#               | N/A                     | `generic`          |
| Rust             | N/A                     | `generic`          |
| Other            | N/A                     | `generic`          |

---

### Question 4 — Services (Multi-service only)

Only ask this question if `app_type` is **Multi-service system**.

> "Please list each service in this project and its stack. For example:
> - `api-gateway`: TypeScript / Express
> - `user-service`: Python / FastAPI
> - `worker`: Go / Standard library
>
> You can type them out in any format — I'll parse it."

Accept free text. Extract service names and their stacks to the best of your ability.

Store answer as `services` (array of `{ name, language, framework }` objects or raw text if parsing is ambiguous).

After this question, continue to Question 5 for the primary service or the most common language across services.

---

### Question 5 — Test Runner

> "Which test runner does this project use (or will use)?"

Show only the options relevant to the detected or selected language:

**TypeScript / JavaScript:**
1. Vitest
2. Jest
3. Mocha
4. None / not set up yet

**Python:**
1. pytest
2. unittest
3. None / not set up yet

**Go:**
> "Go uses the built-in `go test` — I'll configure for that automatically."
(No choice needed; set `test_runner` to `go test` and skip user input.)

**Java:**
1. JUnit (most common — usually bundled with Spring or Maven)
2. TestNG
(If unsure, recommend JUnit as the default.)

**C# / Rust / Other:**
> "I'll note this as unspecified — you can set up test tooling separately."
(Set `test_runner` to `null`.)

Store answer as `test_runner`.

---

## Section 3 — Output

After Quick Path or Guided Path is complete, assemble the following **stack config object** and return it to the equip orchestrator.

### Stack Config Object

```json
{
  "language": "<detected or user-selected language>",
  "framework": "<detected or user-selected framework, or null if none>",
  "app_type": "<web-api | fullstack | cli | library | multi-service>",
  "template": "<typescript-react | angular | python-fastapi | go-api | java-spring | generic>",
  "test_runner": "<vitest | jest | mocha | pytest | unittest | go test | junit | testng | null>",
  "services": null
}
```

For **multi-service** projects, populate `services` as an array:

```json
{
  "language": "<primary language>",
  "framework": "<primary framework or null>",
  "app_type": "multi-service",
  "template": "<template for primary service>",
  "test_runner": "<test runner for primary service or null>",
  "services": [
    { "name": "api-gateway", "language": "TypeScript", "framework": "Express" },
    { "name": "user-service", "language": "Python", "framework": "FastAPI" },
    { "name": "worker", "language": "Go", "framework": "Standard library" }
  ]
}
```

### Return Instruction

Pass this config object back to the equip orchestrator verbatim. The orchestrator uses `template` to select the correct template directory and uses the remaining fields to populate tool configuration, lint rules, CI scaffolding, and documentation stubs.

**Do not proceed with any file generation until the orchestrator has acknowledged receipt of this config.**
