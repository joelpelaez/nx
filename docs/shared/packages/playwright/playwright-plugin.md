Playwright is a modern web test runner. With included features such as:

- Cross browser support, including mobile browsers
- Multi tab, origin, and user support
- Automatic waiting
- Test generation
- Screenshots and videos

## Setting Up Playwright

If the `@nx/playwright` package is not installed, install the version that matches your `nx` package version.

{% tabs %}
{% tab label="npm" %}

```shell
npm install --save-dev @nx/playwright
```

{% /tab %}
{% tab label="yarn" %}

```shell
yarn add --dev @nx/playwright
```

{% /tab %}
{% tab label="pnpm" %}

```shell
pnpm i -D @nx/playwright
```

{% /tab %}
{% /tabs %}

## E2E Testing

By default, when creating a new frontend application, Nx will prompt for which e2e test runner to use. Select `playwright` or pass in the arg `--e2eTestRunner=playwright`

```shell
nx g @nx/web:app frontend --e2eTestRunner=playwright
```

### Add Playwright e2e to an existing project

To generate an E2E project for an existing project, run the following generator

```shell
nx g @nx/playwright:configuration --project=your-app-name
```

Optionally, you can use the `--webServerCommand` and `--webServerAddress` option, to auto setup the [web server option](https://playwright.dev/docs/test-webserver) in the playwright config

```shell
nx g @nx/playwright:configuration --project=your-app-name --webServerCommand="npx serve your-project-name" --webServerAddress="http://localhost:4200"
```

### Testing Applications

Run `nx e2e <your-app-name>` to execute e2e tests with Playwright

{% callout type="note" title="Selecting Specific Specs" %}

You can use the `--grep/-g` flag to glob for test files.
You can use the `--grepInvert/-gv` flag to glob for files to _not_ run.

```bash
# run the tests in the feat-a/ directory
nx e2e frontend-e2e --grep="**feat-a/**"

# run everything except feat-a/ directory
nx e2e frontend-e2e --grepInvert=**feat-a/**
```

{% /callout %}

By default, Playwright will run in headless mode. You will have the result of all the tests and errors (if any) in your
terminal. Test output such as reports, screenshots and videos, will be accessible in `dist/.playwright/apps/<your-app-name>/`. This can be configured with the `outputDir` configuration options.

{% callout type="note" title="Output Caching" %}
If changing the output directory or report output, make sure to update the [target outputs](/concepts/how-caching-works#what-is-cached) so the artifacts are correctly cached
{% /callout %}

### Watching for Changes

With, `nx e2e frontend-e2e --ui` Playwright will start in headed mode where you can see your application being tested.

From, there you can toggle on the watch icon which will rerun the tests when the spec file updates.

```shell
nx e2e <your-app-name> --ui
```

You can also use `--headed` flag to run Playwright where the browser can be seen without using the [Playwright UI](https://playwright.dev/docs/test-ui-mode)

### Specifying a Base Url

The `baseURL` property within the Playwright configuration can control where the tests visit by default.

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // Rest of your config...

  // Run your local dev server before starting the tests
  webServer: {
    command: 'npx serve <your-app-name>',
    url: 'http://localhost:4200',
    reuseExistingServer: !process.env.CI,
  },
  use: {
    baseURL: 'http://localhost:4200', // url playwright visits with `await page.goto('/')`;
  },
});
```

In order to set different `baseURL` values for different environments you can pass them via the [environment variables and nx configurations](/recipes/tips-n-tricks/define-environment-variables) or optionally via setting them per the environment they are needed in such as `CI`

```ts
import { defineConfig } from '@playwright/test';

const baseUrl =
  process.env.BASE_URL ?? process.env.CI
    ? 'https://some-staging-url.example.com'
    : 'http://localhost:4200';

export default defineConfig({
  // Rest of your config...

  // Run your local dev server before starting the tests
  webServer: {
    command: 'npx serve <your-app-name>',
    url: baseUrl,
    reuseExistingServer: !process.env.CI,
  },
  use: {
    baseURL: baseUrl, // url playwright visits with `await page.goto('/')`;
  },
});
```

By default Nx, provides a `nxE2EPreset` with predefined configuration for Playwright.

```ts
import { defineConfig } from '@playwright/test';
import { nxE2EPreset } from '@nx/playwright/preset';
import { workspaceRoot } from '@nx/devkit';

// For CI, you may want to set BASE_URL to the deployed application.
const baseURL = process.env['BASE_URL'] || 'http://localhost:4200';

/**
 * Read environment variables from file.
 * https://github.com/motdotla/dotenv
 */
// require('dotenv').config();

/**
 * See https://playwright.dev/docs/test-configuration.
 */
export default defineConfig({
  ...nxE2EPreset(__filename, { testDir: './e2e' }),
  /* Shared settings for all the projects below. See https://playwright.dev/docs/api/class-testoptions. */
  use: {
    baseURL,
    /* Collect trace when retrying the failed test. See https://playwright.dev/docs/trace-viewer */
    trace: 'on-first-retry',
  },
  /* Run your local dev server before starting the tests */
  webServer: {
    command: 'npx nx serve <your-app-name>',
    url: baseURL,
    reuseExistingServer: !process.env.CI,
    cwd: workspaceRoot,
  },
});
```

This preset sets up the `outputDir` and [HTML reporter](https://playwright.dev/docs/test-reporters#html-reporter) to output in `dist/.playwright/<path-to-project-root>` and sets up chromium, firefox, webkit browsers to be used a browser targets. If you want to use mobile and/or branded browsers you can pass those options into the preset function

```ts
export default defineConfig({
  ...nxE2EPreset(__filename, {
    testDir: './e2e',
    includeMobileBrowsers: true, // includes mobile Chrome and Safari
    includeBrandedBrowsers: true, // includes Google Chrome and Microsoft Edge
  }),
  // other settings
});
```

If you want to override any settings within the `nxE2EPreset`, You can define them after the preset like so

```ts
const config = nxE2EPreset(__filename, {
  testDir: './e2e',
  includeMobileBrowsers: true, // includes mobile Chrome and Safari
  includeBrandedBrowsers: true, // includes Google Chrome and Microsoft Edge
});
export default defineConfig({
  ...config
  retries: 3,
  reporters: [...config.reporters, /* other reporter settings */],
});
```

See the [Playwright configuration docs](https://playwright.dev/docs/test-configuration) for more options for Playwright.