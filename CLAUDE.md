# CLAUDE.md — Hokkaidó interaktivní itinerář

Brief a technická dokumentace pro AI asistenty pracující na tomto projektu.

> **Pozn. k repozitáři:** Do gitu se ukládá **pouze tento soubor `CLAUDE.md`**.
> Zdrojové soubory projektu (HTML/JS/JSON, viz níže) se do gitu **nesourcují** —
> git slouží jen jako úložiště tohoto briefu. Při práci se zdrojovými soubory je
> drž mimo verzování (lokálně / v přílohách), pokud uživatel neřekne jinak.

---

## 1. Co projekt je

Interaktivní webová stránka: **18denní cestovní itinerář po Hokkaidó** (časový
rámec = začátek září). Obsahuje:

- **Reálnou mapu** (Leaflet) s POI body rozdělenými do barevných kategorií.
- **Fotky míst** (Wikimedia Commons) s elegantním fallbackem.
- **Denní rozpis** (11 zastávek / 18 dní) s aktivitami, zajímavostmi a restauracemi.
- **Přehrání trasy** — animovaný průchod zastávkami.
- **Responzivní** chování (na mobilu přepínání panel ↔ mapa).

Jazyk obsahu: **čeština**, japonské názvy míst jako sekundární popisky.

---

## 2. Zdrojové soubory (mimo git)

| Soubor | Role |
|---|---|
| `hokkaido-mapa.html` | **Funkční referenční verze.** Obsahuje veškerá data (`DATA`, `PHOTOS`, `CAT`), mapu, panel, přehrávání trasy. Obsahový základ — data jsou správná. |
| `hokkaido-zari-pruvodce.html` | Sezónní průvodce (text) — zdroj pro úvod/sezónní rámec. Statická typografická stránka. |
| `_photos.json` | Mapování `id zastávky → URL fotky` (Wikimedia Commons, `?width=520`). |

Při **přepracování designu** ber `hokkaido-mapa.html` jako zdroj dat a funkcí,
ale design dělej znovu podle referencí v sekci 5. **Obsah a data nikdy neztrať.**

---

## 3. Datový model

Data žijí v konstantě `DATA` (pole 11 objektů, jeden na zastávku). Klíčová pole:

```js
{
  n: 1,                    // id zastávky (1–11), klíč i do PHOTOS
  d: "Den 1–2",           // rozsah dní (popisek)
  loc: "Hakodate",        // název v češtině
  jp: "函館",              // japonský název (sekundární popisek)
  cat: "city",            // hlavní kategorie zastávky (klíč do CAT)
  lat: 41.7609, lng: 140.7142,  // souřadnice pro pin a flyTo
  tag: "Jih · vstupní bod · …", // jednořádkový tagline
  blurb: "…",             // krátký popis do popupu
  groups: [               // skupiny položek dle kategorie
    { cat: "food",
      items: [
        // [název, japonsky, popis, URL do Google Maps (může být "")]
        ["Ranní rybí trh", "朝市", "Vlastní oliheň…", "https://maps.google.com/?q=…"]
      ],
      note: "…"           // volitelná poznámka pod skupinu
    }
  ]
}
```

Po definici se každé zastávce doplní fotka: `DATA.forEach(d => d.img = PHOTOS[d.n])`.

### Kategorie POI (`CAT`)

| Klíč | Význam | Barva | Ikona |
|---|---|---|---|
| `nature` | Příroda | `#3a7d52` | 🌲 |
| `view` | Vyhlídka | `#7a4b8c` | 🌅 |
| `onsen` | Onsen | `#b5642a` | ♨️ |
| `city` | Město | `#2a5b82` | 🏙️ |
| `culture` | Kultura | `#b08800` | ⛩️ |
| `food` | Jídlo | `#c5402a` | 🍴 |

### Trasa (pořadí zastávek 1→11)

Hakodate → Tója/Noboribecu → Sapporo → Otaru/Šakotan → Cape Sójá → Asahidake →
Furano/Biei → Abaširi → Širetoko → Akan-Mašú → Kuširo (→ zpět do Sappora).

---

## 4. Architektura referenční verze (`hokkaido-mapa.html`)

Jediný HTML soubor, žádný build. Vše inline (`<style>` + `<script>`). Závislost:
**Leaflet 1.9.4** (CSS + JS přes CDN), dlaždice **CARTO Voyager**.

Hlavní stavební bloky JS:

- `pinHtml(d, state)` — SVG pin v `L.divIcon`; stavy `""` / `"cur"` (zvýrazněný,
  zlatý, zvětšený) / `"dim"` (ztlumený). Číslo zastávky je v `.lbl`.
- `popHtml(d, i)` — obsah Leaflet popupu (foto, kategorie, blurb, max 2 odkazy,
  tlačítko „Zobrazit vše v panelu").
- Render panelu — pro každou zastávku `.day` karta s `.daythumb` (foto),
  `.dayrow` (číslo/název/JP) a rozbalitelným `.daybody` (skupiny položek).
- `openDay(i)` — centrální akce: rozbalí kartu, přepne stav pinů (cur/dim),
  dokreslí „zlatou" část trasy (`goldLine`), `map.flyTo` na zastávku, otevře popup.
- Přehrávání trasy — tlačítko `#play` prochází zastávky (`setTimeout`, ~2,6 s/krok),
  ukazuje progress bar `#prog`. `#reset` vrátí výchozí stav a `flyToBounds`.
- Mapa: `baseLine` (přerušovaná celá trasa) + `goldLine` (projitá část).
- Mobil: `#mobtoggle` přepíná `.panel.hidden` (panel se schová pod mapu).

### Konvence kódu reference

- Styly přes **CSS proměnné** v `:root` (paleta `--ink/--paper/--vermilion/…`
  a kategorie `--c-nature` atd.). Drž paletu konzistentní.
- `@media (prefers-reduced-motion: reduce)` vypíná přechody — **zachovej** při
  každém přepracování animací.
- Fotky se vkládají jako `background-image` se dvěma vrstvami:
  `url('…'), linear-gradient(135deg, <barva>, <barva>aa)` → gradient je fallback,
  když se obrázek nenačte. **Tenhle vzor fallbacku zachovej.**
- Externí odkazy vždy `target="_blank" rel="noopener"`.

---

## 5. Designová reference — japonský minimalismus

Cíl: estetika na úrovni **Awwwards** v kategoriích *Minimal* a *Japan*. Reference
(vytěž principy, nekopíruj): **AMAN Resorts**, **Boutique Japan**, galerie
Awwwards *Minimal* / *Japan*.

Principy, které je nutné dodržet:

1. **Bílý prostor jako hlavní materiál** — velkorysé okraje, sekce oddělené
   prostorem, ne rámečky a linkami.
2. **Typografie nese osobnost** — výrazné display písmo + klidné body písmo;
   tenké vertikální kanji popisky, hodně letter-spacingu u majuskulí.
3. **Jeden akcentní tón** — vermilion/červená na jinak monochromatické paletě
   (bílá / uhlová / šedá). Kategorie POI smí mít barvy, ale tlumené.
4. **Pohyb střídmě** — jemné fade/scroll reveal, decentní hover, plynulé přechody
   na mapě. Méně je více.
5. **Mřížka a řád** — baseline, čísla dnů jako typografický prvek (01/02… nebo 一二三).
6. **Fotky dýchají** — velké, konzistentní ořezy; galerie s klidem, ne koláž.

Vhodná písma: Shippori Mincho / Zen Old Mincho (JP nádech) + kvalitní sans pro UI.

---

## 6. Funkční požadavky (zachovat při přepracování)

- [ ] Reálná mapa (Leaflet, CARTO Voyager) se zoomem a posunem.
- [ ] POI body barevně dle kategorie, klikatelné.
- [ ] Klik na bod nebo den → detail s fotkou, popisem, odkazy do Google Maps.
- [ ] Denní rozpis s aktivitami / zajímavostmi / restauracemi.
- [ ] „Přehrát trasu" — animovaný průchod zastávkami.
- [ ] Responzivní (mobil: přepínání panel ↔ mapa).
- [ ] Fotky reálné, s fallbackem; obsah v češtině, JP názvy sekundárně.

---

## 7. Sezónní rámec (začátek září) — promítni do úvodu

- **Asahidake / Daisecuzan** = úplně **první podzimní barvy v celém Japonsku**
  (kolem poloviny září, od vrcholků dolů), proti prvnímu sněhu. Hlavní bod cesty.
- Po letní špičce, mírné počasí (dny 20–25 °C), klidněji a levněji než v létě
  i než v říjnové špičce.
- Jídelní sezona na vrcholu: podzimní losos (aki-zake) a ikura, saury (sanma),
  krab. Korálová tráva u Abaširi rudne koncem září.
- Detailní text k sezoně je v `hokkaido-zari-pruvodce.html`.

---

## 8. Doporučený postup pro AI asistenta

1. Zůstaň u jednoho statického `index.html` (minimalismus snese statiku), nebo
   Vite (vanilla / React + TS) dle uvážení. Leaflet pro mapu, Google Fonts pro pár písem.
2. Překlop `DATA` a `PHOTOS` **beze změny obsahu** z `hokkaido-mapa.html`.
3. Postav bloky: Hero, Mapa, Den (karta), Detail POI, Legenda kategorií.
4. Projdi výsledek proti referencím (sekce 5). Když to vypadá jako generická
   šablona, přepracuj — vsaď boldness do **jednoho** prvku (hero typografie nebo
   mapa), zbytek nech tichý.
5. Otestuj responzivitu a načítání mapy i fotek (včetně fallbacku).

### Akceptační kritéria

- [ ] Vypadá to, že by to mohlo vyhrát Awwwards Minimal/Japan (ne šablona).
- [ ] Hodně bílého prostoru, jeden akcent, výrazná typografie.
- [ ] Mapa reálná, POI barevné a klikatelné, fotky reálné s fallbackem.
- [ ] Data a obsah beze ztráty oproti `hokkaido-mapa.html`.
- [ ] Plynulé, ale střídmé animace. Mobil funguje. `prefers-reduced-motion` respektován.

---

## 9. Git workflow

- Vývoj probíhá na větvi `claude/claude-md-docs-28d17g`.
- **Do gitu commituj pouze `CLAUDE.md`.** Zdrojové soubory projektu nech mimo
  verzování (viz pozn. nahoře), dokud uživatel neřekne jinak.
- Push: `git push -u origin <branch>`. PR vytvářej jen na explicitní žádost.
