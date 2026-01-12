# GGEZ
---
---

## Summary
GGEZ (Good Game Easy) is a lightweight 2D game framework inspired by LÖVE (Love2D). It is designed to be simple, getting out of your way so you can just write code. It is perfect for 2D prototypes, game jams, and learning game dev concepts without the complexity of an ECS.

## Detailed Explanation

### Core Philosophy
"Simplicity." GGEZ provides a basic event loop (update/draw), input handling, and drawing primitives. It doesn't force an architecture on you. You manage your own game state struct.

### Key Features
*   **Simple Loop**: Implement the `EventHandler` trait (update and draw).
*   **Hardware Acceleration**: Uses `wgpu` or OpenGL for fast 2D rendering.
*   **Portable**: Runs on Windows, Linux, and macOS.

### Code Example
*Dependencies: `ggez`*

```rust
use ggez::{Context, ContextBuilder, GameResult};
use ggez::event::{self, EventHandler};

struct MyGame {
    // Your state here
}

impl MyGame {
    pub fn new(_ctx: &mut Context) -> MyGame {
        MyGame { }
    }
}

impl EventHandler for MyGame {
    fn update(&mut self, _ctx: &mut Context) -> GameResult {
        // Update logic
        Ok(())
    }

    fn draw(&mut self, ctx: &mut Context) -> GameResult {
        let mut canvas = ggez::graphics::Canvas::from_frame(ctx, ggez::graphics::Color::WHITE);
        // Draw things
        canvas.finish(ctx)
    }
}

fn main() {
    let (mut ctx, event_loop) = ContextBuilder::new("my_game", "author").build().unwrap();
    let my_game = MyGame::new(&mut ctx);
    event::run(ctx, event_loop, my_game);
}
```

## Interview Questions

1.  **Q: When would you use GGEZ over Bevy?**
    *   **A:** Use GGEZ if you want to make a simple 2D game and don't want the cognitive overhead of learning an Entity Component System. It's much closer to the metal and easier to reason about for small scopes.

2.  **Q: Does GGEZ support 3D?**
    *   **A:** Not really. While you can technically access the underlying graphics context, GGEZ is explicitly designed and optimized for 2D.
