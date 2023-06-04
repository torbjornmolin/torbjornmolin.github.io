---
title: "Kiss 3d"
date: 2023-06-04T10:19:31+02:00
draft: false
---

# Kiss3d

A while back I was browsing [Are we game yet?](https://arewegameyet.rs) looking for a library to easily draw some 3D objects on screen and I came across [Kiss3d](https://github.com/sebcrozet/kiss3d) which does just that in an easy way.

I've always been a person who wants to learn things with a from-the-ground-up approach. It really is the shift left of learning. On the one hand, if you're successful, you'll build a solid foundation. On the other hand, it can quickly become overwhelming and take all the joy out of the process.

When I look back at how I've been most effective at learning over the years I realize that it's always been with the other approach, by starting small and deep diving when running in to problems to solve. Kiss3d, therefore, is perfect for me at this stage in learning Rust an 3D graphics programming.

Here's some ugly code that draws a cone of spheres:

```rust
extern crate kiss3d;

use std::collections::VecDeque;

use kiss3d::light::Light;
use kiss3d::nalgebra::{Point3, Translation3};
use kiss3d::scene::SceneNode;
use kiss3d::window::Window;

fn main() {
    let mut window = Window::new("Kiss3d");
    let mut deq: VecDeque<SceneNode> = VecDeque::new();
    let mut pos: f32 = 0.0;
    window.set_background_color(1.0, 1.0, 1.0);
    while window.render() {
        window.set_light(Light::Absolute(Point3::new(pos * 2.0, 0.0, 0.0)));
        let radius = 4.0;
        if pos < 100.0 {
            let mut c = window.add_sphere(radius);
            c.set_color(1.0, 0.0, 1.0);
            let t = Translation3::new(pos, pos * pos.sin(), pos * pos.cos());
            c.append_translation(&t);
            pos = pos + 1.0;
            deq.push_back(c);
        } else if pos < 200.0 && pos < 300.0 {
            pos = pos + 1.0;
            let c = deq.pop_front();
            match c {
                Some(mut sn) => window.remove_node(&mut sn),
                None => todo!(),
            }
        } else {
            pos = 0.0;
        }
    }
}
```
