# Testing Guidelines

## Overview

This project emphasizes **Test-Driven Development (TDD)** as a core workflow pattern. Tests are not just validation tools—they're your roadmap for implementation and your safety net for refactoring.

## Testing Scope

This project focuses on **unit tests** and **integration tests**:

- ✅ **Backend Unit Tests**: Testing individual functions and API endpoints with Jest
- ✅ **Backend Integration Tests**: Testing API routes end-to-end with Supertest
- ✅ **Frontend Component Tests**: Testing React components with React Testing Library
- ✅ **Manual Browser Testing**: Verifying UI features work in real browsers

We do **NOT** include:

- ❌ **E2E Browser Automation**: No Playwright, Cypress, or Selenium
- ❌ **Full Application E2E Tests**: Manual testing serves this purpose

**Why this scope?** This keeps the lab focused on TDD principles and workflow automation without the complexity of e2e test infrastructure setup, flakiness management, and maintenance overhead.

## Testing Philosophy

### Tests Should Drive Development

1. **Read the test first** to understand requirements
2. **Run the test** to see it fail (Red)
3. **Implement** minimal code to pass (Green)
4. **Refactor** to improve quality
5. **Repeat** with the next test

### Tests Provide Specification

Each test describes:

- **What** the code should do
- **How** it should behave
- **When** it should succeed or fail
- **Why** certain decisions were made

## Backend Testing Strategy

### Test Structure

We use **Jest** and **Supertest** for API testing:

```javascript
describe('Feature Group', () => {
  describe('Specific Endpoint', () => {
    test('should behave in expected way', async () => {
      // Arrange: Set up test data
      // Act: Call the API
      // Assert: Verify results
    });
  });
});
```

### Test Categories

#### 1. Happy Path Tests

Test expected behavior with valid inputs:

```javascript
test('should create a new todo with valid title', async () => {
  const response = await request(app)
    .post('/api/todos')
    .send({ title: 'Valid Todo' });

  expect(response.status).toBe(201);
  expect(response.body).toHaveProperty('id');
});
```

#### 2. Validation Tests

Test error handling with invalid inputs:

```javascript
test('should return 400 when title is missing', async () => {
  const response = await request(app).post('/api/todos').send({});

  expect(response.status).toBe(400);
  expect(response.body).toHaveProperty('error');
});
```

#### 3. Edge Case Tests

Test boundary conditions:

```javascript
test('should handle empty string title', async () => {
  const response = await request(app).post('/api/todos').send({ title: '' });

  expect(response.status).toBe(400);
});
```

#### 4. Integration Tests

Test multiple operations together:

```javascript
test('should handle full CRUD lifecycle', async () => {
  // Create
  const created = await request(app).post('/api/todos').send({ title: 'Test' });

  // Update
  const updated = await request(app)
    .put(`/api/todos/${created.body.id}`)
    .send({ title: 'Updated' });

  // Verify update
  expect(updated.body.title).toBe('Updated');

  // Delete
  await request(app).delete(`/api/todos/${created.body.id}`);

  // Verify deletion
  const todos = await request(app).get('/api/todos');
  expect(todos.body).not.toContainEqual(
    expect.objectContaining({ id: created.body.id })
  );
});
```

## Frontend Testing Strategy

### Test Structure

We use **React Testing Library** with **Material-UI** components and **React Query**:

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// Create a test query client
const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  });

test('should render todo items', () => {
  const testQueryClient = createTestQueryClient();

  render(
    <QueryClientProvider client={testQueryClient}>
      <App />
    </QueryClientProvider>
  );

  const element = screen.getByText(/expected text/i);
  expect(element).toBeInTheDocument();
});
```

### Testing Material-UI Components

MUI components work seamlessly with React Testing Library. Use semantic queries:

```javascript
// Query MUI buttons by role
const addButton = screen.getByRole('button', { name: /add/i });

// Query MUI textfields by label or placeholder
const input = screen.getByPlaceholderText(/what needs to be done/i);

// Query MUI checkboxes
const checkbox = screen.getByRole('checkbox');

// Fire events on MUI components
fireEvent.click(addButton);
fireEvent.change(input, { target: { value: 'New todo' } });
```

### Testing Principles

#### 1. Test User Behavior, Not Implementation

❌ **Don't:**

```javascript
test('state updates correctly', () => {
  const wrapper = shallow(<Component />);
  wrapper.instance().setState({ value: 'test' });
  expect(wrapper.state().value).toBe('test');
});
```

✅ **Do:**

```javascript
test('displays updated value when user types', () => {
  render(<Component />);
  const input = screen.getByRole('textbox');
  fireEvent.change(input, { target: { value: 'test' } });
  expect(screen.getByDisplayValue('test')).toBeInTheDocument();
});
```

#### 2. Query by Accessibility

Prefer queries that reflect how users interact:

```javascript
// Best: By role (most accessible)
screen.getByRole('button', { name: /add/i });

// Good: By label text
screen.getByLabelText('Todo title');

// OK: By placeholder
screen.getByPlaceholderText('Enter todo...');

// Last resort: By test ID
screen.getByTestId('todo-item');
```

## Running Tests

### Run All Tests

```bash
npm test
```

### Run Tests in Watch Mode

```bash
npm run test:watch
```

### Run Specific Test File

```bash
npm test -- app.test.js
```

### Run Specific Test by Name

```bash
npm test -- --testNamePattern="should create a new todo"
```

### Run with Coverage

```bash
npm test -- --coverage
```

## Test-Driven Workflow with Copilot

### Step 1: Understand the Test

```
You: "Explain what this test expects:
[paste test code]"

Copilot: "This test verifies that..."
```

### Step 2: Implement to Pass

```
You: "Implement the POST /api/todos endpoint to pass this test:
[paste test code]"

Copilot: [Provides implementation]
```

### Step 3: Verify

```bash
npm test -- --testNamePattern="should create a new todo"
```

### Step 4: Handle Failures

```
Test output:
  Expected: 201
  Received: 501

You: "The test expects 201 but returns 501. What's wrong with this code:
[paste current implementation]"

Copilot: "The endpoint returns 501 'Not implemented'. You need to..."
```

### Step 5: Iterate

Continue until the test passes, then move to the next test.

## Common Testing Patterns

### Testing Async Operations

```javascript
test('should fetch todos on mount', async () => {
  render(<App />);

  // Wait for async operation
  const todos = await screen.findByText(/first todo/i);

  expect(todos).toBeInTheDocument();
});
```

### Mocking API Calls

```javascript
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve([{ id: 1, title: 'Test' }]),
  })
);

test('should display fetched todos', async () => {
  render(<App />);
  await screen.findByText('Test');
  expect(fetch).toHaveBeenCalledWith('/api/todos');
});
```

### Testing Error States

```javascript
test('should display error message on fetch failure', async () => {
  global.fetch = jest.fn(() => Promise.reject('API Error'));

  render(<App />);

  const errorMessage = await screen.findByText(/error/i);
  expect(errorMessage).toBeInTheDocument();
});
```

## Debugging Failing Tests

### 1. Read the Error Message Carefully

```
Expected: 201
Received: 400

This tells you:
- The test expected a success status (201)
- The API returned a client error (400)
- Check validation logic
```

### 2. Use Console Logs Strategically

```javascript
test('debugging example', async () => {
  const response = await request(app)
    .post('/api/todos')
    .send({ title: 'Test' });

  console.log('Response:', response.status, response.body);
  // Now you can see what's actually being returned

  expect(response.status).toBe(201);
});
```

### 3. Ask Copilot for Help

```
You: "This test is failing with this error: [paste error]
Here's the test: [paste test]
Here's my implementation: [paste code]
What's wrong?"
```

### 4. Isolate the Problem

Run just the failing test:

```bash
npm test -- --testNamePattern="specific failing test"
```

### 5. Check Test Assumptions

Verify:

- Is the test correct?
- Are imports working?
- Is the endpoint path correct?
- Is data persisting between tests?

## Test Organization Best Practices

### Group Related Tests

```javascript
describe('POST /api/todos', () => {
  describe('validation', () => {
    test('requires title', ...);
    test('rejects empty title', ...);
  });

  describe('successful creation', () => {
    test('creates with valid data', ...);
    test('auto-generates ID', ...);
  });
});
```

### Use Descriptive Test Names

❌ **Vague:**

```javascript
test('works', ...);
test('handles error', ...);
```

✅ **Clear:**

```javascript
test('should create todo with valid title', ...);
test('should return 400 when title is missing', ...);
```

### Keep Tests Independent

Each test should:

- Set up its own data
- Clean up after itself
- Not rely on other tests

## Success Criteria

You understand testing when you can:

- ✅ Read a test and explain what it verifies
- ✅ Run tests and interpret failure messages
- ✅ Implement code to make failing tests pass
- ✅ Write new tests for new features
- ✅ Use tests to catch regressions during refactoring
- ✅ Debug failing tests systematically

## Remember

> "Tests are not just about finding bugs—they're about confidence. When all
> tests pass, you know your code works as intended. When you refactor, tests
> ensure you haven't broken anything. Tests are your safety net."

Happy testing! 🧪
