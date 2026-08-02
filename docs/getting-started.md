# Getting started

## Choose a deployment model

Paper has two supported deployment models:

| Model | Server setup | Dependency | Initialization |
| --- | --- | --- | --- |
| Standalone plugin | Install JGuiWrapper as its own Paper plugin | `paper-api` with `compileOnly`/`provided` | Performed by JGuiWrapper |
| Embedded | Package JGuiWrapper inside your plugin | `paper-common` with `implementation` | Call `PaperGuiApiImpl.init(this)` |

Minestom always embeds the `minestom` artifact in the application.

The examples use `v1.0.0.9`. JitPack tags include the leading `v`.

## Repository configuration

### Gradle Kotlin DSL

```kotlin
repositories {
    mavenCentral()
    maven("https://jitpack.io")
}
```

### Maven

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
```

## Paper standalone plugin

Download the JGuiWrapper plugin jar from the project's GitHub releases and place it in the server's `plugins` directory.

Compile your plugin against the Paper-facing API:

```kotlin
dependencies {
    compileOnly("com.github.Jodexx.JGuiWrapper:paper-api:v1.0.0.9")
}
```

```xml
<dependency>
    <groupId>com.github.Jodexx.JGuiWrapper</groupId>
    <artifactId>paper-api</artifactId>
    <version>v1.0.0.9</version>
    <scope>provided</scope>
</dependency>
```

Declare the runtime dependency in `plugin.yml` so Paper loads JGuiWrapper first:

```yaml
depend: [JGuiWrapper]
```

Do not call `PaperGuiApiImpl.init` in this mode. Use `PaperGuiApi.get()` after your plugin has been enabled.

## Paper embedded

Add the full Paper implementation:

```kotlin
dependencies {
    implementation("com.github.Jodexx.JGuiWrapper:paper-common:v1.0.0.9")
    // Optional: in-place title, inventory type, and size updates.
    implementation("com.github.Jodexx.JGuiWrapper:nms:v1.0.0.9")
}
```

```xml
<dependency>
    <groupId>com.github.Jodexx.JGuiWrapper</groupId>
    <artifactId>paper-common</artifactId>
    <version>v1.0.0.9</version>
</dependency>
```

The dependencies must be included in your deployed plugin jar. For Gradle, the Shadow plugin is a common choice:

```kotlin
plugins {
    id("com.gradleup.shadow") version "9.3.1"
}

tasks.build {
    dependsOn(tasks.shadowJar)
}
```

Initialize once in the plugin's main class:

```java
import com.jodexindustries.jguiwrapper.common.PaperGuiApiImpl;
import org.bukkit.plugin.java.JavaPlugin;

public final class ExamplePlugin extends JavaPlugin {
    @Override
    public void onEnable() {
        PaperGuiApiImpl.init(this);
    }
}
```

Initialization registers the inventory listener, detects PlaceholderAPI, creates the built-in GUI factory entries, and attempts to load the optional NMS wrapper.

## Your first Paper GUI

For a menu with named item controllers, use the factory's advanced GUI:

```java
import com.jodexindustries.jguiwrapper.api.gui.factory.GuiOptions;
import com.jodexindustries.jguiwrapper.api.gui.factory.GuiType;
import com.jodexindustries.jguiwrapper.paper.api.PaperGuiApi;
import com.jodexindustries.jguiwrapper.paper.api.gui.types.advanced.PaperAdvancedGui;
import com.jodexindustries.jguiwrapper.paper.api.item.PaperItemWrapper;
import net.kyori.adventure.text.Component;
import org.bukkit.Material;
import org.bukkit.entity.Player;

public void openMenu(Player player) {
    PaperAdvancedGui gui = PaperGuiApi.get().guiFactory().create(
            GuiType.ADVANCED,
            GuiOptions.builder()
                    .size(27)
                    .title(Component.text("Example menu"))
                    .build()
    );

    gui.registerItem("diamond", builder -> builder
            .slots(13)
            .defaultItem(PaperItemWrapper.builder(Material.DIAMOND)
                    .displayName("&bA diamond")
                    .lore("&7Click to receive a message")
                    .build())
            .defaultClickHandler((event, controller) -> {
                event.setCancelled(true);
                event.user().sendMessage("You clicked the diamond.");
            }));

    gui.open(player);
}
```

The default serializer is `LEGACY_AMPERSAND`, so strings such as `&bA diamond` are converted to Adventure components. You can always pass `Component` instances directly.

## Minestom

Add the implementation:

```kotlin
dependencies {
    implementation("com.github.Jodexx.JGuiWrapper:minestom:v1.0.0.9")
}
```

```xml
<dependency>
    <groupId>com.github.Jodexx.JGuiWrapper</groupId>
    <artifactId>minestom</artifactId>
    <version>v1.0.0.9</version>
</dependency>
```

Minestom version `2026.04.13-1.21.11` in this project requires Java 25.

Initialize the server and JGuiWrapper:

```java
import com.jodexindustries.jguiwrapper.minestom.MinestomGuiApi;
import net.minestom.server.MinecraftServer;

var server = MinecraftServer.init();
MinestomGuiApi.init(MinecraftServer.process());
```

Then build a menu with the same factory model:

```java
import com.jodexindustries.jguiwrapper.api.gui.factory.GuiOptions;
import com.jodexindustries.jguiwrapper.api.gui.factory.GuiType;
import com.jodexindustries.jguiwrapper.minestom.MinestomGuiApi;
import com.jodexindustries.jguiwrapper.minestom.gui.types.advanced.MinestomAdvancedGui;
import com.jodexindustries.jguiwrapper.minestom.item.MinestomItemWrapper;
import net.kyori.adventure.text.Component;
import net.minestom.server.entity.Player;
import net.minestom.server.item.Material;

public void openMenu(Player player) {
    MinestomAdvancedGui gui = MinestomGuiApi.get().guiFactory().create(
            GuiType.ADVANCED,
            GuiOptions.builder()
                    .size(27)
                    .title(Component.text("Example menu"))
                    .build()
    );

    gui.registerItem("stone", builder -> builder
            .slots(13)
            .defaultItem(MinestomItemWrapper.builder(Material.STONE)
                    .displayName("&fStone")
                    .build())
            .defaultClickHandler((event, controller) -> {
                event.setCancelled(true);
                event.user().sendMessage("You clicked stone.");
            }));

    gui.open(player);
}
```

## Size rules

Numeric GUI sizes are clamped to `1..54` and rounded up to a multiple of nine:

| Requested | Actual |
| ---: | ---: |
| 1 | 9 |
| 10 | 18 |
| 27 | 27 |
| 60 | 54 |

Both platforms also have constructors that accept a platform-specific `InventoryType`.

## Next steps

- [Choose a GUI type](gui-types.md).
- [Create and update items](items-and-text.md).
- [Understand event cancellation](events-and-users.md).
