# Memory Bank Improvements (Self-Review)

## Feedback Received
- The folder structure might need to be defined more explicitly so the AI doesn't struggle to find the `useBoardStore` path.
- Test files (Vitest/RTL) were generated successfully, but the rules for mocking the Zustand store are not yet fully standardized.

## Changes Made
- The project's MVP (Minimum Viable Product) boundaries, specifically the strict reliance on Local Persistence, were clearly documented in the `domain/glossary.md` file.
- PR and deployment rules tailored specifically for a solo developer workflow were added to the `workflows/development-process.md` file.

## Future Improvements (next iteration)
- Specific guidelines and conventions for mocking state managers like Zustand will be added to the `coding-standards.md` file.
- As the project structure grows, the glossary under the `domain` folder will be split into subcategories such as `domain/tasks` and `domain/ui` for better organization.