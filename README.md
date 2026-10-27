# Surprise

Send Grams as a sealed surprise. The amount stays hidden until the recipient taps to open it.

▶️ [Watch the demo](demo.mp4)

## How it works

1. In a private chat, open the attachment menu and tap **Money**.
2. Enter an amount and an optional note, turn on **Send as Surprise**, and send.
3. The recipient gets the usual Gram transfer card, but the amount is covered with spoiler dust, the note is hidden and the ribbon says **Surprise**. The chat list and the notification only say *"Name sent you a Surprise"*, with no amount.
4. One tap opens it. The dust scatters, the digits decode into the real amount, the address re-hashes for a moment, the diamond spins, the ribbon turns from **Surprise** to **Received**, the note appears and confetti plays.
5. On the sender's side, the card switches from **Surprise** to **Opened** as soon as it's opened.

## Why

- Sending money becomes a moment instead of a receipt: birthdays, holidays, Nowruz *eidi*, red envelopes, a thank-you.
- It gives people a reason to send Grams to friends, not only to pay.
- It's cheap to ship. There are no new screens: it's one chip in the Send Money sheet, and the reveal is built from what the wallet card already has (spoiler effect, ribbon, diamond spin, particles, highlight, fireworks).

## Contents

- `demo.mp4`: the full flow on both sides
- `surprise.patch`: the implementation for [Telegram Android](https://github.com/DrKLO/Telegram) 13.0.2
- `TMessagesProj/`: the changed source files in full, for easy reading
- [IMPLEMENTATION.md](IMPLEMENTATION.md): how it works, what is real and what is demo-only, and what production would need

## Try it

```
git clone https://github.com/DrKLO/Telegram.git
cd Telegram
git checkout f32ffeec763f0bdbf4bec25b52d654f77bff733d
git apply surprise.patch
```

Then build as described in Telegram's own README. It was built and tested on a physical Android phone.

The prototype runs in demo mode: the wallet shows a 2,000 GRAM balance and transfers don't touch the blockchain. Both phones need this build. Details are in [IMPLEMENTATION.md](IMPLEMENTATION.md).

Suggested by [@prsmio](https://t.me/prsmio)
