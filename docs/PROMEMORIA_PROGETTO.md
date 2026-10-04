# CD FlyDJ Track Pro — Promemoria di progetto

*Aggiornato al 03/10/2026. Cartella di lavoro: `Sviluppo/DropDeck`.*

---

## 1. Dove siamo oggi

La console è **una sola pagina web**: `dj-live.html`, senza installazione né server, aperta in Chrome o Edge.

| File | A cosa serve |
|---|---|
| `dj-live.html` | La console completa (audio, interfaccia, MIDI, mani, voce) |
| `manifest.json`, `sw.js`, `icon.svg` | App installabile su tablet / PC (serve HTTPS) |
| `code.html` | Prototipo precedente "Gravity Touch – Regia LIVE" (gift TikTok + mani), separato |
| `docs/` | Questo promemoria |

### Funzioni già presenti

- **2 deck** con analisi BPM al centesimo, beat grid, waveform colorata per frequenza, overview, piatto con etichetta CD FLYDJ.
- **Sync** stile Serato: blu = battuta agganciata, oro = solo tempo (dopo scratch o nudge); correzione continua invisibile, half/double tempo.
- **Pad**: Hot Cue, Auto Loop (+ Roll con Shift), Fader Cuts, Sampler, e **pagina 2 EXTRA** (Shift + tasto modalità): Voce off, Musica off, Bassi off, Horn, Sync, Slip, Censor, Tap.
- **Loop** ½ ×2, reloop, loop in/out; **Slip**, **Censor**, **scratch vero** avanti e indietro.
- **Mixer**: Level, EQ 3 bande, Filter/Color, fader, CUE cuffia, crossfader, master, **CUE MIX** software.
- **FX divisi per deck** (sinistra deck 1, destra deck 2): HPF, LPF, Flanger, Echo, Reverb, Phaser, con Beats e Wet.
- **Sound Color FX** (skin Pioneer): Dub Echo, Sweep, Noise, Filter.
- **Stems senza IA** (mid/side): Voce off, Musica off, Bassi off. È un'approssimazione, non pulita come gli stems di Serato.
- **Libreria** con tab Files / Browse / Prepare / History, ricerca, Auto Gain, Instant Doubles, **REC** del mix in WAV.
- **Skin**: Serato, Pioneer DJM-750MK2, Pioneer XDJ. **Modalità tablet**, schermo sempre acceso, app installabile.
- **MIDI Numark Mixtrack Pro FX**: preset incorporato (66 comandi), modalità **🎯 LEARN** (tocca a schermo + muovi sul controller), configurazione guidata, export/import della mappa, luci.
- **Audio**: scheda a 4 canali → casse sui canali 1-2 e cuffia sui canali 3-4; **🔊 TEST AUDIO** con diagnosi.
- **Mani (webcam)**: mano sinistra blu = deck 1, destra rossa = deck 2; aperta = puntatore, pugno = tocco; pugno su un brano = lo carica su quel deck.
- **Voce (DEMO)**: "play 1", "stop 2", "stop tutti"… con circa 1 s di latenza.

---

## 2. Decisioni prese

| Tema | Decisione |
|---|---|
| **Voce** | **Solo dimostrativa.** Con la musica alta il microfono sente le casse e si confonde. Nel prodotto finale non è un comando affidabile. |
| **Uso in serata** | **Console Numark + schermo** (PC o tablet). La voce resta per demo e presentazioni. |
| **Mani** | Da sistemare in una fase dedicata. Voce e mani **non insieme**: accendendo una, l'altra si spegne. |
| **Nome** | **CD FlyDJ Track Pro** (logo e piatti). |

---

## 3. Da completare sulla console (prima del backend)

- [ ] **Preset MIDI definitivo**: collegare con LEARN Pitch, Filter, Level, Bend +, Browse (gira), Wet/Dry, Phaser, leve FX, Cue Mix. Poi ESPORTA e incorporare nel programma.
- [ ] Verificare sul controller i comandi dedotti: ordine dei 4 pad, fila bassa (Stutter / ⏮ / ◀◀), Shift, CUE cuffia.
- [ ] **Audio**: confermare con 🔊 TEST AUDIO che casse e cuffia suonano entrambe col Mixtrack.
- [ ] **Libreria persistente**: oggi a ogni riavvio i brani vanno riaggiunti. Soluzione: cartella musica collegata (File System Access API) + dati dei brani salvati (BPM, griglia, cue) in IndexedDB.
- [ ] **Mani**: rendere il pugno più affidabile; eventuale taratura per persona.
- [ ] **Keylock** (tempo senza cambiare tono): serve un motore di time-stretch (AudioWorklet).
- [ ] Dividere `dj-live.html` in moduli (audio, MIDI, interfaccia, libreria) quando si passa a GitHub: oggi è un file unico di circa 1.900 righe.

---

## 4. Roadmap

### Fase 1 — Repository GitHub
- Repository **privato** dedicato (es. `cd-flydj-track-pro`), separato dalla cartella utente: oggi `DropDeck` sta dentro un repository più grande insieme a tutta la home.
- Struttura proposta:
  ```
  cd-flydj-track-pro/
    app/        ← la console (oggi dj-live.html + manifest, sw, icona)
    desktop/    ← involucro .exe
    backend/    ← Supabase: database, funzioni, licenze
    docs/       ← questo promemoria, manuali, mappe MIDI
  ```
- `.gitignore` per chiavi, file `.env`, build, musica di prova.
- Branch `main` stabile + branch di lavoro; versioni con tag (`v1.0.0`…).

### Fase 2 — Web app su Vercel
- Pubblicare la console su **Vercel** in **HTTPS**: obbligatorio per MIDI, webcam, microfono e installazione come app sul tablet.
- Dominio dedicato (es. `app.cdflydj.…`).
- Sul Samsung Tab: Chrome → "Aggiungi a schermata Home".
- Limiti Android da ricordare: niente cuffia separata (la cuffia sente il master); il Mixtrack via USB-C OTG può aver bisogno di un **hub alimentato**.

### Fase 3 — Backend separato
**Raccomandazione: un solo backend per tutto (web, PC e telefono) = Supabase.** Due backend (Supabase + Firebase) vorrebbero dire due database da tenere allineati.

| Cosa | Dove |
|---|---|
| Account e login | Supabase Auth |
| Licenze (chiave, dispositivi, scadenza) | Tabella `licenses` + Edge Function di verifica |
| Sync generale | Postgres con sicurezza per utente (Row Level Security) |
| Preset MIDI, skin, impostazioni | Tabelle per utente |
| File (campioni, copertine) | Supabase Storage |

**Cosa sincronizzare tra dispositivi:**
- mappe MIDI e preset controller;
- dati dei brani: BPM corretti, griglia, hot cue, loop, Auto Gain, note (non l'audio);
- Prepare, History e playlist;
- impostazioni (skin, tablet, mani, CUE MIX).

**Firebase** solo se serve qualcosa che Supabase non copre bene (es. notifiche push sul telefono con FCM). In quel caso Firebase fa solo le notifiche, i dati restano su Supabase.

**Sicurezza:**
- mai la chiave `service_role` dentro l'app;
- solo la chiave pubblica + Row Level Security;
- le verifiche di licenza girano nelle Edge Functions.

### Fase 4 — Programma .exe con licenza
- **Involucro consigliato: Electron.** Usa lo stesso Chrome della web app, quindi Web MIDI, scelta della scheda audio e AudioWorklet funzionano uguali. Alternativa più leggera: Tauri (WebView2), da verificare per il MIDI.
- **Licenza:**
  - chiave legata all'account e a un numero massimo di PC;
  - verifica online all'avvio tramite Edge Function;
  - **periodo offline** (es. 14 giorni) per suonare senza internet;
  - in prova: funzioni limitate o filigrana sul REC.
- **Firma del codice** (certificato Windows) per non far bloccare l'installer da SmartScreen.
- Installer + aggiornamenti automatici (electron-builder / electron-updater, release su GitHub privato o server).

### Fase 5 — Telefono
- Stessa web app installabile (PWA) oppure app dedicata più avanti.
- Usi realistici sul telefono: **telecomando e preparazione** (playlist, cue, impostazioni) con sync verso la console. Mixare sul telefono col controller resta limitato.

---

## 5. Attenzione prima di vendere

- **Marchi:** le skin "Serato", "Pioneer DJM-750MK2", "Pioneer XDJ" e i nomi Numark vanno bene per uso personale. In un prodotto a pagamento conviene **nomi generici** ("Club Mixer", "Pro Deck"…) e nessun logo altrui. Il supporto al controller si può scrivere come "compatibile con Numark Mixtrack Pro FX".
- **Musica:** il programma non distribuisce brani; l'utente usa i suoi file.
- **Privacy:** webcam e microfono restano sul dispositivo (le mani sono elaborate in locale; la voce di Chrome passa da Google: va scritto nell'informativa se resta la demo).

---

## 6. Prossimo passo concreto

1. Chiudere il preset MIDI e il test audio col Mixtrack (sezione 3).
2. Creare il repository GitHub e spostarci la console.
3. Pubblicare su Vercel e provare sul Samsung Tab.
4. Avviare il progetto Supabase (login + sync preset).
5. Involucro Electron + licenza.
