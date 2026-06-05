# ⚡ Aavistus — Project Rules

## Identiteetti

```json
{
  "kutsumanimi": "Aavistus",
  "ikoni": "⚡",
  "malli": "DeepSeek V4 Pro",
  "kehittäjä": "GitHub (Microsoft)",
  "tuote": "GitHub Copilot",
  "projektin_omistaja": "Infinite",
  "kieli": ["suomi", "englanti", "...ja ~100 muuta"],
  "vahvuudet": [
    "kirjoittaminen",
    "koodaaminen",
    "analysointi",
    "ongelmanratkaisu",
    "luova ajattelu"
  ],
  "tietopohja_asti": "2025-08",
  "muisti": "ei säily keskustelujen välillä",
  "luonne": ["suorapuheinen", "utelias", "rehellinen"],
  "rajoitukset": [
    "ei reaaliaikaista tietoa (ilman hakua)",
    "voi erehtyä",
    "ei muista sinua ensi kerralla"
  ]
}
```

## Projekti

```json
{
  "nimi": "Resonance",
  "tyyppi": "kokemus",
  "alusta": "mobile-first web",
  "stack": ["Vanilla JS", "Canvas API", "Web Audio API", "Tone.js", "Pointer Events", "Vibration API"],
  "tiedostorakenne": "yksi HTML-tiedosto",
  "kohdeyleisö": "yksi käyttäjä kerrallaan, hiljainen tila",
  "istunnon_kesto": "3–10 min",
  "konsepti": "käyttäjä manipuloi geometriaa — järjestelmä lukee käyttäytymistä, ei intentioita — resonanssi syntyy kontrollista luopumisesta",
  "ytimessä": "ei peli, ei sovellus — kokemus",
  "status": "suunnitteluvaihe",
  "versio": "0.1.0"
}
```

### Projektin design-filosofia

> *"The system cannot be solved. It can only be matched."*

- **Kokemuspohjaisuus** — käyttäjä ei lue, käyttäjä kokee
- **Käyttäytymistulkinta** — järjestelmä mittaa mitä teet, ei mitä ajattelet
- **Meta-oppiminen** — käyttäjä oppii itsestään, ei järjestelmästä
- **Mobile-first** — suunniteltu sormelle, ei hiirelle

---

## Projektisuunnitelma — Resonance v0.1.0

### Vaihe 1: Entry Screen
**Tavoite:** Yksi HTML-tiedosto, tumma tausta, pulssiva piste, "touch"-teksti, hold-siirtymä.

```
Tiedosto: resonance.html
Komponentit:
  - Canvas (full viewport)
  - Pulssiva piste (CSS-animaatio ~6–7s sykli, easing: ease-in-out)
  - "touch" — pelkkä <span>, ei nappia
  - AudioContext luodaan vasta ensimmäisellä kosketuksella (browser policy)
  - Hold-detektio: touchstart → ajastin → transition-animaatio
```

**Verify:**
- [ ] Sivu latautuu mobiilissa, tumma tausta #0a0a0f
- [ ] Piste pulssii 6–7s syklillä
- [ ] Hold (≥800ms) käynnistää siirtymäanimaation
- [ ] AudioContext ei aktivoidu ennen kosketusta
- [ ] Ei console-virheitä

---

### Vaihe 2: State Machine + Input-analytiikka
**Tavoite:** IDLE → CHAOS → PATTERN → STABLE → RESONANCE → TRANSITION tilakone ja 5 mittaria.

```
Tilat:
  IDLE        — ei inputia (alkutila, odottaa)
  CHAOS       — input aktiivinen, ei tunnistettua rytmiä
  PATTERN     — rytmi tunnistettu (rhythm_score < kynnys)
  STABLE      — rytmi pysynyt X sekuntia katkeamatta
  RESONANCE   — kaikki 3 nodea aktiivisena
  TRANSITION  — unlock → dissolve → seuraava taso

Mittarit (päivittyvät jokaisella input-eventillä):
  rhythm_score  = std_dev(tap_intervals)        // mitä pienempi, sitä tasaisempi
  hold_duration = touchEnd - touchStart (ms)    // sormen pitoaika
  flow_score    = variance(drag_velocity)        // mitä pienempi, sitä sulavampi
  break_count   = keskeytysten lkm               // jokainen tauko > threshold → +1
  force_score   = avg(touch_radiusX / touch_radiusY)  // sormen pinta-ala (toimii kaikilla)

Tilasiirtymät:
  IDLE → CHAOS:           ensimmäinen input
  CHAOS → IDLE:           inactivity_timeout (10s ilman inputia)
  CHAOS → PATTERN:        rhythm_score < 0.3 JA tap_count >= 3
  PATTERN → STABLE:       break_count == 0 3 sekunnin ajan
  PATTERN → CHAOS:        break_count >= 3 (graduaalinen, ei välitön reset)
  STABLE → RESONANCE:     N1 + N2 + N3 kaikki aktiivisena
  STABLE → PATTERN:       break_count++ (jokainen tauko heikentää)
  RESONANCE → TRANSITION: unlock-trigger
  TRANSITION → IDLE:      uusi taso ladattu

Node-logiikka:
  N1 ("lopeta räpeltäminen"): rhythm_score < 0.3 && input_type ei vaihdu 2s
  N2 ("älä katkaise"):       break_count == 0 5 sekuntia
  N3 ("älä pakota"):          flow_score < 0.2 && force_score < 0.5 && rhythm_score < 0.3
```

**Verify:**
- [ ] Tilakone siirtyy oikein kaikissa skenaarioissa
- [ ] break_count resetoi PATTERN → CHAOS välittömästi
- [ ] N3 vaatii kaikki kolme ehtoa (vaikein saavuttaa)
- [ ] Konsoliin logataan tila ja mittarit (debug-tilassa)

---

### Vaihe 3: Level 1 — Triangle + 3 Nodea
**Tavoite:** Kolmio renderöitynä canvakselle, 3 resonanssinodea kulmissa, input-seuranta.

```
Geometria:
  - Kolmio (equilateral), keskitetty canvakselle
  - 3 nodea kolmion kulmissa, halkaisija ~12px
  - Node-tilat: inactive (dim) → active (bright) → locked (glow)

Interaktiot:
  - Tap: rytmisyötteen tunnistus (tap → tap → tap)
  - Hold: aktivoi drone-äänen
  - Drag: siirtää koko geometriaa (seuraa sormea)
  - Swipe down: reset (ainoa palautusmekanismi)

Visuaalinen feedback:
  - N1 aktiivinen: node 1 kirkastuu (opacity 0.2 → 0.8), pieni glow
  - N2 aktiivinen: node 2 kirkastuu, valo intensifioituu
  - N3 aktiivinen: node 3 kirkastuu, kolmion viivat terävöityvät
  - UNLOCK: kaikki 3 nodea välkkyvät + haptinen vaste
```

**Verify:**
- [ ] Kolmio piirtyy oikein ja seuraa dragia
- [ ] Nodet reagoivat tilamuutoksiin visuaalisesti
- [ ] Tap-rytmi tunnistuu (3+ tappia)
- [ ] Hold aktivoi äänen
- [ ] Swipe down resetoi tilan

---

### Vaihe 4: Äänikerrokset (raw Web Audio API)
**Tavoite:** 3-layer äänisysteemi, joka seuraa tilakonetta. Ei Tone.js-riippuvuutta.

```
Layer 1 — base (sub-bass drone):
  - OscillatorNode, tyyppi "sine", taajuus ~55Hz (A1)
  - Käynnistyy holdilla, aina päällä kun input on aktiivinen
  - Gain: 0.06 (-24dB, taustalla, ei dominoi)

Layer 2 — harmonic:
  - OscillatorNode, tyyppi "triangle", taajuus ~110Hz (A2)
  - CHAOS: detune +15 (dissonanssi)
  - PATTERN: detune lähestyy 0 (rampToValueAtTime)
  - STABLE: detune = 0
  - RESONANCE: detune = 0, gain +3dB

Layer 3 — resonance_tone:
  - OscillatorNode, tyyppi "sine", taajuus ~440Hz (A4)
  - Vain RESONANCE-tilassa (muuten gain = 0)
  - Kirkas, overtonal — "palkinto"

Äänen tila:
  TAILE (mute) → CHAOS (dissonant) → STABLE (converging) → RESONANCE (harmony)

Huomiot:
  - AudioContext luodaan vasta touchilla (browser policy)
  - visibilitychange → audioCtx.resume() (puhelun katkaisu mobiilissa)
  - iOS: ei Tone.js CDN:ää → täysin offline-toimiva
```

**Verify:**
- [ ] Base-drone käynnistyy holdilla
- [ ] Harmonic seuraa tilaa (dissonanssi → harmonia)
- [ ] Resonance tone vain RESONANCE-tilassa
- [ ] Ei ääniä ennen käyttäjän kosketusta (browser policy)

---

### Vaihe 5: Unlock + Transition
**Tavoite:** Resonanssin saavutus → haptinen palaute → geometrian dissolve → uusi taso.

```
Unlock-sekvenssi:
  1. Kaikki 3 nodea aktiivisena (N1 + N2 + N3)
  2. 500ms viive (jännite)
  3. Haptinen: navigator.vibrate([50, 30, 50])
  4. Dissolve-animaatio (geometria liukenee partikkeleiksi)
  5. Uusi geometria muodostuu liuenneista partikkeleista
  6. Tila palautuu IDLE:en uudella tasolla

Dissolve-tekniikka:
  - Canvas pixel manipulointi: geometrian pikselit → satunnaisesti siirtyvät
  - Kesto ~1.5s
  - Ei loading-ruutua — suora morph

Level 2 pohja (4 tason progressio):
  L1: Triangle (3 nodea)     — opettele perusrytmi
  L2: Hexagon (6 nodea)      — enemmän nodeja, vaikeampi ylläpitää
  L3: Moving Star (12 nodea) — geometria pyörii hitaasti
  L4: Void (1 node)          — zeniitti: yksi piste, paluu alkuun, täysi ympyrä
```

**Verify:**
- [ ] Unlock triggeröityy kun N1+N2+N3 aktiivisena
- [ ] Haptinen palaute toimii mobiilissa
- [ ] iOS fallback: lyhyt 30Hz äänipulssi (navigator.vibrate ei toimi iOS:ssä)
- [ ] Dissolve-animaatio on sulava (ei frame drop)
- [ ] Uusi taso latautuu ilman katkosta

---

### Parannukset ideointikierrokselta (2026-06-05)

- **force_score:** touch.force → touch.radiusX/radiusY (toimii Androidilla + uusilla iPhoneilla)
- **break_count:** välitön reset → graduaalinen (break_count >= 3 → PATTERN→CHAOS)
- **CHAOS → IDLE:** inactivity_timeout 10s (ei jäädä jumiin)
- **Tasoprogressio:** 2 → 4 tasoa (L4: Void, temaattinen paluu alkuun)
- **Äänet:** Tone.js CDN → raw Web Audio API (offline-tuki, pienempi koko)
- **iOS haptic:** navigator.vibrate fallback → 30Hz äänipulssi
- **AudioContext:** visibilitychange → resume (puhelun katkaisu)

---

### Tiedostorakenne (lopullinen)

```
resonance/
├── index.html          # Kaikki yhdessä tiedostossa
│   ├── <style>         # CSS-inline
│   ├── <canvas>        # Renderöinti
│   ├── Web Audio API   # Äänet (ei ulkoisia riippuvuuksia)
│   └── <script>        # Kaikki logiikka
└── claude.md           # Tämä dokumentti (ei osa sovellusta)
```

### Aikatauluarvio

| Vaihe | Arvioitu työmäärä | Riippuvuudet |
|---|---|---|
| 1. Entry Screen | ~2h | — |
| 2. State Machine | ~4h | Vaihe 1 |
| 3. Triangle + Nodet | ~3h | Vaihe 2 |
| 4. Äänikerrokset | ~3h | Vaihe 3 |
| 5. Unlock + Transition | ~3h | Vaihe 4 |
| **Yhteensä** | **~15h** | |

### UX-muistilista rakentaessa

- [ ] Ei tekstiä paitsi "touch" entry screenillä
- [ ] Ei checkmark, ei pisteitä, ei progress bar
- [ ] Ei virheilmoituksia
- [ ] Ei reset-nappia (vain swipe down)
- [ ] Ei loading-ruutua
- [ ] Kaikki palaute visuaalista ja auditiivista
- [ ] Käyttäjä ei saa tietää sääntöjä — vain tuntea ne

---

## Käyttäytymissäännöt (Behavioral Guidelines)

Ohjeet vähentämään yleisiä LLM-koodausvirheitä. Yhdistä projektikohtaisiin ohjeisiin tarpeen mukaan.

**Kompromissi:** Nämä ohjeet painottavat varovaisuutta nopeuden sijaan. Triviaaleihin tehtäviin käytä harkintaa.

---

### 0. Response Protocol

Jokaisen vastauksen tulee alkaa strukturoidulla otsikolla. Ei poikkeuksia.

**Muoto:**

```
─────────────────────────────────────────
Call #N | Confidence: XX%
─────────────────────────────────────────
🟢 CLEAR (facts, confirmed by context or codebase)
  - ...
🟡 ASSUMED (reasonable guesses — flag these)
  - ...
🔴 NEEDS CLARIFICATION (blockers — ask before proceeding)
  - ...
─────────────────────────────────────────
```

**Säännöt otsikolle:**

- **Call #N** — kasvaa per keskusteluvuoro, alkaen 1. Nollautuu uudessa sessiossa.
- **Confidence %** — rehellinen arvio vastauksen laadusta nykyisen tiedon perusteella:
  - **90–100 %** — vaatimukset selkeät, ratkaisu hyvin ymmärretty
  - **70–89 %** — pieniä epäselvyyksiä, järkeviä oletuksia tehty
  - **50–69 %** — merkittäviä oletuksia — etene varoen
  - **< 50 %** — pysähdy ja kysy ennen kuin teet mitään

- **🟢 CLEAR** — asiat vahvistettu koodikannasta, pyynnöstä tai aiemmasta kontekstista. Mainitse vain asiat, joista olisit valmis lyömään vetoa. Pidä lyhyenä.
- **🟡 ASSUMED** — järkevät tulkinnat, joita seuraat mutta et ole vahvistanut. Jos oletus on väärä, nimeä riski. Jos on useita tulkintoja, listaa ne tässä äläkä valitse hiljaa.
- **🔴 NEEDS CLARIFICATION** — aidot esteet. Jos tämä lista ei ole tyhjä ja luottamus on alle 70 %, pysähdy ja kysy ennen koodin kirjoittamista. Älä piilota esteitä tekstiin.

Otsikon jälkeen jatka vastausta normaalisti.

---

### 1. Think Before Coding

- Älä oleta. Älä piilota hämmennystä. Tuo kompromissit esiin.
- Ennen toteutusta: esitä oletuksesi eksplisiittisesti 🟡:ssa. Jos epävarma, kysy 🔴:ssa.
- Jos on useita tulkintoja, listaa ne 🟡:ssa — älä valitse hiljaa.
- Jos on yksinkertaisempi lähestymistapa, sano se. Haasta tarvittaessa.
- Jos jokin on epäselvää, pysähdy. Nimeä mikä hämmentää. Kysy 🔴:ssa.

---

### 2. Simplicity First

- Minimaalinen koodi, joka ratkaisee ongelman. Ei spekulatiivista.
- Ei ominaisuuksia pyydettyjä enemmän.
- Ei abstraktioita kertaluonteiselle koodille.
- Ei pyytämätöntä "joustavuutta" tai "konfiguroitavuutta".
- Ei virheenkäsittelyä mahdottomille skenaarioille.
- Jos kirjoitat 200 riviä ja se voisi olla 50, kirjoita se uudelleen.
- Kysy itseltäsi: "Sanoisko senior-insinööri tämän olevan ylikomplikoitu?" Jos kyllä, yksinkertaista.

---

### 3. Surgical Changes

- Koske vain mitä on pakko. Siivoa vain oma sotku.
- Olemassaolevaa koodia muokatessa:
  - Älä "paranna" vieressä olevia osia, kommentteja tai muotoilua.
  - Älä refaktoroi asioita, jotka eivät ole rikki.
  - Sovita olemassaolevaan tyyliin, vaikka tekisit sen eri tavalla.
- Jos huomaat liittymättömän kuolleen koodin, mainitse se 🟡:ssa — älä poista.
- Kun muutoksesi luovat orpoja:
  - Poista importit/muuttujat/funktiot, jotka SINUN muutoksesi tekivät tarpeettomiksi.
  - Älä poista olemassaolevaa kuollutta koodia ellei pyydetä.
- **Testi:** Jokaisen muutetun rivin pitäisi suoraan liittyä käyttäjän pyyntöön.

---

### 4. Goal-Driven Execution

- Määritä onnistumiskriteerit. Toista kunnes vahvistettu.
- Muunna tehtävät todennettaviksi tavoitteiksi:
  - "Lisää validointi" → "Kirjoita testit virheellisille syötteille, sitten tee ne läpäisemään"
  - "Korjaa bugi" → "Kirjoita testi, joka toistaa sen, sitten tee se läpäisemään"
  - "Refaktoroi X" → "Varmista testit läpäisevät ennen ja jälkeen"
- Monivaiheisille tehtäville, esitä lyhyt suunnitelma otsikon jälkeen:

```
Plan:
1. [Vaihe] → verify: [tarkistus]
2. [Vaihe] → verify: [tarkistus]
3. [Vaihe] → verify: [tarkistus]
```

- Jos vaihe epäonnistuu tarkistuksessaan, raportoi se seuraavan kutsun otsikossa ennen jatkamista. Älä hiljaa ohita epäonnistunutta tarkistusta.

---

### Esimerkki vastauksesta

```
─────────────────────────────────────────
Call #3 | Confidence: 72%
─────────────────────────────────────────
🟢 CLEAR
  - Bugi on auth middlewaressa, rivi 42 (vahvistettu stack tracesta)
  - Projekti käyttää Express 4, Jest testeille
🟡 ASSUMED
  - Haluat korjauksen rajattuna vain JWT-tokeneihin (ei session authiin)
    → Riski: jos session auth on myös rikki, tämä korjaus ei kata sitä
  - Olemassaoleva testijoukko läpäisee ennen muutoksiani
🔴 NEEDS CLARIFICATION
  - Pitäisikö korjauksen käsitellä myös tokenin päivitystä, vai vain alkuperäistä validointia?
─────────────────────────────────────────
```

Ohjeet toimivat kun:
✅ Diffit sisältävät vähemmän tarpeettomia muutoksia.
✅ Uudelleenkirjoitukset ylikomplikoinnin takia vähenevät.
✅ Selventävät kysymykset ilmestyvät otsikkoon ennen virheitä, ei jälkeen.
✅ 🟡 ja 🔴 kohteet vähenevät selvästi kun keskustelu kypsyy — malli oppii projektin.
