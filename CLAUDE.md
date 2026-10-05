# Kalkylator

Miniräknare som ser ut som iOS-räknaren, byggd som installerbar PWA. Har ett dolt läge som
visar ett förvalt resultat efter ett visst antal tryck på `=` (trolleritrick).

- Repo: `hansjnsson/kalkylator` (publikt), branch `main`.
- Publicerad på https://hansjnsson.github.io/kalkylator/ via GitHub Pages, direkt från
  `main` i repo-roten. **En push till `main` är en deploy.**

## Stack

Ren HTML/CSS/JavaScript utan ramverk, byggsteg eller `package.json`. Det ska förbli så.

| Fil | Innehåll |
|---|---|
| `index.html` | Hela appen: CSS, markup och skript i en fil. |
| `sw.js` | Service worker: cache-first för alla GET, faller tillbaka på `index.html` offline. |
| `manifest.webmanifest` | PWA-manifest (standalone, porträtt). |
| `*.png` | Appikoner. |

## Köra lokalt

Service workern kräver http, så öppna inte `index.html` direkt från disk:

```
npx serve .
```

Service workern cachar hårt. Syns inte en ändring: avregistrera den i DevTools
(Application → Service workers) eller testa i ett privat fönster.

## Ny version

Varje ändring som ska nå installerade appar kräver båda stegen, med samma nummer:

1. Höj `CACHE` i `sw.js` (`kalkylator-vN`).
2. Höj `APP_VERSION` i `index.html` (`'N · ÅÅÅÅ-MM-DD'`). Visas under Version i dolda läget.

Nya filer som ska fungera offline läggs också till i `ASSETS` i `sw.js`.

## Så fungerar appen

- Skriptet är en IIFE i ES5-stil (`var`, `function`, inga pilfunktioner). Håll den stilen i
  `index.html`. `sw.js` använder modern syntax.
- Storlekar anges i `cqw` mot `.phone` (container query), så layouten skalar med bredden.
- Uttrycket lagras i `tokens` (omväxlande tal och operator som strängar) plus pågående
  inmatning i `entry`. `evalT` räknar med vanlig prioritet för `×` och `÷`.
- Tal lagras internt med punkt och minus-bindestreck. Visningen (`fmtRaw`, `fmtResult`) byter
  till decimalkomma, `−` och hårt blanksteg som tusentalsavgränsare. Högst 9 siffror.
- Operatorerna är tecknen `+ − × ÷` (riktigt minustecken, inte bindestreck).

### Dolt läge

- Långtryck (650 ms) på räknarikonen uppe till höger öppnar inställningsbladet.
- Kort tryck på samma ikon öppnar en kulissmeny (`#menu`: Enkel, Avancerad,
  Matematikanteckningar, Konvertera) som efterliknar iOS-räknarens lägesmeny. Alla val stänger
  den utan att ändra något.
- Långtryck på klockan uppe till vänster nollställer `=`-räkningen och kvitterar med blink och
  vibration.
- AC nollställer också `=`-räkningen (`pressAc`), men utan blink eller vibration.
- Inställningarna ligger i `cfg` och sparas i `localStorage` under `calc_cfg`.
- `count` räknar tryck på `=`. När `count` når `cfg.n` visas `cfg.target` i stället för det
  riktiga resultatet. `armed` blir falsk efteråt om inte `cfg.repeat` är på.
- Anpassa sista talet (`cfg.adapt`): inför det avgörande trycket räknar `solve` ut vilket
  sista tal som ger målresultatet, och `forced` matar in dess siffror oavsett vilka
  sifferknappar som trycks.
- Bladet visar också `log`: de fem senast inmatade talen (`LOG_MAX`), det senaste överst.

Räknaren ska i övrigt bete sig och se ut som en vanlig räknare. Inget i det synliga
gränssnittet får avslöja det dolda läget.

## Test

Det finns inga automatiska tester. Testa för hand i webbläsaren, helst även på telefon som
installerad app, eftersom långtryck, vibration och safe-area bara märks där.
