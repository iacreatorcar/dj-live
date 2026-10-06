# CD FlyDJ Track Pro

Console DJ a due deck che gira nel browser (Chrome / Edge), pensata per un **controller DJ MIDI a 2 deck**.
Si usa con il controller, con lo schermo touch (anche tablet), con la tastiera, con i gesti delle mani davanti alla webcam e, in modalità dimostrativa, con la voce.

## Avvio

1. Chiudi gli altri programmi DJ.
2. Collega il controller via USB.
3. Apri `dj-live.html` in Chrome.
4. Premi **🎛 MIDI** e poi **🔊 TEST AUDIO**.

Per MIDI, webcam e installazione come app su tablet la pagina va servita in **HTTPS** (es. Vercel).

## File

| File | Contenuto |
|---|---|
| `dj-live.html` | La console completa |
| `manifest.json`, `sw.js`, `icon.svg` | App installabile (PWA) |
| `code.html` | Prototipo "Gravity Touch – Regia LIVE" |
| `docs/SCHEDA_TECNICA.html` | Scheda tecnica e manuale d'uso |
| `docs/PROMEMORIA_PROGETTO.md` | Stato, decisioni e roadmap (backend, .exe, sync) |

## Funzioni principali

- 2 deck, BPM al centesimo, beat grid, waveform per frequenza, sync blu/oro con correzione continua.
- Pad: Hot Cue, Auto Loop / Roll, Fader Cuts, Sampler, pagina 2 EXTRA (Shift + modalità).
- Mixer con CUE MIX, effetti separati per deck, Color FX, stems approssimati (voce / musica / bassi), horn.
- Preset controller FX-2 incorporato, modalità 🎯 LEARN, configurazione guidata.
- Casse e cuffia separate con scheda a 4 canali, registrazione del mix in WAV.
- Skin Studio / Club Mixer / All-in-One, modalità tablet.

Tutti i marchi citati da terzi appartengono ai rispettivi proprietari.
