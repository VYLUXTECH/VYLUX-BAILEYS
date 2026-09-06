# VYLUX-BAILEYS

**Stable WhatsApp Web client library, built and maintained by VYLUX TECH.**

Pinned to the most recent stable 6.x line. Same wire protocol as upstream; this package is owned by VYLUX so the bot controls its client version and UA.

## Features

- **Custom pairing code** — `requestPairingCode(phone, customCode)` with multi-browser support (`getPlatformId`).
- **Native buttons** — legacy `buttonsMessage`, native-flow `interactiveMessage.nativeFlowMessage`, and `viewOnceMessage` for image/audio/video cards.
- **Branded UA** — `Browsers.vylux('Chrome')` returns `['VYLUX', 'Chrome', '6.7.24']`.
- **Self-healing client version** — `fetchLatestBaileysVersion()` reads from our master `lib/Defaults/baileys-version.json`, with a live WhatsApp Web fallback. The bot always reports a current WA version so it won't 405-disconnect.
- **All upstream proto features** — newsletter metadata, business, communities, USync, LID, polls, list, template, view-once — work identically.

## Install

```bash
npm install github:VYLUXTECH/VYLUX-BAILEYS
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

const browser = Browsers.macOS('Safari');
// or branded:  Browsers.vylux('Chrome')   // -> ['VYLUX', 'Chrome', '6.7.24']
```

## Custom pairing code

```js
const sock = makeWASocket({ /* … */ });
if (!sock.authState.creds.registered) {
    const phone  = '256704586844';
    const code   = await sock.requestPairingCode(phone, 'VYLUX1234');
    console.log('Pairing code:', code);   // 8-char code, returned as a string
}
```

## Native buttons

```js
// Legacy buttons (tap returns buttonsResponseMessage.selectedButtonId)
await sock.sendMessage(jid, {
    text: 'Pick a format',
    footer: 'VYLUX-XMD',
    buttons: [
        { buttonId: 'play_audio_xyz', buttonText: { displayText: '🎵 AUDIO' }, type: 1 },
        { buttonId: 'play_video_xyz', buttonText: { displayText: '🎬 VIDEO' }, type: 1 }
    ],
    headerType: 1
});

// Native-flow interactiveMessage
// (tap returns interactiveResponseMessage.nativeFlowResponseMessage.paramsJson)
await sock.sendMessage(jid, {
    viewOnceMessage: {
        message: {
            interactiveMessage: {
                header: { title: 'Song', hasMediaAttachment: false },
                body:  { text:  '👇' },
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

Dispatch on `mtype`:

```js
mtype === 'buttonsResponseMessage'       // selectedButtonId
mtype === 'interactiveResponseMessage'  // JSON.parse(nativeFlowResponseMessage.paramsJson).id
mtype === 'templateButtonReplyMessage'  // selectedId
mtype === 'listResponseMessage'         // singleSelectReply.selectedRowId
```

## Used by

- VYLUX TECH bot fleet.

## License

MIT.
