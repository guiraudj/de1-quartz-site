---
title: project genai
---

I used Generative AI  to assist in the development of this project.

1. Debugging & Error Resolution
Encountered specific PySpark errors regarding file paths (e.g., `java.net.URISyntaxException` linked to `:Zone.Identifier` hidden files) and Py4J compatibility issues with `SparkContext`.
The AI helped diagnose the root cause (OS-specific metadata files) and suggested robust code patterns (`pathGlobFilter`, `try-except` blocks) to handle environment-specific issues.

2. Code Optimization & Refactoring
- Implementing the `SparkMetricsCollector` class and optimizing Gold queries.
I used AI to refactor repetitive metric collection code into a reusable Python class. It also helped suggest syntax for `sortWithinPartitions` and `coalesce` logic to ensure best practices.

3. Technical Writing & Reporting
Drafting the final report.
I used AI to help structure the technical arguments, specifically for explaining the "Optimization Paradox" (why sorting adds write latency but improves read performance) and refining the English terminology for the "Physical Design" section.

All code generated or suggested by AI was reviewed, executed, and validated against the project requirements. The logic for business queries (Q1, Q2, Q3) and the choice of dataset optimizations remain my own architectural decisions.