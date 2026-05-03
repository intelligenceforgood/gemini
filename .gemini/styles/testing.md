---
applyTo: ["**/*test*.{py,ts,tsx}", "**/conftest.py"]
---

# Testing Standards

## Python Testing (`core/`)

- **Framework:** We use `pytest`. Run via `pytest` or your preferred runner.
- **Plugins:** `pytest-mock` is used for mocking, and `pytest-anyio` handles async tests.
- **Directory Structure:** Tests are primarily located in `core/tests/unit/` with subdirectories mapping to the core architecture (e.g., `api/`, `classification/`, `clients/`, etc.).
- **Fixtures:** Shared fixtures are placed in `conftest.py` files at the appropriate directory level. 
  - Use `@pytest.fixture(autouse=True)` for resetting global state between tests (e.g., clearing request logs or rate limit states).
- **Mocking:** Utilize `pytest-mock` for standard mock patterns.
- **Integration Tests:** Mark slower tests with `@pytest.mark.integration`.

## UI / Frontend Testing (`ui/`)

- **Framework:** We use `vitest` as the test runner, orchestrated via `turbo test`.
- **Testing Library:** Use `@testing-library/react` (`render`, `screen`, `fireEvent`, `waitFor`) for testing component behavior and interactions.
- **Component Mocking:** Aggressively mock heavy third-party libraries (e.g., `recharts`, `lucide-react`) and complex shared UI components (`@i4g/ui-kit`) using `vi.mock()`. 
  - Replace them with simplified HTML elements that retain necessary props (like `className`) or add a `data-testid` for easy querying.
- **Network / API Mocking:** We mock global fetch directly in tests.
  - Pattern: Assign `global.fetch = vi.fn().mockResolvedValue({ ok: true, json: async () => mockData });`
  - Use `beforeEach` to reset these mocks and `afterEach` for cleanup (`vi.restoreAllMocks()`).
- **File Naming:** Test files should use `.test.ts` or `.test.tsx` extensions.
