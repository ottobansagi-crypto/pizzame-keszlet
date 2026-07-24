# PizzaMe – Készlet feltöltés

Nyers Wildom készlet-export (HU + SK) → V31-kompatibilis **Készlet feltöltés** Excel.
Két úton is előállítható, ugyanabból az egy repóból: böngészős generátorral azonnal,
vagy GitHub Actionsszel automatikusan.

> Ez a repó **publikus**, mert a böngészős generátor GitHub Pages-en fut, ami
> ingyenes csomaggal csak publikus repóból szolgál ki oldalt. A `bemenet/`/`kimenet/`
> mappákban lévő fájlok emiatt nyilvánosan olvashatók a repóban és a Pages linken
> keresztül is – ugyanaz a helyzet, mint a `pm-kalkuator`/`uzlet-riport` repóknál.

## Mit csinál

A nyers export `Sort Index` oszlopa megegyezik a V31 termékindexszel (1–597). A script
ezen a kulcson párosít, a helyes V31 üzlet-sorrendbe rendezi az adatot, kezeli az
SK-italok „(sk)” duplikációját, megjelöli a negatív értékeket, és külön **Napló**
lapon jelzi a kimaradt (index nélküli) sorokat és a negatívokat. A régi kézi folyamat
2–20. lépését váltja ki.

## Gyors út: vizuális generátor (nincs GitHub feltöltés)

**https://ottobansagi-crypto.github.io/pizzame-keszlet/**

Nyisd meg ezt a linket, húzd be a HU és SK exportot, kattints **Készlet feltöltés
generálása** – az eredmény azonnal letöltődik a böngészőből. A két nyers fájl
soha nem hagyja el a gépedet: a feldolgozás teljes egészében a böngészőben fut
(JavaScript), nem kerül fel sehova. Ez ugyanazt a transzformációt végzi, mint a
lenti GitHub Actions folyamat, csak fájlfeltöltés és várakozás nélkül.

A generátor (`index.html`) és a termékszerkesztő (`szerkeszto.html`) ugyanabban a
repóban van, mint a Python szkript és a `config.json` – nincs mit szinkronizálni,
egy helyen kell csak módosítani.

### Termékek szerkesztése böngészőből

**https://ottobansagi-crypto.github.io/pizzame-keszlet/szerkeszto.html**

Ha csak a termék-mastert kell bővíteni/módosítani (új termék, új index, névváltozás,
egy termék V31/Készlet HU/Készlet SK jelölése), nem kell kézzel piszkálni a
`config.json`-t: ez az oldal betölti a jelenlegi terméklistát, engedi
szerkeszteni/hozzáadni/törölni sorokat, majd megerősítés után commitol ide a GitHub
API-n keresztül. Ehhez egy `repo` jogosultságú GitHub Personal Access Token kell,
amit az oldal csak a böngésződben tárol (localStorage).

Bolt/üzlet-struktúra változás (új üzlet, SK-ital duplikátum pár, oszloprend) ennél
az eszköznél nincs támogatva – az ilyen ritkább, strukturális változásokat egyeztesd
a chatben, ahogy eddig.

## Használat (heti futtatás – alternatíva, ha a fenti linket nem használod)

1. A GitHub weben nyisd meg a repót → `bemenet/` mappa → **Add file → Upload files**.
2. Húzd be a friss **HU** és **SK** exportot (a régieket előbb töröld a mappából).
   Commit.
3. A feltöltés automatikusan elindítja az **Actions** futást (kb. 1 perc).
4. Az eredmény két helyen elérhető:
   - a `kimenet/` mappában (`keszlet_feltoltes_ÉÉÉÉ-HH-NN.xlsx`) – innen letölthető;
   - az Actions futás oldalán **artifactként** is (`keszlet-feltoltes`).
5. Nyisd meg, jelöld ki a **C4:AZ600** blokkot (A és B oszlop nélkül – azok az éles
   tábládban képletek), és illeszd be **csak értékként** az éles „Készlet feltöltés”
   lap **C4** cellájába. Innen a V31 A–E táblák szinkronizálnak.

Kézi indítás fájlcsere nélkül: **Actions** fül → *Készlet feltöltés generálás* →
**Run workflow**.

## Napló lap – amit nézni kell

- **Negatív értékek**: hó végi leltárnál javítandók, mielőtt a V31-be kerülnek.
- **Index nélküli sorok**: nincs Sort Indexük, ezért kimaradtak (nagyrészt nem-leltári
  tétel; ha valamelyik valódi alapanyag, adj neki indexet a Wildomban).
- **ISMERETLEN INDEX**: ha a nyersben olyan index jön, ami nincs a `config.json`
  masterében → a config frissítése kell (lásd lent).

## config.json – mikor kell frissíteni

A `config.json` tartalmazza a V31 termék-mastert, az üzlet-térképeket és az
SK-ital párosítást. Csak akkor kell hozzányúlni, ha **strukturális** változás van:

- új / kikerülő termék (változik az 1–597 indexlista),
- új / bezáró / átrendezett üzlet,
- új SK-ital duplikátum pár.

Puszta névváltozás (ugyanaz az index) **nem** igényel configfrissítést – a párosítás
index alapján megy.

A configot az élő Google-táblából érdemes újragenerálni (a Sort Indexekkel garantáltan
szinkronban). Gyakorlatban: jelezd a chat-Claude-nak, hogy „változott X”, ő
újragenerálja a `config.json`-t, te pedig commitolod ide – ugyanúgy, ahogy az
`uzlet-riport` `data.json`-ját frissíted.

## Helyi futtatás (opcionális)

```bash
pip install -r requirements.txt
python keszlet_transform.py                 # bemenet/ -> kimenet/
python keszlet_transform.py HU.xlsx SK.xlsx OUT.xlsx   # explicit fájlok
```

## GitHub Pages bekapcsolása (egyszeri lépés)

Settings → Pages → Source: **Deploy from a branch**, Branch: `master` / `(root)`.
Utána minden push automatikusan frissíti a fenti linkeket, nincs vele több teendő.

## Fájlok

- `keszlet_transform.py` – a transzformáció (mappa- és kézi mód).
- `index.html` – böngészős generátor (GitHub Pages).
- `szerkeszto.html` – böngészős termékszerkesztő (GitHub Pages).
- `config.json` – V31 master, üzlet-térképek, SK-ital párosítás.
- `.github/workflows/keszlet.yml` – az Actions workflow.
- `bemenet/`, `kimenet/` – be- és kimeneti mappák.
