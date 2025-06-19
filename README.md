# API Quotazioni Immobiliari OMI (con dati ufficiali) 💶🏡
Un'API totalmente gratuita che fornisce le quotazioni immobiliari OMI, basata sui <a href="https://www.agenziaentrate.gov.it/portale/schede/fabbricatiterreni/omi/banche-dati/quotazioni-immobiliari " target="_blank" rel="noopener noreferrer">dati ufficiali dell'Agenzia delle Entrate</a>.

Per altri strumenti gratuiti: [3eurotools.it](https://3eurotools.it)

Pagina d'origine: [3eurotools.it/api-quotazioni-immobiliari-omi](https://3eurotools.it/api-quotazioni-immobiliari-omi)
# F.A.Q. generali
#### Cosa sono le quotazioni immobiliari OMI?
Le quotazioni immobiliari OMI sono dei valori di riferimento forniti dall’Agenzia delle Entrate per stimare in modo rapido il prezzo di vendita o di affitto di un immobile, sulla base della sua tipologia, ubicazione e della superficie espressa in metri quadrati.

#### Queste quotazioni sono affidabili?
Si, rappresentano il miglior per ottenere una stima di un immobile avendo pochissime informazioni, sono rilasciate dall'Agenzia delle Entrate e si basano sugli atti di compravendita e sui contratti di locazione dell'ultimo semestre (attualmente il secondo semestre del 2024).
<a href="https://www.agenziaentrate.gov.it/portale/schede/fabbricatiterreni/omi/banche-dati/quotazioni-immobiliari" target="_blank" rel="noopener noreferrer">Per saperne di più</a>

#### Per utilizzare l'API è necessario effettuare un login?
No, l'API si utilizza grazie a una semplice chiamata che non richiede autenticazione (le istruzioni sono sotto).

Tuttavia, è obbligatorio attribuire i crediti in caso di utilizzo. Dunque si prega di linkare la [pagina web ufficiale dell'API](https://3eurotools.it/api-quotazioni-immobiliari-omi) in caso di utilizzo del servizio in progetti pubblici.

#### Non esistono le API ufficiali fornite dall'Agenzia delle Entrate?
Domanda retorica, no. Questo è il primo (e attualmente unico) servizio libero e gratuito per accedere automaticamente alle quotazioni OMI più aggiornate.

Oltre a non fornire API, il portale dell'Agenzia delle Entrate ha vari limiti: ad esempio è necessario completare manualmente un CAPTCHA ogni volta che si vuole inoltrare una richiesta.

#### Quali sono i limiti di utilizzo?
Il servizio è libero, anche per uso commerciale. Le richieste sono illimitate, ma non troppe in poco tempo: hai un credito di 100 richieste, che si ricarica al ritmo di 1 richiesta al secondo.
Per aumentare i limiti, ottenere risposte più rapide ed eliminare l’obbligo di attribuzione, contattami: d.sagona.20@gmail.com

***

# Come si utilizza?

Per utilizzare il servizio è sufficiente effettuare una semplice richiesta **GET** fornendo i seguenti parametri:

- **`codice_comune`** – **Obbligatorio**. È il <a href="https://www.agenziaentrate.gov.it/portale/documents/20143/448384/Tabella+codici+catastali+comuni_T4_codicicatastali_comuni_24_05_2019.pdf/d4fa70bd-f4bd-caba-24cb-5cc3611237c0" target="_blank" rel="noopener noreferrer">codice catastale del comune</a> (es. `G273` per Palermo).
- **`metri_quadri`** – **Facoltativo**. Numero di metri quadrati commerciali dell'immobile. Il valore predefinito è `1`.
- **`operazione`** – **Facoltativo**. Specifica il tipo di operazione: `affitto` o `acquisto`. Se non indicato, vengono restituiti i dati per entrambe le opzioni.
- **`zona_omi`** – **Facoltativo**. Permette di filtrare i risultati per zona OMI. Se non specificata, vengono mostrati i dati dell’intero comune. Puoi <a href="https://www1.agenziaentrate.gov.it/servizi/geopoi_omi/index.php" target="_blank" rel="noopener noreferrer">trovare la zona OMI di interesse qui</a>.
- **`tipo_immobile`** – **Facoltativo**. Indica la tipologia di immobile (es. `ville_e_villini`). Consulta la sezione qui sotto per l’elenco completo delle tipologie ammesse.

**Elenco di valori ammessi per tipo_immobile (le categorie tra parentesi sono puramente indicative):**

- **abitazioni_civili** (cat. A/2)  
- **ville_e_villini** (cat. A/7, A/8)
- **abitazioni_di_tipo_economico** (cat. A/3, A/4, A/5)  
- **abitazioni_signorili** (cat. A/1)  
- **negozi** (cat. C/1)  
- **uffici** (cat. A/10, B/4)  
- **box** (cat. C/6)  
- **posti_auto_scoperti** (cat. C/7)  
- **posti_auto_coperti** (cat. C/6, C/7)  
- **capannoni_tipici** (cat. C/2)  
- **magazzini** (cat. C/2)  
- **laboratori** (cat. C/3)  
- **autorimesse** (cat. C/6)  
- **capannoni_industriali** (cat. D/7)  
- **abitazioni_tipiche_dei_luoghi** (cat. A/11)  
- **centri_commerciali** (cat. D/8)  
- **uffici_strutturati** (cat. A/10, B/4)

***

### Esempi di richieste tramite browser

#### **Esempio di richiesta completa per un appartamento di 100mq a Palermo, zona omi B3, mostrando solo le quotazioni di compravendita:**
      https://3eurotools.it/api-quotazioni-immobiliari-omi/ricerca?codice_comune=G273&tipo_immobile=abitazioni_civili&metri_quadri=100&zona_omi=B3&operazione=acquisto

#### **Output:**

```json
{
  "abitazioni_civili": {
    "stato_di_conservazione_mediano_della_zona": "normale",
    "prezzo_acquisto_min": 100000.0,
    "prezzo_acquisto_max": 145000.0,
    "prezzo_acquisto_medio": 122500.0
  }
}
```
**Spiegazione output:**

**``"stato_di_conservazione_mediano_della_zona": "normale"``** significa che, in media, in questa zona gli immobili di questa categoria hanno un normale stato di conservazione (se non "normale", può essere "ottimo" o "scadente").

**``"prezzo_acquisto_min"``** e **``"prezzo_acquisto_max"``** rappresentano il range di riferimento per valutare gli immobili in quella zona, mentre **``"prezzo_acquisto_medio"``** è la media di questi due valori.

***

### Esempio di implementazione in Python:
    import requests

    url = "https://3eurotools.it/api-quotazioni-immobiliari-omi/ricerca"
    params = {
        "codice_comune": "G273",
        "metri_quadri": 100,
        "operazione": "acquisto",
        "zona_omi": "B22",
        "tipo_immobile": "abitazioni_di_tipo_economico"
    }
    
    response = requests.get(url, params=params)
    print(response.text)

***

### Esempio di implementazione in Javascript (con fetch)
    const params = new URLSearchParams({
      codice_comune: "G273",
      metri_quadri: 100,
      operazione: "acquisto",
      zona_omi: "B22",
      tipo_immobile: "abitazioni_di_tipo_economico"
    });
    
    fetch("https://3eurotools.it/api-quotazioni-immobiliari-omi/ricerca?" + params)
      .then(response => response.text())
      .then(data => console.log(data));

***
# Use case e benefici di questa API:

### **Confronto: Stima Automatica del Valore di un Immobile (Senza API OMI vs Con API OMI)**

|                   | **Senza API OMI (utilizzando un dataset)**                                                                                     | **Con API OMI**                                                                                       |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **Gestione del dataset**     | Richiede la gestione manuale di un dataset, che può risultare pesante per sistemi con risorse limitate.                   | Nessuna necessità di gestire dataset localmente: l'API fornisce gratuitamente i dati in tempo reale su richiesta.       |
| **Aggiornamenti**            | Il dataset deve essere scaricato e integrato manualmente ogni semestre, con un processo ripetitivo e soggetto a errori.            | Nessun pensiero ad aggiornare i dati, vengono automaticamente utilizzati quelli più recenti.     |
| **Tempo richiesto**          | Processo lungo per scaricare, processare e integrare il dataset nel sistema, con ore di lavoro per ogni aggiornamento.            | Risultati istantanei: basta una chiamata API per ottenere i dati necessari.                              |

