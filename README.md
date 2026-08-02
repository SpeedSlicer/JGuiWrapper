# JGuiWrapper

**A cross-platform Java library for inventory GUIs on Paper and Minestom.**

[![Release](https://github.com/Jodexx/JGuiWrapper/actions/workflows/gradle-publish.yml/badge.svg)](https://github.com/Jodexx/JGuiWrapper/actions/workflows/gradle-publish.yml)
[![JitPack](https://jitpack.io/v/Jodexx/JGuiWrapper.svg)](https://jitpack.io/#Jodexx/JGuiWrapper)
[![License](https://img.shields.io/github/license/Jodexx/JGuiWrapper)](LICENSE)

JGuiWrapper provides one GUI model for Paper and Minestom, including slot-level click handlers, reusable item controllers, paginated menus, placeholder-aware items, and platform-neutral users and events.

## Features

- Paper 1.16.5 through 1.21.11 and Minestom 2026.04.13-1.21.11
- Simple, advanced, and paginated inventory GUIs
- Adventure `Component` titles, display names, and lore
- Platform-neutral click, drag, open, and close events
- Reusable data loaders and item handlers
- Literal and regular-expression placeholders
- Optional Paper NMS support for in-place title and inventory updates
- Standalone Paper plugin or embedded library deployment

## Requirements

| Platform | Supported version | Java |
| --- | --- | --- |
| Paper | 1.16.5–1.21.11 | 16+ |
| Minestom | 2026.04.13-1.21.11+ | 25+ |

## Quick start

The examples below use version `v1.0.0.9` from JitPack.

Add JitPack to `settings.gradle.kts` or `build.gradle.kts`:

```kotlin
repositories {
    maven("https://jitpack.io")
}
```

### Paper: embed JGuiWrapper

Package `paper-common` into your plugin jar:

```kotlin
dependencies {
    implementation("com.github.Jodexx.JGuiWrapper:paper-common:v1.0.0.9")
}
```

Initialize the library once from your plugin:

```java
import com.jodexindustries.jguiwrapper.common.PaperGuiApiImpl;

@Override
public void onEnable() {
    PaperGuiApiImpl.init(this);
}
```

Then create and open a GUI:

```java
import com.jodexindustries.jguiwrapper.api.gui.factory.GuiOptions;
import com.jodexindustries.jguiwrapper.api.gui.factory.GuiType;
import com.jodexindustries.jguiwrapper.paper.api.PaperGuiApi;
import com.jodexindustries.jguiwrapper.paper.api.gui.types.advanced.PaperAdvancedGui;
import com.jodexindustries.jguiwrapper.paper.api.item.PaperItemWrapper;
import net.kyori.adventure.text.Component;
import org.bukkit.Material;
import org.bukkit.entity.Player;

PaperAdvancedGui gui = PaperGuiApi.get().guiFactory().create(
        GuiType.ADVANCED,
        GuiOptions.builder()
                .size(27)
                .title(Component.text("Server menu"))
                .build()
);

gui.registerItem("close", item -> item
        .slots(22)
        .defaultItem(PaperItemWrapper.builder(Material.BARRIER)
                .displayName("&cClose")
                .build())
        .defaultClickHandler((event, controller) -> {
            event.setCancelled(true);
            gui.close(event.user());
        }));

gui.open(player);
```

When embedding a Paper library in a plugin, use a shading plugin so the dependency is included in the deployed jar. Add the optional `nms` artifact if you need in-place title, type, or size updates.

### Minestom

```kotlin
dependencies {
    implementation("com.github.Jodexx.JGuiWrapper:minestom:v1.0.0.9")
}
```

Initialize JGuiWrapper after Minestom:

```java
var server = MinecraftServer.init();
MinestomGuiApi.init(MinecraftServer.process());
```

Create GUIs through `MinestomGuiApi.get().guiFactory()` or extend `MinestomGuiBase`.

## Documentation

- [Documentation home](docs/README.md)
- [Installation and first GUI](docs/getting-started.md)
- [GUI types](docs/gui-types.md)
- [Items, text, and placeholders](docs/items-and-text.md)
- [Events and users](docs/events-and-users.md)
- [Registries, loaders, and handlers](docs/registries-and-loaders.md)
- [API reference](docs/api-reference.md)
- [Project development](docs/development.md)

## Module guide

| Artifact | Use it for |
| --- | --- |
| `api` | Platform-neutral interfaces and models; normally pulled transitively |
| `common` | Shared implementations; normally pulled transitively |
| `paper-api` | Compiling against a separately installed JGuiWrapper Paper plugin |
| `paper-common` | Embedding the complete Paper implementation in your plugin |
| `nms` | Optional Paper in-place menu updates across supported server versions |
| `minestom` | Complete Minestom implementation |

## License and conduct

JGuiWrapper is available under the [MIT License](LICENSE). Contributions and community participation are covered by the [Code of Conduct](CODE_OF_CONDUCT.md) and [Security Policy](SECURITY.md).
