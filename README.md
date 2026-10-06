# CD FlyDJ Track Pro

Console DJ a due deck che gira interamente nel browser: motore audio Web Audio, controller via Web MIDI, gesti delle mani da webcam, installabile come PWA. Nessun server, nessuna build.

**Presentazione e roadmap:** https://flydj.cdalise.com

> Stato: **in sviluppo · early access**. La console (`dj-live.html`) non è pubblicata: online c'è solo la landing in `landing/`.

---

## Indice

1. [Requisiti](#requisiti)
2. [Avvio](#avvio)
3. [Struttura del repository](#struttura-del-repository)
4. [Architettura](#architettura)
5. [Grafo audio](#grafo-audio)
6. [MIDI](#midi)
7. [Persistenza locale](#persistenza-locale)
8. [PWA e offline](#pwa-e-offline)
9. [Landing e deploy](#landing-e-deploy)
10. [Limiti noti](#limiti-noti)
11. [Roadmap](#roadmap)

---

## Requisiti

| Voce | Requisito |
|---|---|
| Browser | Chromium recente (Chrome / Edge). Servono Web Audio, Web MIDI, `AudioContext.setSinkId`. |
| Contesto | `file://` per uso locale; **HTTPS** per MIDI su tablet, webcam, microfono, installazione PWA. |
| Controller | Qualsiasi controller DJ MIDI. Preset incorporato per il controller di riferimento (controller FX-2); gli altri si collegano con LEARN. |
| Audio | Stereo, oppure scheda a 4 canali (casse 1-2, cuffia 3-4). |
| Formati | MP3, WAV, M4A/AAC, FLAC, OGG, OPUS; AIFF dove il browser lo decodifica. |

## Avvio

```text
1. Chiudi gli altri programmi DJ (su Windows una porta MIDI è usabile da un solo processo).
2. Collega il controller via USB.
3. Apri dj-live.html in Chrome (doppio clic). Evita finestre in incognito: le impostazioni non persistono.
4. 🎛 MIDI → consenti l'accesso MIDI → 🔊 TEST AUDIO.
5. + AGGIUNGI BRANI: l'analisi BPM parte in automatico.
```

Per servire in locale via HTTP:

```bash
python -m http.server 8080     # poi http://localhost:8080/dj-live.html
```

## Struttura del repository

| Percorso | Contenuto |
|---|---|
| `dj-live.html` | Console completa, file unico (~2.000 righe: HTML + CSS + JS, nessuna dipendenza a runtime oltre a quelle opzionali sotto). |
| `manifest.json`, `sw.js`, `icon.svg` | PWA: manifest, service worker network-first, icona. |
| `code.html` | Prototipo separato "Gravity Touch – Regia LIVE". Non collegato alla console. |
| `landing/` | Landing pubblica bilingue IT/EN (HTML/CSS/JS statico) con screenshot reali. |
| `vercel.json`, `.vercelignore` | Deploy della sola landing, header `noindex`. |
| `docs/SCHEDA_TECNICA.html` | Scheda tecnica e manuale d'uso. |
| `docs/PROMEMORIA_PROGETTO.md` | Stato, decisioni, roadmap (backend, desktop, sync). |

Dipendenze esterne, caricate **solo** quando servono:

| Dipendenza | Uso | Quando |
|---|---|---|
| `@mediapipe/tasks-vision@0.10.14` (jsDelivr) + modello `hand_landmarker` | Riconoscimento mani on-device | Solo attivando ✋ MANI |
| Google Fonts | Tipografia | All'avvio (fallback di sistema se offline) |
| Riconoscimento vocale del browser | Comandi vocali | Solo attivando 🎤 VOCE DEMO (richiede rete) |

## Architettura

Applicazione single-file, senza framework né bundler. Moduli logici dentro lo stesso `<script>`:

| Modulo | Responsabilità |
|---|---|
| **Deck** | Caricamento e decodifica, analisi BPM al centesimo, beat grid, waveform a 3 bande (bassi/medi/alti), trasporto, pitch (±8/16/50%), hot cue, loop/roll, slip, censor, scratch bidirezionale. |
| **Sync** | Aggancio tempo + fase con loop di correzione continuo (PLL, max ±0,6% di velocità, non udibile); stato blu = fase agganciata, oro = solo tempo; half/double tempo. |
| **Mixer** | Trim, EQ 3 bande, filtro / Color FX, fader di canale, crossfader, master, CUE di canale, CUE MIX software. |
| **FX** | Catena per deck: HPF, LPF, Flanger, Echo, Reverb, Phaser, con Beats e Wet/Dry sincronizzati al BPM. Color FX (Dub Echo, Sweep, Noise, Filter) nelle skin Club/All-in-One. |
| **Stems** | Separazione approssimata mid/side senza IA: voce off, musica off, bassi off. |
| **Sampler** | 4 slot, horn, TAP tempo. |
| **Libreria** | Tab Files / Browse / Prepare / History, ricerca, Auto Gain (−9…+6 dB), Instant Doubles. |
| **REC** | Registrazione del master in WAV 16 bit stereo (~600 MB/h). |
| **MIDI** | Input/output, mappatura, LEARN, configurazione guidata, LED. |
| **Input alternativi** | Touch, tastiera, gesti delle mani (webcam), voce (demo). |
| **UI** | 3 skin (Studio, Club Mixer, All-in-One), modalità tablet, Wake Lock, schermo intero. |

## Grafo audio

```text
 Deck A ─► trim ─► EQ(3) ─► filtro/ColorFX ─► FX deck ─► fader ─┬─► crossfader ─► master ─┬─► ch 1-2 (casse)
                                                                └─► CUE A ─┐               └─► REC
 Deck B ─► trim ─► EQ(3) ─► filtro/ColorFX ─► FX deck ─► fader ─┬─► crossfader             │
                                                                └─► CUE B ─┴─► cueBus ─► CUE MIX ─► ch 3-4 (cuffia)
```

- **Uscita a 4 canali**: `ChannelMerger(4)`; master sui canali 1-2, `cueBus` mixato col master tramite CUE MIX sui canali 3-4. La scheda si sceglie con `AudioContext.setSinkId`; se la scheda non espone 4 canali la cuffia ricade sul master.
- **REC**: `AudioWorkletNode` caricato da Blob URL; su `file://`, dove il worklet può essere bloccato, fallback automatico su `ScriptProcessorNode`.
- **Stems**: `ChannelSplitter(2)` → somma/differenza (mid/side) → `ChannelMerger(2)`.

## MIDI

Formato della mappa (`djfly.midimap`), chiave → azione:

| Chiave | Significato |
|---|---|
| `n:<ch>:<num>` | Note on/off (canale, nota) |
| `c:<ch>:<num>` | Control change |
| suffisso `@s` | Stesso comando con SHIFT premuto |
| `scene:<mode>:<i>` | Pad `i` nella modalità pad `<mode>` |

Modalità di interpretazione: `btn` (note), `ccbtn` (CC come tasto), `ccnz` (CC solo se ≠ 0), `abs` (assoluto, es. fader), `rel` (relativo, es. jog/encoder). Controlli a **14 bit**: MSB sul CC `n`, LSB sul CC `n+32`.

**Preset incorporato** (`PRESET_CTRL`, versione `PRESET_VER = '3'`): mixer, trasporto, SHIFT e scene dei pad, ricavato da una mappatura completa sul controller reale.

- Si carica da solo se la mappa è vuota; se esce una versione nuova viene proposto una volta.
- Il controller è riconosciuto per nome della porta MIDI già usata (`djfly.ctrlName`), senza regex su nomi di prodotto.
- La stessa parola chiave serve a preselezionare la scheda audio del controller in 🎛 MIDI → uscita.
- Export/import della mappa in JSON (`djfly-controller-map.json`).

## Persistenza locale

| Chiave `localStorage` | Contenuto |
|---|---|
| `djfly.settings` | Skin, tablet, mani, voce, CUE MIX, Wake Lock |
| `djfly.midimap` | Mappa MIDI corrente |
| `djfly.presetVer` | Versione del preset già proposta |
| `djfly.ctrlName` | Nome della porta MIDI del controller in uso |

Gli ID skin salvati prima della rinomina vengono migrati all'avvio. I brani **non** sono persistiti: a ogni riavvio vanno riaggiunti (vedi roadmap).

## PWA e offline

- `manifest.json`: `display: fullscreen`, `orientation: landscape`.
- `sw.js`: cache `flydj-v1` con `dj-live.html`, `manifest.json`, `icon.svg`; strategia **network-first** con fallback in cache, solo richieste GET same-origin.
- L'installazione richiede HTTPS.

## Landing e deploy

La landing (`landing/index.html`) è statica e autonoma: bilingue IT/EN con rilevamento della lingua e scelta salvata, responsive da 320 px, nessun framework, screenshot 3200×1800 in WebP.

| File | Ruolo |
|---|---|
| `vercel.json` | `outputDirectory: landing`, nessuna build, `cleanUrls`, header `X-Robots-Tag: noindex, nofollow` su ogni percorso. |
| `.vercelignore` | Esclude tutto tranne `landing/` e `vercel.json` dagli upload CLI. |

Produzione: **https://flydj.cdalise.com** (CNAME su Cloudflare, DNS only, verso Vercel). Verifica:

```bash
curl -sI https://flydj.cdalise.com/ | grep -i x-robots-tag          # noindex, nofollow
curl -sL -o /dev/null -w "%{http_code}\n" https://flydj.cdalise.com/dj-live.html   # 404
```

La pagina non va inserita in sitemap né bloccata in `robots.txt` (altrimenti i crawler non leggono il `noindex`).

## Limiti noti

- Stems approssimati (mid/side), non paragonabili a una separazione con IA.
- Nessun keylock: cambiare tempo cambia anche il tono.
- Libreria non persistente.
- Comandi vocali solo dimostrativi: con musica alta il microfono riprende le casse.
- Su Android la cuffia separata non è disponibile (la cuffia sente il master).
- File unico da ~2.000 righe: da dividere in moduli.

## Roadmap

| Fase | Contenuto | Stato |
|---|---|---|
| 0 | Console web | Fatto |
| 0+ | Libreria persistente (File System Access + IndexedDB), keylock (AudioWorklet time-stretch), gesti più affidabili, moduli | In corso |
| 1 | Repository del codice | Fatto |
| 2 | Web app online in HTTPS e prova su tablet | Prossimo |
| 3 | Backend unico: account, licenze, sync di preset, dati dei brani, playlist, impostazioni | Prossimo |
| 4 | App desktop (.exe) firmata con licenza, installer e aggiornamenti automatici | Prossimo |
| 5 | Telefono: telecomando e preparazione del set | Prossimo |

Dettagli in [`docs/PROMEMORIA_PROGETTO.md`](docs/PROMEMORIA_PROGETTO.md).

---

© 2026 CD FlyDJ Track Pro. Tutti i diritti riservati. I nomi di prodotti di terzi eventualmente citati appartengono ai rispettivi proprietari.
