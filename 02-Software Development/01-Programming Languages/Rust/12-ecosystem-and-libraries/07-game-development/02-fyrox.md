# Fyrox
---
---

## Summary
Fyrox (formerly rg3d) is a production-ready, feature-rich game engine. Unlike Bevy which is code-centric, Fyrox offers a powerful **Scene Editor** (similar to Unity or Godot), making it a strong choice for developers who prefer a visual workflow for level design and asset management.

## Detailed Explanation

### Core Philosophy
Fyrox aims to provide a complete, integrated development environment. It believes that while coding is essential, visual tools are critical for productivity in 3D scene composition, UI layout, and animation blending.

### Key Features
*   **Scene Editor**: A GUI tool to place objects, configure lights, and set up physics.
*   **Animation System**: Advanced state machine for 3D animations.
*   **UI System**: Robust UI toolkit for game menus.
*   **Scripting**: Write game logic in Rust.

### Use Cases
*   **3D Games**: Shooters, RPGs, where level design is complex.
*   **Visual-Heavy Projects**: Projects requiring precise placement of assets.

## Interview Questions

1.  **Q: How does Fyrox differ from Bevy?**
    *   **A:** Bevy is currently more of a "framework" or code-first engine without a native editor (though 3rd party ones exist). Fyrox provides a full-fledged editor experience out of the box, similar to Unity, making it better suited for teams that include level designers who might not code.

2.  **Q: Is Fyrox purely 3D?**
    *   **A:** No, Fyrox supports both 2D and 3D game development, though its feature set is particularly strong for 3D rendering and physics.
