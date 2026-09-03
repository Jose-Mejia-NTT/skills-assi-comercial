# project-alignment-checklist

Use this checklist before generating code.

1. Detect base package from existing source files.
2. Detect module names for core, input, output, and app.
3. Detect existing package conventions:
   - core ports location
   - input controller and DTO location
   - output adapter location
4. Detect wiring location (usually UseCaseConfig).
5. Detect naming style for use case interfaces and services.
6. Detect endpoint path style from existing controllers.
7. Reuse existing mappers and command/query services when possible.
8. If any critical convention cannot be inferred, ask up to 3 questions.
