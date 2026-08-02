# GUI types

JGuiWrapper has three practical GUI styles. Choose the smallest one that matches the menu's behavior.

| Type | Best for | Key abstraction |
| --- | --- | --- |
| Simple | Small menus with direct slot callbacks | `PaperGuiBase` / `MinestomGuiBase` |
| Advanced | Reusable regions, dynamic items, data loading | Named `AdvancedGuiItemController` instances |
| Paginated | Collections spread across multiple screens | Pages of item-controller builders |

## Simple GUIs

A simple GUI subclasses the platform base and sets items and handlers directly. This Paper example protects all GUI slots by default and gives slot `4` a callback:

```java
import com.jodexindustries.jguiwrapper.paper.api.gui.types.PaperGuiBase;
import com.jodexindustries.jguiwrapper.paper.api.item.PaperItemWrapper;
import net.kyori.adventure.text.Component;
import org.bukkit.Material;

public final class ConfirmGui extends PaperGuiBase<ConfirmGui> {
    public ConfirmGui() {
        super(9, Component.text("Confirm"), null);

        holder().setItem(4, PaperItemWrapper.builder(Material.LIME_DYE)
                .displayName("&aConfirm")
                .build());

        setClickHandlers((event, gui) -> {
            event.setCancelled(true);
            event.user().sendMessage("Confirmed.");
            gui.close(event.user());
        }, 4);
    }
}
```

Open it with a Paper player:

```java
new ConfirmGui().open(player);
```

For Minestom, extend `MinestomGuiBase<YourGui>` and use `MinestomItemWrapper` or a generic `ItemWrapper` with a valid Minestom material ID.

### Handler rules

- `setClickHandlers(handler, slots...)` assigns one handler to the supplied raw slots.
- Calling `setClickHandlers(handler)` with no slots assigns it to every GUI slot.
- `removeClickHandlers(slots...)` removes selected handlers.
- Calling `removeClickHandlers()` with no slots removes every handler.
- Empty GUI slots are protected by default. Call `setCancelEmptySlots(false)` to opt out.
- Dragging across any GUI slot is cancelled by default.

You can override `onClick`, `onOpen`, `onClose`, or `onDrag`, or register consumers with `onOpen(...)`, `onClose(...)`, and `onDrag(...)`. When overriding a method on a class that already supplies behavior, call `super` if you want its registered consumers and default protection to continue running.

## Advanced GUIs

An advanced GUI groups one or more slots into a named controller. A controller can have:

- one default item and click handler for all its slots;
- item or click-handler overrides for individual slots;
- a registered item handler for data-driven setup;
- runtime methods for changing slots, items, and callbacks.

Create one directly or through the factory:

```java
PaperAdvancedGui gui = PaperGuiApi.get().guiFactory().create(
        GuiType.ADVANCED,
        GuiOptions.builder()
                .size(54)
                .title(Component.text("Shop"))
                .build()
);
```

Register a controller:

```java
gui.registerItem("border", builder -> builder
        .slots(0, 1, 2, 3, 4, 5, 6, 7, 8)
        .defaultItem(PaperItemWrapper.builder(Material.GRAY_STAINED_GLASS_PANE)
                .displayName(Component.empty())
                .build())
        .defaultClickHandler((event, controller) -> event.setCancelled(true)));
```

Add slot-specific behavior without creating another controller:

```java
gui.registerItem("actions", builder -> builder
        .slots(20, 24)
        .defaultItem(PaperItemWrapper.builder(Material.PAPER).build())
        .slotItem(20, PaperItemWrapper.builder(Material.EMERALD)
                .displayName("&aBuy")
                .build())
        .slotClickHandler(20, (event, controller) -> {
            event.setCancelled(true);
            event.user().sendMessage("Purchase selected.");
        }));
```

Controller keys are unique within one GUI. Registering a second controller with the same key leaves the existing controller unchanged. A slot can belong to only one controller; assigning an occupied slot throws `IllegalArgumentException`.

### Updating controllers

Lookup by key or slot returns an `Optional`:

```java
gui.getController("actions").ifPresent(controller -> {
    controller.updateItems(item -> item.meta().lore("&7Updated now"));
});
```

Useful controller operations include:

| Method | Effect |
| --- | --- |
| `setSlots(...)` | Replaces the managed slot set and redraws it |
| `addSlot(slot)` / `removeSlot(slot)` | Changes one managed slot |
| `defaultItem(item)` | Replaces the fallback item for all managed slots |
| `setItem(slot, item)` | Adds a slot-specific item |
| `updateItems(updater, user)` | Mutates, updates, and redraws all controller items |
| `updateItem(slot, updater, user)` | Mutates, updates, and redraws one resolved item |
| `clear()` | Clears inventory contents and click handlers for managed slots |

## Paginated GUIs

`PaginatedAdvancedGui` on Paper and `MinestomPaginatedGui` on Minestom are experimental advanced GUIs with page management.

This Paper example reserves row six for navigation and creates three content pages:

```java
PaginatedAdvancedGui gui = new PaginatedAdvancedGui(
        54,
        Component.text("Catalog"),
        null
);

gui.registerItem("previous", builder -> builder
        .slots(45)
        .defaultItem(PaperItemWrapper.builder(Material.ARROW)
                .displayName("&ePrevious")
                .build())
        .defaultClickHandler((event, controller) -> {
            event.setCancelled(true);
            gui.previousPage();
        }));

gui.registerItem("next", builder -> builder
        .slots(53)
        .defaultItem(PaperItemWrapper.builder(Material.ARROW)
                .displayName("&eNext")
                .build())
        .defaultClickHandler((event, controller) -> {
            event.setCancelled(true);
            gui.nextPage();
        }));

for (int page = 0; page < 3; page++) {
    final int pageNumber = page;
    gui.addPage(builder -> builder
            .slots(22)
            .defaultItem(PaperItemWrapper.builder(Material.BOOK)
                    .displayName("&fPage " + (pageNumber + 1))
                    .build())
            .defaultClickHandler((event, controller) -> {
                event.setCancelled(true);
                event.user().sendMessage("Page " + (pageNumber + 1));
            }));
}

gui.openPage(0);
gui.open(player);
```

Page indexes are zero-based. `nextPage()` and `previousPage()` return `true` only when navigation succeeded. `openPage(index)` quietly ignores an index outside the available range.

Each page is a list of controller builders. Opening a page unregisters the current page's generated controllers and registers the new page's controllers. Keep permanent navigation and decoration under your own non-page controller keys.

## Titles and holder updates

Changing `title(...)` updates the GUI model, but does not by itself replace the already-open platform inventory.

On Paper, `updateMenu(...)` asks the optional NMS wrapper to update viewers in place. Without a compatible `nms` artifact, these methods return `false` or perform no in-place update. `updateHolder()` closes viewers, replaces the backing holder, and reopens the GUI; it is experimental.

On Minestom, opening with `open(player, title)` sends the requested title. `updateHolder()` recreates the backing inventory and reopens its viewers.
