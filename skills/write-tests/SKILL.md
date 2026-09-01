# Write Tests

1. Inspect the existing test suite structure and discover the project's test
   framework and commands. Never assume a particular language or framework.
   Use semble to locate test files and framework setup efficiently:

   ```bash
   semble search "test setup configuration framework" . --top-k 5
   semble search "describe it expect assert test" . --top-k 5
   ```

2. Before writing tests, identify the observable behaviour that must be
   preserved or introduced.

3. Build a test matrix covering:
   - normal/expected behaviour
   - boundary conditions
   - invalid inputs and error behaviour
   - important state transitions
   - regression cases
   - interactions with external systems where relevant

4. Prefer tests that verify behaviour through public interfaces rather than
   implementation details.

5. Prefer the simplest test level that can meaningfully verify the behaviour:
   unit tests for isolated logic, integration tests for component boundaries,
   and end-to-end tests for critical user-visible workflows.

6. Keep tests deterministic, independent, fast, and readable.

7. Avoid mocking merely to make a test possible. Use real collaborators when
   they are cheap and deterministic; use fakes/stubs/mocks when isolation or
   control is genuinely valuable.

8. Do not write tests merely to increase coverage. Every test should protect
   meaningful behaviour or prevent a plausible regression.

9. After writing tests, run them. If they fail, determine whether the
   implementation or the test is wrong rather than weakening the assertion
   to make it pass.

10. Review the resulting tests for false confidence:
    - Could the implementation be wrong while these tests still pass?
    - Are assertions specific enough?
    - Are important behaviours missing?
    - Is the test coupled unnecessarily to implementation details?

11. Report what was tested, what was not tested, and the commands used to
    verify the result.