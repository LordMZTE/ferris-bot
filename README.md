# rust-twitch-queue-bot
Twitch bot for organising queues - developed live at twitch.tv/stuck_overflow

Use `cargo run -- --help` to see the available options.

## Twitch authentication flow

You need to obtain user credentials to allow the bot to login. The current
version of the bot has a manual flow that requires user assistance to obtain the
right credential.

Prepare a `.toml` file with the correct credentials, see
[`sample.ferrisbot.toml`](sample.ferrisbot.toml) for an example. By default the
app will look for a file named `ferrisbot.toml`, you can override this name with
the `--config_file` flag.

### 1. Obtain user permission

Run `cargo run -- --show_auth_url`. An URL of the following form should be
printed:

```
https://id.twitch.tv/oauth2/authorize?client_id=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&redirect_uri=http://localhost&response_type=code&scope=chat:read%20chat:edit
```

If it's the first time you authenticate this client, you will see a confirmation
dialog. After that, you will be redirected to a URL that will presumably  fail
on your browser, in the form:

```
http://localhost/?code=yyyyyyyyyyyyyyyyyyyyyyyyyyyyyy&scope=chat%3Aread+chat%3Aedit
```

Take the `code` value and copy it (in this example
`yyyyyyyyyyyyyyyyyyyyyyyyyyyyyy`)

### 2. Obtain the first token.

Run `cargo run -- -g --auth-code <code you obtained in step 1>`. The app will
output a `curl` command line in this format:

```sh
curl -X POST 'https://id.twitch.tv/oauth2/token?client_id=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&client_secret=zzzzzzzzzzzzzzzzzzzzzzzzzzzzzz&code=yyyyyyyyyyyyyyyyyyyyyyyyyyyyyy&grant_type=authorization_code&redirect_uri=http://localhost' > /tmp/firsttoken.json
```

### 3. First start of the bot

The first time the bot is started, you need to pass it the token obtained in
step 2. You can do so by using the `--first-token-file` flag:

```sh
cargo run -- --first-token-file /tmp/firsttoken.json
```

At this point the token will be used and stored in the bot's cache. You won't
need to use the `--first-token-file`.

## Running with Docker
It is recommended to run the bot using docker.

### 1. Create `docker-compose.yml` file

The file should have a service for this bot.
```yaml
version: "3.3"
services:
  ferris-bot:
    # TODO: adjust image name once published
    image: ferris-bot
    volumes:
      # Config files are in the /config directory.
      - "./config:/config"
```

### 2. Setup files

Set up the following files in the `/config` directory inside the container

- `ferrisbot.toml` for the config file
- `first-token.json` for the first token

### 3. Run

Start the services with `docker-compose up`

