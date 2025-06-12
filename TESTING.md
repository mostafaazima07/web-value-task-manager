# Testing Guide

## Overview

This document outlines testing procedures and guidelines for Web Value Task Flow. We use PHPUnit for unit tests, Cypress for end-to-end testing, and Jest for JavaScript testing.

## Test Structure

```
tests/
├── Unit/
│   ├── Controllers/
│   ├── Models/
│   └── Services/
├── Integration/
│   ├── API/
│   ├── Database/
│   └── Services/
├── Feature/
│   ├── Auth/
│   ├── Tasks/
│   └── Users/
└── Browser/
    ├── Auth/
    ├── Dashboard/
    └── Tasks/
```

## Setting Up Test Environment

1. Install dependencies:
```bash
composer install --dev
npm install --dev
```

2. Configure test database:
```bash
cp config.example.php config.test.php
mysql -u root -p
CREATE DATABASE task_flow_test;
```

3. Set up test environment:
```bash
cp .env.example .env.testing
php setup-test-environment.php
```

## Running Tests

### Unit Tests

```bash
# Run all unit tests
./vendor/bin/phpunit tests/Unit

# Run specific test file
./vendor/bin/phpunit tests/Unit/Controllers/TaskControllerTest.php

# Run with coverage report
./vendor/bin/phpunit --coverage-html coverage
```

### Integration Tests

```bash
# Run all integration tests
./vendor/bin/phpunit tests/Integration

# Run API tests
./vendor/bin/phpunit tests/Integration/API
```

### End-to-End Tests

```bash
# Start test server
php -S localhost:8000

# Run Cypress tests
npx cypress run

# Open Cypress UI
npx cypress open
```

### JavaScript Tests

```bash
# Run Jest tests
npm test

# Run with watch mode
npm test -- --watch
```

## Writing Tests

### Unit Test Example

```php
class TaskTest extends TestCase
{
    public function testTaskCreation()
    {
        $data = [
            'title' => 'Test Task',
            'description' => 'Test Description',
            'due_date' => '2024-01-31'
        ];

        $task = new Task($data);
        
        $this->assertEquals('Test Task', $task->title);
        $this->assertEquals('Test Description', $task->description);
        $this->assertEquals('2024-01-31', $task->due_date);
    }

    public function testTaskValidation()
    {
        $this->expectException(ValidationException::class);
        
        $data = [
            'title' => '', // Invalid: empty title
            'due_date' => '2024-01-31'
        ];

        new Task($data);
    }
}
```

### Integration Test Example

```php
class TaskAPITest extends TestCase
{
    public function testTaskCreationAPI()
    {
        $response = $this->post('/api/tasks', [
            'title' => 'API Test Task',
            'description' => 'Created via API',
            'due_date' => '2024-01-31'
        ]);

        $response->assertStatus(201)
                 ->assertJson([
                     'title' => 'API Test Task'
                 ]);

        $this->assertDatabaseHas('tasks', [
            'title' => 'API Test Task'
        ]);
    }
}
```

### E2E Test Example

```javascript
describe('Task Management', () => {
    beforeEach(() => {
        cy.login('admin@example.com', 'password');
    });

    it('creates a new task', () => {
        cy.visit('/tasks/create');
        cy.get('#title').type('E2E Test Task');
        cy.get('#description').type('Created via E2E test');
        cy.get('#due_date').type('2024-01-31');
        cy.get('button[type="submit"]').click();
        
        cy.url().should('include', '/tasks');
        cy.contains('E2E Test Task');
    });
});
```

## Test Coverage

Minimum coverage requirements:
- Unit Tests: 80%
- Integration Tests: 70%
- Feature Tests: 60%
- Overall Coverage: 75%

Generate coverage report:
```bash
./vendor/bin/phpunit --coverage-html coverage
```

## Mocking

### Database Mocking

```php
class TaskServiceTest extends TestCase
{
    public function testTaskRetrieval()
    {
        $mockDB = $this->createMock(Database::class);
        $mockDB->expects($this->once())
               ->method('query')
               ->willReturn([
                   ['id' => 1, 'title' => 'Mock Task']
               ]);

        $service = new TaskService($mockDB);
        $tasks = $service->getAllTasks();
        
        $this->assertCount(1, $tasks);
    }
}
```

### API Mocking

```javascript
describe('API Client', () => {
    it('handles API errors', async () => {
        cy.intercept('GET', '/api/tasks', {
            statusCode: 500,
            body: {
                error: 'Server Error'
            }
        }).as('getTasks');

        cy.visit('/dashboard');
        cy.wait('@getTasks');
        cy.contains('Error loading tasks');
    });
});
```

## Continuous Integration

### GitHub Actions Workflow

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.1'
          
      - name: Install Dependencies
        run: composer install
        
      - name: Run Tests
        run: ./vendor/bin/phpunit
```

## Performance Testing

### Load Testing

```bash
# Using k6
k6 run load-tests/tasks-api.js

# Using Apache Benchmark
ab -n 1000 -c 10 http://localhost:8000/api/tasks
```

### Load Test Script Example

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export default function() {
    const res = http.get('http://localhost:8000/api/tasks');
    check(res, {
        'status is 200': (r) => r.status === 200,
        'response time < 200ms': (r) => r.timings.duration < 200
    });
    sleep(1);
}
```

## Security Testing

### OWASP ZAP Integration

```bash
# Run security scan
zap-cli quick-scan --self-contained \
    --start-options "-config api.disablekey=true" \
    http://localhost:8000
```

## Test Data

### Factories

```php
class TaskFactory
{
    public static function create(array $attributes = [])
    {
        return array_merge([
            'title' => 'Test Task',
            'description' => 'Test Description',
            'due_date' => '2024-01-31',
            'status' => 'pending'
        ], $attributes);
    }
}
```

### Seeders

```php
class TestDatabaseSeeder
{
    public function run()
    {
        $this->call([
            UserSeeder::class,
            TaskSeeder::class,
            CommentSeeder::class
        ]);
    }
}
```

## Debugging Tests

### PHPUnit

```bash
# Run with debug output
./vendor/bin/phpunit --debug

# Run specific test with verbose output
./vendor/bin/phpunit --filter testTaskCreation -v
```

### Cypress

```javascript
// Add debugging pause
cy.pause();

// Log to console
cy.log('Debug message');

// Screenshot
cy.screenshot('debug-state');
```

## Best Practices

1. Follow AAA pattern (Arrange, Act, Assert)
2. One assertion per test when possible
3. Use meaningful test names
4. Keep tests independent
5. Clean up test data
6. Use appropriate mocks
7. Test edge cases
8. Maintain test documentation

## Support

For testing support:
- Email: testing@webvalue.com
- Documentation: /docs/testing
- Issue Tracker: GitHub Issues
