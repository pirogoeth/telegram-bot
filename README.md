Rust Telegram Bot Library
=========================
[![Build Status](https://img.shields.io/travis/telegram-rs/telegram-bot/master.svg)](https://travis-ci.org/telegram-rs/telegram-bot)
[![License](https://img.shields.io/github/license/telegram-rs/telegram-bot.svg)]()
[![Crates.io](https://img.shields.io/crates/v/telegram-bot.svg)](https://crates.io/crates/telegram-bot)

<table>
  <tbody>
    <tr>
      <td><b>Documentation:</b></td>
      <td><a href="https://docs.rs/telegram-bot/">Latest crates.io version</a></td>
      <td><a href="https://telegram-rs.github.io/telegram-bot/telegram_bot/"><code>master</code></a></td>
    </tr>
  </tbody>
</table>

A library for writing your own [Telegram](https://telegram.org/) bots. More information [here](https://core.telegram.org/bots). Official API [here](https://core.telegram.org/bots/api).

## Example
Here is a simple example (see [`example/simple.rs`](https://github.com/telegram-rs/telegram-bot/blob/master/lib/examples/simple.rs)):

``` rust
extern crate futures;
extern crate telegram_bot;
extern crate tokio;

use std::env;

use futures::{Stream, future::lazy};

use telegram_bot::*;

fn main() {
    let mut runtime = tokio::runtime::current_thread::Runtime::new().unwrap();
    runtime.block_on(lazy(|| {
        let token = env::var("TELEGRAM_BOT_TOKEN").unwrap();
        let api = Api::configure(token).build().unwrap();

        // Convert stream to the stream with errors in result
        let stream = api.stream().then(|mb_update| {
            let res: Result<Result<Update, Error>, ()> = Ok(mb_update);
            res
        });

        // Print update or error for each update.
        tokio::executor::current_thread::spawn(
            stream.for_each(move |update| {
                match update {
                    Ok(update) => {
                        // If the received update contains a new message...
                        if let UpdateKind::Message(message) = update.kind {
                            if let MessageKind::Text { ref data, .. } = message.kind {
                                // Print received text message to stdout.
                                println!("<{}>: {}", &message.from.first_name, data);

                                // Answer message with "Hi".
                                api.spawn(message.text_reply(
                                    format!("Hi, {}! You just wrote '{}'", &message.from.first_name, data)
                                ));
                            }
                        }
                    }
                    Err(_) => {}
                }

                Ok(())
            })
        );

        Ok::<_, ()>(())
    })).unwrap();

    runtime.run().unwrap();
}
```
You can find a bigger examples in the `examples`.

## Usage
This library is available via `crates.io`. In order to use it, just add this to your `Cargo.toml`:

```
telegram-bot = "0.7"
```

## Collaboration
Yes please! Every type of contribution is welcome: Create issues, hack some code or make suggestions. Don't know where to start? Good first issues are tagged with [up for grab](https://github.com/telegram-rs/telegram-bot/issues?q=is%3Aissue+is%3Aopen+label%3A%22up+for+grab%22).
