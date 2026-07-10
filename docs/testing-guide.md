# BrainBytes Testing Guide

## Overview

BrainBytes uses a combination of automated testing tools to ensure application quality and reliability:

* **Jest** for unit and integration testing
* **@testing-library/react** for frontend component testing
* **Playwright** for end-to-end (E2E) browser testing

---

# Test Directory Structure

```text
brainbytes-multi-container/
├── frontend/
│   ├── tests/
│   │   ├── components/
│   │   │   └── ChatInterface.test.js
│   │   └── endpoint/
│   │       └── chat.test.js
│   ├── jest.config.js
│   └── jest.setup.js
│
├── backend/
│   ├── tests/
│   │   ├── unit/
│   │   │   ├── aiService.test.js
│   │   │   └── messageService.test.js
│   │   └── integration/
│   │       ├── messages.test.js
│   │       ├── profiles.test.js
│   │       ├── materials.test.js
│   │       └── errorHandling.test.js
│   └── jest.config.js
│
└── e2e-tests/
    ├── tests/
    │   └── smoke.spec.js
    ├── playwright.config.js
    └── package.json
```

---

# Running Tests

## Frontend Tests

```bash
cd brainbytes-multi-container/frontend

# Run all tests
npm test

# Watch mode
npm test -- --watch

# Generate coverage report
npm test -- --coverage

# Run a test by name
npm test -- -t "ChatInterface"

# Run a specific test file
npm test -- tests/components/ChatInterface.test.js
```

---

## Backend Tests

```bash
cd brainbytes-multi-container/backend

# Run all tests
npm test

# Watch mode
npm test -- --watch

# Generate coverage
npm test -- --coverage

# Run tests matching a pattern
npm test -- -t "profiles"
```

---

## End-to-End Tests

```bash
cd brainbytes-multi-container/e2e-tests

# Execute all Playwright tests
npm run test:e2e

# Run with browser visible
npx playwright test --headed

# Debug mode
npx playwright test --debug

# View HTML report
npx playwright show-report
```

---

# Writing Tests

## Frontend Component Testing

Use **React Testing Library** to render components and simulate user interactions.

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import MyComponent from '../components/MyComponent';

test('renders and responds to user input', async () => {
  render(<MyComponent />);

  const input = screen.getByPlaceholderText(/enter text/i);

  await userEvent.type(input, 'Hello');

  fireEvent.click(screen.getByRole('button'));

  expect(await screen.findByText('Hello')).toBeInTheDocument();
});
```

---

## Testing Loading States

Simulate delayed API responses to verify loading indicators.

```javascript
let resolvePromise;

global.fetch.mockReturnValueOnce(
  new Promise((resolve) => {
    resolvePromise = resolve;
  })
);

// Trigger component action

resolvePromise({
  ok: true,
  json: async () => ({ data: 'result' }),
});

// Verify rendered result
```

---

## Testing Error Handling

Mock failed API requests to validate error messages.

```javascript
global.fetch.mockRejectedValueOnce(
  new Error('Network error')
);

// Trigger component action

// Verify error message appears
```

---

## Backend Integration Testing

Use **Supertest** to test Express API endpoints.

```javascript
const request = require('supertest');
const app = require('../app');

test('GET /api/materials returns a list', async () => {
  const response = await request(app).get('/api/materials');

  expect(response.statusCode).toBe(200);
  expect(Array.isArray(response.body)).toBe(true);
});
```

---

## Backend Unit Testing

Mock Mongoose models to isolate business logic.

```javascript
const Message = require('../../models/Message');

jest.mock('../../models/Message');

test('saves a message successfully', async () => {
  Message.mockImplementation(() => ({
    save: jest.fn().mockResolvedValue({
      text: 'Hello',
    }),
  }));

  const message = new Message({
    text: 'Hello',
  });

  const saved = await message.save();

  expect(saved.text).toBe('Hello');
});
```

---

# Debugging Tests

## Using Console Output

```javascript
test('debug example', () => {
  const result = myFunction();

  console.log(result);

  expect(result).toBe(expected);
});
```

---

## Debugging the Rendered DOM

```javascript
import { render } from '@testing-library/react';

test('debug rendered component', () => {
  const { debug } = render(<MyComponent />);

  debug();
});
```

---

## Running Individual Tests

Run a specific test file:

```bash
npm test -- --testPathPattern="messages"
```

Run a test by its name:

```bash
npm test -- -t "creates a new profile"
```

---

# CI/CD Integration

BrainBytes integrates automated testing into its GitHub Actions workflow.

| **Pipeline Stage**           | **Purpose**                                                      |
| ---------------------------- | ---------------------------------------------------------------- |
| **Lint**                     | Performs code quality checks using ESLint.                       |
| **Unit & Integration Tests** | Executes Jest test suites and generates code coverage reports.   |
| **End-to-End Tests**         | Runs Playwright browser tests against the deployed application.  |
| **Coverage Reports**         | Uploads coverage reports as workflow artifacts for later review. |

To review test results:

1. Open the GitHub repository.
2. Navigate to the **Actions** tab.
3. Select the desired workflow run.
4. Review the logs, annotations, and uploaded coverage artifacts.
