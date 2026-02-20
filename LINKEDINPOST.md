# Using Maveniverse Toolbox in Initializer for Apache Maven™

[Initializer for Apache Maven™](https://maven-starter.io/) helps you bootstrap production-ready Maven-based Java projects: a well-structured `pom.xml`, sensible defaults, and up-to-date dependencies in one click. Under the hood, the backend relies on **[Maveniverse Toolbox](https://github.com/maveniverse/toolbox)** for the Maven-heavy work. This short post is about that choice and where it fits.

## Why the Toolbox?

We needed a way to generate and edit POMs programmatically, resolve dependency and plugin versions from Maven repositories, and keep the logic maintainable. After evaluating archetype components and custom template engines, we chose **Maveniverse Toolbox**: a library from the Apache Maven™ ecosystem that offers a Java DSL for POM creation and editing, plus version resolution. It’s actively maintained and keeps Maven-specific details in one place instead of scattering them across our codebase.

The Toolbox gives us methods like `addDependencies` and a clear editing model (e.g. transactional sessions so the POM stays valid), which made it straightforward to implement our generation pipeline.

## Where It Fits

The backend uses the Toolbox in two main ways: **version resolution** (latest or requested versions for dependencies and plugins) and **POM generation/editing** (building the project file and applying changes in a single session). All of this is orchestrated by our services; the Toolbox stays a library we call, not the entry point.

For a concise picture of how the Initializer is structured and how the Toolbox fits next to the REST API, project generation, and config, see the **[architecture overview in our technical documentation](https://support-and-care.github.io/maven-initializer/architecture/)**. There you’ll find high-level diagrams, the end-to-end generation flow, and pointers to the ADRs (including the decision to use the Toolbox).

## Try It and Read More

- **Use the app:** [maven-starter.io](https://maven-starter.io/)
- **Technical docs (architecture + ADRs):** [support-and-care.github.io/maven-initializer](https://support-and-care.github.io/maven-initializer/)
- **Source:** [GitHub — support-and-care/maven-initializer](https://github.com/support-and-care/maven-initializer)
