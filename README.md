# dioxus_timer
Simple timer that works with dioxus.<br>
Provide use_timer and use_shared_timer.

## Setup
Add the following crate
```
cargo add dioxus-timer
```

### If dioxus-web, also add
```
cargo add instant -F "wasm-bindgen"
cargo add async_std
```

## Example
```rust
use dioxus::prelude::*;
use dioxus_timer::{DioxusTimer, TimerState};

#[cfg(target_arch = "wasm32")]
use instant::{Duration, Instant};
#[cfg(not(target_arch = "wasm32"))]
use std::time::{Duration, Instant};

#[component]
fn App(cx: Scope) -> Element {
    dioxus_timer::use_shared_timer(cx, Duration::from_millis(16));
    let timer = use_shared_state::<DioxusTimer>(cx)?;

    let state = timer.read().state();
    use_effect(cx, &state, |_| async move {
        if state == TimerState::Finished {
            println!("finished!");
        }
    });

    let time = timer.read().to_string();
    render! {
        div {
            h1 { "Timer" }
            p { "{time}" }
            p { "{state}" }
            TimerControll {}
            TimerSet {}
        }
    }
}

#[component]
fn TimerControll(cx: Scope) -> Element {
    let timer = use_shared_state::<DioxusTimer>(cx)?;

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
        dioxus_timer::TimerState::Inactive => rsx! {
            button { onclick: start_handle, "Start" }
            button { onclick: reset_handle, "Reset" }
        },
        dioxus_timer::TimerState::Working => rsx! {
            button { onclick: pause_handle, "Pause" }
            button { onclick: reset_handle, "Reset" }
        },
        dioxus_timer::TimerState::Paused => rsx! {
            button { onclick: start_handle, "Resume" }
            button { onclick: reset_handle, "Reset" }
        },
        dioxus_timer::TimerState::Finished => rsx! {
            button { onclick: reset_handle, "Reset" }
        },
    };
    cx.render(controller)
}

#[component]
fn TimerSet(cx: Scope) -> Element {
    let timer = use_shared_state::<DioxusTimer>(cx)?;

    let submit_handle = move |evt: FormEvent| {
        let hours = evt.values["hours"][0].parse::<u64>().unwrap();
        let minutes = evt.values["minutes"][0].parse::<u64>().unwrap();
        let seconds = evt.values["seconds"][0].parse::<u64>().unwrap();
        let preset_dur = instant::Duration::from_secs(hours * 3600 + minutes * 60 + seconds);
        timer.write().set_preset_time(preset_dur);
    };

    render! {
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