# Using Maveniverse Toolbox in Initializer for Apache Maven™

[Initializer for Apache Maven™](https://maven-starter.io/) helps you bootstrap production-ready Maven-based Java projects: a well-structured `pom.xml`, sensible defaults, and up-to-date dependencies in one click. Under the hood, the backend relies on **[Maveniverse Toolbox](https://github.com/maveniverse/toolbox)** for the Maven-heavy work. This short post is about that choice and where it fits.

## The problem: programmatic Maven project generation

The core challenge is **programmatic Maven project generation**: producing a full, valid Maven project (directory layout, `pom.xml`, Maven Wrapper, README) from code, based on user choices in the web UI. We don’t just fill in a static template, we need to create and edit the POM dynamically (add dependencies, plugins, and properties), resolve real versions from Maven repositories, and ensure the result is a valid, buildable project every time. Doing that reliably and maintainably meant we had to choose how to handle the Maven-specific logic (POM model, version resolution, session handling) instead of reimplementing or gluing it ourselves.

## Why the Toolbox?

We needed a way to generate and edit POMs programmatically, resolve dependency and plugin versions from Maven repositories, and keep the logic maintainable. After evaluating archetype components and custom template engines, we chose **Maveniverse Toolbox**: a library from the Apache Maven™ ecosystem that offers a Java DSL for POM creation and editing, plus version resolution. It’s actively maintained and keeps Maven-specific details in one place instead of scattering them across our codebase.

The Toolbox gives us methods like `addDependencies` and a clear editing model (e.g. transactional sessions so the POM stays valid), which made it straightforward to implement our generation pipeline.

## Version resolution: how the Toolbox helps

Generated projects should use **current stable versions** of dependencies and plugins, not hard-coded ones. We need to query Maven repositories (e.g. Maven Central) and pick the newest non-snapshot version. The Toolbox gives us exactly that through `ToolboxResolver`, which we obtain from `ToolboxCommando`.

Our `ArtifactVersionService` wraps the resolver and exposes methods like `resolveLatestDependencyVersion` and `resolveLatestPluginVersion`. Under the hood we call `findNewestVersion` with an artifact coordinate and a version matcher that excludes snapshots and previews:

```java
private static final ArtifactVersionMatcher VERSION_MATCHER =
    ArtifactVersionMatcher.noSnapshotsAndPreviews();

private String resolveLatestVersion(
    String groupId, String artifactId, String classifier, String extension) {
  Version newestVersion =
      toolboxResolver.findNewestVersion(
          new DefaultArtifact(groupId, artifactId, classifier, extension, "LATEST"),
          VERSION_MATCHER);
  return newestVersion != null ? newestVersion.toString() : "TODO";
}
```

So for each dependency or plugin we add to the generated POM, we first resolve its latest version via the Toolbox; the resolver talks to the configured repositories and returns a single version string we can plug into the POM. No manual HTTP calls or metadata parsing—the Toolbox encapsulates that.

## POM generation and editing: how the Toolbox helps

Building the POM is the other half: we start from a minimal `pom.xml` (group, artifact, version) and then add properties, dependency management, dependencies, and plugins. The Toolbox supports this with an **edit session**: we open a session on the POM file, apply all changes through a `PomEditor` callback, and either commit (POM stays valid) or roll back on failure.

We create a session with `toolboxCommando.createEditSession(pomFile)` and pass one or more editors to `editPom`. Each editor receives a `PomEditor` and can set packaging, update properties, insert elements, add dependencies, and configure plugins:

```java
try (ToolboxCommando.EditSession editSession = toolboxCommando.createEditSession(pomFile)) {
  toolboxCommando.editPom(
      editSession,
      Collections.singletonList(
          editor -> {
            editor.setPackaging("jar");
            editor.properties()
                .updateProperty(true, "maven.compiler.release", request.getJavaVersion());
            editor.properties().updateProperty(true, "project.build.sourceEncoding", "UTF-8");
            editor.insertMavenElement(editor.root(), "description", request.getDescription());

            addDependencyManagement(editor, request);
            addDependencies(editor, request);

            plugins.forEach(plugin ->
                editor.plugins().updatePlugin(true, toCoordinates(plugin)));
          }));
}
```

Adding a dependency is done by finding or creating the `<dependencies>` element and inserting `<dependency>` children with `<groupId>`, `<artifactId>`, `<scope>`, and optionally `<version>` (when not managed by a BOM). The Toolbox’s `PomEditor` and Maven element helpers keep the structure valid and avoid manual XML string building. Plugin coordinates (including versions we resolved earlier) are applied with `updatePlugin`. So the flow is: resolve versions with the Toolbox → write minimal POM → edit in one session with the Toolbox → formatted POM on disk.

For a concise picture of how the Initializer is structured and how the Toolbox fits next to the REST API, project generation, and config, see the **[architecture overview in our technical documentation](https://support-and-care.github.io/maven-initializer/architecture/)**. There you’ll find high-level diagrams, the end-to-end generation flow, and pointers to the ADRs (including the decision to use the Toolbox).

## Try It and Read More

- **Use the app:** [maven-starter.io](https://maven-starter.io/)
- **Technical docs (architecture + ADRs):** [support-and-care.github.io/maven-initializer](https://support-and-care.github.io/maven-initializer/)
- **Source:** [GitHub — support-and-care/maven-initializer](https://github.com/support-and-care/maven-initializer)
