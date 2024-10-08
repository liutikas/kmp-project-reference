# Repro of KGP project isolation violation

1. `./gradlew libraryA:build -Dorg.gradle.unsafe.isolated-projects=true`

## Expected

Success

## Actual

```
- Build file 'libraryA/build.gradle': line 12: Project ':libraryA' cannot dynamically look up a method in the parent project ':'
```

and a report has a stacktrace

## Additional info

`project(":libraryB")` call within `sourceSets` scope in groovy end up calling `getProject` on the `HasProject.getProject`
that is part of `org.jetbrains.kotlin.gradle.plugin.KotlinSourceSet` interface, then due to Gradle-ism it tries to call
`getProperty(":libraryB"` on that `Project` object.

As a user of KGP that uses Gradle Groovy build files, there is only workaround which is to use `implementation(project.project(":libraryB"))`
`project.project` to force Groovy to pick the right call that doesn't violate project isolation, however this is a fairly
ugly workaround.

Fixes for KGP could include:
- Removing getProject() from the extensions (might be hard due to binary compatibility)
- Add project(name: String) to every place that has getProject (likely the easiest solution)
