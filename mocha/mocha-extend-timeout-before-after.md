# Extending the Timeout for before() and after()

## Do not use arrow function in `describe`

To give the entire test file (including the before() hook, S0100.1, S0100.2, and after()) enough time, follow the pattern used in 
S1100-InstagramTestPrefix.spec.ts
:

Use a standard function (not an arrow function) for describe and set this.timeout(...) at the suite level:

```
// NOTE: This uses function() so that the global timeout can be applied
// for before and after hooks, as well as for each test case. Using arrow
// functions would not allow this.timeout() to work properly.

describe("S0100 - Test the Test Setup Testing", function () {
  const TEST_TIMEOUT = 60 * 1000;
  this.timeout(TEST_TIMEOUT);
  ```

