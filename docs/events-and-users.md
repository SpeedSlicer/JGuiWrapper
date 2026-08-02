# Events and users

JGuiWrapper wraps platform events so most menu logic can use one API on Paper and Minestom.

## Event model

| Event | Important data | Cancellable |
| --- | --- | --- |
| `GuiOpenEvent` | GUI and user | No |
| `GuiCloseEvent` | GUI and user | No |
| `GuiClickEvent` | Raw slot, player-inventory flag, action, click type | Yes |
| `GuiDragEvent` | Raw slots touched by the drag | Yes |

Every event exposes:

```java
event.gui();
event.user();
```

Paper and Minestom listeners propagate `setCancelled(true)` back to the underlying click or drag event.

## Click handling

Use slot handlers for most simple menus:

```java
setClickHandlers((event, gui) -> {
    event.setCancelled(true);

    if (event.clickType().isRightClick()) {
        event.user().sendMessage("Right click");
    }

    if (event.action() == GuiClickEvent.InventoryAction.MOVE_TO_OTHER_INVENTORY) {
        event.user().sendMessage("Shift transfer blocked");
    }
}, 10, 11, 12);
```

`rawSlot()` is relative to the combined inventory view. A raw slot below `gui.size()` is in the GUI's top inventory. Check `playerInventory()` when handling clicks from the player's inventory.

Simple GUIs have `cancelEmptySlots` enabled by default. Consequently:

- clicks in top-inventory slots without a handler are cancelled;
- shift moves and collect-to-cursor actions from the player inventory are cancelled;
- drags touching any top-inventory slot are cancelled.

Handlers are responsible for cancellation when they exist. Call `event.setCancelled(true)` in button handlers unless moving items is intentional.

## Lifecycle consumers

Register lifecycle callbacks in a constructor or setup method:

```java
onOpen(event -> event.user().sendMessage("Menu opened"));
onClose(event -> event.user().sendMessage("Menu closed"));
onDrag(event -> {
    if (event.rawSlots().contains(0)) {
        event.setCancelled(true);
    }
});
```

The current Paper listener dispatches close, click, and drag events. `PaperGuiBase` dispatches `GuiOpenEvent` only when the active NMS wrapper opens the inventory and returns a native view; the non-NMS Bukkit fallback does not currently synthesize it. Programmatic close invokes the GUI close callback before closing the Bukkit inventory and filters the resulting Bukkit close event to avoid a second callback.

The current Minestom listener dispatches client close, click, and drag events. Opening a Minestom GUI does not currently synthesize `GuiOpenEvent`. For portable behavior, run per-user setup explicitly before `open` instead of relying on the open callback.

## Access the platform event

`GuiEvent.as(Class<E>)` unwraps the native event:

```java
InventoryClickEvent paperEvent = event.as(InventoryClickEvent.class);
```

```java
InventoryPreClickEvent minestomEvent = event.as(InventoryPreClickEvent.class);
```

Requesting an incompatible class throws `IllegalStateException`. Keep unwraps in platform-specific code.

## Users

`User` supports platform-neutral messages:

```java
event.user().sendMessage("Plain message");
event.user().sendMessage(Component.text("Adventure message"));
```

Unwrap the player when platform APIs are needed:

```java
Player paperPlayer = event.user().as(Player.class);
```

For Minestom, import `net.minestom.server.entity.Player` instead. Requesting the wrong type throws `IllegalStateException`.

You can create a wrapper outside an event:

```java
User paperUser = PaperGuiApi.get().user(player);
MinestomUser minestomUser = MinestomGuiApi.get().user(player);
```

## Threading

JGuiWrapper does not make inventory mutations thread-safe. Follow the platform's threading rules.

`PaperGuiBase` includes `runTask` and `runTaskAsync` convenience methods. Schedule Bukkit inventory changes back onto the server thread. Minestom code should follow Minestom's instance and scheduler requirements.
