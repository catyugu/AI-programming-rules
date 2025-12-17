# MetaHotSpot: Code Standards & Quality Gates

## 1\. Core Philosophy: "Zero-Cost Abstraction"

We are building a simulator where performance is paramount.

1.  **No Hidden Costs:** Avoid features that incur implicit overhead (e.g., `virtual` functions in the hot path, excessive `std::shared_ptr`, accidental copies).
2.  **Data First:** Design your data layout *before* you write the code. If the memory layout is bad, the code is bad.
3.  **Explicit is Better:** Type conversions, memory allocations, and ownership transfers must be explicit.

-----

## 2\. Naming Conventions

We strictly separate the "Physics Core" (snake\_case) from the "UI Framework" (CamelCase) to visual distinguish the two domains.

| Element               | Style             | Example                         | Reasoning                        |
|:----------------------|:------------------|:--------------------------------|:---------------------------------|
| **Namespace**         | `snake_case`      | `namespace metahotspot`         | Standard C++ library style.      |
| **Structs (POD)**     | `PascalCase`      | `struct SimulationModel`        | Distinct from variables.         |
| **Classes (Logic)**   | `PascalCase`      | `class ModelManager`            | Distinct from variables.         |
| **Methods/Functions** | `snake_case`      | `void add_layer(...)`           | Matches STL/Boost style.         |
| **Variables**         | `snake_case`      | `double thermal_k;`             | Easy to read.                    |
| **Private Members**   | `m_camelCase`     | `m_dataModel`                   | *Only* in UI Classes (Qt style). |
| **Constants**         | `SCREAMING_SNAKE` | `constexpr int MAX_ITER = 100;` | Immediate visual warning.        |
| **Template Types**    | `PascalCase`      | `template <typename Precision>` | Standard practice.               |

**Special Rule for IDs:**
Always suffix ID variables with `_id` or `_idx`.

* `MaterialID material_id;` (Good)
* `MaterialID material;` (Bad - looks like the object itself)

-----

## 3\. C++ Coding Rules (The "Hot Path" Restrictions)

These rules apply strictly to the **Solver**, **Mesher**, and **Baker** modules.

### 3.1. Memory Management

* **Rule:** **No `new` / `delete`**. Use `std::vector` for dynamic arrays and `std::unique_ptr` for singletons.
* **Rule:** **Contiguity is King.**
    * *Bad:* `std::vector<Unit*> units;` (Pointer chasing kills cache).
    * *Good:* `std::vector<Unit> units;` (Contiguous memory).
* **Rule:** **Reserve First.** Always call `.reserve()` on vectors before pushing in a loop.

### 3.2. Object Orientation

* **Rule:** **No Virtual Functions** in the `Solver` or `Mesher` inner loops.
    * *Exception:* `QAbstractItemModel` in the UI requires inheritance. This is allowed in the UI layer *only*.
* **Rule:** **Structs are Public.** Core data structures (`Types.h`) must be `struct` with all members public. No getters/setters.
* **Rule:** **Functions are Stateless.** Prefer "Pure Functions" that take data in and return data out.

### 3.3. Performance Safety

* **Rule:** **Pass by `const &`.** Never pass complex structs by value unless you explicitly intend to copy.
    * *Right:* `void process(const Layer& layer);`
    * *Wrong:* `void process(Layer layer);`
* **Rule:** **No Exceptions in Hot Loops.** The solver loop must be exception-free. Return error codes or status enums. Exceptions are for *configuration* and *I/O* only.

-----

## 4\. Qt / UI Guidelines

### 4.1. The "No-Touch" Rule

* **Rule:** The UI classes (`QWidget` derivatives) must **NEVER** modify `SimulationModel` directly.
* **Mechanism:** They must call `ModelManager` methods (`add_layer`, `remove_unit`). The UI *observes* the model, it does not *own* it.

### 4.2. Signals & Slots

* **Rule:** Use New Syntax (`&Class::method`). Never use the old `SIGNAL()`/`SLOT()` macros.
* **Rule:** Keep slots lightweight. If a slot triggers a heavy calculation (Soling/Baking), spawn a `QThread` or `QtConcurrent::run`. **Do not freeze the GUI.**

-----

## 5\. Quality Gates (CI/CD)

Every Pull Request (PR) must pass these gates before merging.

### 5.1. Static Analysis

* **Compiler Warnings:** `-Wall -Wextra -Werror` (GCC/Clang) / `/W4 /WX` (MSVC). Treat warnings as errors.
* **Clang-Tidy:** Must pass standard checks for modern C++ (e.g., `modernize-*`, `performance-*`).
    * *Check:* Implicit conversions that lose precision (double $\to$ float).
    * *Check:* Unnecessary copies in `for-range` loops.

### 5.2. Unit Testing (GoogleTest)

* **Coverage Target:** 100% coverage for "Logic" (`Utils.cpp`, `ModelManager`), 80% for "Solver". UI testing is optional but recommended.
* **Sanitizers:** Tests must run with **AddressSanitizer (ASan)** and **UndefinedBehaviorSanitizer (UBSan)** to catch buffer overflows and memory leaks.

### 5.3. Benchmark Gate

* **Regression Check:** A standard benchmark (e.g., `ev6.flp` steady state) must not degrade in performance by more than 5% compared to the `main` branch.

-----

## 6\. Documentation Standards

* **API Docs:** Doxygen style comments in headers (`.h`).
  ```cpp
  /**
   * @brief Converts the string-based model into a linear solver context.
   * @param model The user configuration.
   * @return SolverContext optimized for FVM operations.
   * @throws std::runtime_error if referenced materials are missing.
   */
  SolverContext bake_simulation_context(const SimulationModel& model);
  ```
* **Implementation Comments:** "Why," not "What."
    * *Bad:* `// Loop through units`
    * *Good:* `// Iterate units to calculate effective conductivity based on occupancy ratio.`

-----

## 7\. Version Control Workflow

* **Branching:** Feature Branch Workflow (`feature/new-solver`, `fix/ui-crash`).
* **Commit Messages:** Conventional Commits.
    * `feat: add transient solver loop`
    * `fix: resolve crash when deleting material used by layer`
    * `perf: vectorize FVM stencil calculation`
    * `docs: update build instructions`

-----

### **Summary Checklist for Developers**

* [ ] Is my struct POD? (No private, no virtuals).
* [ ] Did I reserve my vectors?
* [ ] Am I using `const string&` instead of `string`?
* [ ] Did I use `snake_case` for logic and `CamelCase` for Types?
* [ ] Does `clang-tidy` pass?
* [ ] Do the Unit Tests pass with ASan enabled?