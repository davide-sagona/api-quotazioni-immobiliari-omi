# API Quotazioni Immobiliari OMI (con dati ufficiali) 💶🏡

Dai le coordinate di un punto qualsiasi d'Italia, oppure il nome di un comune e di una zona, e ricevi in JSON la sua **zona OMI** con i **prezzi di vendita e di affitto al metro quadro, dal 2004 a oggi**.
I valori sono quelli ufficiali delle <a href="https://www.agenziaentrate.gov.it/portale/schede/fabbricatiterreni/omi/banche-dati/quotazioni-immobiliari" target="_blank" rel="noopener noreferrer">quotazioni immobiliari dell'Agenzia delle Entrate</a>.

- 📍 Ricerca da coordinate GPS, oppure per nome o codice di comune e zona
- 📅 Quotazioni storiche di ogni anno dal 2004
- 🆓 Fino a 5.000 richieste al giorno gratis

Pagina ufficiale e chiave gratuita: [3eurotools.it/api-quotazioni-immobiliari](https://3eurotools.it/api-quotazioni-immobiliari)
Per altri strumenti gratuiti: [3eurotools.it](https://3eurotools.it)

> ⚠️ **La vecchia API senza chiave (`/api-quotazioni-immobiliari-omi/ricerca`) smette di rispondere il 15 dicembre 2026.** Le quotazioni sono le stesse: cambiano l'indirizzo, l'autenticazione e la forma della risposta. Vedi [Migrare dalla vecchia API](#migrare-dalla-vecchia-api).

***

## Indice

- [F.A.Q. generali](#faq-generali)
- [Avvio rapido](#avvio-rapido)
- [Autenticazione](#autenticazione)
- [Endpoint](#endpoint)
  - [`/quotazioni`](#quotazioni-prezzi-di-un-punto-di-una-zona-o-di-un-intero-comune)
  - [`/zone`](#zone-le-zone-omi-di-un-punto-o-di-un-comune)
  - [`/me`](#me-piano-e-consumi)
- [Tipologie di immobile](#tipologie-di-immobile)
- [Come leggere la risposta](#come-leggere-la-risposta)
- [Limiti](#limiti)
- [Errori](#errori)
- [Esempi di codice](#esempi-di-codice)
- [Migrare dalla vecchia API](#migrare-dalla-vecchia-api)
- [Use case e benefici](#use-case-e-benefici)

***

## F.A.Q. generali

#### Cosa sono le quotazioni immobiliari OMI?
Sono valori di riferimento pubblicati dall'Agenzia delle Entrate per stimare rapidamente il prezzo di vendita o di affitto di un immobile, in base alla tipologia, all'ubicazione (zona OMI) e alla superficie in metri quadrati.

#### Queste quotazioni sono affidabili?
Sì: sono il modo migliore per stimare un immobile avendo pochissime informazioni. Sono rilasciate dall'Agenzia delle Entrate e si basano sugli atti di compravendita e sui contratti di locazione.
<a href="https://www.agenziaentrate.gov.it/portale/schede/fabbricatiterreni/omi/banche-dati/quotazioni-immobiliari" target="_blank" rel="noopener noreferrer">Per saperne di più</a>

#### Da che anno sono disponibili?
Dal 2004 a oggi. Il valore di un anno è la media dei semestri pubblicati per quell'anno.

#### Serve una chiave?
Sì. La chiave è **gratuita**: ti registri sulla [pagina ufficiale](https://3eurotools.it/api-quotazioni-immobiliari) e la generi dalla dashboard.

#### Non esistono API ufficiali dell'Agenzia delle Entrate?
No. Oltre a non fornire API, il portale dell'Agenzia delle Entrate richiede di completare a mano un CAPTCHA a ogni ricerca.

#### Posso partire da un indirizzo?
Sì: converti l'indirizzo in coordinate con un servizio di geocoding (Google Maps, OpenStreetMap Nominatim e simili) e passa `lat` e `lon` all'API.

***

## Avvio rapido

1. **Crea la chiave** sulla [pagina ufficiale](https://3eurotools.it/api-quotazioni-immobiliari). La vedi una sola volta: salvala in una variabile d'ambiente, mai nel codice pubblico.

   ```bash
   export OMI_API_KEY="la_tua_chiave"
   ```

2. **Fai la prima richiesta**: abitazioni civili di 80 m² vicino al Colosseo.

   ```bash
   curl -H "X-API-Key: $OMI_API_KEY" \
     "https://3eurotools.it/api/v1/quotazioni?lat=41.8902&lon=12.4922&tipologia=abitazioni_civili&mq=80"
   ```

3. **Risposta:**

   ```json
   {
     "anno": 2026,
     "risolto": {
       "metodo": "coordinate",
       "comune": { "codice": "H501", "nome": "Roma", "provincia": "RM", "regione": "Lazio" },
       "zona": { "codice": "B3", "descrizione": "Celio (Via Claudia)", "fascia": "B" }
     },
     "zone": {
       "B3": {
         "descrizione": "Celio (Via Claudia)",
         "tipologie": {
           "abitazioni_civili": {
             "stato": "normale",
             "semestri": 1,
             "acquisto": { "min": 352000.0, "max": 512000.0, "medio": 432000.0 },
             "affitto":  { "min": 1280.0, "max": 1840.0, "medio": 1560.0 }
           }
         }
       }
     }
   }
   ```

   Un'abitazione civile di 80 m² al Celio vale in media **432.000 €** e si affitta in media a **1.560 € al mese**.

***

## Autenticazione

- **Indirizzo base:** `https://3eurotools.it/api/v1`
- Tutte le richieste sono **GET** e le risposte sono in **JSON**.
- La chiave va inviata in un header:
  - `X-API-Key: la_tua_chiave`, oppure
  - `Authorization: Bearer la_tua_chiave`
- La chiave **nell'URL non viene accettata**.

> 🔒 Chiama l'API dal tuo server, non dal browser dei visitatori: chiunque potrebbe leggere la chiave dal codice della pagina.

***

## Endpoint

### `/quotazioni`: prezzi di un punto, di una zona o di un intero comune

L'area si indica in **uno** di questi modi:

| Modo | Parametri | Esempio |
|---|---|---|
| Coordinate | `lat`, `lon` | `lat=41.8902&lon=12.4922` |
| Testo libero | `q` | `q=Roma, San Lorenzo` · `q=H501/C5` · `q=Roma/C5` · `q=Roma` |
| Comune e zona separati | `comune`, `zona` | `comune=H501&zona=C5` |

**Tutti i parametri:**

| Parametro | Descrizione |
|---|---|
| `lat`, `lon` | Coordinate di un punto (WGS84, le stesse di Google Maps o di un GPS). Non si combinano con gli altri modi. |
| `q` | Comune e zona insieme, separati da virgola o barra: `Roma, San Lorenzo`, `H501/C5`, `Roma/C5`. Con il solo comune restituisce tutte le sue zone. |
| `comune` | In alternativa a `q`: codice catastale (es. `H501`) o nome del comune (es. `Roma`). |
| `zona` | Codice o descrizione della zona OMI. Se manca, tutte le zone del comune. |
| `provincia` | Sigla (es. `LE`). Serve solo per i comuni con lo stesso nome, come Castro (BG) e Castro (LE). |
| `tipologia` | Una delle [tipologie OMI](#tipologie-di-immobile) (es. `abitazioni_civili`) oppure il codice numerico. Se manca, tutte le tipologie della zona. |
| `anno` | Dal 2004 a oggi. L'anno è la media dei semestri pubblicati. |
| `mq` | Superficie, da 1 a 100.000.000. I prezzi vengono moltiplicati per questo valore. Default `1`, cioè prezzo al metro quadro. |
| `operazione` | `acquisto` o `affitto`. Se manca, entrambe. |

**Come si trova la zona**

- **Da coordinate:** l'API trova la zona OMI che contiene il punto, usando i perimetri ufficiali delle zone pubblicati dall'Agenzia delle Entrate. Non serve sapere in che comune o zona ti trovi: te lo dice il blocco `risolto` della risposta. Se il punto cade in mare o fuori da ogni zona ricevi `404 zona_not_found`.
- **Per nome:** una zona si indica con il codice (`B1`), con il nome (`Testaccio`) o con la descrizione intera (`Testaccio (Piazza S.Maria Liberatrice)`). Maiuscole, accenti e apostrofi non contano: `Forlì`, `forli` e `FORLI`` sono la stessa cosa.
  - Se più zone corrispondono ricevi `409 ambiguous_zona` con l'elenco dei candidati.
  - Se nessuna corrisponde, `404 zona_not_found` suggerisce le più simili.

Sulla [pagina ufficiale](https://3eurotools.it/api-quotazioni-immobiliari) puoi vedere tutte le zone di un comune senza chiave, oppure scaricare il JSON con le zone di tutti i comuni (con, per ognuna, l'input più semplice che la individua).

### `/zone`: le zone OMI di un punto o di un comune

Accetta gli stessi parametri di area di `/quotazioni` e restituisce solo le zone, senza prezzi: descrizione, fascia e tipologia prevalente.

```bash
curl -H "X-API-Key: $OMI_API_KEY" "https://3eurotools.it/api/v1/zone?lat=41.8902&lon=12.4922"
```

Con le coordinate del Colosseo restituisce la zona `B3` di Roma, "Celio (Via Claudia)".

### `/me`: piano e consumi

Restituisce il piano, i limiti, le richieste usate oggi e le tue chiavi (solo il prefisso). **Non consuma la quota giornaliera.**

```bash
curl -H "X-API-Key: $OMI_API_KEY" "https://3eurotools.it/api/v1/me"
```

***

## Tipologie di immobile

Valori ammessi per `tipologia` (le categorie catastali tra parentesi sono puramente indicative). È accettato anche il codice numerico OMI della tipologia (es. `20`).

| `tipologia` | Categorie indicative |
|---|---|
| `abitazioni_civili` | A/2 |
| `ville_e_villini` | A/7, A/8 |
| `abitazioni_di_tipo_economico` | A/3, A/4, A/5 |
| `abitazioni_signorili` | A/1 |
| `abitazioni_tipiche_dei_luoghi` | A/11 |
| `negozi` | C/1 |
| `uffici` | A/10, B/4 |
| `uffici_strutturati` | A/10, B/4 |
| `box` | C/6 |
| `autorimesse` | C/6 |
| `posti_auto_coperti` | C/6, C/7 |
| `posti_auto_scoperti` | C/7 |
| `magazzini` | C/2 |
| `capannoni_tipici` | C/2 |
| `laboratori` | C/3 |
| `capannoni_industriali` | D/7 |
| `centri_commerciali` | D/8 |

***

## Come leggere la risposta

```
risolto                      → come è stata interpretata la richiesta (metodo, comune, zona)
zone → <codice zona>
     → tipologie → <tipologia>
                 → acquisto / affitto → min, max, medio
```

| Campo | Significato |
|---|---|
| `anno` | Anno delle quotazioni restituite. |
| `risolto.metodo` | Come è stata trovata l'area (es. `coordinate`). |
| `risolto.comune` | Codice catastale, nome, provincia e regione del comune. |
| `risolto.zona` | Codice, descrizione e fascia della zona OMI. |
| `acquisto` | Valore di compravendita, già moltiplicato per `mq`. |
| `affitto` | Canone **mensile**, già moltiplicato per `mq`. |
| `min`, `max` | Non sono i prezzi più bassi e più alti registrati, ma il **range** in cui si colloca un immobile della zona: un attico ristrutturato sta in alto, un piano terra da sistemare in basso. |
| `medio` | Media tra `min` e `max`. |
| `stato` | Stato di conservazione della quotazione: `normale`, `ottimo` o `scadente`. |
| `semestri` | Quanti semestri compongono la media dell'anno. |

***

## Limiti

| Situazione | Limite |
|---|---|
| Chiave appena creata | fino a **100 richieste al giorno** |
| Piano free, dopo l'approvazione dell'account (si richiede con un clic dalla dashboard) | fino a **5.000 richieste al giorno** e **30 al minuto** |

- Il conteggio giornaliero si azzera a **mezzanotte, ora italiana**.
- Ogni risposta riporta lo stato dei limiti negli header:

  ```
  X-RateLimit-Limit-Minute: 30
  X-RateLimit-Remaining-Minute: 27
  X-RateLimit-Limit-Day: 5000
  X-RateLimit-Remaining-Day: 4123
  X-RateLimit-Reset-Day: 1794265200   # prossima mezzanotte, timestamp Unix
  ```

- Superato un limite ricevi `429` con l'header `Retry-After` (i secondi da attendere). Le richieste respinte **non consumano la quota**.
- Per limiti più alti scrivi a d.sagona.20@gmail.com.

***

## Errori

Gli errori hanno sempre la stessa forma, con un `code` da usare nel programma e un `message` leggibile:

```json
{ "error": { "code": "ambiguous_comune",
             "message": "Esistono 2 comuni chiamati 'Castro': indica la provincia.",
             "candidates": [ { "codice": "C337", "nome": "Castro", "provincia": "BG" },
                             { "codice": "M261", "nome": "Castro", "provincia": "LE" } ] } }
```

| Stato | Codice | Cosa fare |
|---|---|---|
| 401 | `missing_api_key`, `invalid_api_key`, `api_key_revoked`, `api_key_expired` | Controlla l'header `X-API-Key` o genera una nuova chiave. |
| 403 | `account_suspended` | Contatta il supporto. |
| 400 | `invalid_parameters` | Correggi i parametri indicati in `details`. |
| 404 | `comune_not_found`, `zona_not_found`, `anno_not_available` | Usa uno dei suggerimenti in `candidates` o un anno di `anni_disponibili`. |
| 409 | `ambiguous_comune`, `ambiguous_zona` | Scegli tra i `candidates`: aggiungi la provincia o usa il codice. |
| 429 | `rate_limited` | Attendi i secondi indicati in `Retry-After`. |

***

## Esempi di codice

### cURL

Da coordinate:

```bash
curl -H "X-API-Key: $OMI_API_KEY" \
  "https://3eurotools.it/api/v1/quotazioni?lat=41.8902&lon=12.4922&tipologia=abitazioni_civili&mq=80"
```

Per nome di comune e zona (con `--data-urlencode` per spazi e virgole):

```bash
curl -G -H "X-API-Key: $OMI_API_KEY" \
  --data-urlencode "q=Roma, San Lorenzo" \
  --data-urlencode "tipologia=abitazioni_civili" \
  --data-urlencode "mq=100" \
  --data-urlencode "operazione=acquisto" \
  "https://3eurotools.it/api/v1/quotazioni"
```

Quotazioni di un anno passato:

```bash
curl -H "X-API-Key: $OMI_API_KEY" \
  "https://3eurotools.it/api/v1/quotazioni?comune=H501&zona=B3&tipologia=abitazioni_civili&anno=2010"
```

### Python

```python
import os
import requests

URL = "https://3eurotools.it/api/v1/quotazioni"
CHIAVE = os.environ["OMI_API_KEY"]

risposta = requests.get(
    URL,
    params={
        "lat": 41.8902,
        "lon": 12.4922,
        "tipologia": "abitazioni_civili",
        "mq": 80,
    },
    headers={"X-API-Key": CHIAVE},
    timeout=10,
)

if risposta.status_code == 429:
    attesa = risposta.headers["Retry-After"]
    raise SystemExit("Limite raggiunto, riprova tra " + attesa + " secondi")
risposta.raise_for_status()

dati = risposta.json()
zona = dati["risolto"]["zona"]
prezzi = dati["zone"][zona["codice"]]["tipologie"]["abitazioni_civili"]
print(zona["descrizione"], prezzi["acquisto"]["medio"])
```

### JavaScript (Node.js, con fetch)

```javascript
const params = new URLSearchParams({ lat: 41.8902, lon: 12.4922, tipologia: "abitazioni_civili", mq: 80 });
const risposta = await fetch("https://3eurotools.it/api/v1/quotazioni?" + params, {
  headers: { "X-API-Key": process.env.OMI_API_KEY },
});
const dati = await risposta.json();
if (!risposta.ok) throw new Error(dati.error.message);
const zona = dati.risolto.zona.codice;
console.log(dati.zone[zona].tipologie.abitazioni_civili.acquisto.medio);
```

***

## Migrare dalla vecchia API

La vecchia API senza chiave smette di rispondere il **15 dicembre 2026**. Ci vogliono pochi minuti:

1. **Crea la chiave** gratuita sulla [pagina ufficiale](https://3eurotools.it/api-quotazioni-immobiliari) e salvala in una variabile d'ambiente.
2. **Cambia indirizzo** da `https://3eurotools.it/api-quotazioni-immobiliari-omi/ricerca` a `https://3eurotools.it/api/v1/quotazioni`.
3. **Invia la chiave** nell'header `X-API-Key: la_tua_chiave` (o `Authorization: Bearer`).
4. **Rinomina i parametri** come nella tabella qui sotto.
5. **Leggi la risposta** nel nuovo formato: i prezzi stanno in `zone → <zona> → tipologie → <tipologia> → acquisto / affitto`, con `min`, `max` e `medio`.

| Vecchia API | API v1 | Note |
|---|---|---|
| `codice_comune` | `comune` o `q` | Accetta il codice (`H501`) o il nome (`Roma`). |
| `zona_omi` | `zona` o `q` | Accetta il codice (`C5`) o la descrizione (`San Lorenzo`). |
| `tipo_immobile` / `cod_tip_filtro` | `tipologia` | Stessi nomi (`abitazioni_civili`) oppure il codice numerico (`20`). |
| `metri_quadri` | `mq` | Da 1 a 100.000.000. Default 1, cioè prezzo al metro quadro. |
| `operazione`, `anno` | invariati | Stesso significato di prima. |

**Prima e dopo**

```diff
- GET https://3eurotools.it/api-quotazioni-immobiliari-omi/ricerca?codice_comune=G273&tipo_immobile=abitazioni_civili&metri_quadri=100&zona_omi=B3&operazione=acquisto
+ GET https://3eurotools.it/api/v1/quotazioni?comune=G273&tipologia=abitazioni_civili&mq=100&zona=B3&operazione=acquisto
+ X-API-Key: la_tua_chiave
```

```diff
- prezzo = dati["abitazioni_civili"]["prezzo_acquisto_medio"]
+ zona = dati["risolto"]["zona"]["codice"]
+ prezzo = dati["zone"][zona]["tipologie"]["abitazioni_civili"]["acquisto"]["medio"]
```

La documentazione della vecchia API resta consultabile dalla [pagina ufficiale](https://3eurotools.it/api-quotazioni-immobiliari) fino alla sospensione.

***

## Use case e benefici

Stime del valore degli immobili nel gestionale di un'agenzia, prezzi di zona accanto agli annunci di un portale, app di valutazione, analisi del mercato e del rendimento degli affitti: l'API restituisce i valori OMI già pronti per zona e tipologia.

|  | **Senza API (scaricando i dataset)** | **Con l'API OMI** |
|---|---|---|
| **Gestione dei dati** | Bisogna scaricare e incrociare a mano i file dell'Agenzia delle Entrate, con un dataset pesante da gestire. | Nessun dataset locale: i dati arrivano su richiesta. |
| **Trovare la zona** | Serve incrociare a mano indirizzi e perimetri delle zone OMI. | Basta passare `lat` e `lon`: l'API trova la zona sui perimetri ufficiali. |
| **Aggiornamenti** | Il dataset va riscaricato e reintegrato a ogni semestre. | Vengono usati automaticamente i dati più recenti. |
| **Storico** | Bisogna archiviare e unire i file di ogni semestre. | Con `anno` ottieni le quotazioni di ogni anno dal 2004. |
| **Tempo richiesto** | Ore di lavoro a ogni aggiornamento. | Una chiamata API. |

***

© [3eurotools.it](https://3eurotools.it) · Dati OMI dell'Agenzia delle Entrate
