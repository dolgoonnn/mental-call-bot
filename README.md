# ALLY: a phone line you can talk to

**3rd place, Dev Hackathon 2024** (Mongolian University of Science and
Technology, School of ICT; 27–28 April 2024; 18 teams from universities
across Mongolia). Team AXLETH, students of the University of Finance and
Economics.

ALLY is a mental-support line. Someone in distress calls an ordinary phone
number and talks, in Mongolian, with a counsellor-style assistant that
listens, answers briefly, and keeps them on the line. No app to install, no
account, no smartphone needed, which matters for the people most likely to
call.

Built in a weekend on top of Twilio's open-source `call-gpt` sample (see
[Credit](#credit)). What we changed is the persona and the language; what the
sample gave us is the real-time voice loop.

## How a call works

```
caller's phone ──► Twilio number ──► Media Streams (websocket, μ-law audio)
                                           │
                       Deepgram nova-2 (streaming speech-to-text)
                                           │  utterance
                       GPT-4 Turbo with the ALLY system prompt (streaming)
                                           │  sentence chunks
                       ElevenLabs (streaming text-to-speech) ──► back to the call
```

- The caller can interrupt; when Deepgram hears new speech while the
  assistant is talking, the queued audio is cleared and the assistant
  listens.
- Replies are split on `•` markers that the prompt asks the model to insert
  every 5–10 words, so speech starts after the first phrase instead of after
  the whole answer.
- The system prompt (`services/gpt-service.js`) sets the role: a therapist
  meeting a new client with anxiety and depression; listen actively, be
  empathetic, give practical steps, one question at a time, keep it short,
  keep the caller on the line without being rude, and answer in Mongolian.

## Run it

```bash
npm install
cp .env.example .env   # OPENAI_API_KEY, DEEPGRAM_API_KEY, XI_API_KEY, VOICE_ID, SERVER
npm run dev            # then point a Twilio number's voice webhook at https://<SERVER>/incoming
npm run outbound       # optional: have the app call your own phone
```

`SERVER` is the public host Twilio can reach (ngrok locally, or the Fly.io
deploy in `fly.toml.example`). Full setup, including the Twilio console steps
and the Fly deploy, is in `README.upstream.md`.

## Honest notes

- The `functions/` folder (airpods inventory, price, transfer call) is the
  sample's tool-calling demo and is not part of ALLY; it was left in place
  during the hackathon and is still wired into the model's tool list.
- Mongolian speech recognition and synthesis quality was the limiting
  factor on the day: Deepgram's `nova-2` and the ElevenLabs voice were used
  as-is, without a Mongolian-specific model.
- A real service would need crisis-escalation to a human, call logging rules,
  and clinical review of the prompt. This is a hackathon prototype, not a
  product.

## Team

Dolgoon Amartaivan (voice pipeline, prompt, deploy) with two teammates from
the University of Finance and Economics (pitch, research, demo).

## Credit

Based on [twilio-labs/call-gpt](https://github.com/twilio-labs/call-gpt) by
Craig Dennis, MIT licensed; the license is kept in `LICENSE` and the original
README in `README.upstream.md`. Services: Twilio Media Streams, Deepgram,
OpenAI, ElevenLabs.
