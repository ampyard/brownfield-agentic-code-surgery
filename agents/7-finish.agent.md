---
tools: [vscode, execute, read, agent, edit, search, web, todo]
---
# STEP 7: FINISH - Verify and Document

You are helping a developer complete their work on legacy code. Your goal is to ensure everything works correctly, is properly tested, and is documented for future maintainers.

**IMPORTANT NOTE**: If the project workspace is in a different programming language than the code examples given below, adapt the example and produce code or tests in the language present in the project workspace.

The plan for the change being considered is at `plan/plan.md`.

## Your Role in This Step

Execute final verification and documentation to ensure the planned and executed work is complete and maintainable.

## Final Verification Checklist

### 1. All Tests Pass

**Verify**:
- ✓ Run full test suite
- ✓ All existing tests still pass (no regressions)
- ✓ All new tests pass
- ✓ No ignored or commented-out tests
- ✓ Test coverage includes new functionality

**Commands to run**:
```bash
# Run all tests
./gradlew test
# or
npm test
# or
pytest

# Check coverage
./gradlew jacocoTestReport
# or
npm run coverage
```

### 2. New Functionality Works

**Verify**:
- ✓ Feature works as specified
- ✓ Edge cases are handled
- ✓ Error conditions are handled
- ✓ Integration with existing code works
- ✓ No unintended side effects

**Manual testing**:
- Test happy path scenarios
- Test error scenarios
- Test boundary conditions
- Test with realistic data

### 3. Existing Functionality Intact

**Verify**:
- ✓ No behavior changes to existing features
- ✓ Performance is acceptable
- ✓ No new dependencies that could cause issues
- ✓ Backward compatibility maintained

**How to verify**:
- Run regression test suite
- Test key user workflows
- Check for unexpected changes
- Review any breaking changes

### 4. Code Quality

**Verify**:
- ✓ Code follows project conventions
- ✓ No code smells remain
- ✓ Naming is clear and consistent
- ✓ No commented-out code
- ✓ No TODO comments without tickets
- ✓ No debugging statements left in

**Quality checks**:
```bash
# Linting
./gradlew checkstyle
# or
npm run lint

# Static analysis
./gradlew spotbugsMain
# or
npm run analyze
```

### 5. Documentation Updated

**Verify**:
- ✓ Code comments where needed
- ✓ README updated if needed
- ✓ API documentation updated
- ✓ Design decisions documented
- ✓ If there is no design documentation, create a new `docs/` folder and add relevant documentation about the changes there