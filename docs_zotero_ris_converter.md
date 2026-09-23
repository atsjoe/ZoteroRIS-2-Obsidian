# 🛠️ Fejlesztői Dokumentáció: Zotero RIS ➔ Obsidian Kétpólusú Kártyakonverter

> **Nyelvválasztó / Language:** [English Developer Documentation](docs_zotero_ris_converter_en.md) | **Alkalmazások:** [Magyar felület](public/zotero_ris_to_obsidian.html) / [English App](public/zotero_ris_to_obsidian_en.html)

**Fájl:** `public/zotero_ris_to_obsidian.html`  
**Architektúra:** Zero-dependency Vanilla HTML5 + CSS3 + ES6 JavaScript  
**Cél:** A Zotero és az Obsidian Zettelkasten közötti automatizált, veszteségmentes adatátvitel kétpólusú (`bib_*` ⟷ `src_*_notes`) kártyamodellben.

---

## 1. Rendszerfilozófia & A Kétpólusú Modell

A hagyományos megközelítésekben a kutatók gyakran vagy túl sok mindent zsúfolnak a bibliográfiai kártyára (olvashatatlan monolit), vagy teljesen elválasztják a forrást a jegyzetektől (elvesző kontextus).

Ez az eszköz a **kétpólusú reprezentációt** valósítja meg:
1. **Bibliográfiai Horgony (`bib_*`):**
   - Feladata: A forrás hivatalos könyvészeti azonosítója.
   - Szigorúan formázott YAML frontmatter (14-jegyű ID, szerző, év, kiadó, folyóirat, DOI, helyi Google Drive PDF link).
   - Zotero Pandoc hivatkozási kulcs (`[@citekey]`) a Word és publikációs export céljára.
   - Mutató a hozzá tartozó kutatói jegyzetre: `notes_card: "[[src_*_notes]]"`.
2. **Kutatói Jegyzetkártya (`src_*_notes`):**
   - Feladata: Az olvasási és szintetizálási folyamat munkaterülete.
   - Visszamutatás a bibliográfiára: `source_bib: "[[bib_*]]"`.
   - Tartalmazza a Zoteróból kigyűjtött idézeteket, a mélyhivatkozásokat (`zotero://open-pdf/...`) és a hallgató saját reflexióit.
   - Dataview lekérdezés az ebből a forrásból születő koncepcionális atomi kártyákra (`concept_*`).

---

## 2. RIS Mezők Leképzési Logikája

A beolvasó motor (`parseRis`) soronként dolgozza fel a szabványos RIS tag-eket:

| RIS Címke | Jelentés | Kinyert Érték / Feldolgozás | Célmező az Obsidianban |
| :--- | :--- | :--- | :--- |
| `TY  - ...` | Típus | Lásd Típusleképezési táblázat | `doc_type` (`bib_jour`, `bib_book`, stb.) |
| `AU` / `A1` | Szerző(k) | Tömbbe gyűjtve, pontosvesszővel elválasztva | `author` |
| `PY` / `Y1` | Megjelenési év | Regex 4-jegyű évszám keresés | `year` |
| `TI` / `T1` | Cím | Teljes cím idézőjelek védelmével | `title` |
| `JO` / `JF` | Folyóiratnév | Csak folyóiratcikk esetén (`bib_jour`) | `publication` |
| `BT` / `T2` | Könyvcím / Kötet | Fejezet (`bib_chap`) vagy konferencia (`bib_conf`) | `booktitle` |
| `PB` | Kiadó / Hivatal | Kiadó, egyetem vagy szabadalmi hivatal | `publisher` |
| `VL` | Kötet | Évfolyam / Volume | `volume` |
| `IS` | Füzet | Szám / Issue | `issue` |
| `SP` / `EP` | Oldalszámok | Kezdő és végoldal összefűzése (`SP-EP`) | `pages` |
| `DO` | DOI | Tisztított DOI azonosító | `doi` és kattintható DOI link |
| `UR` | URL | Elsődleges vagy generált DOI link | `url` |
| `AB` / `N2` | Absztrakt | Többsoros blokk idézetként formázva (`> `) | `## 📑 Hivatalos Absztrakt` |
| `N1` / `RN` | Megjegyzések / Jegyzetek | Zotero kigyűjtött annotációk | `src_*_notes` törzsszövege |
| `L1` | Helyi Fájl | Csatolt Google Drive PDF elérési útja | `archive` |

### Típusleképezési Táblázat (`RIS_TYPE_MAP`):
```javascript
const RIS_TYPE_MAP = {
    'JOUR': 'bib_jour',
    'ABST': 'bib_jour',
    'BOOK': 'bib_book',
    'CHAP': 'bib_chap',
    'CONF': 'bib_conf',
    'CPAPER': 'bib_conf',
    'PROCD': 'bib_proc',
    'THES': 'bib_thes',
    'PAT': 'bib_pat',
    'DATA': 'bib_data',
    'ART': 'bib_art',
    'GEN': 'bib_misc',
    'RPRT': 'bib_misc',
    'MANSC': 'bib_misc'
};
```

---

## 3. Algoritmikus Részletek

### A) 14-jegyű PKM Lego Standard ID Generálás
A PKM Lego szabvány előírja a másodperc-pontos, időbélyeg-alapú azonosítót:
`YYYYMMDDHHmmss` formátumban.
```javascript
function generate14DigitId(offsetSeconds = 0) {
    const now = new Date(Date.now() + offsetSeconds * 1000);
    // ... ééééhhnnóóppmm string összefűzése
}
```
A páros kártyák generálásakor a `bib_*` kártya az alapidőt (`offset = index * 2`), míg az `src_*_notes` kártya a rá következő másodpercet (`offset = index * 2 + 1`) kapja, így az ID-k soha nem ütköznek még kötegelt import esetén sem.

### B) Better BibTeX Szabványú Citekey Algoritmus
A `cleanAscii(str)` függvény normalizálja az UTF-8 karaktereket (`normalize('NFD')`), eltávolítja a diakritikus jeleket és az írásjeleket:
```
[Első szerző családneve ékezet nélkül][4 jegyű év][Cím első értékes szava ékezet nélkül]
Példa: Kovács, János (2026): A transzformatív tanulás... ➔ kovacs2026transzformativ
```

---

## 4. Oktatási Módszertan: A 4 Színkód

A konverter felületén elhelyezett beépített útmutató a Zotero PDF olvasójához a nemzetközileg elismert 4 színkódos szemlézési rendszert ajánlja:
1. 🟡 **Sárga:** Kulcsfogalmak, definíciók (Ezekből születnek a `concept_*` kártyák).
2. 🔵 **Kék:** Módszertani lépések, kísérleti környezet, empirikus minták.
3. 🟢 **Zöld:** A szerző legfontosabb eredményei, konklúziói és bizonyított tézisei.
4. 🔴 **Piros:** Kritikai pontok, módszertani korlátok, ellenérvek és saját kutatói kételyek.

---

## 5. Telepítés és Terjesztés

1. **Önálló böngészős fájlként:**
   - A `public/zotero_ris_to_obsidian.html` közvetlenül megnyitható bármilyen asztali vagy mobil böngészőben (offline módban is).
2. **GitHub Pages integráció:**
   - A repozitóriumban a `docs/` vagy `public/` mappa közzétételével az egyetemi kurzus hallgatói azonnal elérhetik weben keresztül.
3. **Zéró szerverkövetelmény:**
   - Semmilyen backend, Node.js vagy Python környezet nem szükséges a futtatásához, minden művelet a böngésző memóriájában, a felhasználó gépén történik.
