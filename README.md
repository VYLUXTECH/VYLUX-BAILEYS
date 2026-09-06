# VYLUX-BAILEYS

**Stable, owned, branded fork of [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys) `6.7.24`.**

Built and maintained by **VYLUX TECH** for **VYLUX-XMD**. Same upstream source — vendored as a separate package so the bot owns its WhatsApp client.

## Why fork?

- Stable, recent base: `6.7.24` (released 2026-07-29) — the last 6.x stable; 7.x is still in `release-candidate`.
- **Custom pairing code** (`requestPairingCode`) with multi-browser support (`getPlatformId`).
- **Native buttons**: legacy `buttonsMessage` + native-flow `interactiveMessage` + `viewOnceMessage` for song-card/audio/video buttons.
- Branded under VYLUX (`vylux` browser UA, own version.json endpoint).
- Self-healing client version via `fetchLatestBaileysVersion()` (uses our master `lib/Defaults/baileys-version.json`, with the live WhatsApp Web fallback as backup).

## What changed vs upstream?

- `package.json` → name `vylux-baileys`, version `6.7.24-vylux.1`, devDependencies and TS build scripts removed (this repo ships only the compiled `lib/` + `WAProto/`).
- `lib/Utils/generics.js` → adds a `Browsers.vylux` UA helper (`['VYLUX', browser, '6.7.24']`).
- `lib/Utils/generics.js` → `fetchLatestBaileysVersion` fetches from `VYLUXTECH/VYLUX-BAILEYS/master/lib/Defaults/baileys-version.json`. Falls back to live WhatsApp Web sw.js if our endpoint is unreachable. Bottom line: the bot always reports a current WA version and won't 405-disconnect.

Nothing else is patched. All upstream proto features (button responses, newsletter metadata, business, communities, USync, LID, polls) work identically.

## Install

```bash
# in your bot's package.json
"vylux-baileys": "github:VYLUXTECH/VYLUX-BAILEYS"
```

```js
// CommonJS (Node >= 22 — require(esm) is stable)
const {
    makeWASocket,
    useMultiFileAuthState,
    fetchLatestBaileysVersion,
    Browsers,
    DisconnectReason,
    proto,
    generateWAMessageFromContent,
    prepareWAMessageMedia,
    downloadMediaMessage,
    getContentType,
    jidNormalizedUser,
    areJidsSameUser,
    S_WHATSAPP_NET,
    getBinaryNodeChild,
    downloadContentFromMessage,
    jidDecode,
    generateMessageID,
    makeCacheableSignalKeyStore
} = require('vylux-baileys');

const browser = Browsers.macOS('Safari');   // any upstream helper
// or: Browsers.vylux('Chrome')             // our branded UA
```

## Custom pairing code

```js
const sock = makeWASocket({ /* … */ });
if (!sock.authState.creds.registered) {
    const phone = '256704586844';
    const code = await sock.requestPairingCode(phone, 'VYLUX1234');
    console.log('Pairing code:', code);
}
```

## Native buttons

```js
// Legacy buttons (returns buttonsResponseMessage.selectedButtonId on tap)
await sock.sendMessage(jid, {
    text: 'Pick a format',
    footer: 'VYLUX-XMD',
    buttons: [
        { buttonId: 'play_audio_xyz', buttonText: { displayText: '🎵 AUDIO' }, type: 1 },
        { buttonId: 'play_video_xyz', buttonText: { displayText: '🎬 VIDEO' }, type: 1 }
    ],
    headerType: 1
});

// Native-flow interactiveMessage (returns interactiveResponseMessage.nativeFlowResponseMessage.paramsJson)
await sock.sendMessage(jid, {
    viewOnceMessage: {
        message: {
            interactiveMessage: {
                header: { title: 'Song', hasMediaAttachment: false },
                body: { text: '👇' },
                nativeFlowMessage: {
                    buttons: [{
                        name: 'quick_reply',
                        buttonParamsJson: JSON.stringify({
                            display_text: '🎵 AUDIO',
                            id: 'play_audio_xyz'
                        })
                    }]
                }
            }
        }
    }
});
```

## Receiving button taps

The message handler can dispatch on `mtype`:

```js
mtype === 'buttonsResponseMessage'        // legacy → message.buttonsResponseMessage.selectedButtonId
mtype === 'interactiveResponseMessage'   // native  → JSON.parse(msg.nativeFlowResponseMessage.paramsJson).id
mtype === 'templateButtonReplyMessage'    // template→ message.templateButtonReplyMessage.selectedId
mtype === 'listResponseMessage'           // list   → message.listResponseMessage.singleSelectReply.selectedRowId
```

## Used by

- [VYLUX-XMD](https://github.com/VYLUXTECH/VYLUX-XMD) — the multi-session WhatsApp bot.

## License

MIT — inherited from upstream WhiskeySockets/Baileys.
