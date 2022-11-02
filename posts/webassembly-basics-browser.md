---
title: WebAssembly in the Browser (Basics)
description: A simple example of how to use WebAssembly in the browser and what the toolchain looks like.
date: 2022-11-02
tags:
    - software
    - rust
    - webassembly
    - wasm

layout: layouts/post.njk
---


(As part of my WebAssembly explorations I'll be sharing some of my learnings as a mix of notes/tutorial/high level overview.
I won't detail every step, but I'll try to explain the concepts and the decisions at least at a high level.)

In this post I'll show you how to get started with WebAssembly in the browser using the Rust programming language to create the WebAssembly module. 

With WebAssembly, we have a way to run code created in other languages inside the browser. (You can also use it in the server but I'll focus on the browser here)

## The process (high level overview)

1. Write the WebAssembly module in a supported language (Rust, Go, C#, AssemblyScript, etc)
2. Compile (instructions depend on the selected language) and generate the binary file with .wasm extension
3. Include the .wasm file in your web application, it must be published with your web application
4. Load the file in JavaScript and run the WebAssembly exported methods.

Let's go through each step.

## Installing rust toolchain (optional)

(If you don't want do do it you can skip this step and just download the pre-built package here https://github.com/hugozap/rust-format-message-example-library/releases/download/v1/pkg.zip)

If you want to generate the WebAssembly module from Rust you need to install the Rust toolchain from (https://rustup.rs/). 

To install wasm-pack (a tool that will help us to generate the npm compatible package):

```bash
cargo install wasm-pack
```

## Writing the WebAssembly module (optional)


I'll use Rust for this example. I'll use the `wasm-bindgen` crate to generate the bindings between Rust and JavaScript.

The first step is to create a new Rust project with `cargo new rust-simple-library --lib`

Then we need to add the `wasm-bindgen` dependency to our `Cargo.toml` file

```toml
[dependencies]
wasm-bindgen = "0.2.83"
```

Now we can start writing our code. We'll create a simple library that returns an ascii message with the text we pass

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
fn format_message(t:&str)->String {

    let mut result = String::new();
    let mut max = 0;
    //find the longest line
    for line in t.lines() {
        if line.len() > max {
            max = line.len();
        }
    }
    //print the top line
    result.push_str(&"*".repeat(max+4));
    result.push_str("\n");
    //print the text
    for line in t.lines() {
        result.push_str("* ");
        result.push_str(line);
        //add spaces to the end of the line
        result.push_str(&" ".repeat(max-line.len()));
        result.push_str(" *\n");
    }
    //print the bottom line
    result.push_str(&"*".repeat(max+4));
    result.push_str("\n");
    return result;
}
```

## Compiling the WebAssembly module (optional)

Now we'll compile the code with `wasm-pack build --target web`

This will generate the `pkg` folder with the npm compatible package. We can include it from our web application now.


## Including the wasm module in our web application

We need the .wasm and js helper files to be published with our web application. Depending on the bundler we use, we'll have different ways to include it.

If you don't use a bundler you can compile the module with `--target no_modules` and include the .wasm file in your web site contents folder.

### Common bundler wasm plugins

For Webpack:
https://github.com/wasm-tool/wasm-pack-plugin

For Vite:
https://github.com/nshen/vite-plugin-wasm-pack/


## Using our wasm module from JS

I'll import the wasm module in my web application with `import init, { printTextInsideRectangle } from "rust-simple-library"`

This is possible because wasm-pack generated the bindings for us.

```js
import init, { format_message } from "rust-simple-library";

const runWasm = async () => {
    //init will fetch our .wasm file
    const wasm = await init();
    //let's call the exported method
    const result = format_message("Hello World!");
    console.log(result);
}

runWasm();
```

Note: We need to wrap our code in an async function and call it. This is because we need to wait for the wasm module to be loaded and top level await is not supported yet.

If we run this code we'll see the following output in the console

```
****************
* Hello World! *
****************
```

## What's going on under the hood?

If you open the js bindings (ascii_simple_message.js) file you'll see glue code that will call the WebAssembly object to instantiate the module and call the exported methods.

This is how the .wasm file is loaded

```js
async function init(input) {
    if (typeof input === 'undefined') {
        input = new URL('ascii_simple_message_bg.wasm', import.meta.url);
    }
    ...
}
```
It uses a simple fetch to load the .wasm file.

I recommend taking a look at the generated bindings to understand how it works and see the glue code generated by wasm-bindgen.It basically creates proxy objects for each exported method.

There are some helper methods to convert the arguments and return values to the correct types. This is required because only basic types can be passed/received to the WebAssembly module.

## Conclusion

This was a simple example to show how to get started with WebAssembly in the browser. There are many more things to explore like memory management, calling JavaScript from WebAssembly, serializing/deserializing objects, etc. 

I'll cover some of these topics in future posts.