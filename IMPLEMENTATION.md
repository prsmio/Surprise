# Implementation

Base: Telegram Android 13.0.2, commit `f32ffeec763f0bdbf4bec25b52d654f77bff733d`.

## What is real and what is demo

Everything the user sees is Telegram's own code. The transfer card is the real `WalletActionLayout`, sending uses the real send flight, and the reveal is built from the card's own pieces: the spoiler effect, the ribbon, the diamond spin, sent particles, the highlight sweep and the chat fireworks.

Only the transport is a demo. Real Gram transfers can't carry a "surprise" flag today, so `SurpriseDemo.java` stands in for that part. In production this one file goes away and the rest stays as it is.

## How a Surprise travels in the demo

1. **Send as Surprise** in the Money tab sets `SurpriseDemo.pendingSurprise`.
2. While the demo is on, `GramWallet.send()` hands Gram transfers to `SurpriseDemo.send()`. Instead of a blockchain transaction it sends a normal message whose visible text is only `🎁 Surprise`. The amount, both addresses, the note and a transaction id ride in an invisible text link:

   ```
   https://telegram.org/gram#v2.<amount>.<to>.<from>.<note>.<txId>
   ```

   Addresses and the note are base64url. The `txId` starts with `surprise-` for Surprises and `demo-` for plain demo transfers.
3. On both phones, `SurpriseDemo.convert()` turns that message, in memory only, into a `TL_messageService` with a `TL_messageActionGramTransfer`. From then on Telegram draws it with its own wallet card. `MessageObject` calls it for every message, and `ChatActivity` calls it again when the server confirms a sent message, so the card doesn't flip back to text.
4. The opened state is stored per account and per transaction in local `SharedPreferences`. When the recipient opens a Surprise, the app sends a 🎉 reaction to it, and the sender's card reads any reaction from the other side as **Opened**.

A client without this build sees the carrier as a plain `🎁 Surprise` link.

## Changes by file

| File | Change |
| --- | --- |
| `ui/Wallet/SurpriseDemo.java` | New. Demo balance, demo send, carrier ⇄ transfer conversion, sealed/opened state, the **Send as Surprise** chip, sealed texts. |
| `ui/Wallet/WalletActionLayout.java` | The sealed card and the reveal: spoiler dust over the amount, digit decode, address re-hash, ribbon text and color, hidden note, tap handling. |
| `ui/Cells/ChatActionCell.java` | Binds the Surprise state to the card before it is set; no fiat value on the service line while it's sealed. |
| `messenger/MessageObject.java` | Converts carrier messages; sealed text for the chat list and service line. |
| `messenger/NotificationsController.java` | Sealed notification text, without the amount. |
| `ui/ChatActivity.java` | Keeps the card after the server confirms the sent message. |
| `ui/Wallet/GramWallet.java` | Demo balance and demo send while the demo is on. |
| `ui/Components/ChatAttachAlertGramLayout.java` | Adds the **Send as Surprise** chip to the Send Money sheet. |

All paths are under `TMessagesProj/src/main/java/org/telegram/`.

## The sealed card

- The amount is covered by Telegram's spoiler dust. The dust always has the same width, so it never hints at how big the number is.
- The note is hidden the same way an encrypted note is.
- The ribbon says **Surprise** in violet (`#9466FF`).
- The chat list, the service line and the notification say *"Name sent you a 🎁 Surprise"*, with no amount and no fiat value.
- The automatic "received" animation is skipped for incoming Surprises, so the reveal is the moment.

## The reveal

Times are from the tap, in milliseconds.

| Time | What happens |
| --- | --- |
| 0 | Light haptic, card bounce, diamond spin with extra momentum, sent particles, spoiler dust starts to scatter |
| 0–700 | The wallet address re-hashes, then settles |
| 60–1150 | The amount decodes: digits cycle and lock in one by one, left to right |
| 650 | Ribbon turns from **Surprise** to **Received**, violet to blue |
| 900 | The note appears |
| 1150 | Strong haptic, highlight sweep, fireworks, 🎉 reaction to the sender |

On the sender's side, the card bounces, spins and highlights when the other person opens it, and the ribbon changes from **Surprise** to **Opened**.

## Testing

1. Build the app from the patched source and install it on two phones, or use two accounts.
2. On phone A, open a private chat with B, tap the attachment icon, then **Money**. The demo balance is 2,000 GRAM.
3. Enter an amount and a note, tap **Send as Surprise**, and send.
4. On phone B, check the notification and the chat list (no amount), then open the chat and tap the card.
5. On phone A, the card changes to **Opened**.

To go back to real transfers, set `SurpriseDemo.ENABLED = false`.

## What production would need

- A `surprise` flag on the transfer request and on `messageActionGramTransfer`, instead of the carrier message.
- An "opened" state kept by the server, with an update to the sender, instead of local storage and a reaction.
- The server leaving the amount out of push notifications and previews while a Surprise is sealed. This prototype covers the in-app texts and local notifications only.
- The fiat-value check in `ChatActionCell` reading the flag instead of the message text.
- Moving the English strings (**Surprise**, **Opened**, **Send as Surprise**, the sealed texts) into `strings.xml`.
- Removing the demo balance and demo send.
