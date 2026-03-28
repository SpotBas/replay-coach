# ReplayCoach — CLAUDE.md

## Wat is dit?

ReplayCoach is een **Progressive Web App (PWA)** voor sportcoaches. Het toont een vertraagd camerabeeld zodat een sporter zijn eigen techniek direct na uitvoering kan terugkijken — zonder opname te hoeven starten of stoppen. Alles draait in de browser, geen app-store nodig.

**Typisch gebruik:** zet een tablet/telefoon op een statief, start de app, speel sport. De sporter loopt na een actie naar het scherm en ziet zichzelf X seconden terug.

---

## Architectuur

Eén enkel HTML-bestand: `index.html`. Geen build-tooling, geen frameworks, geen npm.

```
index.html          — alles: HTML, CSS en JS in één file
sw.js               — Service Worker voor PWA offline-support
manifest.json       — PWA manifest (naam, iconen, display)
icon-192.png        — app-icoon
```

---

## Hoe het technisch werkt

### Frame buffer strategie

iOS Safari crasht bij grote canvas-buffers of ImageBitmap arrays. Oplossing:

1. **Capture canvas** (onzichtbaar, 640×360): elk frame van de live video wordt hierop getekend via `drawImage`.
2. **JPEG dataURL** (`canvas.toDataURL('image/jpeg', 0.5)`): het captured frame wordt direct als compacte string opgeslagen — ~10–20 KB per frame.
3. **Frame buffer array** `[{ dataUrl, t }]`: circulaire buffer, maximaal 32 seconden. Oudste frames worden verwijderd.
4. **Display canvas** (fullscreen, bovenop de live video): het gewenste vertraagde frame wordt opgezocht via binair zoeken en getekend.

**Geheugenberekening:** 15fps × 32s = 480 frames × ~15KB = ~7MB totaal — trivial voor iOS.

### Render loop

```
requestAnimationFrame loop
  ├── elke ~66ms: captureer frame naar capCv → toDataURL → push naar frames[]
  ├── prune frames ouder dan MAX_BUFFER_S
  ├── update buffer-voortgangsbalk
  └── als bufferReady: zoek frame op targetT = now - delayMs → displayImg.src = dataUrl
                        displayImg.onload → drawImage op replayCanvas (cover-gedrag)
```

### Cover-tekening (geen CSS object-fit)

`object-fit: cover` werkt **niet** op `<canvas>` elementen. De drawImage implementeert dit zelf:
- Als bronafbeelding breder is dan canvas: bijsnijden links/rechts
- Als bronafbeelding smaller is: bijsnijden boven/onder
- Resultaat: altijd gevuld scherm zonder vervorming

### Camera

- Vraagt 720p via `getUserMedia` (wordt intern toch gedownsampled naar 640×360 bij capture)
- Ondersteunt voor- en achtercamera wisselen (herstart de loop)
- Front-camera: `scaleX(-1)` spiegel zowel in live video (CSS) als in de capture (canvas transform)
- iOS fallback: toont `#permOverlay` bij camera-weigering

---

## UI-onderdelen

| Element | Functie |
|---|---|
| `#liveVideo` | Live camera stream, altijd actief op achtergrond |
| `#replayCanvas` | Vertraagd beeld, zit boven liveVideo. Opacity 0 in live-modus |
| Sidebar (`.ctrl-col`) | Delay-slider (1–30s), preset-knoppen (3/5/10/15/20s), camera-switch |
| `#startBtn` | Start/stop de loop en buffer |
| `#viewToggleBtn` | Wisselen live ↔ vertraagd (alleen zichtbaar als actief) |
| `#bufOverlay` | Voortgang buffer-opbouw (verbergt zodra buffer vol genoeg is) |
| Toggle sidebar | Knop linksonder in videogebied, slideout animatie |

---

## Constanten (aanpasbaar)

```js
CAP_W = 640        // capture breedte (pixels)
CAP_H = 360        // capture hoogte
FPS   = 15         // frames per seconde capture
MAX_BUFFER_S = 32  // maximale buffer in seconden
delayMs = 5000     // standaard vertraging (ms), via slider instelbaar 1–30s
```

---

## Bekende keuzes / trade-offs

- **Geen MediaRecorder**: eerder geprobeerd, gaf iOS problemen met seekable blobs en timing.
- **JPEG dataURL ipv ImageBitmap**: dataURL leeft in CPU-geheugen (geen GPU-limiet), sync API, betrouwbaar op iOS.
- **640×360 capture**: 4× minder pixels dan 720p, voldoende voor replay op ~3m afstand.
- **15fps**: halve framerate is prima voor techniek-replay, halveert geheugengebruik.
- **Single HTML file**: bewuste keuze — geen build-stap nodig, makkelijk te hosten op GitHub Pages of lokale server.

---

## PWA / offline

- `sw.js` cacht de app-shell voor offline gebruik.
- `manifest.json` maakt installatie mogelijk als "app" op iOS/Android.
- Wake Lock API wordt aangevraagd zodat het scherm niet dimmt tijdens gebruik.
