# maven-buildpack

A Cloud Foundry **supply** buildpack that installs [Apache Maven](https://maven.apache.org/) into the container during staging so that `mvn` is available in `PATH` at runtime.

Primary use case: Goose agent running inside CF needs to execute `mvn` builds as part of automated tasks.

## Usage

Add this buildpack **before** `java_buildpack_offline` in your `manifest.yml`:

```yaml
buildpacks:
  - https://github.com/cpage-pivotal/goose-buildpack
  - https://github.com/0pens0/maven-buildpack
  - java_buildpack_offline
```

## What it does

1. Downloads Apache Maven 3.9.9 during staging (cached for subsequent pushes)
2. Extracts Maven to `$DEPS_DIR/$INDEX/maven/`
3. Writes a `profile.d/maven.sh` that sets `M2_HOME` and adds `mvn` to `PATH` at container startup

## Notes

- The `java_buildpack_offline` provides the JRE. Maven uses it automatically via `JAVA_HOME`.
- `MAVEN_OPTS=-Xmx512m` is set to limit Maven's heap footprint inside the CF container.
- Maven's local repository (`~/.m2/repository`) is ephemeral — it is recreated on each container start. For projects with many dependencies this adds startup time on the first `mvn` invocation.
