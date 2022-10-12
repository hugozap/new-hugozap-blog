---
title: Using Constraint Solvers with Rust and WebAssembly (Exploration)
description: 
date: 2022-10-12
tags:
  - software
  - rust
  - ui
layout: layouts/post.njk
---

Yesterday I spent some time experimenting with linear constraint solvers using Rust and WebAssembly.

([Demo repository](https://github.com/hugozap/rust-constraint-solver-wasm-demo))

What is a constraint solver?

The basic idea is that you specify a problem in terms of restrictions. This restrictions should be expressed as linear expressions.

UI Layouts are a good fit for linear constraint systems, because you can express the relations among elements in terms of these linear expressions.

- the top of the box2 element should be equal to the bottom of box3
- the x position of box4 should be equal to the total width/2
- the total width is >= than box2
- the total width is >= than box5

When you have the set of expressions you ask the solver to generate a solution and it will try to produce an optimized set of values that satisfy the constraints.

When a variable changes, the solver will re compute the solution again.

## Expressing problems as constraints

I think this way of approaching problems is very interesting and aligned with the current trends of AI assisted workflows, that's one of the reasons I find it useful to improve the problem definition skills and defining constraints is one way to do it.

## Simple experiment with WebAssembly

As part of my Rust and WebAssembly learning journey I like to experiment with topics I find interesting, so let's try to run a solver in WASM and use it from JavaScript.

## Creating the rust project

I used `wasm-pack new` to generate the starter project for my library. It will export a struct that JS will consume and will contain a method to compute the layout.

Then, I needed to install the cassowary rust crate. At this point I wasn't sure if it would work in Wasm.

(Rust project dependencies)
```
[dependencies]
wasm-bindgen = "0.2.63"
cassowary = "0.3.0"
js-sys = "0.3.44"
```


## Using the cassowary-rust library

There's an implementation of the cassowary algorithm to find solutions to a set of constraints.
The high level idea is:

- Create a new solver instance
- Create variables (with `Variable::new()`)

Our model only has 2 points, each point with a location. Note that we use Variable for the values, because they will be used by the solver.

```rust
    let mut names = HashMap::new();
    let window_width = Variable::new();
    names.insert(window_width, "window_width");

    let p1 = Point {
        x: Variable::new(),
        y: Variable::new(),
    };

    let p2 = Point {
        x: Variable::new(),
        y: Variable::new(),
    };

    names.insert(p1.x, "p1.x");
    names.insert(p1.y, "p1.y");
    names.insert(p2.x, "p2.x");
    names.insert(p2.y, "p2.y");
```


- Add the constraints (EQ is the equals operator, there's also support operators for inequalities)

```rust
 solver.add_constraints(&[
                p1.x | EQ(REQUIRED) | 0.0,
                p1.y | EQ(REQUIRED) | 0.0,
                p2.x | EQ(REQUIRED) | window_width,
                p2.y | EQ(REQUIRED) | window_width/2.0,
            ])
            .unwrap();
```

Our constrained system is simple, there are 2 points and the location of point 2 is determined by the value of the `window_width` variable. If `window_width` variable changes, p2.y will be updated.

- Set known values

We know the window_width variable so let's tell the solver we want to start with a value for the window_width variable (first we add it as editable)

```rust
solver.add_edit_variable(window_width, STRONG).unwrap();
solver.suggest_value(window_width, 100.0).unwrap();
 ```

To retrieve the calculated values we use the solver `get_value` method.
Here we populate a hash map with (name,value) pairs.


```rust
    let mut result = HashMap::new();
    for (var, name) in names {
        result.insert(String::from(name), solver.get_value(var));
    }
```

## Testing that the solver works

Let's add a simple unit test to make sure we set the solver parameters correctly

```rust
    #[cfg(test)]
    mod tests {
        use super::*;

        #[test]
        fn test_calculateLayout() {
            let app = App::new();
            let result = app.calculateLayout(); 
            assert_eq!(result.get("p1.x"), Some(&0.0));
            assert_eq!(result.get("p1.y"), Some(&0.0));
            assert_eq!(result.get("p2.x"), Some(&100.0));
            assert_eq!(result.get("p2.y"), Some(&50.0));
        }
    }
```

## How to return the values to JS?

With WebAssembly we have different options to return data.

- Return an array of bytes
- Return a pointer and let JS access the WebAssembly memory to retrieve the values
- Serialize to json and return the string

I'll use the 2nd approach because it's the fastest. In terms of performance we don't want to transfer a lot of data between JS and Wasm. A pointer is very small and because JS has access to the WebAssembly memory this will be very fast!

To return the pointer we have this method

```rust
    #[wasm_bindgen]
        pub fn get_points(&self) -> *const PointLocation {
            self.points.as_ptr()
    }
```

PointLocation has this shape

```rust
    #[wasm_bindgen]
    struct PointLocation {
        x: f64,
        y: f64,
    }
```

## Generating the WebAssembly package

With `wasm-pack build --target web` the npm compatible package will be generated in the `pkg` folder.
We can include it from our web application now.

I used the `Vite` Bundler, but this is compatible with most bundlers.

## Using our wasm module from JS

After generating the package I added it as a dependency in my web application.
This step is different for each bundler (vanilla JS is also supported).

Now in my web application I can access the exported Rust structs and methods

```js
    import init, { App } from "rust-constraint-solver-wasm";

    const runWasm = async () => {
    //init will fetch our .wasm file
    const wasm = await init();
    //Let's create an instance of the exported App struct (defined in rust)
    const app = new App();

    //the main method that will run the solver
    //for demo purposes we are not passing any parameter. In real life 
    //we should pass the initial values (e.g the width of the window, positions, etc)
    //to get the updated values
    app.update_locations();
    
    //We didn't return the values from update_locations,
    //just updated the memory with the values of interest, 
    //now we need to access the wasm memory space and retrieve them.
    const points_ptr = app.get_points();
    const points = new Float64Array(wasm.memory.buffer, points_ptr, 4);
    console.log({ points });
};
```

When running the app you'll see in the log, the list of updated values.
Not very exciting but this opens the opportunity to solve much more interesting problems that can be expressed with linear constraints.

[console output](/img/cassowary-experiment-log.png)

## What's next

Now that we have a proof of concept, we could try creating layout systems for Canvas, WebGL. I see some potential for layout systems that offer a more natural way to arrange elements. 