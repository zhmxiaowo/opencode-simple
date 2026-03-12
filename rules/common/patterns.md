# Common Patterns

## Skeleton Projects

When implementing new functionality:
1. Search for battle-tested skeleton projects
2. Use parallel agents to evaluate options:
   - Security assessment
   - Extensibility analysis
   - Relevance scoring
   - Implementation planning
3. Clone best match as foundation
4. Iterate within proven structure

## Spec-First Pattern

Before writing ANY implementation code:
1. **Research**: Use `context7` + `exa-web-search` to gather official docs
2. **Draft SPEC.md**: Document API design, data models, constraints
3. **User Confirm**: Wait for explicit user approval
4. **Then Implement**: Follow the spec precisely

## CDD (Compile-Driven Development) Pattern

The sole verification method for code:
1. **Write code** based on SPEC
2. **Run compiler/type-checker** for the project type
3. **Check LSP diagnostics** for zero Errors
4. **If compile fails**: Invoke `sequential-thinking` for root cause analysis — NEVER blindly guess
5. **Apply minimal fix** and recompile
6. **Exit Code 0 + 0 LSP Errors** = ✅ Task Complete

## Design Patterns

### Repository Pattern

Encapsulate data access behind a consistent interface:
- Define standard operations: findAll, findById, create, update, delete
- Concrete implementations handle storage details (database, API, file, etc.)
- Business logic depends on the abstract interface, not the storage mechanism
- Enables easy swapping of data sources

### API Response Format

Use a consistent envelope for all API responses:
- Include a success/status indicator
- Include the data payload (nullable on error)
- Include an error message field (nullable on success)
- Include metadata for paginated responses (total, page, limit)
