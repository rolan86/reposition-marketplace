# Test Scaffolding — Config and Example Templates

This file is read by the deliver skill orchestrator. The orchestrator selects the
section matching `.reposition/equip/stack.json`, checks for existing test config,
and creates files only when they are absent.

---

## Deliver Skill — Install Logic

Before creating any file, the orchestrator checks whether test infrastructure already exists.

### Existence Checks (Per Stack)

| Stack | Skip condition |
|---|---|
| Python/FastAPI | `pytest.ini`, `pyproject.toml [tool.pytest]`, or `setup.cfg [tool:pytest]` exists |
| TypeScript/React | `vitest.config.ts`, `vitest.config.js`, or `jest.config.*` exists |
| Go | No config needed — skip scaffolding, go to example file check only |
| Java/Spring | `pom.xml` already present (do not create or modify it) — skip to example file |
| Angular | `karma.conf.js` or `angular.json` present (CLI manages test config) — skip to example |
| Generic | Skip entirely — no assumptions about stack |

If the skip condition is met, log:
```
[reposition-equip] Test config already exists — skipping [stack] test scaffolding.
Example test template saved to: .reposition/equip/example-test.[ext]
```

If no config exists, create the files listed below and log each creation.

---

## Python/FastAPI

### pytest.ini

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
addopts = --tb=short -q --cov=app --cov-report=term-missing
```

### tests/__init__.py

```python
```

*(Empty file — makes `tests/` a package so relative imports work.)*

### tests/conftest.py

```python
import pytest
import pytest_asyncio
from httpx import AsyncClient, ASGITransport
from app.main import app  # adjust import to match your project structure


@pytest_asyncio.fixture
async def client():
    """Async test client for the FastAPI app."""
    async with AsyncClient(
        transport=ASGITransport(app=app), base_url="http://test"
    ) as ac:
        yield ac
```

### tests/test_example.py

```python
import pytest


class TestHealthCheck:
    """Example test class — replace with real tests."""

    async def test_health_returns_200(self, client):
        response = await client.get("/health")
        assert response.status_code == 200

    async def test_health_returns_expected_body(self, client):
        response = await client.get("/health")
        data = response.json()
        assert "status" in data

    async def test_missing_endpoint_returns_404(self, client):
        response = await client.get("/does-not-exist")
        assert response.status_code == 404
```

---

## TypeScript/React

### vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./src/__tests__/setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
    },
  },
});
```

### src/__tests__/setup.ts

```typescript
import "@testing-library/jest-dom";
```

### src/__tests__/example.test.tsx

```typescript
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";

// Replace ExampleComponent with your real component
function ExampleComponent({ onAction }: { onAction: () => void }) {
  return <button onClick={onAction}>Click me</button>;
}

describe("ExampleComponent", () => {
  it("renders the action button", () => {
    render(<ExampleComponent onAction={vi.fn()} />);
    expect(screen.getByRole("button", { name: /click me/i })).toBeInTheDocument();
  });

  it("calls onAction when button is clicked", async () => {
    const user = userEvent.setup();
    const handleAction = vi.fn();

    render(<ExampleComponent onAction={handleAction} />);
    await user.click(screen.getByRole("button", { name: /click me/i }));

    expect(handleAction).toHaveBeenCalledOnce();
  });
});
```

---

## Go

No configuration file needed — the standard `go test ./...` command discovers tests
automatically. There is no `go.test.json` or equivalent.

### example_test.go

Place alongside the package being tested (e.g., `internal/calculator/calculator_test.go`).

```go
package calculator_test

import (
	"testing"

	"github.com/your-org/your-repo/internal/calculator" // adjust import path
)

func TestAdd(t *testing.T) {
	tests := []struct {
		name     string
		a, b     int
		expected int
	}{
		{name: "positive numbers", a: 2, b: 3, expected: 5},
		{name: "zero values", a: 0, b: 0, expected: 0},
		{name: "negative numbers", a: -1, b: -2, expected: -3},
		{name: "mixed sign", a: 5, b: -3, expected: 2},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := calculator.Add(tt.a, tt.b)
			if got != tt.expected {
				t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.expected)
			}
		})
	}
}
```

---

## Java/Spring

The deliver skill does **not** create or modify `pom.xml`. JUnit 5 and Spring Boot Test
are assumed to be present via `spring-boot-starter-test` in the existing dependency manifest.

### src/test/java/com/example/ExampleTest.java

*(Adjust the package path to match your project structure.)*

```java
package com.example;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class ExampleTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @DisplayName("GET /health returns 200 OK")
    void healthEndpointReturns200() throws Exception {
        mockMvc.perform(get("/health").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk());
    }

    @Test
    @DisplayName("GET /does-not-exist returns 404 Not Found")
    void unknownEndpointReturns404() throws Exception {
        mockMvc.perform(get("/does-not-exist"))
                .andExpect(status().isNotFound());
    }
}
```

---

## Angular

The Angular CLI manages test configuration via `angular.json` and sets up Karma/Jasmine
by default. The deliver skill does not create `karma.conf.js` if it already exists or
if `angular.json` is present — the CLI configuration is authoritative.

To run tests: `ng test --watch=false`

### example.component.spec.ts

Place alongside the component being tested (e.g., `src/app/example/example.component.spec.ts`).

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { By } from "@angular/platform-browser";
import { ExampleComponent } from "./example.component";

describe("ExampleComponent", () => {
  let component: ExampleComponent;
  let fixture: ComponentFixture<ExampleComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ExampleComponent], // standalone component — import directly
    }).compileComponents();

    fixture = TestBed.createComponent(ExampleComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("should create the component", () => {
    expect(component).toBeTruthy();
  });

  it("should render the heading", () => {
    const heading = fixture.debugElement.query(By.css("h1"));
    expect(heading).toBeTruthy();
  });

  it("should update state when action is triggered", () => {
    // Arrange: set up initial signal value
    // component.count.set(0);

    // Act: trigger the action
    // const button = fixture.debugElement.query(By.css("button[data-action='increment']"));
    // button.triggerEventHandler("click", null);
    // fixture.detectChanges();

    // Assert: verify the new state
    // expect(component.count()).toBe(1);

    // Replace the above with your real component's signal and action.
    expect(true).toBeTrue(); // placeholder — replace with real assertions
  });
});
```

---

## Generic

The deliver skill skips test scaffolding for projects with a `generic` stack config.

No assumptions are made about the test framework, file layout, or configuration format.
Attempting to scaffold tests without knowing the stack would produce incorrect output.

To add tests to a generic project:
1. Choose a test framework appropriate for your language.
2. Follow its getting-started guide to create a config file and first test.
3. Update `.reposition/equip/stack.json` with the correct stack once identified
   so future equip runs can provide stack-specific scaffolding.
