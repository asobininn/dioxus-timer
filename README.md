# dioxus_timer
Simple timer that works with dioxus.<br>
Provide use_timer.

## Setup
Add the following crate
```
cargo add dioxus-timer
```

### If dioxus-web, also add
```
cargo add async_std
```

## Dioxus support table
| dioxus | dioxus-timer |
| ------ | ------------ |
| ^0.5   | 0.3          |
| ^0.4   | 0.2          |

## Example
```rust
#![allow(non_snake_case)]
use dioxus::prelude::*;
use dioxus_timer::{use_timer, DioxusTimer, TimerState};
use std::time::Duration;
use tracing::Level;

fn main() {
    dioxus_logger::init(Level::INFO).expect("failed to init logger");
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    let timer = use_timer(Duration::from_millis(16));

    let state = timer.read().state();
    use_effect(move || {
        if state == TimerState::Finished {
            println!("finished!");
        }
    });

    let time = timer.read().to_string();
    rsx! {
        div {
            h1 {"Timer"}
            p {"{time}"}
            p {"{state}"}
            TimerControll {timer}
            TimerSet {timer}
        }
    }
}

#[component]
fn TimerControll(timer: Signal<DioxusTimer>) -> Element {
    let start_handle = move |_| {
        timer.write().start();
    };
    let pause_handle = move |_| {
        timer.write().pause();
    };
    let reset_handle = move |_| {
        timer.write().reset();
    };

    let controller = match timer.read().state() {
        TimerState::Inactive => rsx! {
            button { onclick: start_handle, "Start" }
            button { onclick: reset_handle, "Reset" }
        },
        TimerState::Working => rsx! {
            button { onclick: pause_handle, "Pause" }
            button { onclick: reset_handle, "Reset" }
        },
        TimerState::Paused => rsx! {
            button { onclick: start_handle, "Resume" }
            button { onclick: reset_handle, "Reset" }
        },
        TimerState::Finished => rsx! {
            button { onclick: reset_handle, "Reset" }
        },
    };
    controller
}

#[component]
fn TimerSet(timer: Signal<DioxusTimer>) -> Element {
    let submit_handle = move |ev: Event<FormData>| {
        let values = ev.values();
        let hours = values["hours"].first().unwrap().parse::<u64>().unwrap();
        let minutes = values["minutes"].first().unwrap().parse::<u64>().unwrap();
        let seconds = values["seconds"].first().unwrap().parse::<u64>().unwrap();
        let preset_dur = Duration::from_secs(hours * 3600 + minutes * 60 + seconds);
        timer.write().set_preset_time(preset_dur);
    };

    rsx! {
        div {
            form {
                onsubmit: submit_handle,
                label { "hour:" }
                input { r#type: "number", name: "hours", value: "0", min: "0", max: "23" }
                label { "minutes:" }
                input { r#type: "number", name: "minutes", value: "0", min: "0", max: "59" }
                label { "seconds:" }
                input { r#type: "number", name: "seconds", value: "0", min: "0", max: "59" }
                button { r#type: "submit", "set" }
            }
        }
    }
}
```