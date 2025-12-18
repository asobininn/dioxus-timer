# dioxus_timer

Simple timer that works with dioxus.  
Provide use_timer.

## Setup

Add the following crate

```bash
cargo add dioxus-timer
```

## Dioxus support table

| dioxus | dioxus-timer |
| ------ | ------------ |
| ^0.7   | 0.5          |
| ^0.6   | 0.4          |
| ^0.5   | 0.3          |
| ^0.4   | 0.2          |

## Example

```rust
use dioxus::prelude::*;
use dioxus_timer::{use_timer, DioxusTimer, TimerState};
use std::{collections::HashMap, time::Duration};

fn main() {
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

fn formdata_to_map(data: &[(String, FormValue)]) -> HashMap<String, String> {
    let mut map = HashMap::new();
    for (key, value) in data {
        if let FormValue::Text(text) = value {
            map.insert(key.to_owned(), text.to_owned());
        }
    }
    map
}

#[component]
fn TimerSet(timer: Signal<DioxusTimer>) -> Element {
    let submit_handle = move |ev: Event<FormData>| {
        ev.prevent_default();
        let values = formdata_to_map(&ev.values());
        let hours = values["hours"].parse::<u64>().unwrap();
        let minutes = values["minutes"].parse::<u64>().unwrap();
        let seconds = values["seconds"].parse::<u64>().unwrap();
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
