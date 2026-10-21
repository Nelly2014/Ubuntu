# UbuntuVoice (self-hosted)

Two-way voice and text translation for South African public-service settings. The page runs in the browser; a small Node server holds your Anthropic API key and does the translating.

## Run it
1. Install Node 20.6 or newer.
2. `cp .env.example .env`, then set `ANTHROPIC_API_KEY` and long random `ACCESS_CODES`.
3. `npm install`
4. `npm run dev` and open http://localhost:3000 (microphone needs HTTPS or localhost).

## Deploy
- **Docker:** `docker build -t ubuntuvoice .` then `docker run -p 3000:3000 --env-file .env -v $PWD/logs:/app/logs ubuntuvoice`
- **Azure App Service, a VPS, or any Node host:** set the same variables as app settings, start with `npm start`.
- Always serve over HTTPS (the host or a reverse proxy does this). Set `TRUST_PROXY=1` behind a proxy so rate limits use the real client IP.
- GitHub Pages cannot host this: the API key must stay on a server.

## What it does
- Staff enter an access code once per browser session. Codes are compared in constant time.
- The server builds the translation prompt itself, accepts only listed languages and settings, caps messages at 1,000 characters, and rate-limits to 30 requests a minute per IP. It is not an open proxy.
- Each translation returns the translation, a literal English back-translation, and a caution if a term or number is risky.
- `logs/audit.jsonl` records time, access-code ID, languages, setting and message length. Message text is not stored unless `LOG_CONTENT=true`.

## Before real use
- This is AI translation, not a certified interpreter. Have fluent staff test it on your real phrases, especially in the smaller languages, before relying on it.
- Keep consent, medication doses, arrests and court matters with qualified interpreters.
- Health and legal conversations are personal information under POPIA. Check the Anthropic data-handling terms for your account, agree retention rules, and have your Information Officer sign off before deployment.
- Voice input and output come from the user's browser and depend on device language support. Many of the 11 languages have no speech recognition or voice on most devices; typing and reading still work.
- South African Sign Language is not recognised. The simplify button only writes short English sentences for an interpreter.
