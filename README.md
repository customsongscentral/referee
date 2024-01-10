# Refsheet

This is what has been allowing referees to control stuff for stream scenes and Discord logging.
Warning: very dirty.

[Download refsheet HTML file, and launch the file locally on your browser (communication with Bastion requires extra setup)](https://raw.githubusercontent.com/customsongscentral/referee/main/docs/index.html)

## Quickstart

`yarn` to install dependencies

`cp .env.example .env`, fiddle with the env vars according to your [Bastion](https://github.com/customsongscentral/bastion) settings

`yarn build` to build `./docs/index.html`

Share the `index.html` to your referees (everything should be contained in there)

## Notes

While you could technically host the HTML online, the `ws://` calls to Bastion won't work because of cross-domain shenanigans.
Running the `index.html` file locally is actually a workaround that allows us to perform those `ws://` calls (this is so cursed).

In a way, this is somewhat more secure because you're not hosting the server credentials online (because yea, with this building process,
the passwords are embedded in the `index.html`). This is a fucked up model anyways, but it works for all intents and purposes :shrug: