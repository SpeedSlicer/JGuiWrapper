# Items, text, and placeholders

## Choose an item wrapper

`ItemWrapper` is the shared representation. Platform wrappers provide native material and item-stack access:

| Platform | Wrapper | Native access |
| --- | --- | --- |
| Paper | `PaperItemWrapper` | Bukkit `Material` and `ItemStack` |
| Minestom | `MinestomItemWrapper` | Minestom `Material` and immutable `ItemStack` |
| Shared code | `ItemWrapper` | String material ID |

Prefer a platform wrapper when platform types are already available:

```java
PaperItemWrapper icon = PaperItemWrapper.builder(Material.DIAMOND)
        .amount(3)
        .displayName("&bDiamonds")
        .lore("&7Click to inspect", "&8Three items")
        .customModelData(1201)
        .enchanted(true)
        .build();
```

```java
MinestomItemWrapper icon = MinestomItemWrapper.builder(Material.DIAMOND)
        .amount(3)
        .displayName(Component.text("Diamonds"))
        .enchanted(true)
        .build();
```

A generic wrapper can be adapted when it is placed into a platform holder:

```java
ItemWrapper item = ItemWrapper.builder("stone")
        .displayName("&fStone")
        .build();
```

The ID must resolve to a valid platform material. Paper uses `Material.matchMaterial`; Minestom uses `Material.fromKey`. Invalid IDs throw `IllegalArgumentException` during adaptation.

## Update lifecycle

The builder calls `update()` by default, which renders the wrapper metadata into its native item stack.

After construction, changing wrapper fields or metadata marks the item as needing an update. There are two update modes:

```java
item.meta().displayName("&aNew name");
item.update(user); // resolves placeholders for this user and updates the native stack
```

```java
item.autoFlushUpdate(true);
item.amount(2); // automatically invokes update()
```

`update(user)` uses temporary processed components, so placeholder output is rendered into the native item without replacing the original template stored in `ItemMeta`. This allows the same template to be processed for different users.

Controller helpers such as `updateItems` call `update(user)` when an item is dirty, then redraw the relevant slots.

For platform wrappers, post-construction metadata and the platform-specific `material(...)` setter are applied to the native stack during update. The shared `id(...)` and `amount(...)` setters do not currently rebuild a Paper or Minestom native stack; construct a new wrapper or replace the native stack when changing those values after creation.

## Serializers

JGuiWrapper represents text with Adventure `Component`. String convenience methods use a `SerializerType`:

| Serializer | Input style |
| --- | --- |
| `LEGACY_AMPERSAND` | `&aGreen` |
| `LEGACY_SECTION` | Section-sign legacy formatting |
| `MINI_MESSAGE` | `<green>Green</green>` |
| `PLAIN` | Plain text |
| `JSON` | Adventure JSON |
| `GSON` | Gson-based Adventure JSON |

The default is `LEGACY_AMPERSAND`. Set another default after API initialization:

```java
GuiApi.get().defaultSerializer(SerializerType.MINI_MESSAGE);
```

You can also choose a serializer per GUI or item:

```java
PaperItemWrapper item = PaperItemWrapper.builder(
        Material.EMERALD,
        SerializerType.MINI_MESSAGE
).displayName("<green>Emerald</green>").build();
```

Serializer implementations are discovered from the runtime classpath. If an Adventure serializer module is unavailable, deserialization returns an empty component and serialization returns an empty string, with a one-time warning after the API is initialized.

## Placeholders

Create an engine after JGuiWrapper has been initialized:

```java
PlaceholderEngine placeholders = PlaceholderEngine.of();
```

Register literal replacements:

```java
placeholders.register("%server_name%", "Example Network");
placeholders.register("%balance%", user -> lookupBalance(user));
```

Literal values are exact substring matches. Include delimiters such as `%...%` in the registered key when your item text uses them.

Register a regular-expression replacement when the matched text carries an argument:

```java
placeholders.registerRegex(
        "%uppercase:([^%]+)%",
        (match, user) -> match
                .substring("%uppercase:".length(), match.length() - 1)
                .toUpperCase()
);
```

Attach the engine to an item template:

```java
PaperItemWrapper profile = PaperItemWrapper.builder(Material.PLAYER_HEAD)
        .displayName("&e%server_name%")
        .lore("&7Balance: &a%balance%")
        .placeholderEngine(placeholders)
        .build(false);

profile.update(PaperGuiApi.get().user(player));
```

Use `build(false)` when placeholder resolution requires a user and the initial user-neutral render is not useful.

### Platform processing

After custom replacements, the platform engine performs its platform-specific pass:

- Paper delegates to PlaceholderAPI when it is installed and a Paper user is supplied.
- Minestom performs only the replacements registered with the JGuiWrapper engine.

You can merge engines with `addAll`. Both engines must be JGuiWrapper engine implementations created by the active platform API.
