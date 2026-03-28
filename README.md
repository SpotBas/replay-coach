# ReplayCoach — Deployment instructies

## Bestanden
```
replaycoach/
├── index.html      ← de complete app
├── manifest.json   ← PWA metadata
├── sw.js           ← service worker (offline cache)
├── icon-192.png    ← iPad homescreen icon (maak zelf aan, 192×192px)
└── icon-512.png    ← iPad homescreen icon (maak zelf aan, 512×512px)
```

## HTTPS vereiste
iOS vereist HTTPS voor camera-toegang. Opties:

### Optie A — Gratis hosting (eenvoudigst)
1. Maak account op **Netlify** (netlify.com)
2. Sleep de `replaycoach/` map naar het Netlify-dashboard
3. Je krijgt direct een `https://....netlify.app` URL
4. Open die URL op de iPad in Safari

### Optie B — GitHub Pages
1. Maak een repo aan op GitHub
2. Push de bestanden
3. Ga naar Settings → Pages → Deploy from branch
4. Je krijgt een `https://jouwname.github.io/replaycoach/` URL

### Optie C — Lokaal (voor thuis/gym via eigen wifi)
Vereist een lokale HTTPS server. Met Node.js:
```bash
npx serve --ssl-cert cert.pem --ssl-key key.pem replaycoach/
```
Of gebruik **Caddy** (gratis, automatisch HTTPS op lokaal netwerk).

## Installeren als PWA op iPad
1. Open de URL in **Safari** (Chrome/Firefox werkt niet voor PWA installatie op iOS)
2. Tik op het **Deel-icoontje** (vierkantje met pijl omhoog)
3. Kies **"Zet op beginscherm"**
4. De app start voortaan fullscreen, zonder Safari-balk

## Iconen aanmaken
De app werkt zonder iconen, maar voor een nette installatie:
- Maak een vierkant logo van 512×512px (PNG)
- Sla op als `icon-512.png` en schaal naar `icon-192.png`
- Upload beide samen met de andere bestanden

## Gebruik
| Actie | Werking |
|-------|---------|
| **START** | Start camera + opname in rollende buffer |
| **Slider / preset-knoppen** | Stel vertraging in (3–30 seconden) |
| **BEKIJK REPLAY** | Speelt het moment van X seconden geleden af |
| **⏹ STOP** | Stopt de opname |
| Camera wisselen | Tik Voorkant / Achterkant (achtercamera aanbevolen) |

## Performance-notities
- Buffer gebruikt ~15–25 MB/s RAM bij 1080p — geen probleem voor een iPad
- App werkt volledig offline na eerste load (service worker)
- Scherm dimmen/vergrendelen stopt de opname (iOS-beperking, niet op te lossen)
