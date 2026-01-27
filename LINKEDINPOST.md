# Maven Initializer v1.2.0 🚀

We’ve released **Maven Initializer v1.2.0**, a modern web app that makes bootstrapping Maven-based Java projects fast and painless.

To generate valid, production-ready `pom.xml` files programmatically, we built the backend on **Maveniverse Toolbox**. Its Java DSL, automatic version resolution, and transactional `EditSession` model gave us a clean and reliable way to work with Maven without touching raw XML.

## Why Toolbox?

- **Type-safe, fluent POM editing**
- **Latest versions resolved from Maven Central**
- **Atomic, transactional updates**
- **Clean abstraction over Maven internals**

The result is a maintainable generator that produces ready-to-build projects with sensible defaults and best practices baked in.

👉 Want the technical details? Read the full deep dive on our architecture and design decisions.

## Thanks to

- **The Maveniverse Toolbox team**
- **Sandra Parsick** (project lead)
- **Noah Tayebwa** (active development)

Development is ongoing and feedback is very welcome!

🔗 Try it out: [https://maven-starter.io](https://maven-starter.io)