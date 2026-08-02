# API reference

This page is a practical map of the public API. Consult generated Javadocs or source for every overload.

## Entry points

### `GuiApi`

| Member | Purpose |
| --- | --- |
| `GuiApi.get()` | Returns the initialized platform API or throws `IllegalStateException` |
| `GuiApi.getOptional()` | Safe initialization check |
| `defaultSerializer()` | Returns the current global default serializer |
| `defaultSerializer(type)` | Changes the default used by subsequently constructed GUIs and items |
| `createPlaceholderEngine()` | Creates the active platform's engine |
| `guiFactory()` | Returns the built-in and user-extensible GUI factory |
| `getModule()` | Provides the platform logger through `Module` |

Only one `GuiApi` instance can be set in a class loader.

### `PaperGuiApi`

| Member | Purpose |
| --- | --- |
| `PaperGuiApi.get()` / `getOptional()` | Paper-specific singleton access |
| `getPlugin()` | Owning Bukkit plugin |
| `getRegistry()` | Global loaders and item handlers |
| `getNMSWrapper()` | Active NMS wrapper or an inert fallback |
| `getOpenedGui(player)` | JGuiWrapper holder in the player's top inventory, or `null` |
| `isPAPI()` | Whether PlaceholderAPI was detected at initialization |
| `user(player)` | Wraps a Bukkit player or human entity |

Embedded mode initializes this API with `PaperGuiApiImpl.init(plugin)`.

### `MinestomGuiApi`

| Member | Purpose |
| --- | --- |
| `MinestomGuiApi.init(process)` | Initializes the API and registers its inventory listener |
| `MinestomGuiApi.init(process, logger)` | Initializes with a custom logger, without registering the listener |
| `get()` / `getOptional()` | Minestom-specific singleton access |
| `server()` | Returns the `ServerProcess` |
| `getRegistry()` | Global loaders and item handlers |
| `user(player)` | Wraps a Minestom player |

If you use the two-argument initialization overload, register `GuiListener` yourself or call the one-argument overload instead.

## GUI factory

Built-in types:

```java
GuiType.ADVANCED
GuiType.PAGINATED
```

`GuiOptions` fields:

| Option | Default | Notes |
| --- | --- | --- |
| `size` | `54` | Adapted to a chest-row size by the GUI constructor |
| `title` | `Component.empty()` | Adventure component |
| `serializer` | `null` | `null` means the API default |
| `attributes` | Empty map | Available to custom creators |

Register a custom factory type by string key:

```java
GuiFactory factory = GuiApi.get().guiFactory();
factory.register("example:confirm", options ->
        new ConfirmGui(options.size(), options.title(), options.serializer()));

ConfirmGui gui = factory.create(
        "example:confirm",
        GuiOptions.builder().size(9).build()
);
```

Built-in enum registration uses the lower-case enum name as its key. Registering an existing key replaces its creator; creating an unknown key throws `IllegalArgumentException`.

## `Gui`

| Member | Purpose |
| --- | --- |
| `holder()` | Returns the platform holder and inventory |
| `open(user)` | Opens with the model title |
| `open(user, title)` | Opens with a one-time title |
| `close(user)` | Closes for one viewer |
| `close()` | Closes for all viewers |
| `updateHolder()` | Recreates the backing holder; experimental in the shared API |
| `title()` / `size()` | Current model properties |
| `Gui.getActiveInstances()` | Snapshot of weakly referenced live GUI objects |

GUI instances are added to a weak-reference set at construction. The active snapshot does not keep otherwise unreachable GUIs alive.

## Holders

The shared `GuiHolder` supports:

```java
holder.gui();
holder.setItem(slot, itemWrapper);
holder.clear(slot);
```

`PaperGuiHolder` also exposes Bukkit `getInventory()` and accepts a native `ItemStack`. `MinestomGuiHolder` exposes `getInventory()` as `MinestomInventory`.

## Simple GUI members

| Member | Purpose |
| --- | --- |
| `setClickHandlers(handler, slots...)` | Assigns a click handler; no slots means all GUI slots |
| `removeClickHandlers(slots...)` | Removes handlers; no slots means all |
| `setCancelEmptySlots(boolean)` | Controls default click and drag protection |
| `onOpen(consumer)` | Adds an open consumer |
| `onClose(consumer)` | Adds a close consumer |
| `onDrag(consumer)` | Adds a drag consumer |

`title(String)` uses the GUI's serializer. `size(int)` changes the model size but does not recreate the holder.

## Advanced GUI members

| Member | Purpose |
| --- | --- |
| `registerItem(key, builder)` | Builds, registers, attaches, and draws a controller |
| `unregister(key)` | Clears and removes a named controller |
| `getController(key)` / `getController(slot)` | Looks up a controller |
| `getControllers()` | Unmodifiable controller collection |
| `registerLoader(key)` / `registerLoader(loader)` | Attaches data-loading behavior |
| `loadData(user)` | Invokes every attached loader |
| `loadItemHandlers(type, user)` | Invokes each resolved controller handler |
| `attach(controller, slot)` / `detach(slot)` | Low-level slot-map operations |

Controller builder members:

```java
defaultItem(item)
defaultClickHandler(handler)
slots(int...)
slots(collection)
slotItem(slot, item)
slotClickHandler(slot, handler)
itemHandler(key)
build()
```

## Paginated members

| Member | Purpose |
| --- | --- |
| `addPage(builders...)` | Adds one page made from one or more controller builders |
| `pages()` | Page count |
| `currentPage()` | Zero-based current page |
| `openPage(index)` | Replaces generated page controllers |
| `nextPage()` / `previousPage()` | Navigates and reports success |

Paginated classes are currently marked experimental.

## Items

Common builder members:

```java
amount(value)
displayName(String or Component)
lore(String... or List<Component>)
customModelData(value)
enchanted(value)
autoFlushUpdate(value)
placeholderEngine(engine)
build()
build(updateNow)
```

`ItemMeta` offers fluent getters and setters for the serializer, display name, lore, custom model data, and enchanted flag. Call `ItemWrapper.update(user)` after runtime metadata changes unless a controller update helper or auto-flush performs it.

## Events

`GuiClickEvent` exposes `rawSlot`, `playerInventory`, `InventoryAction`, and `ClickType`. Its click helpers include `isLeftClick`, `isRightClick`, `isShiftClick`, `isKeyboardClick`, and `isCreativeAction`.

`GuiDragEvent.rawSlots()` returns all raw slots touched by the drag. Both click and drag events extend `CancellableGuiEvent`.

## Packages

| Package prefix | Contents |
| --- | --- |
| `...jguiwrapper.api` | Shared entry point and module abstraction |
| `...jguiwrapper.api.gui` | Core GUI, holder, factory, events, and GUI types |
| `...jguiwrapper.api.item` | Shared items and metadata |
| `...jguiwrapper.api.text` | Serializers and placeholder API |
| `...jguiwrapper.paper.api` | Public Paper types |
| `...jguiwrapper.common` | Embedded Paper implementation |
| `...jguiwrapper.minestom` | Complete Minestom implementation |

## Behavioral cautions

- Initialize the platform API before constructing GUIs or item builders; constructors consult `GuiApi.get()`.
- Numeric sizes are clamped and rounded to chest rows.
- Changing a GUI's size or title does not automatically rebuild its holder.
- Handler-backed controllers resolve their global handler when registered; register the handler first.
- Loader and handler invocation is explicit, not automatic on open.
- Platform unwrapping with `as(Class<T>)` throws for an incompatible type.
- A serializer not present on the runtime classpath produces empty output and logs once.
- The current registry implementation requires globally unique component IDs across namespaces.
