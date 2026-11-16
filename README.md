# UbuntuVoice with a key-holding proxy

Users no longer paste an API key. The page calls a small Cloudflare Worker, and the Worker holds your key.

```
ubuntuvoice/
  index.html          the app (replaces the one in your GitHub "Ubuntu" repo)
  worker/
    worker.js         the proxy
    wrangler.toml     its config
```

## Deploy the Worker (about 10 minutes, free tier is enough)

1. Create a free account at cloudflare.com.
2. In a terminal inside `worker/`:
   ```
   npx wrangler login
   npx wrangler secret put ANTHROPIC_API_KEY
   npx wrangler deploy
   ```
   Paste your `sk-ant-` key when asked. It is stored encrypted at Cloudflare, never in the code.
3. `deploy` prints a URL like `https://ubuntuvoice-proxy.<you>.workers.dev`.

## Point the app at it

1. Open `index.html` and set `PROXY_URL` (near the top of the script) to that URL.
2. Replace `index.html` in your GitHub repo and commit. GitHub Pages updates in a minute or two.

## Lock it down

- In `wrangler.toml`, `ALLOWED_ORIGINS` lists the sites allowed to call the Worker. Remove the `localhost` entry once you've tested, then run `npx wrangler deploy` again.
- In the Anthropic Console, put the key in its own workspace and set a monthly spend limit.
- In the Cloudflare dashboard, add a rate limiting rule for the Worker route. The Worker also limits each IP to 20 requests a minute, but that counter is per instance and only best effort.
- The Worker builds the prompts itself and caps input at 1,500 characters, so it can't be used as a general Claude endpoint. The origin check stops other websites from using it from a browser; it does not stop someone calling it directly with a script, which is what the rate limit and spend limit are for.

## Test locally

```
cd worker && npx wrangler dev          # Worker on http://localhost:8787
python3 -m http.server 8080            # in the folder with index.html
```
Set `PROXY_URL` to `http://localhost:8787`, open http://localhost:8080.

## Notes

- Model is set by `MODEL` in `wrangler.toml` (default `claude-sonnet-5-5`). Use `claude-haiku-5-5` for faster, cheaper replies.
- Voice input and read-aloud use the browser's built-in speech features. Support for languages like isiZulu and isiXhosa varies by browser and device (Edge and Chrome on Windows/Android are the best bet). Where a language isn't supported, the app says so and the typed and on-screen text still work.
- Conversations are held in memory only and disappear on refresh.
