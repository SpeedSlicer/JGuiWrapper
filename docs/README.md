# JGuiWrapper documentation

This documentation describes the API represented by the current source tree and release version `1.0.0.9`.

## Start here

1. [Install JGuiWrapper and create your first GUI](getting-started.md).
2. Choose between [simple, advanced, and paginated GUIs](gui-types.md).
3. Build platform-native [items, text, and placeholders](items-and-text.md).
4. Handle [GUI lifecycle events and platform users](events-and-users.md).

## Deeper topics

- [Registries, data loaders, and item handlers](registries-and-loaders.md)
- [API reference and behavioral notes](api-reference.md)
- [Repository architecture and local development](development.md)

## Concepts at a glance

| Concept | Responsibility |
| --- | --- |
| `GuiApi` | Initialized platform entry point, serializer, placeholder factory, and GUI factory |
| `Gui` | Opens, closes, updates, and exposes a holder |
| `GuiHolder` | Owns the platform inventory and changes items by slot |
| `SimpleGui` | Lifecycle consumers and direct slot click handlers |
| `AdvancedGui` | Named item controllers, data loaders, and item handlers |
| Paginated GUI | Adds pages and next/previous navigation to an advanced GUI |
| `ItemWrapper` | Platform-adaptable item data, display metadata, and placeholder processing |
| `User` | Platform-neutral messaging and access to the underlying player object |

JGuiWrapper uses zero-based raw inventory slots. A 27-slot chest therefore has GUI slots `0` through `26`.
