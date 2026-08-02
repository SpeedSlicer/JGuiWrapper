# Registries, loaders, and handlers

Advanced GUIs can separate reusable data access from item presentation:

- A `GuiDataLoader<G>` loads or prepares data for one GUI instance and user.
- An `ItemHandler<T>` configures a controller using a compatible loader type.
- A `GlobalRegistry` assigns Adventure `Key` values to reusable loaders and handlers.
- A controller references an item handler by key.

This is useful when several menus share the same data source or controller behavior.

## Define a loader

```java
import com.jodexindustries.jguiwrapper.api.gui.types.advanced.GuiDataLoader;
import com.jodexindustries.jguiwrapper.api.user.User;
import com.jodexindustries.jguiwrapper.paper.api.gui.types.advanced.PaperAdvancedGui;
import org.jetbrains.annotations.NotNull;

public final class BalanceLoader implements GuiDataLoader<PaperAdvancedGui> {
    private int balance;

    @Override
    public void load(@NotNull PaperAdvancedGui gui, @NotNull User user) {
        balance = loadBalance(user);
    }

    public int balance() {
        return balance;
    }

    private int loadBalance(User user) {
        return 100;
    }
}
```

Loader instances are registered by their concrete class inside a GUI. Registering another loader of the same class replaces the earlier one for that GUI.

## Define a typed item handler

```java
import com.jodexindustries.jguiwrapper.api.gui.types.advanced.AdvancedGuiItemController;
import com.jodexindustries.jguiwrapper.api.gui.types.advanced.item.HandlerContext;
import com.jodexindustries.jguiwrapper.api.gui.types.advanced.item.ItemHandler;
import org.jetbrains.annotations.NotNull;

public final class BalanceItemHandler implements ItemHandler<BalanceLoader> {
    @Override
    public void load(
            @NotNull BalanceLoader loader,
            @NotNull AdvancedGuiItemController<?, ?> controller,
            @NotNull HandlerContext context
    ) {
        controller.updateItems(item -> item.meta()
                .displayName("&aBalance")
                .lore("&7Coins: &e" + loader.balance()), context.user());

        controller.defaultClickHandler((event, ignored) -> event.setCancelled(true));
    }
}
```

Use a concrete handler class that directly declares `ItemHandler<YourLoader>`. The registry reflects that generic parameter to decide which loader the handler requires; raw types, erased proxies, and some indirect generic hierarchies cannot provide the expected class reliably.

## Register both components

Register reusable components during platform initialization, before creating controllers that refer to them:

```java
import com.jodexindustries.jguiwrapper.api.gui.types.advanced.registry.GlobalRegistry;
import com.jodexindustries.jguiwrapper.paper.api.PaperGuiApi;
import net.kyori.adventure.key.Key;

public static final Key BALANCE_KEY = Key.key("example", "balance");

GlobalRegistry registry = PaperGuiApi.get().getRegistry();
registry.registerLoader(BALANCE_KEY, new BalanceLoader());
registry.registerHandler(BALANCE_KEY, new BalanceItemHandler());
```

For Minestom, obtain the registry from `MinestomGuiApi.get().getRegistry()`.

`registerLoader` and `registerHandler` create the namespace registry when necessary. You may instead create one explicitly:

```java
DataRegistry registry = globalRegistry.register("example");
registry.registerLoader("balance", new BalanceLoader());
registry.registerHandler("balance", new BalanceItemHandler());
```

Registering an existing namespace or duplicate component ID throws `IllegalStateException`. In the current implementation, loader and handler backing maps are shared across namespace registries, so keep each loader ID and each handler ID globally unique even when namespaces differ.

## Attach the components to a GUI

```java
gui.registerLoader(BALANCE_KEY);

gui.registerItem("balance", builder -> builder
        .slots(13)
        .defaultItem(PaperItemWrapper.builder(Material.GOLD_INGOT).build())
        .itemHandler(BALANCE_KEY));
```

When the controller is registered, JGuiWrapper resolves its item-handler key and stores the typed handler association. Register the global handler before `gui.registerItem(...)`.

Loading is explicit:

```java
User user = PaperGuiApi.get().user(player);
gui.loadData(user);
gui.loadItemHandlers(LoadType.ON_OPEN, user);
gui.open(player);
```

JGuiWrapper does not automatically call loaders or item handlers when a GUI opens. This lets your application control ordering, error handling, caching, and asynchronous work.

## Load types

`HandlerContext` supplies a `LoadType` and optional user:

| Value | Intended use |
| --- | --- |
| `ON_LOAD` | Initial or user-neutral construction |
| `ON_OPEN` | Refreshing content for a specific viewer |

The enum communicates intent to your handler; JGuiWrapper does not impose different behavior for the two values.

## Direct loaders

For one-off behavior, skip the global registry:

```java
gui.registerLoader(new BalanceLoader());
gui.loadData(user);

gui.getTypedLoader(BalanceLoader.class)
        .ifPresent(loader -> user.sendMessage("Balance: " + loader.balance()));
```

Direct loaders participate in `loadData` but cannot be selected by a keyed item handler unless the handler is registered and resolved through the global registry.
