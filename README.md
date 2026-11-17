# Metastar AI NPC SDK for three.js

**Metastar AI NPC SDK is a free JavaScript SDK that adds a talking AI NPC to a three.js scene with one ES module import.** Visitors talk to the character by voice or text, and it answers aloud with lip sync. You do not need your own LLM, speech-to-text, or text-to-speech API keys. They stay on Metastar's servers.

- **Live demo:** https://www.metastar.world/static/sdk-demo/paste-three.html
- **Docs:** https://metastar.world/sdk/v1/docs
- **Dashboard (get your key):** https://metastar.world/sdk/v1/dashboard
- **Using Babylon.js?** See [[metastar-ai-npc-sdk-babylonjs](../metastar-ai-npc-sdk-babylonjs)](https://github.com/singeryoa/metastar-ai-npc-sdk-babylonjs/tree/main)

## What you get

- A 3D avatar with a name tag and an AI badge, placed in your existing three.js scene
- Voice chat (hold the microphone button) and text chat in a built-in chat panel
- Spoken replies with lip sync, blinking, and turning toward the visitor
- A proximity greeting when a visitor walks up to the NPC
- A persona, banned topics, and greeting lines you set in a dashboard, with no code
- Events and methods (`sendText`, `say`, `on("reply")`, …) if you want to build your own UI
- No second copy of three: you pass the `THREE` you already use
- Nothing to call per frame: your render loop stays as it is

## How do I add an AI NPC to a three.js scene?

**1. Get a public key and an NPC ID (about 5 minutes).**
Open the [dashboard](https://metastar.world/sdk/v1/dashboard), sign in with Google, and press **Register as operator** (approved instantly). Register your site's domain, for example `https://example.com` or `http://localhost:8080` while developing, then create an NPC: name, avatar, voice, a short persona, and a greeting line.

**2. Paste this after your scene is created.**

```html
<script type="module">
  import * as THREE from "three";
  import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
  import { MetastarNPC } from "https://metastar.world/static/sdk/v0/npc.three.js";

  MetastarNPC.init({
    publicKey: "pk_live_xxxxxxxx",          // from the dashboard
    npcId: "npc_xxxxxxxx",                  // from the dashboard
    THREE, GLTFLoader,                      // the same three your scene uses
    scene, camera, renderer,                // your THREE.Scene, camera, WebGLRenderer
    position: { x: 0, y: 0, z: 3 },         // where the NPC stands (feet)
    rotationY: Math.PI,                     // 0 faces +Z, Math.PI faces -Z
    getPlayerPosition: () => camera.position
  });
</script>
```

`three` and `three/addons/` must be resolvable through an import map or your bundler.

## Run the complete example

[`index.html`](index.html) in this repository is a complete, minimal three.js page with one AI NPC. It uses an import map, so there is no build step.

1. Replace `pk_live_xxxxxxxx` and `npc_xxxxxxxx` with the values from your dashboard.
2. Serve it from a domain you registered:

```bash
python -m http.server 8080
# then open http://localhost:8080
```

Press **Talk**, or tap the avatar, and type or hold the microphone button.

## Using a bundler (Vite, webpack)

Keep the SDK out of your bundle and load it at runtime:

```js
const { MetastarNPC } = await import(
  /* @vite-ignore */ /* webpackIgnore: true */
  "https://metastar.world/static/sdk/v0/npc.three.js"
);
```

## Options

`MetastarNPC.init(options)`

| Option | Description |
|---|---|
| `publicKey` * | Your site's public key. It only works on the domains you registered. |
| `npcId` * | The NPC to place. |
| `THREE` * | The three module your scene uses (`import * as THREE from "three"`). r150 or later. |
| `GLTFLoader` * | The `GLTFLoader` class from three's addons. You can pass a ready loader instance as `gltfLoader` instead. |
| `scene` * | Your `THREE.Scene`. |
| `camera` | The camera you render with. Needed for opening the chat by tapping the avatar. |
| `renderer` | Your `WebGLRenderer` (or pass its canvas as `domElement`). The SDK only listens for taps; it never renders or resizes. |
| `position` | `{x, y, z}` of the feet. Default: origin. |
| `rotationY` | Facing in radians. `0` faces +Z, `Math.PI` faces −Z. |
| `scale` | Avatar scale. Default `1` (about 1.75 m tall). |
| `getPlayerPosition` | Function returning the visitor's `{x, y, z}`. Used for the greeting and the Talk button. |
| `greet` | `false` turns off the proximity greeting. |
| `ui` | `false` hides the built-in panel so you can build your own UI. |
| `onError` | Called with `{code, message}` if initialisation fails. `init()` never throws. |

\* required. Call `init()` several times to place several NPCs.

## API

```js
const npc = await MetastarNPC.init({ /* ... */ });
if (!npc.ready) return;                          // see onError

npc.on("reply", (e) => console.log(e.text));
npc.sendText("What are your opening hours?");    // ask as the visitor (LLM)
npc.say("We close at six today.");               // fixed line, no LLM
npc.openChat(); npc.closeChat();
npc.setPosition({ x: 2, y: 0, z: 5 });
npc.destroy();                                   // removes the avatar and frees geometries, materials, textures
```

Events: `ready`, `greeting`, `listening`, `thinking`, `reply`, `speaking`, `speakend`, `quota_exceeded`, `login_required`, `login`, `logout`, `limit`, `error`. See the [docs](https://metastar.world/sdk/v1/docs) for details.

## FAQ

### Which LLM does it use?
Replies are generated by Anthropic's Claude (currently Claude Sonnet 4.6). Speech recognition uses OpenAI's gpt-4o-mini-transcribe, and the voice is Google's Gemini 2.5 Flash TTS. All of it runs on Metastar's servers, so there are no API keys in your page.

### Which languages are supported?
You pick the NPC's default language in the dashboard from 8 options: Korean, English, Japanese, Chinese, Spanish, Vietnamese, Indonesian, and Hindi. If a visitor clearly speaks another language, the NPC answers in the visitor's language. The chat panel's own buttons and hints are in Korean or English. Korean and English are the most tested.

### Is it free?
Yes, for site operators. There is no setup fee, no usage fee, and no card required. Each operator account has a safety cap on usage; if you reach it, Metastar raises it at no cost on request.

### Is there a limit for visitors?
Yes. Without signing in, a visitor gets 3 text turns a day. Signed-in visitors (Google sign-in with a Metastar account) get 10 text turns and 3 voice turns a day across all sites, and voice chat requires signing in. Metastar's paid plans raise the daily turns for that visitor.

### Does the NPC remember visitors?
For visitors who are not signed in, it remembers the last few turns of the current visit. For signed-in visitors, it remembers their recent chats with that NPC until midnight (KST) of the following day, and longer for chats they save on the "My NPC Chats" board in Metastar Main Town.

### Which three.js versions work?
three.js r150 or later (checked on r150 and r187), with `GLTFLoader`.

### Does it work with React Three Fiber?
It has not been tested with React Three Fiber. The SDK only needs your `THREE`, `GLTFLoader`, scene, camera, and renderer, so it may work if you pass those in, but treat it as unverified.

### Does voice chat need HTTPS?
Yes. Voice input needs a secure page (`https://` or `localhost`). On other pages the microphone button is hidden and text chat still works.

### Can I use my own avatar model?
Not yet. You choose from the avatars in the dashboard.

### What has not been tested yet?
Immersive WebXR sessions, `WebGPURenderer`, and voice quality in languages other than Korean and English.

## Troubleshooting

- **Nothing appears, and the console shows `npc_not_found`:** the `npcId` is wrong, or the NPC was deleted in the dashboard.
- **`origin_not_allowed` or a similar key error:** the page's domain is not registered for that public key. Register the exact origin, including scheme and port (for example `http://localhost:8080`). Changes take about a minute to apply.
- **`Failed to resolve module specifier "three"`:** add an import map for `three` and `three/addons/` (see `index.html`), or use a bundler.
- **The page was opened as `file://`:** serve it over `http://localhost` instead.

More in the [docs](https://metastar.world/sdk/v1/docs).

## 한국어 요약

**메타스타 AI NPC SDK** 는 three.js 웹 월드에 ES 모듈 한 줄로 말하는 AI NPC 를 넣는 무료 SDK 입니다. 방문자가 음성이나 글로 말을 걸면 NPC 가 입 모양에 맞춰 소리로 답합니다. LLM·음성 인식·음성 합성 API 키를 직접 준비할 필요가 없고, 사이트 운영자에게는 무료입니다. [대시보드](https://metastar.world/sdk/v1/dashboard)에서 사이트와 NPC 를 만든 뒤 이 저장소의 `index.html` 에서 ID 두 개만 바꾸면 됩니다. 자세한 내용은 [문서](https://metastar.world/sdk/v1/docs)(한국어 지원)를 보세요.

## License

The example code in this repository is released under the MIT License. The SDK itself (`npc.three.js`) and the Metastar service are provided under [Metastar's terms](https://metastar.world/terms).
````
