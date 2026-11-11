# Rechts-Check pizzeriapinocchiobeckum.de (10 Punkte, nur Technik-Check)

- **Stand:** 10.11.2026, Repo `C:/Users/Abed Falah/dev/gastronomie/PinoccioBeckum`, Zweig `main` = `b0d5fea`
  (= `origin/main`). `index.html`, `impressum.html`, `datenschutz.html`, `account.html` und `dashboard.html` sind
  byte-gleich mit der Live-Seite (per curl abgeglichen). `config.js` unterscheidet sich nur in den Zeilenenden.
- **Backend** `api.pizzeriapinocchiobeckum.de` (Coolify): `server.js` ist identisch mit `main`, die mitgelieferten
  Dateien stammen aber vom älteren Stand `bf92f57` (vor dem 04.11.). Das wird unter 1 und 6 wichtig.
- **Geprüft:** alle Dateien im Repo (ohne `node_modules`), die Git-Historie (Schlüsselmuster nur gezählt, nichts
  ausgegeben), die Live-Seite im frischen Browser **ohne Einwilligung** (Netzwerk, Cookies, Storage), dazu
  Tastaturbedienung und die Kasse bis vor den Bestellbutton. **Es wurde nichts bestellt und nichts abgeschickt.**
  Die Kontraste habe ich nachgerechnet.
- **Keine Rechtsberatung.** Das ist ein technischer Abgleich zwischen Code, Live-Seite und Rechtstexten. Was rechtlich
  zu bewerten ist, steht unter „Für den Anwalt“. Die Seite ist dadurch **nicht** „rechtssicher“.
- **Es wurde nichts geändert:** kein Code, kein Zweig, kein Commit. Diese Datei ist ungetrackt und soll nicht
  committet werden.

## Übersicht

| # | Punkt | Status | Fund (Datei:Zeile) | Vorschlag |
|---|---|---|---|---|
| 1 | Impressum | **Problem (Platzhalter)** | Inhaber nur „Nissan“ (`impressum.html:29`), USt-IdNr `DE123456789` ist ein Beispielwert (`:33`). Dieselben Werte stehen in `index.html:3859` (Modal), im AGB-Kopf `index.html:3869`, in `datenschutz.html:58` und in `stammdaten.json`. Anschrift, Telefon und E-Mail sind echt (`:29-31`). Erreichbar per Footer-Knopf (`index.html:3822`): 1 Klick, aber am Ende einer etwa 11.800 px langen Seite. Von `account.html` sind es 2 Klicks. Unter `api.pizzeriapinocchiobeckum.de/` liegt eine alte Kopie der Seite mit `Nissan@beispiel.de`. | Abed holt den vollen Namen und die USt-IdNr (oder: es gibt keine, dann Zeile raus). Danach an allen 7 Stellen ersetzen (siehe Details 1). Alte Backend-Kopie abstellen (siehe 6). |
| 2 | Datenschutzerklärung | **Problem (groß)** | Genannt ist nur Stripe (`datenschutz.html:113-116`). **Es fehlen:** GitHub Pages (Hosting der Seite), Hetzner/Coolify (Backend), MongoDB Atlas, Resend, Kundenkonto, Bestellformular mit Speicherdauer, Stripe.js-Cookies und FlueVate als Auftragsverarbeiter. Als „Hoster“ steht „Abed Rachman Falah/ Fluevate, Zur Goldbrede 30“ (`:50`). Es gibt Bausteine ohne Gegenstück: Nutzeranalyse (`:39`), Transportunternehmen/Versandstatus (`:109`), Lastschrift/Kontonummer (`:86-87`). Dieselbe Fassung steht noch in `index.html:3864` und `account.html:816`. | Die DSE bei eRecht24 mit der Dienste-Liste unter 2 neu erzeugen (wie bei fluevate.de) und an allen 3 Stellen einsetzen. Vorher vom Anwalt prüfen lassen. |
| 3 | AGB | **Problem** | Die AGB sind vorhanden (Modal `index.html:3867-3870`, eRecht24 1-11 plus Ziffer 12). 12.1 (Servicegebühr 0,99 €) und 12.2 (Lieferzonen) passen zu `server.js:183,490-499` und `index.html:2269-2275`. **12.3 und 4.2 sagen aber „ausschließlich Barzahlung, Online-Zahlung wird nicht angeboten“.** Stripe ist seit 04.11. live (`index.html:2153,2225`). Der Commit `4e0d2ff` hat nur den Schalter umgelegt. Der Link heißt „AGB & Widerrufsbelehrung“, eine Belehrung gibt es aber nicht, nur den Ausschluss in 12.4. | 4.2 und 12.3 an die echten Zahlungsarten anpassen. Ob es einen Widerrufsbutton und eine Belehrung braucht, hängt an der Getränke-Frage (Anwalt 1). |
| 4 | Google Fonts / CDNs | **Problem (klein)** | Keine Google Fonts, kein unpkg/jsdelivr/cdnjs. Font Awesome liegt lokal unter `vendor/` (mit Lizenzkopf), DM Sans und Playfair liegen lokal unter `fonts/`. **Aber:** `https://js.stripe.com/v3/` wird auf jeder Seite geladen (`index.html:12`), und der Code nutzt es gar nicht (Zahlung per Weiterleitung, `index.html:2526`). | Zeile 12 entfernen. Die Zahlung funktioniert weiter, weil Stripe Checkout auf Stripes eigener Seite läuft. |
| 5 | Cookie-Banner | **fehlt, und es wird vor der Einwilligung geladen** | Es gibt keinen Banner. Beim ersten Aufruf ohne jede Aktion setzt Stripe.js **3 Cookies**: `__stripe_mid` (365 Tage) und `__stripe_sid` (30 Min.) auf `.pizzeriapinocchiobeckum.de`, dazu `m` auf `m.stripe.com` (400 Tage). Außerdem lädt es das Betrugserkennungs-Iframe `m-outer` (live geprüft). | Mit der entfernten Zeile 12 fallen alle Cookies weg. Dann braucht die Seite nach meinem Verständnis keinen Banner, das klärt der Anwalt (Frage 6). |
| 6 | Datenbank / Keys / Zugriffsschutz | **Keys OK, Integrität: Problem** | Keine Schlüssel im Frontend oder in der Historie, `.env` wurde nie committet. Admin- und Kunden-Endpunkte antworten live mit 401. **Aber:** `POST /api/orders` übernimmt den ganzen Request (`server.js:619-625`). Ein Gast kann damit `paymentStatus:"paid"` oder `source:"pos"` mitschicken (dann gilt die Bestellung als bezahlt bzw. bestätigt, mit Mails und Druck). Die Preise kommen ungeprüft aus dem Browser (`:614-617,664-684`), und so viel verlangt auch Stripe. Das Backend liefert seinen ganzen Ordner aus (`:124-131`), live mit 200: `/server.js`, `/node_modules/…` und die alte `index.html`. Es gibt keine Löschfristen, und die Kunden-E-Mail steht im Log (`:1320`). Das Repo ist öffentlich. | Bestell-API härten (nur erlaubte Felder, Preise serverseitig aus `Menue/menu.json`), `express.static` auf nichts oder nötige Dateien begrenzen, Löschfrist festlegen, Log ohne E-Mail. Eigener Zweig mit Opus, weil es das Live-Bestellsystem ist. |
| 7 | Barrierefreiheit | **Problem** | `lang="de"` und Alttexte überall vorhanden (13/13 Bilder). **Per Tastatur kann man nicht bestellen:** Größenwahl und „Hinzufügen“ in allen Produkt-Dialogen sind `<div onclick>` (`index.html:2899-2908,3080,3192,3302,3401,3490,3637`). 153 klickbare Elemente sind nicht fokussierbar. Kein Feld der Kasse hat ein Label (nur Platzhalter, `:2132-2173`). **Kontraste:** Die Auswahlfelder „Liefergebiet“ und „Zahlungsmethode“ stehen schwarz auf Dunkelgrün, **1,21:1**, also kaum lesbar (`:2143,2152`). Der Bestellbutton weiß auf Grün hat 2,76:1, Impressum und DSE haben grün auf hell 2,57:1, der Footer dort 1,99:1. Die Dialoge haben keine Dialog-Semantik. | An den Frontend-Bot geben: `div` durch `button` ersetzen, Labels ergänzen, Select-Schriftfarbe setzen, Kontraste der Rechtsseiten anheben, Dialoge mit `role="dialog"` und Fokus. Ob das BFSG gilt, klärt der Anwalt (Frage 10). |
| 8 | Bilder / Lizenzen | **OK** | Die 9 Kategorie-Banner stehen mit Quelle und ID in `img/menu/QUELLEN.md` (Pexels, Unsplash, Pixabay, kommerziell frei ohne Namensnennung). Weitere Fotos oder ein Logo-Bild gibt es nicht. Font Awesome Free hat den Lizenzhinweis in der CSS-Datei. 4 Banner fehlen absichtlich und erzeugen 404-Abrufe. | Lizenzseiten zu den IDs einmal als PDF/Screenshot sichern (Beweis, falls ein Bild bei der Plattform verschwindet). |
| 9 | Fake-Bewertungen | **OK** | Keine Bewertungen, Sterne, Zitate oder Siegel. Die Stempelkarte ist aus (`index.html:4893`). Die einzige Beliebtheits-Aussage ist „Viele bestellen das auch dazu:“ (`:2084`) über einer festen Liste (Cola, Pizzabrötchen). | Optional neutral formulieren („Passt dazu:“). |
| 10 | Checkout / Bestellung | **Problem** | Der Button heißt „✅ Kostenpflichtig bestellen“ (`index.html:2258`), das ist ok. **Aber:** Die Kasse zeigt **keine Artikel**, nur „Speisen 4,00 €“, Liefer- und Servicegebühr (`:2326`, Screenshot geprüft). Ein Hinweis „inkl. MwSt.“ steht weder an der Karte noch in der Kasse, nur in AGB 12.1. Die Zwischensumme „Gesamt“ im Warenkorb rechnet die Servicegebühr nicht mit (`:1693,1706`), anders als AGB 12.1 sagt. Es gibt keine Widerrufsbelehrung und kein Muster-Formular, nur den Ausschluss 12.4. Eine Eingangsbestätigung per Mail gibt es erst, wenn das Restaurant annimmt (`server.js:1117`). Die Datenschutz-Checkbox ist eine Pflicht-„Zustimmung“ (`index.html:2200`). | Artikelliste (Name, Größe, Extras, Menge, Preis) in die Kasse, „Alle Preise inkl. MwSt.“ an Karte und Kasse, Servicegebühr schon im Warenkorb zeigen. Widerruf, Bestätigungen und Checkbox klärt der Anwalt. |

---

## Details

### 1. Impressum (`impressum.html`, gleicher Text als Modal in `index.html:3857-3860`)

| Angabe | Stand | Zeile |
|---|---|---|
| Name des Inhabers | **„Nissan“**, nur ein Vorname und laut claude-wissen ein Beispielwert. Bei einem Einzelunternehmen gehört der volle Vor- und Nachname hinein | `impressum.html:29` |
| Geschäftsbezeichnung / Rechtsform | Pizzeria Pinocchio, Einzelunternehmen | `:29` |
| Anschrift | Stromberger Str. 135, 59269 Beckum | `:29` |
| Telefon | 02521 / 857 94 79 (nur Text, ohne `tel:`-Link) | `:31` |
| E-Mail | nissanhass2@gmail.com, seit 04.11. echt (Commit `6f4f039`) | `:31` |
| USt-IdNr | **DE123456789**, ein Beispielwert. Gibt es keine, muss die Zeile weg; eine erfundene Nummer ist schlechter als keine | `:33` |
| Aufsichtsbehörde | Stadt Beckum, Fachdienst Recht und Ordnung (ob nötig, siehe Anwalt 9) | `:35` |
| Verbraucherstreitbeilegung | Hinweis vorhanden, kein OS-Link (die EU-Plattform gibt es seit 20.07.2025 nicht mehr, so ist es richtig) | `:37` |
| Handelsregister | keins genannt (bei einem nicht eingetragenen Einzelunternehmen üblich; ist es ein e.K., fehlt es) | – |

- **Platzhalter an 7 Stellen** (6× „Inhaber: Nissan“, 3× DE123456789): `impressum.html:29,33`, `index.html:3859` (Impressum-Modal), `index.html:3869`
  (AGB-Kopf „Inhaber: Nissan“), `datenschutz.html:58` und der DSE-Modal-Text in `index.html:3864` und `account.html:816`
  („Inhaber: Nissan“), dazu `stammdaten.json` (`inhaber`, `ustid`, `steuernummer: FEHLT_STEUERNUMMER`).
- **Erreichbarkeit:**
  - Startseite: Footer-Knopf „Impressum“ (`index.html:3822`) öffnet das Modal, also 1 Klick. Der Footer steht aber
    am Ende einer sehr langen Speisekarte. Der Knopf ist ein `<button>` mit JavaScript, kein Link.
  - `account.html`: kein Impressum-Link. Über „zurück“ (`account.html:315`) und den Footer sind es 2 Klicks.
  - `impressum.html` und `datenschutz.html` verlinken sich gegenseitig, werden aber von keiner Seite aus verlinkt.
    Gefunden werden sie nur über Suchmaschinen oder die direkte URL.
  - Kasse: Dort gibt es kein Impressum, aber den AGB-Link mit den Anbieterdaten im Kopf.
- **Alte Kopie im Backend:** `https://api.pizzeriapinocchiobeckum.de/` liefert die ganze Bestellseite aus (Stand
  `bf92f57`) mit Impressum „Nissan@beispiel.de“ und abgeschalteter Online-Zahlung. Das kommt von `express.static`
  in `server.js:124` (siehe 6).

### 2. Datenschutzerklärung: Dienste im Code gegen DSE

Die DSE steht dreimal mit gleichem Inhalt: `datenschutz.html`, `index.html:3864` (Modal) und `account.html:816`
(Modal bei der Registrierung). Ich habe die Fassungen per Skript verglichen, sie sind inhaltsgleich.

| Dienst | Wo im Code / Betrieb | Zweck, Daten | In der DSE? |
|---|---|---|---|
| **GitHub Pages** (GitHub Inc., USA) | `CNAME`, `.github/workflows/deploy.yml`, live Header `Server: GitHub.com` | Hosting der Bestellseite, IP-Logs bei jedem Aufruf | **nein**. Als Hoster steht FlueVate (`datenschutz.html:50`) |
| **Hetzner** (Coolify `fluevate-vps`, Nürnberg) | `config.js:16` → `api.pizzeriapinocchiobeckum.de`, laut claude-wissen `fluevate-vps-coolify.md` | Backend, Bestell-API, Container-Logs | **nein** |
| **MongoDB Atlas** | `server.js:143,159-231`, DB `pinoccio_beckum` (live `/api/health`) | Bestellungen (Name, Telefon, E-Mail, Adresse, Artikel, Anmerkung), Kundenkonten (E-Mail, Passwort-Hash, Name, Telefon, Adressen) | **nein**. Region des Clusters unbekannt |
| **Resend** (USA) | `server.js:58-74,1272-1380`, Absender `bestellungen@pizzeriapinocchiobeckum.de` | Bestellbestätigung und Stornomail an den Gast, Bestellmail mit Name, Telefon und Adresse an das Restaurant, Berichte | **nein** |
| **Stripe** (Checkout, Connect) | `server.js:661-742`, `index.html:2537-2539` | Zahlung auf Stripes Seite. An Stripe gehen E-Mail, Artikel mit Anmerkungen und Betrag | **ja** (`datenschutz.html:113-116`) |
| **Stripe.js auf jeder Seite** | `index.html:12` | ungenutzt. Setzt Cookies und lädt das Betrugserkennungs-Iframe, siehe 5 | **nein** (nur ein allgemeiner Cookie-Absatz `:93-98`) |
| **FlueVate** (Abed) als Dienstleister | betreibt das System, bekommt Berichte (`OWNER_EMAIL`, `server.js:2102-2110,2263-2270`) und über Stripe Connect die Gebühr (`server.js:686-713`) | Auftragsverarbeitung, Zahlungsfluss | nur als „Hoster“ (`:50`) |
| **Kundenkonto** | `account.html`, `server.js:290-439`, Token 30 Tage in `localStorage` | Registrierung, Bestellhistorie | **nein** |
| **Bestellformular** | `index.html:2132-2173` → `/api/orders` bzw. Stripe | Name, Telefon, E-Mail, Adresse, Anmerkung | nur allgemein (`:105-107`), ohne Speicherdauer |
| `localStorage` / `sessionStorage` | `index.html:2590,2636` (Bestellstatus), `:4835` (Login-Token) | technisch nötig, erst nach Bestellung oder Login | nein (nach meinem Verständnis § 25 Abs. 2 TDDDG, siehe Anwalt) |
| Restaurant-Postfach | `RESTAURANT_EMAIL` (Wert nur in Coolify). Inhaber-Adresse ist Gmail | Bestellmails, Berichte | nein. Ist es Gmail, kommt Google als Empfänger dazu |
| **PrintNode** (Bondruck-Cloud) | `server.js:1388-1392`, `printnode-helper.js:15` | nur aktiv, wenn `PRINTNODE_API_KEY` gesetzt ist. In Nordkirchen war es leer, hier unbekannt | nein. Nötig nur, wenn aktiv |
| **PayPal** | `server.js:24-55,798-923` | im Frontend nicht angeboten (`index.html:2221-2225`) | nein, nicht nötig, solange es aus ist |
| Sunmi-Kasse (FlueVate-Kasse) | holt Bestellungen über `/api/admin/orders/pending` (`server.js:1055`) | Bondruck im Laden | nicht nötig als eigener Dienst (Gerät des Restaurants) |
| Server-Logs | Coolify-Container: Kunden-E-Mail (`server.js:1320`), IP bei falschem Admin-Login (`:1043`) | Betrieb | nur allgemein („Server-Log-Dateien“, `:99-103`) |
| Schriften, Font Awesome | lokal (`fonts/`, `vendor/`) | – | nicht nötig (kein Dritter) |
| Analytics, Pixel, Maps, Videos, WhatsApp, Newsletter, Consent-Tool | – | – | **gibt es im Code nicht** (`WHATSAPP_NUMBER` leer, `/api/config` live) |

**Texte ohne Gegenstück auf der Seite** (eRecht24-Bausteine):
- „Andere Daten können zur Analyse Ihres Nutzerverhaltens verwendet werden“ (`datenschutz.html:39`). Es gibt keine Analyse.
- „Transportunternehmen … Versandstatus per E-Mail“ (`:108-109`). Die Pizzeria liefert selbst.
- „Kontonummer bei Einzugsermächtigung … Lastschriftverfahren“ (`:86-87`). Lastschrift gibt es nicht, die Karte läuft über Stripe.

**Versprechen, die belegt sein müssen:** Löschung nach Zweckwegfall (`:62,107`). Im Code wird nichts gelöscht
(kein TTL, kein Lösch-Endpunkt für Bestellungen oder Konten). Auftragsverarbeitung nur mit AV-Vertrag (`:66`).

### 3. AGB (`index.html:3867-3870`, Modal „AGB & Widerrufsbelehrung“)

- **Aufbau:** eRecht24 „Online-Shop“ 1-11 plus **Ziffer 12 „Ergänzende Bedingungen der Pizzeria Pinocchio“**.
- **12.1 Servicegebühr 0,99 €**, je Bestellung, zählt nicht zum Mindestbestellwert. Das passt zu `server.js:183,492`,
  `index.html:2275` (`SERVICE_FEE`) und `:2324,2339` (wird bei Lieferung und Abholung berechnet). Der MBW-Vergleich
  nutzt nur den Warenwert (`:1700,2451`), auch das passt. **Abweichung:** „wird im Warenkorb gesondert ausgewiesen“.
  Im Warenkorb steht sie nicht, erst in der Kasse (`:1684-1707` gegen `:2309-2348`).
- **12.2 Lieferzonen:** Beckum 20,00 €/1,50 €, Neubeckum, Roland und Vellern je 25,00 €/3,00 €. Das stimmt mit
  `server.js:493-498`, `index.html:2144-2147,2269-2274` und `stammdaten.json` überein.
- **12.3 Zahlungsarten: „ausschließlich Barzahlung bei Übergabe“**, ebenso **4.2: „Eine Zahlung per Karte,
  Online-Zahlungsdienst oder auf andere Weise wird nicht angeboten.“** Das ist **falsch seit 04.11.**: Die Kasse
  bietet „Online bezahlen (Stripe)“ sogar als erste Wahl an (`index.html:2153`), dazu Logos für Visa, Mastercard,
  AMEX, Apple Pay, Google Pay und „& mehr“ (`:2164-2171`). Mit Connect lässt der Server nur `card` zu
  (`server.js:709`). Ob Apple Pay und Google Pay auf der Stripe-Seite wirklich erscheinen, habe ich nicht geprüft,
  weil das eine echte Checkout-Sitzung angelegt hätte.
- **12.4 „Kein Widerrufsrecht … § 312g Abs. 2 Nr. 2 BGB, da alle Speisen frisch zubereitet werden“.** Auf der Karte
  stehen aber auch Getränke in Flaschen und Dosen (z. B. Coca-Cola 0,33 L, `index.html:2090`). Siehe Anwalt 1.
- **Widerrufsbelehrung / Widerrufsbutton:** Link und Checkbox heißen „AGB & Widerrufsbelehrung“
  (`index.html:2186,3830`). Eine Belehrung oder ein Muster-Widerrufsformular gibt es nicht, nur den Ausschluss.
  Einen Widerrufsbutton (EU-Richtlinie 2023/2673, seit 19.06.2026) gibt es auch nicht. Nach meinem Verständnis
  braucht es ihn nur, wenn überhaupt ein Widerrufsrecht besteht.
- **2.2 Vertragsschluss:** Der Vertrag kommt u. a. zustande, wenn der Anbieter „zur Zahlung auffordert (z. B.
  Kreditkartenzahlung im Bestellprozess)“. Bei Stripe zahlt der Gast also, bevor das Restaurant annimmt
  (`server.js:756` „wartet auf Admin-Bestätigung“). Lehnt das Restaurant ab, wird erstattet (`server.js:1145-1171`).
  Siehe Anwalt 2.
- **2.3** verspricht, dass „alle Eingaben vor der verbindlichen Abgabe … in einem Bestätigungsfenster angezeigt“
  werden. Die Artikel fehlen dort (siehe 10).
- **2.4** Vertragstext per E-Mail: Die Bestätigungsmail (`server.js:1272-1322`) enthält Artikel, Preise und
  Zahlart, aber nicht die AGB.

### 4. Google Fonts und externe Schriften/CDNs

- Suche über alle HTML-, CSS-, JS- und JSON-Dateien (ohne `node_modules`) nach `fonts.googleapis`, `fonts.gstatic`,
  `unpkg`, `jsdelivr`, `cdnjs`, `typekit` und `use.fontawesome`: **0 Treffer**.
- Font Awesome 6 Free liegt lokal (`vendor/fontawesome/`, eingebunden `index.html:11`), der Lizenzkopf mit
  `fontawesome.com/license/free` steht in der CSS. DM Sans und Playfair Display liegen lokal (`fonts/*.woff2`) und
  werden nur in `account.html:7` und `dashboard.html:11` eingebunden.
- `index.html` nennt „Playfair Display“ und „DM Sans“ (`:943,1029`), lädt die Schriften aber nicht und fällt auf
  Systemschriften zurück. Das ist nur Optik, kein Rechtspunkt.
- **Einziges externes Skript:** `https://js.stripe.com/v3/` (`index.html:12`). Es gibt keinen `Stripe(…)`-Aufruf im
  Code. Die Zahlung läuft über `window.location.href = data.url` zu Stripe Checkout (`:2526`).
- **Live beim ersten Aufruf geladen:** `pizzeriapinocchiobeckum.de` (CSS, `config.js`, Schrift, Favicon),
  `api.pizzeriapinocchiobeckum.de` (`/api/status`, `/api/availability`) und `js.stripe.com` (Skript plus Iframe
  `m-outer-….html`).

### 5. Cookie-Banner (live geprüft, frischer Browser, keine Aktion)

- **Kein Banner, kein Consent-Tool** im Code.
- **Cookies direkt nach dem Laden:**

| Cookie | Domain | Laufzeit | Herkunft |
|---|---|---|---|
| `__stripe_mid` | `.pizzeriapinocchiobeckum.de` | 365 Tage | Stripe.js |
| `__stripe_sid` | `.pizzeriapinocchiobeckum.de` | 30 Minuten | Stripe.js |
| `m` | `m.stripe.com` (Drittanbieter) | 400 Tage | Stripe-Betrugserkennung (`m-outer`-Iframe) |

- `localStorage`/`sessionStorage` sind nach dem Laden leer. Sie werden erst nach einer Bestellung (Status der
  wartenden Bestellung) oder einem Login (Token) beschrieben.
- **Bewertung technisch:** Die Cookies entstehen nur durch die ungenutzte Zeile `index.html:12`. Ohne sie setzt
  die Seite vor der Bestellung nichts. Stripe setzt auf seiner eigenen Checkout-Seite weiter Cookies, das ist
  dann Stripes Seite.

### 6. Datenbank, Speicher, Keys, Endpunkte

- **Keys:** Im Frontend stehen keine (auch kein Stripe-Publishable-Key). `.env` steht in `.gitignore` und wurde laut
  Historie nie committet. Muster wie `sk_live_`, `whsec_`, `re_…`, Mongo-URI mit Passwort, `AIza…` oder
  `ADMIN_PASSWORD=` kommen in der ganzen Historie 0 Mal vor (nur gezählt). `.env` liefert live 404 (Frontend und
  Backend). Die Schlüssel liegen in Coolify (`render.yaml` nennt sie nur mit `sync: false`).
- **Repo öffentlich:** `github.com/falahabed007/pizzeria-pinocchio-beckum` antwortet ohne Login mit 200. GitHub
  Pages veröffentlicht zusätzlich das ganze Repo-Verzeichnis (`deploy.yml` `path: '.'`). Live erreichbar sind
  `/server.js`, `/stammdaten.json`, `/render.yaml`, `/package.json`, `/Menue/menu.json` und `/.gitignore`.
  Geheimnisse sind dort nicht drin, aber FlueVates Gebührenlogik (`server.js:686-690`) und die Berichtslogik sind
  öffentlich.
- **Zugriffsschutz, live geprüft:** `/api/admin/orders` und `/api/admin/customers` → 401, `/api/account/orders` → 401.
  `/api/orders/status/<token>` gibt nur Status und Nummer zurück, keinen Namen und keine Adresse (`server.js:639-658`).
  Der Admin-Login ist gebremst (`:1005-1052`), Passwörter werden mit bcrypt gespeichert (`:302`).
- **Lücken:**
  1. **Bestell-API übernimmt alles vom Browser** (`server.js:619-625`, `new Order({...req.body, …})`). Wer die API
     direkt aufruft, kann `paymentStatus:"paid"` und `payment:"stripe"` mitschicken. Dashboard, Bon und Mail zeigen
     dann „BEZAHLT“, ohne dass Geld floss. Mit `source:"pos"` wird die Bestellung sofort bestätigt, Mails gehen raus
     und der Druck startet (`:627-631`).
  2. **Preise aus dem Browser:** Artikelpreise, Zwischensumme, Liefer- und Servicegebühr kommen vom Gast
     (`:614-617`, Stripe `:664-684`). Mit einem veränderten Warenkorb zahlt man online weniger. Mindestbestellwert und
     Liefergebiet prüft der Server nicht.
  3. **Backend liefert seinen Ordner aus** (`server.js:124-131`). Live 200 auf `api.pizzeriapinocchiobeckum.de`:
     `/server.js`, `/stammdaten.json`, `/package.json`, `/render.yaml`, `/Menue/menu.json`,
     `/node_modules/express/package.json` und die alte `index.html` (siehe 1). `.env` ist durch Express' Standard
     (Dotfiles ignorieren) geschützt.
  4. **Keine Löschfrist:** Bestellungen und Kundenkonten bleiben unbegrenzt in Atlas. Einen Lösch-Endpunkt für
     Kunden gibt es nicht.
  5. **Logs:** Die Kunden-E-Mail steht im Containerlog (`server.js:1320`).
  6. **Mails ohne Escaping:** Name und Anmerkung des Gasts gehen ungeprüft in HTML-Mails (`server.js:1280,1294,1315`).
  7. Kein Limit auf Kunden-Login/Registrierung (`:290-334`), Mindestlänge des Passworts 6 Zeichen.
     `/api/create-stripe-checkout` legt ohne Limit Bestellungen und Stripe-Sitzungen an.
  8. Optional, kein Rechtspunkt: Das Backend sendet `X-Powered-By: Express`, kein HSTS und keine CSP.
     `/api/health` nennt den Datenbanknamen.

### 7. Barrierefreiheit

- **OK:** `lang="de"` auf allen Seiten (jeweils Zeile 2). Alle 13 `<img>` haben `alt`. Die Produkt-„+“-Knöpfe und
  die Kategorie-Links sind echte `<button>`/`<a>` und per Tab erreichbar, der Fokusrahmen bleibt sichtbar.
  Der Hauptkontrast ist gut (Text 15,5:1, leiser Text und Preise 6,3:1, Hero 7,0:1).
- **Tastatur:**
  - Im Pizza-Dialog sind die Größen (`index.html:2899-2901`) und „Hinzufügen“ (`:2908`) `<div onclick>`. Dasselbe
    gilt für Pasta (`:3080`), Salat (`:3192`), die drei kleinen Dialoge (`:3302,3401,3490`) und die Familienpizza
    (`:3637`). **Ohne Maus kommt nichts in den Warenkorb.** Insgesamt sind 153 `onclick`-Elemente `div`/`span` ohne
    `tabindex`.
  - Das unsichtbare Suchfeld (`:822`) und die Links des geschlossenen Handymenüs stehen in der Tab-Reihenfolge.
  - Dialoge (Produkt, Upsell, Kasse, Rechtstexte) haben kein `role="dialog"`/`aria-modal` und keine
    Fokusführung. Escape schließt nur die Rechtstexte (`:3935-3937`) und die Suche (`:4540`). Im Quelltext steht kein einziges `aria-`.
  - Die „+“-Knöpfe heißen für Screenreader nur „+“, ohne Produktnamen. Es gibt keinen Skip-Link.
- **Formular:** 18 Felder ohne `<label>`, nur mit Platzhalter (Kasse `:2132-2173`, Suche, Anmerkungen in den
  Dialogen). Fehler kommen als `alert()`.
- **Kontraste (WCAG AA verlangt 4,5:1 für normalen Text):**

| Stelle | Verhältnis | Zeile |
|---|---|---|
| Auswahlfelder „Liefergebiet“ und „Zahlungsmethode“: Schrift schwarz (Browser-Standard) auf `#181b16` | **1,21:1** (kaum lesbar, im Screenshot bestätigt) | `index.html:2143,2152` (ebenso `cart-stadt` `:895`) |
| Bestellbutton weiß auf `#72ab3d`, 15 px fett | **2,76:1** | `index.html:2255-2256` |
| Impressum/DSE: Überschriften und Links `#72ab3d` auf `#f4f7fb` | **2,57:1** | `impressum.html:13-19`, `datenschutz.html:13-19` |
| Impressum/DSE: Footer-Links (75 % Deckkraft) | **1,99:1** | `impressum.html:21`, `datenschutz.html:21` |
| Impressum/DSE: „Zurück zur Speisekarte“ weiß auf Grün | **2,76:1** | `:10-11` |
| Allergen-Hochzahlen, 9,3 px, 85 % Deckkraft | 4,88:1 (ok, aber sehr klein) | per JS |
| MBW-Warnung rot auf dunkel, 12 px | 4,54:1 (knapp ok) | `index.html:2335` |
| Fließtext, leiser Text, Preise, Hero | 6,3 bis 15,5:1 (ok) | – |

### 8. Bilder

| Datei | Quelle laut `img/menu/QUELLEN.md` | Lizenz nachweisbar? |
|---|---|---|
| `pizza.jpg`, `nudeln.jpg`, `gnocchi.jpg`, `pizzabroetchen.jpg`, `getraenke.jpg` | Pexels (IDs 10880324, 36430297, 15580891, 27126839, 4113660) | ja, Pexels-Lizenz |
| `penne.jpg` | Pixabay 10238319 | ja, Pixabay-Lizenz |
| `tortellini.jpg`, `salat.jpg`, `spezialangebote.jpg` | Unsplash (IDs in QUELLEN.md) | ja, Unsplash-Lizenz |
| `vendor/fontawesome/*` | Font Awesome Free (Icons CC BY 4.0, Schrift OFL, Code MIT) | ja, Hinweis in der CSS |
| `fonts/*.woff2` | DM Sans, Playfair Display (OFL) | ja |

- Die Herkunftskette ist dokumentiert (Nordkirchen, Amoura, Italia mit Commit). Nordkirchens Bilder mit unbelegter
  Herkunft wurden bewusst **nicht** übernommen.
- `getraenke.jpg` zeigt die Marke Coca-Cola. Das passt, weil die Pizzeria Cola verkauft. QUELLEN.md vermerkt es.
- Banner für `haehnchen-curry`, `desserts`, `mittagsangebote` und `abholer-angebote` fehlen absichtlich. Die Seite
  fragt sie trotzdem ab (live 404, `index.html:4325`). Das ist kein Rechtspunkt.

### 9. Bewertungen, Werbeaussagen

- Suche nach ★, Sterne, Bewertung, Review, Rating, Testimonial, Kundenstimme, Bestseller, beliebt, „seit 19xx“,
  „Nr. 1“, Lieferando, Google: **0 Treffer** (außer dem ⭐-Emoji am Menüpunkt „Spezial“).
- Die Stempelkarte („3 Bestellungen = 1 Gratis-Margherita“) steht noch im Code, ist aber aus
  (`STEMPELKARTE_AKTIV = false`, `index.html:4893`). Live erscheint kein Werbe-Popup, und der Konto-Knopf führt
  direkt zu `account.html`.
- „Viele bestellen das auch dazu:“ (`index.html:2084`) steht über einer festen Liste, nicht aus Bestelldaten.

### 10. Checkout / Bestellung (live bis vor den Button geprüft, nichts abgeschickt)

- **Ablauf:** „+“ → Produkt-Dialog → Warenkorb → Upsell „Dazu noch etwas?“ → Kasse (Name, Telefon, E-Mail, Adresse,
  Liefergebiet, Zahlart, Anmerkung) → 2 Pflicht-Checkboxen → „✅ Kostenpflichtig bestellen“. Bei Stripe folgt
  die Weiterleitung zu Stripe Checkout, bei Bar geht die Bestellung direkt ans Backend.
- **Button-Lösung (§ 312j Abs. 3 BGB):** „Kostenpflichtig bestellen“ (`index.html:2258,2529,2533`), ok. Vor dem
  Anhaken steht „🔒 Bitte AGB & Datenschutz bestätigen“.
- **Zusammenfassung direkt vor dem Button** (`index.html:2309-2348`): Es stehen nur „Speisen 4,00 €“,
  „Liefergebühr (Beckum) 1,50 €“, „Servicegebühr 0,99 €“ und „Gesamt 6,49 €“ da. **Keine Artikel, keine Größen,
  keine Extras.** Das widerspricht AGB 2.3 und nach meinem Verständnis § 312j Abs. 2 BGB (wesentliche Eigenschaften
  unmittelbar vor der Bestellung).
- **Preisangaben:** Ein Hinweis „inkl. MwSt.“ fehlt an der Speisekarte, im Warenkorb und in der Kasse. Er steht nur
  in AGB 12.1 („7 % auf Speisen, 19 % auf Getränke“). Nach meinem Stand verlangt § 6 PAngV im Fernabsatz diesen
  Hinweis und den Hinweis auf Liefer- und sonstige Kosten.
- **Servicegebühr:** Sie fällt in der Kasse an (`:2328,2341`) und wird per Checkbox bestätigt („gem. § 312a BGB“,
  `:2188`). Im Warenkorb heißt die Summe aber schon „Gesamt“, ohne die 0,99 € (`:1696,1706`).
- **Widerruf:** keine Widerrufsbelehrung, kein Muster-Widerrufsformular, kein Widerrufsbutton, nur der Ausschluss
  12.4 (siehe 3).
- **Bestätigungen:** Nach dem Absenden zeigt die Seite einen Wartebildschirm mit Status. Eine E-Mail geht erst raus,
  wenn das Restaurant annimmt (`server.js:1110-1123`). Die Stornomail enthält den Erstattungshinweis
  (`:1357-1380`). Die E-Mail-Adresse ist Pflicht (`index.html:2437`), im Feld steht aber kein Sternchen.
- **Datenschutz-Checkbox** (Pflicht): „… stimme der Datenverarbeitung zur Bestellabwicklung zu“ (`index.html:2200`).
  Die DSE stützt die Bestellung auf lit. b (`datenschutz.html:106`), eine Einwilligung ist dafür nicht nötig.
- **Zahlungsempfänger:** Stripe Connect als „Destination Charge“ ohne `on_behalf_of` (`server.js:707-713`). Die
  Zahlung läuft über FlueVates Plattformkonto, das 0,99 € plus die Stripe-Gebühr einbehält (`:686-690`), den Rest
  bekommt das Konto der Pizzeria. Welcher Name dem Gast auf der Stripe-Seite und im Kontoauszug erscheint, habe ich
  nicht geprüft (dafür müsste eine echte Sitzung angelegt werden). Laut Stripe-Doku ist es bei diesem Modell das
  Plattformkonto.

### Nebenbei

- Die Funde zu Stripe.js (4/5), zur Bestell-API (6), zu `express.static` und zur Kasse ohne Artikel kommen aus
  der gemeinsamen Vorlage (Pinocchio Nordkirchen). **Vermutlich betrifft das alle Klone.** Geprüft habe ich das nicht.
- Seitentitel live: „🐴 Pizzeria Pinocchio – Bestellen“ (Emoji per JS gesetzt). Das ist nur Optik.

---

## Für den Anwalt

1. **Widerrufsrecht bei Getränken:** AGB 12.4 schließt den Widerruf über § 312g Abs. 2 Nr. 2 BGB aus, „da alle
   Speisen frisch zubereitet werden“. Gilt das auch für Getränke in verschlossenen Flaschen und Dosen? Wenn nicht,
   brauchen wir dann eine Widerrufsbelehrung, ein Muster-Widerrufsformular und seit 19.06.2026 den Widerrufsbutton?
2. **Vertragsschluss bei Online-Zahlung** (AGB 2.2): Mit Stripe zahlt der Gast, bevor das Restaurant annimmt, und das
   Restaurant kann danach noch ablehnen (automatische Erstattung). Passt 2.2 so, oder braucht es eine Regel für die
   Annahme durch das Restaurant?
3. **Servicegebühr 0,99 €** (Pflicht bei jeder Bestellung): Muss sie in den Gesamtpreis nach PAngV? Ist „Gesamt“ im
   Warenkorb ohne die Gebühr irreführend? Stimmt der Verweis „gem. § 312a BGB“, und reicht die Checkbox als
   ausdrückliche Vereinbarung?
4. **Pflicht-Checkbox Datenschutz** („stimme der Datenverarbeitung zur Bestellabwicklung zu“): ersetzen durch einen
   Hinweis auf die DSE, weil die Bestellung auf lit. b läuft?
5. **Eingangs- und Vertragsbestätigung** (§ 312i Abs. 1 Nr. 3, § 312f Abs. 2 BGB): Reichen der Wartebildschirm und die
   Mail nach Annahme? Müssen AGB und Anbieterdaten in die Bestätigungsmail?
6. **Banner:** Wenn Stripe.js von der Seite entfernt ist, setzt sie vor der Bestellung nichts mehr. Später gibt es nur
   `localStorage` für den Bestellstatus und das Login. Reicht dann der Hinweis in der DSE, ohne Banner?
7. **Rolle von FlueVate:** FlueVate betreibt das System, bekommt Berichte und kassiert über Stripe Connect (Zahlung
   über FlueVates Plattformkonto, ohne `on_behalf_of`). Ist FlueVate Auftragsverarbeiter oder (teilweise) eigener
   Verantwortlicher? Muss der Zahlungsweg in AGB und DSE stehen? Welche Verträge braucht es zwischen Pizzeria und
   FlueVate (AVV)?
8. **Drittland:** GitHub Pages (USA), Resend (USA), Stripe, MongoDB Atlas (Region noch offen). Welche Formulierung
   braucht die DSE (DPF/Standardvertragsklauseln)?
9. **Aufsichtsbehörde im Impressum:** Eine Gaststätte in NRW ist nur anzeigepflichtig. Muss „Stadt Beckum,
   Fachdienst Recht und Ordnung“ drinbleiben, oder ist die Angabe unnötig?
10. **Barrierefreiheitsstärkungsgesetz:** Gilt es für die Pizzeria (Ausnahme für Kleinstunternehmen bei
    Dienstleistungen?) oder für FlueVate als Anbieter der Bestellplattform? Davon hängt ab, wie dringend Punkt 7 ist
    (vor allem: Bestellen ohne Maus ist unmöglich).
11. **Impressum-Erreichbarkeit:** Reicht ein JavaScript-Modal am Ende einer langen Speisekarte und auf `account.html`
    der Weg über die Startseite (2 Klicks)?
12. **Link-Titel „AGB & Widerrufsbelehrung“**, obwohl es nur einen Ausschluss gibt: umbenennen (z. B. „AGB &
    Widerrufsrecht“)?
13. **Mehrwertsteuer-Hinweis:** Reicht „Alle Preise inkl. MwSt., zzgl. Liefer- und Servicegebühr“ einmal an der
    Karte und in der Kasse?

## Von Abed noch zu erledigen

1. **Echte Impressum-Daten vom Inhaber holen** (nur Abed hat sie):
   - vollständiger Vor- und Nachname des Inhabers (statt „Nissan“),
   - USt-IdNr. **oder** die Bestätigung, dass es keine gibt (dann fliegt die Zeile raus),
   - Rechtsform bestätigen (Einzelunternehmen? e.K. mit Handelsregister?),
   - Telefonnummer 02521 / 857 94 79 und E-Mail nissanhass2@gmail.com bestätigen,
   - Steuernummer nur, falls der Inhaber sie in Berichten haben will (`STEUERNUMMER`, kein Impressum-Pflichtfeld).
2. **Zahlungsarten festlegen:** bleibt die Online-Zahlung? Welche Arten sollen genannt werden (Karte, Apple Pay,
   Google Pay, bar)? Danach AGB 4.2 und 12.3 anpassen lassen. Einmal selbst testen, was die Stripe-Seite anbietet.
3. **In Coolify nachsehen** (ohne Werte weiterzugeben): Ist `PRINTNODE_API_KEY` oder `PAYPAL_CLIENT_ID` gesetzt?
   Welche Art Adresse ist `RESTAURANT_EMAIL` (Gmail?)? Das braucht die DSE.
4. **Atlas:** die Region des Clusters nennen. **Löschfrist** festlegen: Bestellungen (Aufbewahrung?) und
   Kundenkonten (z. B. nach X Monaten ohne Login).
5. **AVVs belegen:** Hetzner, MongoDB Atlas, Resend, Stripe und GitHub, dazu einen Vertrag Pizzeria ↔ FlueVate.
6. **Neue DSE bei eRecht24 erzeugen** mit der Dienste-Liste unter 2 (wie bei fluevate.de), danach vom Anwalt prüfen lassen.
7. **Repo öffentlich?** Sollen Gebührenlogik und Code öffentlich bleiben? Privat geht GitHub Pages nur mit
   bezahltem Plan, alternativ läuft das Frontend über Coolify.
8. **BFSG:** beim Inhaber Beschäftigtenzahl und Umsatz grob erfragen (Ausnahme für Kleinstunternehmen, siehe Anwalt 10).
9. Die Lizenzseiten der 9 Banner-Bilder als Beleg sichern (optional, kann ich übernehmen).

## Was ich nach Freigabe technisch ändern würde (je Punkt ein eigener Zweig)

Bei diesem Repo ist **jeder Push auf `main` ein Live-Deploy** (GitHub Pages sofort, das Backend über Coolify).

- **A, klein, Sonnet:** `index.html:12` (Stripe.js) entfernen. Danach setzt die Seite keine Cookies mehr.
- **B, Text, nach Abeds Daten:** die Platzhalter an allen 7 Stellen ersetzen, AGB 4.2 und 12.3
  (Zahlungsarten) anpassen, den Link-Titel ändern.
- **C, Kasse, Sonnet oder Opus:** die Artikelliste in die Kassen-Zusammenfassung, „Alle Preise inkl. MwSt.“ an
  Karte und Kasse, die Servicegebühr schon im Warenkorb. Die Checkbox-Texte erst nach Rückmeldung des Anwalts.
- **D, Live-Bestellsystem, Opus:** `POST /api/orders` nur mit erlaubten Feldern (kein `paymentStatus`/`source` vom
  Gast), Preise, Gebühren und Mindestbestellwert serverseitig aus `Menue/menu.json` und `/api/config` rechnen,
  `express.static` im Backend abschalten (die Seite liegt auf GitHub Pages), `deploy.yml` nur Frontend-Dateien
  veröffentlichen lassen, Log ohne E-Mail, HTML-Escaping in den Mails. Mit Tests. Gehört in die Vorlage, also
  ein Kandidat für alle Klone.
- **E, Frontend-Bot:** Barrierefreiheit: `div` → `button` in den Dialogen, Labels, Schriftfarbe der Auswahlfelder,
  Kontraste auf Impressum und DSE, Dialog-Semantik und Fokus, Namen für die „+“-Knöpfe.
- **F, nach Abeds Frist:** TTL-Index für Bestellungen und alte Konten.
- **G, nach Anwalt:** DSE einsetzen (eRecht24 neu) an den 3 Stellen.
