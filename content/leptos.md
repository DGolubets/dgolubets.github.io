+++
title = "Web UI in Rust with Leptos"
template = "leptos-address-parser.html"
date = 2024-10-23
weight = 0
[taxonomies] 
tags = ["rust", "leptos", "frontend"]
+++

Rust was originally created to be a system programming language. 
But it turned out rather quickly there is much more potential to it. 
Nowdays it's a truly general purpose language, perfectly suitable for building various applications: REST APIs, game engines and more.

Nevertheless, I was surprised to discover a frontend/fullstack web framework in Rust, a niche usually dominated by JavaScript. That framework is [Leptos](https://leptos.dev/).

So I gave it a try.
<!-- more -->

## Prelude
At work we analyze UK property market.
There are different address and property databases available, but none of them is perfect.
We have put a lot of work to combine different sources to build the most complete database.
That involves various techniques of address strings parsing and matching to correctly combine different records.

To facilitate that process I developed an utility to parse an address string into a structure.
I chose Rust, because it has:

- great tools for string processing: 
    - nice parsing combinator libraries
    - great implementation of regex
    - access to strings via references and slices allows to work with them in very efficient manner
- strong static type system that gives overall confidence in complex code

The tool worked, I was quite happy with the result and left it there.
That was a few years ago. 

However there was something I wanted to build alongside the address parsing tool, but never did before recently - a UI for it. 
The moment I discovered Leptos, I thought: "That's what I'll spend next evening working on!".

## Building address parsing UI

I will describe my experience and takeaways building that UI.

### Principles

Leptos should be very easy to pick up for anyone who's familiar with React model. 
It's based on the same simple concepts:

- you have **pure functions** describing your components markup
- to keep state or run effects you use **signals** (similar to **hooks**)

Leptos provides a convinient macros to define the markup, making components' code surprisingly easy to understand.

Worth mentioning all that works on both stable and nightly! Though on nightly there is a more convinient way to use signals, whereas on stable you have to use `.get` and `.set` methods working with them. 

### Code example
Here is how I defined the root component:
```Rust
#[component]
pub fn App(default_address: String, example_address: String) -> impl IntoView {
    let (text, set_text) = create_signal(default_address);

    view! {
        <div class="address-parser">
            <h1>Address Parser</h1>
            <input
                class="address-text"
                type="text"
                placeholder="Enter an address..."
                value=text.get()
                on:input=move |e| {
                    set_text.set(event_target_value(&e));
                }
            />
            {move || {
                let text = text.get();
                let mut addresses = parse(&text).unwrap_or_else(|_| vec![]);
                addresses.reverse();
                if addresses.is_empty() {
                    return vec![
                        view! {
                            <div class="no-results">
                                No results. <br />Make sure the address is a valid UK address, like:
                                <pre>{&example_address}</pre>
                            </div>
                        }
                            .into_view(),
                    ];
                }
                addresses
                    .iter()
                    .map(|address| {
                        view! { <AddressDetails text=&text address=&address /> }
                    })
                    .collect::<Vec<_>>()
            }}
        </div>
    }
}
```

Pretty self explanatory, isn't it?

### Web assembly

Now's the cool part: Leptos compiles your code into [web assembly](https://webassembly.org/).

For my little UI tool that means I don't even have to use any web-server, I can just run my parsing algorithm in the browser!

### Demo
Here is a live demo of the UI.
(Using parser stub, to not leak our internal product).


