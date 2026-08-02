# Project development

## Repository layout

JGuiWrapper is a Gradle multi-project build:

```text
JGuiWrapper/
├── api/                    Platform-neutral public API
├── common/                 Shared implementations
├── paper/
│   ├── paper-api/          Public Paper-facing API
│   ├── paper-common/       Embedded Paper implementation and listeners
│   ├── paper-plugin/       Standalone plugin and executable examples
│   └── nms/                Aggregated, version-specific Paper wrappers
└── minestom/               Complete Minestom implementation and tests
```

The published module chain is:

```text
paper-common -> common -> api
             -> paper-api -> api

minestom -> common -> api

nms -> paper-api -> api
```

## Toolchains

- Shared and Paper modules declare Java 16.
- Minestom declares Java 25.
- The Foojay toolchain resolver is configured in `settings.gradle.kts`.

Use the checked-in Gradle wrapper:

```powershell
.\gradlew.bat build
```

Useful focused tasks:

```powershell
.\gradlew.bat :api:build
.\gradlew.bat :paper:paper-api:javadoc
.\gradlew.bat :paper:paper-plugin:shadowJar
.\gradlew.bat :minestom:test
```

The standalone Paper jar is produced under `paper/paper-plugin/build/libs`.

## Adding shared behavior

Put platform-neutral interfaces and models in `api`. Put reusable implementations that do not depend on Paper or Minestom in `common`. Keep all Bukkit types in `paper-api` or `paper-common`, and all Minestom types in `minestom`.

A shared API change should be checked against both platform implementations. The two platforms translate their native inventory events into the same `GuiClickEvent` action and click enums.

## Adding a GUI factory type

The platform initialization classes register built-in `ADVANCED` and `PAGINATED` creators. A new shared `GuiType` needs a creator in both `PaperGuiApiImpl` and `MinestomGuiApi`.

Application-specific types do not require an enum change; register a string key:

```java
api.guiFactory().register("plugin:type", options -> createGui(options));
```

## Paper NMS wrappers

`paper/nms` builds an aggregate jar from version-specific child projects. `NMSMatcher` maps the running Minecraft version to a class named:

```text
com.jodexindustries.jguiwrapper.paper.nms.Wrapper<version>
```

When adding a server version:

1. Add its project to `settings.gradle.kts`.
2. Implement `NMSWrapper` in the expected package and class name.
3. Add or update the version mapping in `NMSMatcher`.
4. Build the aggregate `:paper:nms` module on the target toolchain.
5. Test open and update behavior on the corresponding Paper server.

Do not relocate JGuiWrapper's NMS packages when shading the optional `nms` artifact. The matcher loads wrapper classes by their exact fully qualified name.

## Documentation conventions

- Keep README installation coordinates aligned with the published `artifactId` values in each module build file.
- Prefer examples based on public APIs rather than implementation details.
- Use zero-based raw slots and state whether an example is Paper or Minestom.
- Mark experimental APIs such as paginated GUIs and holder replacement.
- Update the version and platform table when Gradle dependencies or NMS mappings change.

## Verification checklist

Before publishing a change:

1. Run the full Gradle build.
2. Run Minestom tests.
3. Generate Paper Javadocs.
4. Build the standalone shaded plugin.
5. Verify Markdown links and code identifiers in `README.md` and `docs/`.
6. Smoke-test affected GUI actions on their target platform versions.
