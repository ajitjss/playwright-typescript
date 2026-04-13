# 🎭 Playwright TypeScript Framework

A robust end-to-end test automation framework built with [Playwright](https://playwright.dev/) and TypeScript.

---

## 👨‍💻 Author

**Ajeet Yadav**
📧 [ajit091@gmail.com](mailto:ajit091@gmail.com)
🔗 [github.com/ajeetyadawa/playwright-typescript](https://github.com/ajeetyadawa/playwright-typescript)

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Running Tests](#running-tests)
- [Writing Tests](#writing-tests)
- [Page Object Model](#page-object-model)
- [Reporting](#reporting)
- [CI/CD Integration](#cicd-integration)
- [Best Practices](#best-practices)

---

## ✅ Prerequisites

Make sure you have the following installed before setting up the project:

| Tool | Version | Download |
|------|---------|----------|
| Node.js | >= 18.x | [nodejs.org](https://nodejs.org/) |
| npm | >= 9.x | Comes with Node.js |
| Git | Latest | [git-scm.com](https://git-scm.com/) |

---

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/ajeetyadawa/playwright-typescript.git
cd playwright-typescript
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Install Playwright Browsers

```bash
npx playwright install
```

> To install only specific browsers:
> ```bash
> npx playwright install chromium
> npx playwright install firefox
> npx playwright install webkit
> ```

---

## 📁 Project Structure

```
playwright-typescript/
│
├── tests/                    # Test spec files
│   ├── example.spec.ts
│   └── login.spec.ts
│
├── pages/                    # Page Object Model classes
│   ├── BasePage.ts
│   └── LoginPage.ts
│
├── fixtures/                 # Custom fixtures and test setup
│   └── index.ts
│
├── utils/                    # Helper utilities
│   └── helpers.ts
│
├── test-data/                # Test data (JSON/CSV)
│   └── users.json
│
├── playwright.config.ts      # Playwright configuration
├── tsconfig.json             # TypeScript configuration
├── package.json
└── README.md
```

---

## ⚙️ Configuration

The `playwright.config.ts` file controls global test settings:

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 2 : undefined,
  reporter: [
    ['list'],
    ['html', { open: 'never' }],
  ],
  use: {
    baseURL: 'https://your-app-url.com',
    headless: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'Chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'Firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'WebKit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
});
```

### Environment Variables

Create a `.env` file in the root directory:

```env
BASE_URL=https://your-app-url.com
USERNAME=testuser@example.com
PASSWORD=yourpassword
```

---

## ▶️ Running Tests

### Run All Tests

```bash
npx playwright test
```

### Run Tests in Headed Mode (visible browser)

```bash
npx playwright test --headed
```

### Run Tests in Interactive UI Mode

```bash
npx playwright test --ui
```

### Run a Specific Test File

```bash
npx playwright test tests/login.spec.ts
```

### Run Tests Matching a Name

```bash
npx playwright test -g "login with valid credentials"
```

### Run Tests on a Specific Browser

```bash
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit
```

### Run Tests in Debug Mode

```bash
npx playwright test --debug
```

### Run Tests with a Specific Number of Workers

```bash
npx playwright test --workers=4
```

### Retry Failed Tests

```bash
npx playwright test --retries=2
```

---

## ✍️ Writing Tests

### Basic Test Example

```ts
import { test, expect } from '@playwright/test';

test.describe('Homepage', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should display the page title', async ({ page }) => {
    await expect(page).toHaveTitle(/My App/);
  });

  test('should navigate to login page', async ({ page }) => {
    await page.getByRole('link', { name: 'Login' }).click();
    await expect(page).toHaveURL('/login');
  });
});
```

### Recommended Locator Strategies

```ts
// ✅ Preferred — role-based
page.getByRole('button', { name: 'Submit' })

// ✅ Preferred — label-based
page.getByLabel('Email Address')

// ✅ Preferred — placeholder
page.getByPlaceholder('Enter your email')

// ✅ Preferred — text content
page.getByText('Welcome back')

// ✅ Reliable — test IDs
page.getByTestId('submit-btn')

// ⚠️ Fragile — avoid when possible
page.locator('.css-class')
page.locator('//xpath')
```

### Common Assertions

```ts
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveTitle('My App');
await expect(locator).toBeVisible();
await expect(locator).toBeEnabled();
await expect(locator).toHaveText('Hello World');
await expect(locator).toHaveValue('typed value');
await expect(locator).toHaveCount(5);
await expect(locator).toBeChecked();
```

---

## 🗂️ Page Object Model

The framework uses the **Page Object Model (POM)** pattern to keep tests clean and maintainable.

### Creating a Page Class

```ts
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput    = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton  = page.getByRole('button', { name: 'Login' });
    this.errorMessage  = page.getByTestId('error-msg');
  }

  async navigate() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async verifyErrorMessage(message: string) {
    await expect(this.errorMessage).toHaveText(message);
  }
}
```

### Using a Page Class in Tests

```ts
// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test.describe('Login Tests', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.navigate();
  });

  test('should login with valid credentials', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123');
    await expect(page).toHaveURL('/dashboard');
  });

  test('should show error for invalid credentials', async () => {
    await loginPage.login('wrong@example.com', 'wrongpass');
    await loginPage.verifyErrorMessage('Invalid email or password');
  });
});
```

---

## 📊 Reporting

### View HTML Report

```bash
npx playwright show-report
```

### Generate Report After Test Run

The HTML report is automatically generated in the `playwright-report/` directory after each test run.

### Other Reporter Options

```ts
// playwright.config.ts
reporter: [
  ['list'],                              // Console output
  ['html', { open: 'never' }],          // HTML report
  ['json', { outputFile: 'results.json' }],  // JSON output
  ['junit', { outputFile: 'results.xml' }],  // JUnit for CI
],
```

---

## 🔄 CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps

      - name: Run Playwright Tests
        run: npx playwright test

      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## 💡 Best Practices

- **Use Page Object Model** — keep locators and actions out of test files
- **Prefer role-based locators** — `getByRole`, `getByLabel` over CSS/XPath
- **Use `test.describe` blocks** — group related tests logically
- **Use `test.beforeEach`** — set up shared state without duplication
- **Avoid hard waits** (`page.waitForTimeout`) — use Playwright's auto-waiting instead
- **Use `data-testid` attributes** — coordinate with developers for stable selectors
- **Store auth state** — use `storageState` to avoid repeated logins
- **Isolate tests** — each test should be independent and not rely on another's state
- **Run tests in parallel** — Playwright supports parallelism out of the box
- **Use environment variables** — never hardcode credentials in test files

---

## 📦 Useful Scripts (`package.json`)

```json
{
  "scripts": {
    "test": "npx playwright test",
    "test:headed": "npx playwright test --headed",
    "test:ui": "npx playwright test --ui",
    "test:debug": "npx playwright test --debug",
    "test:report": "npx playwright show-report",
    "test:chromium": "npx playwright test --project=chromium",
    "test:firefox": "npx playwright test --project=firefox",
    "test:webkit": "npx playwright test --project=webkit"
  }
}
```

---

## 📚 Resources

- [Playwright Official Docs](https://playwright.dev/docs/intro)
- [Playwright API Reference](https://playwright.dev/docs/api/class-playwright)
- [TypeScript Docs](https://www.typescriptlang.org/docs/)
- [Playwright GitHub](https://github.com/microsoft/playwright)

---

## 📄 License

This project is licensed under the MIT License.

---

> **Questions or Issues?**
> Reach out to **Ajeet Yadav** at [ajit091@gmail.com](mailto:ajit091@gmail.com) or open an [issue on GitHub](https://github.com/ajeetyadawa/playwright-typescript/issues).