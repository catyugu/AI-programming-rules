# DEVELOPING GUIDELINES

1.  **Core Philosophy: Functional & Composable.** All new components MUST be written as **Functional Components** using **React Hooks**. Strive for small, reusable, and composable components.
2.  **Project Setup.** Use modern build tools like **Vite** or **Create React App** for bootstrapping. All environment variables and configurations MUST be clearly documented in a `.env.example` file.
3.  **State Management.**
      * **Local State:** Use `useState` or `useReducer` for component-local state.
      * **Shared State:** Use `React Context` for low-frequency updates of prop-drilling.
      * **Global State:** For complex, high-frequency global state, use a dedicated library (e.g., **Zustand** or **Redux Toolkit**). Avoid prop-drilling complex state through many layers.
4.  **Component Design.**
      * **Single Responsibility:** A component should do one thing well. If a component becomes too large (e.g., over 200-300 lines) or handles too much logic, it MUST be refactored and broken down.
      * **Pure Components:** Keep components as pure as possible. Business logic and side effects (e.g., API calls) should be isolated in custom hooks (`useMyLogic`) or service modules.
      * **Props:** Use **TypeScript** (or `PropTypes` if in a JS-only project) to define and enforce prop types for every component.
5.  **File Structure.** Maintain a clean and scalable project structure.
   
    ```bash
    src/
    |-- assets/         (images, fonts, etc.)
    |-- components/     (small, reusable UI components)
    |-- hooks/          (custom React hooks)
    |-- layouts/        (page layout components, e.g., Navbar, Footer)
    |-- pages/          (top-level route components)
    |-- services/       (API calls, external service logic)
    |-- store/          (global state management)
    |-- styles/         (global styles, theme)
    |-- utils/          (helper functions)
    |-- App.jsx
    |-- main.jsx
    ```
7.  **Naming Traditions.**
      * Components (Files & export): `PascalCase` (e.g., `Button.jsx`, `UserProfile.jsx`)
      * Custom Hooks (Files & export): `useCamelCase` (e.g., `useAuth.js`, `useFetch.js`)
      * Functions/Variables: `camelCase`
      * Constants: `UPPER_SNAKE_CASE` (e.g., `API_BASE_URL`)
      * Test Files: `MyComponent.test.jsx`
8.  **Styling.** Choose **one** consistent styling approach for the project and adhere to it.
      * **Recommended:** **CSS Modules** (for component-scoped CSS) or a utility-first framework like **Tailwind CSS**.
      * Avoid mixing styling methods (e.g., global CSS, styled-components, and CSS Modules) in the same project.
9.  **Documentation.** Use **JSDoc-style comments** for all components, hooks, and complex utility functions. Clearly document the *purpose* of props and returned values.
10.  **Testing (TDD Principle).** Adhere to the TDD workflow: PLANNING -\> WRITING TESTS -\> IMPLEMENTING -\> TESTING -\> REFACTORING.
      * Use **Jest** and **React Testing Library (RTL)** for unit and integration tests.
      * Focus on testing *user behavior* (e.g., "when the user clicks this button, this text appears") rather than implementation details.
      * All new features MUST be accompanied by tests, and all tests MUST pass before a `commit`.
11. **Version Control (`git`).**
      * All work MUST be done on a `dev` or feature branch (e.g., `feat/user-login`).
      * If a large-scale refactor is needed, open an `unstable` branch.
      * Merge to `dev` after feature completion and test passing.
      * `master` (or `main`) branch is for production-ready milestones only.
12. **Performance.**
      * **Keys:** Always provide stable, unique `key` props when rendering lists. Do NOT use array indices as keys.
      * **Memoization:** Use `React.memo`, `useMemo`, and `useCallback` *strategically* to prevent unnecessary re-renders, but only after profiling and identifying a real performance bottleneck. Do not memoize prematurely.
      * **Lazy Loading:** Use `React.lazy()` and `<Suspense>` to code-split pages and large components.
13. **NO MAGIC VALUES.** No magic strings, numbers, or other constants in the component logic. Define them with meaningful names at the top of the file or in a separate `constants.js` file.
14. **Error Handling.** Implement **Error Boundaries** at the application or page level to catch rendering errors and display a fallback UI. All API calls or async operations MUST have proper `.catch()` or `try...catch` blocks.
15. **Clean Imports.** No relative path "spaghetti" (e.g., `../../../../utils`). Configure **absolute imports** or path aliases in your `vite.config.js` or `jsconfig.json` (e.g., `@/components/Button`).
16. **What if you get stuck:** If you try more than 3 times but still fail on the same problem (e.g., a bug, a state logic issue), you MUST stop. Take a step back, review the React docs or data flow, and try to identify the root cause from a higher (architectural) perspective.
17. **Plan and Checklists:** Before starting a new feature, create a plan document or issue with clear todos and checklists. Update the checklist as you complete tasks. This is crucial for tracking progress and for future developers.
