+++
title = "Cypress with TypeScript"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Install Cypress with npm

From terminal:

```bash
> npm init -y
> npm install cypress --save-dev
```

Expect package.json file to be created an contain the following:

```cli
> cat package.json

{
  "devDependencies": {
    "cypress": "^13.15.0",
  },
  "dependencies": {
  }
}
```

## Open Cypress from CLI

```bash
> npx cypress open
```

## Using TypeScript with Cypress

Information may be found on [Cypress TypeScript] documentation page.

### Dependencies installation

Install TypeScript locally:

```bash
npm install typescript --save-dev
```

Expect the following:

```cli
> cat package.json

{
  "devDependencies": {
    "cypress": "^13.15.0",
    "typescript": "^5.6.3"    
  },
  "dependencies": {
  }
}
```

Optionally, install the ESLinter for TypesSript:

```bash
> npm install eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev
> npx eslint --init
```

Pick the right configuration:

```text
✔ How would you like to use ESLint? · problems
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · none
✔ Does your project use TypeScript? · typescript
✔ Where does your code run? · browser
The config that you've selected requires the following dependencies:

eslint, globals, @eslint/js, typescript-eslint
✔ Would you like to install them now? · Yes
✔ Which package manager do you want to use? · npm
```

Install Cypress Plugin:
```bash
> npm install eslint-plugin-cypress --save-dev
```



### Configure TypeScript tsconfig.ts File

Configure TypeScript for Cypress:

From the cypress directory, run the following command.

```bash
> npx tsc --init --types cypress,node,jquery --lib dom,es6
```

Expect a `tsconfig.json` file to be created at the project's `cypress` directory:

```cli
Created a new tsconfig.json with:                                                  target: es2016
  module: commonjs
  lib: dom,es6
  strict: true
  types: cypress
  esModuleInterop: true
  skipLibCheck: true
  forceConsistentCasingInFileNames: true
```

Note: the `tsconfig.json` should be in the `cypress` directory, therefore it should be moved if it was created at the project root level.

```json
{
  "compilerOptions": {
    "types": ["cypress", "jquery"],
    "target": "es6",
    "lib": ["es6", "dom"],  // Ensure DOM types are included
    "moduleResolution": "node",
    "esModuleInterop": true
  },
  "include": ["cypress/**/*.ts", "cypress/**/*.d.ts"]  // Ensure it includes Cypress TypeScript files
}
```

## ESLint Installation for Cypress and TypeScript

You’ll need to install the following npm packages to configure ESLint for TypeScript and Cypress:

Core ESLint Packages:

- eslint: The core ESLint package.
- @eslint/js: JavaScript-specific ESLint configurations (recommended rules).
- @typescript-eslint/parser: ESLint parser for TypeScript.
- @typescript-eslint/eslint-plugin: Plugin for TypeScript-specific ESLint rules.
- eslint-plugin-cypress: ESLint plugin for Cypress-specific rules.

Globals Package:

- globals: Provides global variables for environments like browsers, Node.js, Cypress, Mocha, etc.

```bash
> npm install eslint @eslint/js @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-cypress globals --save-dev
```

Installation command:
```bash
> npm install eslint @eslint/js @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-cypress globals --save-dev
```

ESLint Configuration File:

```json
import globals from 'globals';
import pluginJs from '@eslint/js';
import tseslint from '@typescript-eslint/eslint-plugin';  // Import TypeScript plugin
import tsParser from '@typescript-eslint/parser';  // Use TypeScript parser
import cypressPlugin from 'eslint-plugin-cypress';  // Import Cypress plugin

export default [
  // Apply to general JavaScript and TypeScript files
  {
    files: ['**/*.{js,mjs,cjs,ts,tsx}'],  // Apply to all JS and TS files
    languageOptions: {
      parser: tsParser,  // Use TypeScript parser for TS files
      globals: {
        ...globals.browser,  // Include browser-specific global variables
        ...globals.node,     // Include Node.js globals
        JQuery: 'readonly',  // Add JQuery global for Cypress files
      },
    },
    plugins: {
      '@typescript-eslint': tseslint,
    },
    rules: {
      'quotes': ['error', 'single'],  // Enforce single quotes
      'semi': ['error', 'always'],    // Enforce semicolons
      '@typescript-eslint/no-unused-vars': 'error',  // Disallow unused variables
    },
  },

  // Apply specifically to Cypress test files
  {
    files: ['**/*.cy.{js,ts,tsx}', 'cypress/**/*.{js,ts,tsx}'],  // Target Cypress test files
    languageOptions: {
      parser: tsParser,  // Use TypeScript parser for Cypress files
      globals: {
        ...globals.browser,  // Include browser-specific global variables
        cy: 'readonly',  // Explicitly define 'cy' as a global variable
        Cypress: 'readonly',  // Explicitly define 'Cypress' as a global variable
        ...globals.cypress,  // Include Cypress globals (cy, Cypress, etc.)
        ...globals.mocha,  // Include Mocha globals (describe, it, beforeEach, etc.)
        JQuery: 'readonly',  // Add JQuery global for Cypress files
      },
    },
    plugins: {
      '@typescript-eslint': tseslint,
      'cypress': cypressPlugin,  // Enable Cypress plugin
    },
    rules: {
      'quotes': ['error', 'single'],  // Enforce single quotes
      'semi': ['error', 'always'],    // Enforce semicolons
      '@typescript-eslint/no-unused-vars': 'error',  // Disallow unused variables
      'cypress/no-unnecessary-waiting': 'error',  // Enforce Cypress rule
      'cypress/assertion-before-screenshot': 'warn',  // Enforce Cypress rule
    },
  },

  pluginJs.configs.recommended,  // Recommended JavaScript rules
  {
    rules: tseslint.configs.recommended.rules,  // Include recommended TypeScript rules
  },
];
```

ESLint Commands:

```bash
> npx eslint '**/*.cy.{ts,js,tsx}'
> npx eslint '**/*.{ts,js,mjs,tsx}'
```

Adding Eslint to the package.json scripts section:

The following must be added to the 'package.json' (comments are for doc only, please remove them):

```json
  "scripts": {
    "lint": "eslint '**/*.{js,ts,tsx}'",   // Lint all JS/TS files
    "lint:fix": "eslint '**/*.{js,ts,tsx}' --fix",  // Fix linting issues
    "lint:cypress": "eslint 'cypress/**/*.{js,ts,tsx}'"  // Lint only Cypress test files
  },
```

The npm command to run it:

```bash
> npm run lint
> npm run lint:fix
> npm run lint:cypress
```

<!-- References and links -->

[Cypress TypeScript]: https://docs.cypress.io/guides/tooling/typescript-support

