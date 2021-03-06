# Bamboo Mocha Jest Code Coverage
## Convert Jest Code Coverage Reports for Atlassian's Bamboo and Mocha Test Parser

Bamboo Mocha Jest Code Coverage is a utility that converts the JSON Summary generated by Jest's code coverage into a JSON object that Bamboo's Mocha Test Parser task can understand.

This lets developers using Atlassian's Bamboo to make code coverage part of a build's stages and jobs. If code coverage thresholds are not met then the tests fail, resulting in a failed build.

If you are using the Atlassian suite with BitBucket server, then a failed build will prevent a pull request from being merged and prevent changes that drop the code coverage below acceptable thresholds.

### Note

I found tools that convert Jest automated tests to Mocha Test Parser format, but was unable to find a tool that converts Jest's code coverage into Mocha Test Parser format. This utility is something custom built that allowed us to implement our code coverage into our Bamboo jobs.

I tried to make it reusable enough to be used across multiple projects and open sourced it so other developers who may also want to use Jest's code coverage in their Bamboo jobs can do. That being said, I know there are opportunities for further configuration and customization available and encourage anyone who makes customizations or configurations to submit a pull request.

## Set Up

Install `bamboo-jest-mocha-parser` in your project:

```
npm install bamboo-mocha-jest-code-coverage --save-dev
```

### Configure Jest

Make sure you have your Jest code coverage configuration in package.json. By default, Jest outputs code coverage as text, json, and lcov. We will need to add a fourth format: json-summary. The json-summary is what Bamboo Mocha Jest Code Coverage parses for Bamboo's Mocha Test Parser.

Example Jest configuration:
```
"jest": {
  "collectCoverage": true,
  "coverageDirectory": "coverage",
  "collectCoverageFrom": "app/**/*.jsx",
  "coverageReporters": [
    "text",
    "json",
    "json-summary",
    "lcov"
  ],
  "coverageThreshold": {
    "global": {
      "branches": 100,
      "functions": 100,
      "lines": 90,
      "statements": 100
    }
  }
}
```

### Create NPM Script

Create an NPM script for Bamboo to run that executes Jest and then Bamboo Jest Mocha Parser.

```
"bamboo:coverage": "jest || true && bamboo-jest-coverage-parser"
```

*NOTE:* Jest exits with `Exit status 1` if code coverage thresholds are not met. There is currently **no way** to suppress this or force Jest to exit without `Exit status 1`.

`Exit status 1` causes NPM to abort, meaning if your code coverage **does not** meet thresholds, Bamboo Mocha Jest Code Coverage will *not* be executed. Executing `jest || true` lets the script continue, even if Jest exits with an error.

This introduces risk, as **any** error will be ignored (not just failed code coverage thresholds). However, until there is a way to suppress `Exit status 1` from Jest's code coverage, or a way to format code coverage results like test results, this is the only way to chain the command together for Bamboo.

### Configure Bamboo

Now configure a Bamboo job/task that executes the NPM script `bamboo:coverage`.

After Jest is finished, Bamboo Mocha Jest Code Coverage will convert the `summary-json.json` file into `coverage.json` at the root of the project.

Use Bamboo's Mocha Test Parser to parse `coverage.json`.

Now your test coverage results are available via Bamboo!
