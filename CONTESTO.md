# Pappe — contesto del progetto (per riprendere in una nuova chat)

> Allega questo file all'inizio della nuova chat e scrivi cosa vuoi fare. Se la chat ha accesso al repository GitHub `abbassalatesta/mio-sito1`, l'app è il file `index.html` (ramo `main`); altrimenti allega anche `index.html`.

## 1. Cos'è
App web mobile "Pappe": un unico file HTML/JS (~155 KB, in italiano). Crea menu settimanali di pranzo e cena per tutta la famiglia, con:
- ricette e porzioni adattate all'età;
- valori nutrizionali (carboidrati, grassi, proteine, fibre) ed equilibrio settimanale;
- lista della spesa e dispensa, calendario, colazioni e spuntini;
- più componenti della famiglia, allergeni e avvisi per età;
- piatti creati dall'utente.

Famiglia dell'utente: una bambina di circa 9 mesi (nata a gennaio 2026) e un adulto.

**Dove si trova**
- **Artifact** (privato finché l'utente non lo condivide): https://claude.ai/artifact/4725yDgpfi7AhSKptXK1Ju — **versione 34**.
- **GitHub**: `abbassalatesta/mio-sito1`, file `index.html` su `main`, identico all'artifact v34. Ramo di lavoro usato finora: `claude/ripartiamo-da-qui-t9rraa`.
- `backup/` nel repository: copie delle versioni precedenti (`pappe-v3-backup-pre-*.html`).
- Dati dell'utente: nel browser (`localStorage`, chiave `pappe`) e nel suo account (capability `db` + `user`, documento `data/users/{id}/pappe`, campo `json`).

## 2. Regole di lavoro dell'utente
- "Parliamone prima" / "non fare modifiche": si discute senza toccare il codice. Quando una scelta cambia l'aspetto o il comportamento, chiedere prima.
- **Backup** prima di ogni modifica importante: `backup/pappe-v3-backup-pre-<argomento>.html`.
- **UI pulita**: niente pulsanti in più, stile coerente fra le sezioni, testi brevi.
- **Pubblicazione in due passi**:
  1. prima solo l'artifact (stesso URL), così l'utente prova;
  2. poi `main` su GitHub (pull request + merge) quando l'utente dice "puoi pubblicare".
  Quando l'utente dice "rilascia", fare entrambi.
- **Regola dei rilasci** (vale per tutti i rilasci futuri):
  - un aggiornamento non deve mai cambiare la settimana in corso dell'utente;
  - le modifiche al motore del menu valgono dalla settimana successiva;
  - la settimana in corso cambia solo se l'utente la rigenera o sceglie un piatto;
  - mai toccare pasti preparati, scelti dall'utente o giorni passati;
  - se cambiano le regole del motore, alzare `PV`: `upgradePlans` ricalibra una volta solo le settimane future già create.
- **Verifiche**: provare in Chromium con Playwright (viewport 390×844), confrontare schermate prima/dopo quando la modifica non deve cambiare l'aspetto, dire sempre che non è stato provato su un telefono vero.
- La funzione "pasti fuori casa + adattamento automatico dei pasti successivi" è stata provata e **rifiutata**: non rimetterla se non richiesta.
- Lo **schema settimanale dei gruppi** dell'utente (uova 1, formaggio 2, legumi 3, pesce 3, carne bianca 2, carne rossa 1, più 3 "quadratini bianchi" opzionali per uova, legumi e pesce) è **in pausa**: non introdurlo finché non lo chiede. Oggi il motore usa frequenze più larghe (vedi §4).
- L'ultima prova di spostare i suggerimenti degli ingredienti sopra il campo (per la tastiera del telefono) e il numero di versione nel foglio "i" è stata **scartata** dall'utente: la versione corretta è quella su GitHub.

## 3. Funzioni presenti (v34)
- **Piano**: selettore Giorno/Settimana, striscia dei giorni, pranzo e cena, colazioni e spuntini (3 preparazioni a settimana), "Segna come preparata" (scala la dispensa).
  - Sotto i pasti, una riga con l'equilibrio del giorno ("Giorno in equilibrio · carboidrati x%, grassi y%, proteine z%" oppure "Poche proteine" ecc.).
  - Foglio "i": spiegazione dei valori e dell'equilibrio.
- **Scheda del pasto**: "Segna come preparata", poi la riga "Modifica | Sostituisci | Elimina" (Elimina solo per i piatti tuoi), "Chiudi" in fondo.
- **Modifica**:
  - su un piatto tuo modifica il piatto ovunque compaia;
  - su un piatto dell'app crea un piatto tuo **solo per quel pasto**, con nome e ingredienti dell'originale e i grammi originali; l'originale resta negli altri giorni;
  - si conferma con **Salva**: niente cambiamenti → "Nessuna modifica"; solo il nome → aggiorna il nome senza ricalcolare; ingredienti cambiati → ricalcola i valori e ricalibra la settimana.
- **Elimina** (solo piatti tuoi): schermata di conferma che dice quanti pasti futuri e di quali componenti verranno sostituiti con proposte equilibrate. I pasti preparati e i giorni passati lo conservano.
- **Sostituisci** (foglio "Cambia il pasto"):
  - **Crea il tuo piatto**: nome + ingredienti scelti dal ricettario ingredienti, con suggerimenti mentre si scrive, etichette rimovibili e Invio per testo libero separato da virgole. Gli ingredienti sconosciuti restano segnati "non nel ricettario" ed esclusi dai valori. Poi **Calcola** → anteprima → "Salva e metti nel menu".
  - **Cerca nel ricettario**: cerca piatti o ingredienti; suggerisce tutti gli ingredienti e i piatti del ricettario.
  - **Oppure scegli**: elenco dei piatti adatti; poi lo scambio con un altro pasto.
- **Ricalibrazione automatica**: quando l'utente sceglie, modifica o scambia un piatto, l'app ricalibra prima l'altro pasto del giorno e poi la settimana, toccando solo i pasti proposti dall'app. Un avviso dice quali pasti sono cambiati, con "Annulla".
- **Avvisi per età**: ingredienti sconsigliati sotto una certa età.
  - Limite rigido (piatto bloccato per quel componente): miele, sale, zuccheri, salumi, formaggi salati/affumicati/erborinati, molluschi, pesce affumicato o sotto sale, dado, funghi, olive e capperi, cioccolato e cacao, caffè, alcol.
  - Solo nota sotto i 4 anni: frutta secca, uva/ciliegie/pomodorini, peperoncino.
  - Un piatto tuo troppo "adulto" per il componente mostra "da N mesi" e il pulsante "Non adatto a …".
- **Spesa/dispensa**, **Calendario** (puntini per i pasti preparati, vista Famiglia), **Famiglia** (più componenti, allergeni glutine/latte/uova/pesce/legumi + testo libero), **Segnala** (solo anteprima, nessun invio).
- Selettore del componente come chip nell'header; fogli chiudibili trascinando verso il basso.
- Backup: export/import (base64 JSON, validazione, doppia conferma); salvataggio locale + account (sospeso sopra 240 KB).

## 4. Dettagli tecnici
- **Stato `S`**:
  - `kids[]`: id, name, sex, dob, `plans{lunedìISO:[14 id]}`, `done{data-slot:id+1}`, stock, bought, avoid[], avoidTxt, ageKey, snk, `fix{data-slot:id+1}` (pasti scelti dall'utente), `pv` (versione motore applicata);
  - `cur`, `custom[]` (piatti tuoi `{id 100–199, n, i0, m0, x, h}`).
  - `S.plans/done/stock/bought/ageKey` sono proxy sul componente corrente `K()`.
- **Piatti** (`B` mappa id → piatto; `R[id]={t,s[]}` ricette):
  - `D` = piatti di base usati per generare il menu:
    - 0–13 pappe 6–24 mesi;
    - 14–20 piatti 10 mesi–12 anni;
    - 21–29 piatti per bambini dai 2 anni;
    - 30–49 adulti (internazionali segnati `int`: 37–47, 49);
    - `XD` aggiunti: 50–59 piatti 10 mesi–12 anni, 70–74 pappe 6–12 mesi, 76–86 piatti italiani per adulti.
  - `SND` 60–66: colazioni e spuntini, fuori dal menu.
  - `CB` 200–249: ricettario (36 italiani + 14 pappe 236–249), **solo su ricerca** (scelta dell'utente).
  - Piatti tuoi: 100–199.
  - Ogni piatto `{id,n,t,a:[minMesi,maxMesi],i0,m0,...}`; porzioni scalate da `fac(mesi)` in `applyAge()` (<8:.75, <10:1, <12:1.25, <24:1.5, <48:2, <84:3, <144:3.5, ≥144:4).
- **Ricettario ingredienti**:
  - `NU`: 288 voci, nome → [carboidrati, grassi, proteine, fibre per 100 g, porzione base; 0 = non entra nelle porzioni];
  - `ALI`: 146 sinonimi;
  - funzioni `parseIng`, `resolveIng` (prima il nome intero, poi senza parole vuote; vince il nome o sinonimo più specifico), `calcM`;
  - allergeni `GR`, gruppi proteici `GI`/`gOf`, basi `BASEI`/`bOf`, regole per età `AGE`/`minA`/`ageInfo`.
- **Compositore**: `cmpHTML`, `cSug`, `cAdd`, `cTxt`, `compose`, `ideaView`, `putIdea`, `editDish`/`saveEdit`, `copyDish`/`saveCopy`, `delCustom`/`delDo`; stato `cName`, `cSel`, `cOrig` ('pick' | 'ideas' | 'edit' | 'copy'), `cEdit`, `cSlot`.
- **Motore del menu**:
  - obiettivi per età `TGT`, come quota di energia di pranzo + cena:
    - <12 mesi: carboidrati 42±10, grassi 40±8, proteine 18±6;
    - <36 mesi: carboidrati 45, grassi 36, proteine 18;
    - <12 anni: carboidrati 50, grassi 32, proteine 18, fibre ≥8 g/1000 kcal;
    - adulti: carboidrati 50, grassi 30, proteine 20, fibre ≥12;
  - frequenze settimanali `GW`: legumi 3–4, pesce 2–3, carne bianca 2–3, carne rossa 1–2, uova 1–2, latticini 1–3; abbassate se per l'età ci sono pochi piatti del gruppo (`wCtx`);
  - `wCost` penalizza:
    - giorni fuori equilibrio (peso maggiore);
    - stesso piatto nello stesso giorno;
    - ripetizioni nella settimana, di più se in giorni consecutivi;
    - stessa base o stessi ingredienti principali a pranzo e cena;
    - basi oltre il limite settimanale (riso 3, polenta 2, altre 4);
    - più di 2 piatti internazionali a settimana;
  - `balance` fa una ricerca locale sui pasti liberi (sostituzioni + scambi); `rebal` agisce dopo una scelta dell'utente; `upgradePlans` con `PV=2`;
  - risultati di prova: nessun doppione nello stesso giorno, nessuna stessa base a pranzo e cena, giorni in equilibrio ~100% per tutte le età.
- **Trappole note**:
  - negli handler inline non usare una funzione globale `open` (usare `openPick`);
  - gli attributi con apostrofi vanno passati da `E()`;
  - i nomi dei piatti tuoi vengono ripuliti con `cleanN`;
  - `fixWeeks` non sostituisce id ≥100;
  - `bkOk` accetta nei piani ogni id esistente tranne gli spuntini (`SND`);
  - i test che dipendono dalla data vanno lanciati con l'orologio fissato (`page.clock.install`).
- **Test**: script Playwright scritti di volta in volta nello scratchpad (non salvati nel repository): generazione di 20 settimane per 8 fasce d'età, compositore, avvisi per età, ricalibrazione/Annulla, modifica/eliminazione, backup, confronto pixel delle schermate.

## 5. Sicurezza
L'app non ha codice server. Rischi reali:
- dati familiari sensibili nei backup;
- condivisione del link;
- nessun audit indipendente.

L'artifact resta privato finché l'utente non lo condivide. I valori nutrizionali sono stime da tabelle italiane arrotondate e le regole per età seguono indicazioni pediatriche comuni: da confermare con il pediatra.

## 6. Decisioni prese (storico)
- Niente AI a runtime e niente server: solo ricettario integrato + ingredienti scelti dall'utente.
- Il menu usa piatti pronti (ricette fisse), non composizioni automatiche dagli ingredienti: più sicuro per lo svezzamento.
- Il ricettario `CB` non entra nella generazione del menu.
- Menu degli adulti misto, ma con al massimo 2 piatti internazionali a settimana.
- Ricalibrazione automatica con avviso e Annulla, ma solo sui pasti proposti dall'app.

## 7. Da fare / idee aperte
- **Segnala**: manca l'indirizzo email di destinazione (deve fornirlo l'utente).
- Quantità facoltative quando l'utente crea o modifica un piatto.
- "Salva" invece di "Calcola" anche nella creazione di un piatto nuovo.
- Decidere se modificare un pasto già mangiato debba ricalibrare gli altri pasti (oggi lo fa).
- Schema settimanale dei gruppi dell'utente (in pausa, vedi §2).
- Sul telefono la tastiera può coprire i suggerimenti degli ingredienti, che compaiono sotto il campo. Una correzione è stata provata e scartata: riparlarne con l'utente se il problema si ripresenta.
- Possibili: salvataggio dei piatti tuoi per singolo componente invece che per famiglia, più piatti di base.
- Mai provata su un telefono vero: chiedere riscontri all'utente.

## 8. Come riprendere
1. Apri la nuova chat con accesso al repository `abbassalatesta/mio-sito1` (oppure allega `index.html`) e allega questo file.
2. Per una modifica: backup in `backup/` → modifica → prova in Chromium → pubblica l'artifact sullo stesso URL (passando l'URL come `url`; leggerlo prima con `read` se la chat è nuova) → quando l'utente conferma, pull request + merge su `main`.
3. A ogni rilascio aggiornare questo file (versione e funzioni).
