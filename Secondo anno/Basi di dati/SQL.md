
# DataBase
spesso per fare gli esempi si utilizza la seguente realtà medica:
**MEDICO** (Matricola, Cognome, Nome, Specializzazione, Parcella, Citta, PRIMARY KEY (Matricola))
**PAZIENTE**(CodFiscale, Cognome, Nome, Sesso, DataNascita, Citta, Reddito,PRIMARY KEY (CodFiscale))
**VISITA**(Medico, Paziente, Data, Mutuata, PRIMARY KEY (Medico,Paziente,Data))

## Schemino per risoluzione esercizi
- **Per ogni / per ciascuno** → **Raggruppamento** → `'GROUP BY'` in SQL _(Esempio: numero di studenti per corso)_
    
- **Solo / solamente / esclusivamente** → **'Differenza'** _(Si usa con `EXCEPT`, `NOT EXISTS` o `LEFT JOIN ... WHERE ... IS NULL`)_
    
- **Tutti / tutte / ogni** → **'Divisione'** _(Si risolve con doppio `NOT EXISTS` oppure con il raggruppamento; esempio tipico: "chi ha superato _tutti_ gli esami")_
    
- **Condizioni aggregate** (quelle con funzioni tipo `COUNT`, `SUM`, `AVG`) → **NON VANNO MESSE NEL `WHERE`, MA NEL `HAVING`.** _(Es: `HAVING COUNT(*) > 5`)_

|**Parola nella traccia**|**Cosa usare in SQL**|
|---|---|
|Per ogni / Ciascuno|`GROUP BY`|
|Solo / Solamente|**Differenza** ➔ `NOT EXISTS`, `EXCEPT`, `LEFT JOIN ... IS NULL`|
|Tutti / Tutte / Ogni|**Divisione** ➔ `NOT EXISTS` doppio|
|Condizioni aggregate (`COUNT`, `SUM`, `AVG`)|➔ **nel** `HAVING` **(non** `WHERE` **)**|
|Diversi / Unici|`DISTINCT`|
|Se esiste almeno uno|`EXISTS`|
|Non esiste / Nessuno ha fatto|`NOT EXISTS`|

## Introduzione 
le query si fanno seguendo la struttura 
```sql
SELECT Lista_Attributi
FROM Insieme_Tabelle
WHERE Lista_Condizioni;
```

facciamo un primo esempio di query

- Indicare cognome e nome delle persone di età inferiore a 40 anni
```sql
SELECT Cognome, Nome
FROM Persona
WHERE Età < 40;
```

se voglio indicare tutte le informazioni (tutte le colonne): **SELECT * **

Si possono usare operatori logici (AND, NOT, OR)
- Indicare il codice fiscale delle persone di età inferiore a 40 anni il cui cognome è Lepre
```sql
SELECT CodFiscale
FROM Persona
WHERE Età < 40
	AND Cognome = 'Lepre';
```

## Intervallo di valori
per verificare se il valore di un attributo appartiene a un intervallo si usa **BETWEEN** oppure **>= <=**.

- Indicare Cognome e Nome delle persone di età compresa fra 45 e 60 anni
```sql
SELECT Cognome, Nome
FROM Persona
WHERE Età BETWEEN 45 AND 60;
```
alternativa
```sql
SELECT Cognome, Nome
FROM Persona
WHERE Età >= 45 
	AND Età <= 60;
```

## Duplicati
- Indicare i cognomi delle persone di età almeno pari a 38 anni
```sql
SELECT Cognome
FROM Persone
WHERE Età >= 38;
```

se io ho delle persone omonime esse vengono ripetute più volte, per ovviare a questa cosa uso **DISTINCT**

```sql
SELECT DISTINCT Cognome
FROM Persona
WHERE Età >= 38;
```

il distinct viene processato come ultima cosa

## Valori NULL
un valore NULL è un valore mancante 
- indicare matricola e data d laurea degli studenti laureati immatricolati fa l'anno 2001 e l'anno 2005
```sql
SELECT Matricola, DataLaurea
FROM Studente
WHERE DataLaurea IS NOT NULL
	AND DataIscrizione BETWEEN '2001-01-01' AND '2005-12-31';
```
>Le date vengono scritte nel formato 'anno-mese-giorno'
in questo esempio si utilizza IS NOT NULL se uno studente ha NULL la DataLaurea viene automaticamente rimosso dalla query

Si può utilizzare **IS NULL** che elimina tutti i record che assumono valore diverso da NULL sull'attributo considerato

## Gestione date
ci sono due tipi di date
- DATE con formato YYYY-MM-DD
- TIMESTAMP con formato YYYY-MM-DD HH:MM:SS

esiste la funzione **DATE_FORMAT** per manipolare le date (guarda slide 60 lezione1)

- indicare matricola e data di laurea (nel formato ‘dd|mm|yyyy, nome_giorno’)  degli studenti iscritti prima del 2005
```sql
SELECT Matricola, DATE_FORMAT(DataLaurea, ‘%d|%m|%Y, %W’)
FROM Studente
WHERE DataIscrizine < '2005-01-01';
```
- indicare la matricola degli studenti che si sono laureati di mercoledì
```sql
SELECT Matricola
FROM Studente
WHERE DATE_FORMAT(DataLaurea, '%w') = 3;
```
>DATE_FORMAT(DataLaurea) = 3 indica che si verifica che sia il terzo giorno della settimana

 Sulle date si possono fare anche dei confronti: usando gli operatori logici o il between. (il sistema tratta le date come lunghezze)

-Indicare matricola e mese di laurea degli studenti immatricolati dopo il 2000
```sql
SELECT Matricola, MONTH(DataLaurea)
FROM Studente
WHERE DataLaurea IS NOT NULL
	AND YEAR(DataIscrizione) > 2000;
```
> Le funzioni **DAY()**, **MONTH()**, **YEAR()** prendono come argomento una data e ne restituiscono rispettivamente il giorno, mese e anno.

-Indicare il cognome degli studenti che si sono laureati 5 anni fa
```sql
SELECT Cognome
FROM Studente 
WHERE DataLaurea IS NOT NULL
	AND YEAR(DataLaurea) = YEAR(CURRENT_DATE) -5;
```
> CURRENT_DATE corrisponde alla data corrente

> [!warning] Attenzione
> - non si possono sottrarre dei valori di tipo data:  '2005-07-15' - '2005-09-34'
> - si può fare: YEAR('2005-07-15') - YEAR('2005-09-34') perchè li convertisco in interi

- Indicare matricola e da quanti giorni risultavano iscritti gli studenti, ad oggi laureati, che non si erano ancora laureati il 15 luglio 2005
```sql
SELECT Matricola, DATEDIFF('2005-07-15', DataIscrzione)
FROM Studente
WHERE DataIscrizione < '2005-07-15'
	AND DataLaurea > '2005-07-15'
```
> datediff(data1, data2) esprime di quanti giorni data1 è più recente di data2

- Indicare la matricola e il mese di iscrizione degli studenti che si sono laureati dopo 5 anni esatti dal giorno dell'iscrizione
```sql
SELECT Matricola, MONTH(DataIscrizione)
FROM Studente
WHERE DataLaurea = DATE_ADD(DataIscrizine, INTERVAL 5 YEAR);
```
non importa aggiungere DataLaurea is not null perchè l'uguaglianza dell'ultima riga scarta già i record con DataLaurea pari a NULL
> le funzioni DATE_ADD e DATE_SUB permettono di sommare o sottrarre intervalli di tempo a una data
> INTERVAL esprime un intervallo di tempo espresso in giorni , mesi o anni

alternativa:

```sql
SELECT Matricola, MONTH(DataIscrizone)
FROM Studente
WHERE DataLaurea IS NOT NULL
	AND DataLaurea = DataIscrizione + INTERVAL 5 YEAR;
```

ci sono delle funzioni utili sulle date a slide 75 della prima lezione

## Operatori di aggregazione
Si applicano nel SELECT. Questi operatori comprimono gli attributi (selezionati) che arrivano nel select e producono uno scalare

1. COUNT( )
2. SUM ( )
3. AVG( )
4. MAX ( )
5. MIN ( )

> 
1. Il COUNT è una funzione che conta le righe.
- indiare il numero di visite effettuate in data 1° Marzo 2013
```sql
SELECT COUNT(*) AS VisitePrimoMarzo
FROM Visita
WHERE Data = '2013-03-01';
```

>COUNT( * ) conta quello che rimane dopo il where e produce uno scalare che viene collocato in una cella chiamata VisitePrimoMarzo (ciò dovuto dal fatto che ho messo AS)

N.B. occhio alla spaziatura, non deve esserci tra COUNT e le parentesi, altrimenti non funziona 

- indicare il numero di pazienti visitati nel mese di Marzo 2013
```sql
SELECT COUNT(DISTINCT Paziente)
FROM Visita
WHERE MONTH(Data) = '03'
	AND YEAR(Data) = '2013';
```
se avessi scritto COUNT( * ) avrebbe contato le visite, invece noi vogliamo il numero di pazienti --> usiamo il DISTINCT perchè un paziente potrebbe andare più volte

2. La SUM è una funzione che permette la somma.
- supponendo che Umberto Manzi sia il marito di Antonella Lepre, calcolare il reddito totale della famiglia Manzi
per questo esercizio consideriamo la relazione Paziente(<u> CodFiscale </u>, Cognome, Nome, DataNascita, Reddito)

```sql
SELECT SUM(Reddito) AS RedditoTotale
FROM PAZIENTE 
WHERE (Cognome = 'Lepre'
	AND Nome='Antonella')
	OR
	(Cognome = 'Manzi'
	AND
	Nome='Umberto');
```
sta cosa prende i redditi dei bro e li somma
N.B. occhio al connettivo logico -- > serve un OR --> una persona non può essere sia Antonella che Umberto

3. Il AVG è una funzione che permette la media.
- calcolare il reddito medio dei pazienti dopo il 1950
```sql
SELECT AVG(Reddito) AS RedditoMedio
FROM Pazienti
WHERE DataNascita >= '01-01-1950';
```

4. il MIN è una funzione che individua il valore minimo
- ricavare il reddito minimo fra quelli di tutti i pazienti
```sql
SELECT MIN(Reddito)
FROM Pazienti;
```

5. il MAX è una funzione che individua il valore massimo
- ricavare il reddito massimo fra quelli di tutti i pazienti
```sql
SELECT MAX(Reddito)
FROM Pazienti;
```

> [!warning] Manifestazione del maligno
> indicare reddito massimo con relativo cognome e nome di chi lo detiene
> ```sql
>SELECT MAX(Reddito), Nome, Cognome
>FROM Paziente
>```
>una cosa così non si può fare


## Query su più tabelle
vi può essere la necessità di riunire (combinare) le informazioni provenienti da più tabelle, vi sono vari modi per farlo:
1. Inner join
2. Join naturale
3. Cross Join
4. Join esterno
5. Join multiplo
6. Self join

> 
1. Inner Join
date due tabelle combina tutti i record record della prima con tutti i record della seconda che verificano un determinato predicato
- Indicare nome e cognome dei medici che hanno effettuato almeno una visita
```sql
SELECT DISTINCT M.Nome, M.Cognome
FROM Visita V
	INNER JOIN
	Medico M ON V.Medico = M.Matricola;
```
le informazioni richieste provenivano dalla relazione Medico (nome e cognome) e dalla relazione Visita (l'eventualità di aver fatto una visita).
Inner Join ha lavorato scorrendo le tabelle e cercando delle corrispondenze (le tuple che avevano stessa matricola di medico) (come se avessi fatto un equi join in algebra relazionale)
> quando scrivo Medico M sto indicando un alias da usare per richiamare Medico

2. Join naturale
Questo join combina i record delle tabelle che hanno valori uguali sugli attributi omonimi 
> nel DataBase della clinica sta roba non è fattibile perchè non ci sono attributi con lo stesso nome 

3. Cross Join
corrisponde al prodotto cartesiano: restituisce tutte le possibili combinazioni

- indicare il numero di pazienti aventi nome uguale ad almeno un medico della clinica
```sql
SELECT COUNT(DISTINCT P.CodFiscale)
FROM Paziente P
	CROSS JOIN 
	Medico M
WHERE P.Nome=M.Nome;
```

4. Join esterni
Si dividono in:
	- join esterno sinistro: LEFT OUTER JOIN
	- join esterno destro: RIGHT OUTER JOIN
	
date due tabelle, combina ogni record della prima tabella con i record della seconda tabella che soddisfano la condizione, mantenendo tutti i record di una delle due tabelle

- indicare le visite effettuate da medici che non lavorano più presso la clinica
(consideriamo i medici dei quali non abbiamo più i dati nella tabella medici ma abbiamo ancora le loro visite)
```sql
SELECT V.*
FROM Visita V
	LEFT OUTER JOIN
	Medico M ON V.Medico = M.Matricola
WHERE M.Matricola IS NULL;
```

scrivendo V.* proietta solo le cose che provengono dalla tabella Visita

il join esterno sinistro mantiene tutti i record della tabella di sinistra mettendo null a destra per i record non joinabili

> il where contiene condizione per mantenere i record di interesse, dopo l'esecuzione del join

5. Join Multiplo
il join può essere fatto su più di due tabelle
- indicare nome e cognome dei pazienti visitati dal dott. Rino Neri
```sql
SELECT 
FROM Paziente P
	INNER JOIN
	Visita V ON P.CodFiscale = V.Paziente
	INNER JOIN 
	Medico M ON V.Medico = M.Matricola
WHERE M.Nome = 'Rino'
	AND M.Cognome = 'Neri';
```

qua è obbligatorio usare gli alias

6. Self join
combina i record di una tabella con i record della stessa tabella (che rispettano una determinata condizione)

- indicare il codice fiscale dei pazienti che sono stati visitati più di una volta da uno stesso medico della clinica, nel mese corrente
```sql
SELECT V1.Paziente
FROM Visita V1
	INNER JOIN
	Visita V2 ON (
					V2.Medico=V1.Medico
					AND V2.Paziente=V1.Paziente
					AND V2.Data <> V1.Data
					)
WHERE MONTH(V1.Data) = MONTH(CURRENT_DATE) 
	AND YEAR(V1.Data) = YEAR(CURRENT_DATE)
	AND MONTH(V2.Data) = MONTH(CURRENT_DATE) 
	AND YEAR(V2.Data) = YEAR(CURRENT_DATE);
```

si possono fare anche dei self join esterni

## Derived tabel
Sono tabelle volatili che possono essere incapsulate nel join

- indicare la matricola dei medici che non hanno mai visitato di giovedì
```sql
SELECT DISTINCT V1.Medico
FROM Visita V1 
LEFT OUTER JOIN	(
				SELECT V2.Medico 
				FROM Visita V2
				WHERE DAYOFWEEK(V2.Data) = 4;
				) AS D
				ON V1.Medico = D.Medico
WHERE D.Medico IS NULL;
```

la sotto query seleziona i medici che hanno visitato di giovedì, facendo poi un join completo a sx unisco le due tabelle e i medici che non trovano corrispondenza nella tabella D (ovvero D.Matricola=NULL) vuol dire che sono quelli che non hanno mai visitato di giovedì --> top

## Raggruppamento e condizioni su di essi
il raggruppamento suddivide un insieme di record in gruppi di record. In ogni gruppo il valore di un attributo (anche più di uno) è costante in tutti i record. (siparietto capelli biondi)

- Data ogni specializzazione, indicarne il nome e la parcella media dei medici
(devo dividere in gruppi caratterizzati dalla stessa tipologia di specializzazione --> ricordiamoci che ogni gruppo collassa in un unico record)

```sql
SELECT  Specializzazione, AVG(Parcella) AS ParcellaMedia
FROM Medico
GROUP BY Specializzazione;
```

- Restituire valor medio e deviazione standard delle parcelle medie dei medici delle varie specializzazioni
```sql
SELECT AVG(D.ParcellaMedia) AS ValorMedio, STDDEV(D.ParcellaMedia) AS DerivazioneStandard
FROM 
	(
	SELECT AVG(Parcella) AS ParcellaMedia
	FROM Medico
	GROUP BY Specializzazione
	) AS D;
```

> [!warning] Manifestazione del maligno
>la proiezione può contenere solo attributi presenti nel predicato di raggruppamento, gli altri attributi proiettati devono essere argomento di funzioni di aggregazione!
> ```sql
>SELECT Specializzazione, MIN(Parcella), **Cognome**
>FROM Medico
>GROUP BY Specializzazione;
>```
>una cosa così non si può fare

- per ogni specializzazione medica, indicarne il nome, la parcella minima e il cognome del medico a cui appartiene
per sconfiggere malefica...
```sql
SELECT M.Specializzazione, D.ParcellaMinima, M.Cognome
FROM Medico M
	NATURAL JOIN
	(
	SELECT Specializzazione, MIN(Parcella) AS ParcellaMInima
	FROM Medico
	GROUP BY Specializzazione
	) AS D
WHERE M.Parcella = D.ParcellaMinima;
```

**CONDIZIONI SUI RAGGRUPPAMENTI**
Le condizioni sui gruppi sono espresse esclusivamente tramite operatori di aggregazione e permettono di scartare gruppi, qualora non siano soddisfatte.
Queste condizioni sono controllate gruppo per gruppo, non record per record.

-Indicare le specializzazioni della clinica con più di due medici
```sql
SELECT Specializzazione
FROM Medico
GROUP BY Specializzazione
HAVING COUNT(*) > 2;
```

-Indicare le specializzazioni con la più alta parcella media (occhio al maligno)
```sql
SELECT M.Specializzazione
FROM Medico M
GROUP BY M.Specializzazione
HAVING AVG(M.Parcella) = -- cerco quella che è la parcella max per poterci affiancare il nome della specializzazione
	(
	SELECT MAX(D.MediaParcelle) --di D seleziono il massimo
	FROM 
		(
		SELECT M2.Specializzazione, AVG(M2.Parcella) AS MediaParcelle
		FROM Medico M2
		GROUP BY M2.Specializzazione
		) AS D --media parcella relativa a ogni specializzazione
	);
```
ragionamento:
 - Media delle parcelle dei medici di ciascuna specializzazione 
 - Calcolo della più alta parcella media 
 - Specializzazioni con media parcelle uguale alla più alta 

>con il codice che abbiamo scritto proietta gli eventuali pari merito

**CONDIZIONI SUI GRUPPI VS CONDIZIOI SUI RECORD**
- Le condizioni nel WHERE sono applicate ai record prima del raggruppamento
- Le condizioni nell'HAVING sono applicate ai gruppi, dopo il raggruppamento
> Nelle interrogazioni con raggruppamento, le condizioni esprimibili con operatori di aggregazione devono sempre essere argomento della clausola having

## Common Table Expression (CTE)
Sono result set dotati di identificatori che possono essere usati prima di una query per costruire risultati intermedi.
--> Parti di codice i cui risultati sono stoccati in memoria e nominati

**SINTASSI**
```sql
WITH
	name1 AS (query1)
	,
	name2 AS (query2)
	,
	...
	name2 AS (query2)
	,
query finale;
```

- indicare il  numero di pazienti di Siena, mai visitati da ortopedici
```sql
WITH pazienti_visitati_ortopedici AS 
	(
	SELECT V.Paziente AS CodFiscale
	FROM Visita V
		INNER JOIN 
		Medico M ON V.Medico = M.Matricola
	WHERE M.Specializzazione = 'Ortopedia'
		AND P.Città = 'Siena'
	) -- questa è la CTE
	SELECT COUNT(*)
	FROM Paziente P
		NATURAL RIGHT OUTER JOIN
		pazienti_visitati_ortopedici PVO
	WHERE PVO.CodFiscale IS NULL;
```
quella appena scritta è al soluzione del prof PER ME è SBAGLIATA

a seguito le soluzioni di chat, me e lore
```sql
WITH pazienti_visitati_ortopedici AS 
	(
	SELECT V.Paziente AS CodFiscale
	FROM Visita V
		INNER JOIN 
		Medico M ON V.Medico = M.Matricola
        INNER JOIN
        Paziente P ON V.Paziente = P.CodFiscale
	WHERE M.Specializzazione = 'Ortopedia'
		AND P.Citta = 'Siena'
	) -- questa è la CTE
	SELECT COUNT(*)
	FROM Paziente P
		LEFT JOIN
		pazienti_visitati_ortopedici PVO ON P.CodFiscale = PVO.CodFiscale
	WHERE P.Città = 'Siena' 
		AND PVO.CodFiscale IS NULL;






WITH pazienti_visitati_ortopedici AS 
	(
	SELECT V.Paziente AS CodFiscale
	FROM Visita V
		INNER JOIN 
		Medico M ON V.Medico = M.Matricola
        INNER JOIN
        Paziente P ON V.Paziente = P.CodFiscale
	WHERE M.Specializzazione = 'Ortopedici'
		AND P.Citta = 'Siena'
	) -- questa è la CTE
	-- nella CTE ho tutti i pazienti di siena visitati da Ortopedici

-- nella query principale controllo se i pazienti appaiono nella CTE, se non vi appaiono vanno nel conteggio
	SELECT COUNT(*)
	FROM Paziente P
		LEFT OUTER JOIN
		pazienti_visitati_ortopedici PVO ON P.CodFiscale = PVO.CodFiscale
	WHERE P.Citta = 'Siena' 
		AND PVO.CodFiscale IS NULL;

-- Il LEFT OUTER JOIN serve per mantenere tutti i pazienti anche se non hanno match con la CTE
```

> [! warning ]
> VERIFCA CHE SUCCEDE QUA E SUBITO DOPO QUA
> slide 52 e 53 della presentazione 3, argomento CTE

versione di lore della slide 53
```sql
WITH ortopedici AS (
  SELECT M.Matricola
  FROM Medico M
  WHERE M.Specializzazione = 'kkk'
),
paz_visitati_ortopedici AS (
  SELECT V.Paziente
  FROM Visita V
  INNER JOIN ortopedici O ON V.Medico = O.Matricola
)
SELECT COUNT(*)
FROM Paziente P
LEFT OUTER JOIN paz_visitati_ortopedici PVO ON P.CodFiscale = PVO.Paziente
WHERE P.Citta = 'pisa'
  AND PVO.Paziente IS NULL;
```

## Subquery
Sono query (dette subquery) che possono essere incapsulate in un'altra query. Esse rappresentano un'alternativa al join per query su più tabelle.
##### Noncorrelated subquery
Si incapsulano nel WHERE per ottenere risultati usati dalla outer query per determinare il result set finale.
(Immagine del bidone, il contenuto del bidone è usato dalla query esterna per cercarci le cose dentro)
I record ottenuti dalla subquery non dipendono dalla outer query.
Un record fa parte del risultato se i valori che assume su un sottoinsieme di attributi si trova anche in (alemeno) un record del result set della sebquery.

- Indicare nome, cognome e parcella degli ortopedici che hanno effettuato almeno una visita nell'anno 2013
```sql
SELECT M.Nome, M.Cognome, M.Parcella
FROM Medico M
WHERE M.Specializzazione = 'Ortopedia'
	AND M.Matricola IN (
						SELECT V.Medico
						FROM Visita V
						WHERE YEAR(V.Data) = 13
						); 
```
ho diviso la query in:
1. Indicare nome, cognome e parcella degli ortopedici --> questa è la outer query
2. che hanno effettuato almeno una visita nell'anno 2013 --> questa è la subquery
'IN' permette di controllare la presenza della matricola all'interno del risultato della subquery (il bidone) 

- indicare i cognomi dei pazienti che non appartengono anche a un medico
```sql
SELECT DISTINCT P.Cognome
FROM Paziente P
WHERE P.Cognome NOT IN (
						SELECT M.Cognome 
						FROM Medico M
						);
```
avrei potuto farlo con un join esterno
>usando NOT IN un record della outer query va nel risultato solo se non è presente nel result set della subquery

Data una query con subquery è sempre possibile passare alla versione basata su join, e viceversa.

è possibile effettuare degli annidamenti multipli-

La visibilità delle subquery è così strutturata:
-Da dentro vedo tutto 
-Da fuori non vedo dentro 
ovvero: in una subquery si possono riferire tutti gli attributi delle query esterne, a qualsiasi livello di annidamento esterno essi si trovino (si esce ma non si entra).

***SUBQUERY SCALARI***
Restituiscono un unico record, composto da un unico attributo
- indicare il numero degli otorini aventi parcella più alta della media delle parcelle dei medici della loro specializzazione
```sql
SELECT COUNT(*)
FROM Medico M1
WHERE M1.Specializzazione = 'Otorinolaringoiatria'
	AND M1.Parcella > (
						SELECT AVG(M2.Parcella)
						FROM Medico M2
						WHERE M2.Specializzazione = 'Otorinolaringoiatria'
					);
```
la subquery restituisce un numero



#### Correlated subquery
Il loro risultato dipende da ciascuna tupla della query esterna.
(concetto simile a una chiamata di funzione)
- Indicare matricola dei medici che hanno visitato per la prima volta almeno un paziente nel mese di Ottobre 2014
(per la prima volta --> il paziente non è mai stato visitato da quel medico prima)
```sql
SELECT DISTINCT V1.Medico
FROM Visita V1
WHERE YEAR(V1.Data) = 2014
	AND MONTH(V1.Data) =10
	AND V1.Paziente NOT IN (
							SELECT V2.Paziente
							FROM Visita V2
							WHERE V2.Medico = V1.Medico
								AND V2.Data < V1.Data
							) 
```
la query esterna contiene le visite di ottobre 2013
la subquery viene eseguita per ogni tupla selezionata dalla query esterna
(riconosco che è una subquery correlated perchè viene eseguita più volte --> una per riga)

nella subquery compaiono **riferimenti alla riga corrente della query esterna** --> la subquery non può essere eseguita da sola

______________

> le correlated subquery possono essere inserite anche nel SELECT per calcolare un valore da inserire, come attributo, nel risultato (parliamo di subquery scalari)

- Considerato ciascun paziente di sesso maschile, indicarne il nome e il numero di visite effettuate
```sql
SELECT P. Nome, ( SELECT COUNT(*)
					FROM Visita V
					WHERE V.Paziente = P.CodFiscale) AS TotaleVisite
FROM Paziente P
WHERE P.Sesso = 'M';
```
(il numero di visite effettuate è un'informazione da proiettare, non una condizione)

--------------
**COSTRUTTO EXIST**
Permette di verificare che il result set di una correlated subquery contenga almeno un record
(la sua negazione controlla che il result set sia vuoto)

- una visita di controllo è una visita in cui un medico visita un paziente già visitato precedentemente almeno una volta. Indicare medico, paziente e date delle visite di controllo del mese di Gennaio 2016
```sql
SELECT V1.Medico, V1.Paziente, V1.Data
FROM Visita V1
WHERE MONTH(V1.Data) = 1
	AND YEAR(V1.Data) =2016
	AND EXISTS (
				SELECT *
				FROM Visita V2
				WHERE V2.Medico = V1.Medico
					AND V2.Paziente = V1.Paziente
					AND V2.Data < V1.Data
				)
```

-prendi tutte le visite di gennaio 2016
-per ciascuna, controlla se il medico aveva già visitato quel paziente in passato
-se si --> è una visita di controllo

--> alternativa con il NOT EXIST
- Indicare la matricola dei medici che hanno visitato **per la prima volta** almeno un paziente nel mese di Ottobre
```sql
SELECT V1.Medico, V1.Paziente, V1.Data
FROM Visita V1
WHERE MONTH(V1.Data) = 1
	AND YEAR(V1.Data) =2016
	AND NOT EXISTS (
				SELECT *
				FROM Visita V2
				WHERE V2.Medico = V1.Medico
					AND V2.Paziente = V1.Paziente
					AND V2.Data < V1.Data
				)
```

## Divisione
Operatore insiemistico derivato, utile per interrogazioni che contengono condizione esaustive espresse mediante l'avverbio *tutti*

------- divisione con doppio NOT EXIST ---------
-indicare i pazienti visitati da tutti i medici
pensala come: trovare i pazienti per cui non esiste nemmeno un medico che non li ha visitati
```sql
SELECT P.CodFiscale -- 3
FROM Paziente P 
WHERE NOT EXISTS ( 
					SELECT * -- 2
					FROM Medico M 
					WHERE NOT EXISTS
					(
						SELECT * -- 1
						FROM Visita V
						WHERE V.Medico = M.Matricola
							AND V.Paziente = P.CodFiscale )
					);
```
Per ogni paziente verifichiamo se esiste anche un solo medico che *non* lo ha mai visitato. Se *non* esiste un tale medico, allora vuol dire che P è stato visitato da tutti i medici
- lettura dal basso verso l'alto:
1)Il medico M ha visitato il paziente P?
2)Esiste un medico che NON ha mai visitato P?
3)Non deve esistere nemmeno un medico che non ha visitato P

- lettura dall'alto verso il basso 
3)Stiamo scorrendo tutti i pazienti
2)il paziente P sarà incluso solo se NON esiste alcun medico M per cui non si trovi una visita con quel paziente
1)controlla se non esiste alcuna riga della tabella Visita dove il medico M ha visitato il paziente P
quindi se non troviamo nulla il medico M non ha mai visitato p --> paziente scartato
se trovo qualcosa --> M ha visitato P

il punto due scorre tutti i medici, il punto 1 verifica la visita e il punto 1 scorre tutti i pazienti

------- divisione con raggruppamento ---------
```sql
SELECT V.Paziente 
FROM Visita V 
HAVING COUNT(DISTINCT V.Medico) = (
									SELECT COUNT(*)
									FROM Medico
									);
```

COUNT(DISTINCT V.Medico) --> sono medici diversi che hanno visitato il paziente

------------
## Linguaggio procedurale
da qua in poi teoricamente non c'è nel primo appello
Questo tipo di programmazione si chiama BL SQL

**STORED PROCEDURE**
Sono programmi dichiarativo-procedurali memorizzati nel DBMS, che vengono invocati tramite chiamata e possono ricevere e/o restituire valori.
Grazie al loro utilizzo, il traffico sulla rete viene drasticamente ridotto.

Una **stored procedure** è un **blocco di codice SQL predefinito e memorizzato nel database** che puoi eseguire ogni volta che vuoi, con un semplice comando.
(è come una funzione in un linguaggio di programmazione).
```sql
DELIMITER //

CREATE PROCEDURE nome_procedura (IN param1 INT, OUT param2 INT)
BEGIN
    -- blocco di istruzioni SQL
    SELECT COUNT(*) INTO param2
    FROM tabella
    WHERE colonna = param1;
END //

DELIMITER ;

```

-scrivere una stored proedure che restituisca le specializzazioni mediche offerte dalla clinica
```sql
DROP PROCEDURE IF EXIST mostra_applicazioni;

DELIMITER $$

CREATE PROCEDURE mostra_specializzazioni()
	BEGIN
		SELECT DISTINCT Specializzazione
		FROM Medico;
	END $$

DELIMITER
```

per eseguire la chiamata e ottenere il risultato restituito dall'esecuzione del body
```sql
CALL mostra_specializzzazioni();
```

- possiamo avere delle *variabili locali* che vengono usate all'interno di una stored procedure, per memorizzare informazioni intermedie di ausilio.
sintassi delle variabili locali:
```sql
DECLARE nome_variabile tipo(size) DEFAULT valore_default;
```

---
#### **ASSEGNAMENTO**
- è possibile assegnare un valore a una variabile in due modalità :
	- istruzione SET  
	- oppure SELECT + INTO

**ASSEGNAMENTO STATICO CON SET**
supporre di essere nel body di una stored procedure e creare una variabile contenente il minimo numero di visite mensili da effettuare, impostato a 20
```sql
DECLARE min_visite_mensili INT DEFAULT 0;
.
.
.
SET min_visite_mensili = 20;
```

- creare una variabile contenete il numero di visite effettuate nel mese in corso
```sql
SET visite_mese_attuale =
	()
	SELECT COUNT(*) 
	FROM Visita V
	WHERE MONTH(V.Data) = MONTH(CURRENT_DATE)
		AND YEAR(V.Data) = YEAR(CURRENT_DATE)
	);
```
attenzione: una variabile non può contenere un result set

**ASSEGNAMENTO STATICO CON SELECT + INTO**
supporre di essere nel body di una stored procedure e creare una variabile contenente il numero di visite effettuate nel mese in corso
```sql
DECLARE visite_mese_attuale INT DEFAULT 0; 
.
.
SELECT COUNT(*) INTO visite_mese_attuale
FROM Visita V
WHERE MONTH(V.Data) = MONTH(CURRENT_DATE)
	AND YEAR(V.Data) = YEAR(CURRENT_DATE)
```
oppure
```sql
SELECT COUNT(*) INTO visite_mese_attuale
FROM Visita V
WHERE MONTH(V.Data) = MONTH(CURRENT_DATE)
	AND YEAR(V.Data) = YEAR(CURRENT_DATE)
INTO visite_mese_attuale
```

-----
#### **VARIABILI USER-DEFINED** @
Sono inizializzate dall'utente senza necessità di dichiarazione, e il loro ciclo di vita equivale alla durata della connection a MySQL server.
(sono scalari )

---
#### **PARAMETRI DI UNA STORED PROCEDURE**
Una stored procedure MySQL accetta parametri di tipo 
- ingresso (IN) (passaggio per copia, valore)
- uscita (OUT) (passaggio per copia, valore)
- ingresso-uscita (corrispondente delle cose passate per riferimento in cpp)

- scrivere una stored procedure che restituisca il numero di pazienti visitati da medici di una data specializzazione, ricevuta come parametro
```sql
DROP PROCEDURE IF EXISTS tot_pazienti_visitati_spec; 

DELIMITER $$
CREATE PROCEDURE tot_pazienti_visitati_spec(
						IN _specializzazione VARCHAR(100), --1
						OUT totale_pazienti_ INT) 
	BEGIN
		SELECT COUNT(DISTINCT V.Paziente) INTO totale_pazienti_ 
		FROM Visita V
			INNER JOIN
			Medico M ON V.Medico = M.Matricola 
		WHERE M.Specializzazione = _specializzazione;
	END $$ 
DELIMITER ;

CALL tot_pazienti_visitati_spec('Neurologia', @quantiPazienti); 

SELECT @quantiPazienti;
```
1)stringa variabile al massimo di 100 caratteri

> per convenzione scriviamo le variabili in ingresso _ nome e le variabili in uscita con nome_ 

---
#### **ISTRUZIONI CONDIZIONALI**
Le istruzioni condizionali permettono di esprimere condizioni, modificando il flusso di esecuzione
1. ISTRUZIONE IF
```sql
IF if_condition THEN
	-- blocco istruzioni if true

ELSEIF elseif_1_condition THEN
	-- blocco istruzioni elseif_1
.
.
.

ELSEIF elseif_N_condition THEN
	-- blocco istruzioni elseif_N

ELSE
	-- blocco istruzioni else

END IF ;
```
gli else/else if sono facoltativi

- scrivere una stored procedure che riceva come parametro un intero t e una specializzazione s e restituisca in uscita true se il numero di visite della specializzazione s nel mese in corso è superiore a t, false se è inferiore, e NULL se è uguale 
```sql
DROP PROCEDURE IF EXISTS visite_sopra_soglia;

DELIMITER $$
CREATE PROCEDURE visite_sopra_soglia(IN _t INT, IN _s VARCHAR(100), OUT passed BOOLEAN)
BEGIN
    DECLARE visite_mese_attuale INT DEFAULT 0;
    SET visite_mese_attuale = (
				        SELECT COUNT(*)
				        FROM Visita V
					        INNER JOIN 
					        Medico M ON V.Medico = M.Matricola
				        WHERE M.Specializzazione = _s
				          AND MONTH(V.`Data`) = MONTH(CURRENT_DATE)
				          AND YEAR(V.`Data`) = YEAR(CURRENT_DATE)
					    );

    IF visite_mese_attuale > _t THEN
        SET passed = TRUE;
    ELSEIF visite_mese_attuale < _t THEN
        SET passed = FALSE;
    ELSE
        SET passed = NULL;
    END IF;
END $$

DELIMITER ;

-- Esempio di chiamata
CALL visite_sopra_soglia(10, 'Otorinolaringoiatria', @controllo);

```

2. ISTRUZIONE CASE (equivale all'istruzione 'swith' del C++)
```sql
CASE
WHEN condition_1 THEN
	-- blocco istruzioni_1
.
.

WHEN condition_N THEN
	-- blocco istruzioni_N

END CASE ;
```

- Scrivere una stored procedure che restituisca la data in cui un paziente, il cui codice fiscale è passato come parametro, è stato visitato per la prima volta, e il nome e cognome del medico che lo ha visitato in tale circostanza. In caso di più medici, per semplicità, selezionarne uno.
```sql
DELIMITER $$
CREATE PROCEDURE primaVisitaPaziente(IN paziente VARCHAR(16),
								    OUT dataPrimaVisita DATE,
								    OUT cognomeMedico VARCHAR(100),
								    OUT nomeMedico VARCHAR(100)
									)
BEGIN
    SELECT MIN(V.`Data`) INTO dataPrimaVisita
    FROM Visita V
    WHERE V.Paziente = paziente;

    SELECT M.Cognome, M.Nome INTO cognomeMedico, nomeMedico
    FROM Visita V
	    INNER JOIN 
	    Medico M ON V.Medico = M.Matricola
    WHERE V.Paziente = paziente
      AND V.`Data` = dataPrimaVisita
    LIMIT 1;
END $$
DELIMITER ;
```

---
#### **ISTRUZIONI ITERATIVE**
Le istruzioni iterative permettono di ripetere blocchi di codice, dipendentemente dalla veridicità di una condizione
1. WHILE
```sql
WHILE condition DO
	-- blocco istruzioni
END WHILE
```
La condizione è controllata prima di ogni iterazione (si esegue finchè è true)

2. REPEAT
```sql
REPEAT
	-- blocco istruzioni
UNTIL condition 
END REPEAT;
```
UNTIL contiene una condizione d'uscita
la condizione è controllata DOPO ogni iterazione
3. LOOP
```sql
loop_label: LOOP
	-- blocco istruzioni e check di condizioni
END LOOP;
```
le condizioni d'uscita sono gestite dal programmatore

---
#### **ISTRUZIONI DI SALTO**
Le istruzioni di salto permettono, rispettivamente, di interrompere un ciclo e passare all'iterazione successiva. Le istruzioni di salto sono:
- LEAVE (controparte del break in cpp)
- ITERATE (controparte del continue in cpp)

- scrivere una stored procedure che riceve in ingresso un intero i e stampa a video i numeri dispari da 1 a i, separati da virgola
```sql
SELECT DROP PROCEDURE IF EXIST stampa_dispari;
DELIMITER $$
CREATE PROCEDURE stampa_dispari (IN I INT) -- prende in ingresso un intero chiamato i
BEGIN 
	DECLARE s VACHER(225) DEFAULT ''; -- dichiara la stringa s = ''
	DECLARE counter INT DEFAULT 1; -- dichiara contatore = 1

	IF i>0 THEN
		SET s= '1';
	END IF;

	scan: LOOP
		SET counter = counter +1;

		IF counter = i+1 THEN
			ITERATE scan;
		END IF;

		IF (counter%2) = 0 THEN
			ITERATE scan;
		ELSE
			SET s =CONCAT(s, ',', counter);
		END IF;
	END LOOP;

	SELECT s;
END $$
DELIMITER;
```

il LEAVE dalla riga 16 va alla 25.
riga 12-24 abbiamo il loop 

---
#### **CURSORI**
Puntano a un result set, e permettono di scorrerlo procedendo in avanti, per effettuare delle azioni all'interno di istruzioni iterative.
Dichiarazione: 
```sql
DECLERE NomeCursore CURSOR FOR
SQL query;
```
i cursori vanno dichiarati immediatamente dopo la dichiarazione di tutte le variabili.
La vita del cursore è divisa in tre fasi:
-OPEN
-FETCH
-CLOSE
Infatti per usare un cursore lo si deve aprire, poi è possibile effettuare il prelievo riga per riga, infine lo si deve chiudere.
```sql
OPEN NomeCursore; -- apertura

FETCH NomeCursore INTO ListaVariabili -- prelievo
									--   sempre inserito in un ciclo
CLOSE NomeCursore -- chiusura
```

---
#### **HANDLER**
Sono gestori di situazioni, utili (tra l'altro) quando si usano i cursori per riconoscere la fine del result set.

Definito subito dopo la definizione di variabili e cursori.
```sql
DECLARE CONTINUE HANDLER FOR NON FOUND
SET finito = 0;
```

Pensiamolo come un orecchio che sente quando ci schiantiamo sulla fine di un fetch --> quando 'finito' da 0 diventa 1 vuol dire che abbiamo terminato il fetch, la roba da prelevare.
È quindi tipicamente utilizzato per gestire il fine corsa nella scansione di un result set effettuata all'interno di un ciclo dove si scorre un cursore. 

- Scrivere una stored procedure che riceve in ingresso una specializzazione s e restituisca i codici fiscali dei pazienti visitati da un solo medico di s, in una stringa del tipo “codFiscale1, ... , codFiscaleN”

```sql
DELIMITER $$
CREATE PROCEDURE PazientiSingoloMedico(IN specializzazione VARCHAR(50),
   OUT codiciFiscali VARCHAR (255)) -- in ingresso la specializzazione, in uscita la stringa dei codici fiscali -- il prf ha dichiarato specializzazione come CHAR, è un errore, deve essere VARCHAR
BEGIN
   DECLARE finito INTEGER DEFAULT 0;
   DECLARE codiceFiscale VARCHAR(255) DEFAULT '';

   -- dichiarazione del cursore
   DECLARE cursoreCodici CURSOR FOR
   SELECT V.Paziente
   FROM Visita V INNER JOIN Medico M ON V.Medico = M.Matricola
   WHERE M.Specializzazione = specializzazione
   GROUP BY V.Paziente
   HAVING COUNT(DISTINCT V.Medico) = 1;

   -- dichiarazione handler
   DECLARE CONTINUE HANDLER
   FOR NOT FOUND SET finito = 1;

   OPEN cursoreCodici;

   -- ciclo di fetch per il prelievo
   preleva: LOOP -- facciamo un inserimento in testa
   FETCH cursoreCodici INTO codiceFiscale;
   IF finito = 1 THEN
   LEAVE preleva;
   END IF;
   SET codiciFiscali = CONCAT(codiceFiscale, ';', codiciFiscali);
   END LOOP preleva;
   CLOSE cursoreCodici;
END $$
DELIMITER ;
```
abbiamo usato il linguaggio procedurale perché ci serve una concatenazione 

per eseguire la chiamata di questa procedura
```sql
SET @codiciPazienti = '';
CALL PazientiSingoloMedico('Ortopedia', @codiciPazienti); 
SELECT @codiciPazienti;
```

---
## **Data Manipulation**
Capiamo come si aggiornano i dati nel database.

La Data Manipulation è parte del linguaggio che permette di inserire, cancellare e aggiornare interi record o valori di attributi delle tabelle di un database.
comandi: INSERT - DELETE -UPDATE
1. INSERT 
Inserire nel database un nuovo paziente, le cui informazioni sono:
- Nome: Elvira
- Cognome: Passerotti
- Sesso: F
- Data di nascita: 27 Ottobre 1965
- Città: Pisa
- Reddito: 1500 €
- Codice fiscale: PSSLVR65R67G702U
```sql
INSERT INTO Paziente
VALUES (‘PSSLVR65R67G702U’, ‘Passerotti’, ‘Elvira’, ‘F’,‘1965-10-27’, ‘Pisa’, 1500);
```
2. UPDATE
Modificare in “mutuata” tutte le visite del mese corrente, effettuate da pazienti nati prima del 1925
```sql
UPDATE Visita
SET Mutuata = 1
WHERE MONTH(Data) = MONTH(CURRENT_DATE) 
	AND YEAR(Data) = YEAR(CURRENT_DATE) 
	AND Paziente IN (SELECT CodFiscale
					FROM Paziente
					WHERE YEAR(DataNascita) < 1925);
```
La formula è UPDATE - SET - WHERE
3. DELETE
Il dottor Ettore Grigi ha lasciato la clinica. Rimuovere tutte le sue visite dal database, supponendo che il medico non avesse omonimi nella clinica.
```sql
DELETE FROM Visita
WHERE Medico = (SELECT Matricola FROM Medico
				WHERE Nome = ‘Ettore’ 
					AND Cognome = ‘Grigi’);
```

occhio al **maligno** non mettere nel WHERE la tabella sulla quale vuoi fare le modifiche --> al mattino, pane e volpe! --> usiamo le derived table

## **Data analytics**
sono chiamate anche Window functions: affiancano a ogni record r un valore ottenuto da un'operazione (conteggio, media..) eseguita su un insieme di record logicamente connessi a r.
- Scrivere una query che indichi, per ogni cardiologo, la matricola, la parcella e la parcella media della sua specializzazione
se dovessi farlo senza la conoscenza dei data analytics sarebbe un bordello perchè per trovare la parcella media dovrei fare un raggruppamento e quindi perdere i dati dei medici
Utilizziamo la **clausola OVER**, essa applica a una funzione di tipo aggregate (sum, avg, count, max, min) o non-aggregate a un insieme di record (detto 'partition') associati a un record di un result set
```sql
SELECT M.Matricola, M.Parcella, **AVG(Parcella) OVER()**
FROM Medico M
WHERE M.Specializzazione = 'Cardiologia';
```

se OVER() non ha contenuto, la partition è sempre la stessa per ogni record, e coincide con il risult set della query senza la parte in grassetto
detto da chat: se OVER() è vuoto, operazione viene eseguita su tutte le righe che sono rimaste dopo il WHERE --> in questo caso fa la media tra tutte le parcelle rimaste ovvero tra tutte le parcelle dei cardiologi, top è quello che volevamo

LA **PARTITION** è l'insieme di record a cui si applica una funzione aggregate/non-aggregate e dipende dal record processato 
- Scrivere una query che indichi, per ogni medico, la matricola, la specializzazione, la parcella, e la parcella media della sua specializzazione
```sql
SELECT M.Matricola, M.Specializzazione, AVG(M.Parcella) OVER(
												PARTITION BY M.Specializzazione
														)
FROM Medico M;
```
la partition by partiziona la tabella per specializzazione e per ogni specializzazione calcola la parcella media --> quindi AVG() è applicato partizionando i record del result set della query per specializzazione
Il PARTITION BY crea gruppi logici (come un GROUP BY, ma senza ridurre le righe).

---
Non-aggregate functions utilizzabili con OVER:
associano a ciascun record di un result set un valore ottenuto dalla partiton _senza fondere i record_ in un'informazione riepilogativa

**FUNZIONE ROW_NUMBER**
- assegnare un numero a ogni medico nella sua specializzazione
```sql
SELECT M.Matricola, 
		M.Cognome, 
		M.Specializzazione, 
		ROW_NUMBER() OVER(PARTITION BY M.Specializzazione)
FROM Medico M;
```
la **ROW_NUMBER** assegna un numero progressivo a ogni riga all’interno della partizione.

---
**FUNZIONE RANK**
Serve per stilare una classifica dipendentemente da un criterio.
Va usata con ordinamento --> ORDER BY
- Classificare i medici in base alla loro convenienza. Restituire matricola, cognome, specializzazione, parcella e posizione in classifica.
(parcelle basse hanno maggior convenienza, quindi rank migliori)
```sql
SELECT M.Matricola, M.Cognome, M.Specializzazione, M.Parcella,
		RANK() OVER(ORDER BY M.Parcella)
FROM Medico M;
```

> se ho due posizioni a parimerito poi avrò un salto di posizione 

- Effettuare una classifica dei medici di ogni specializzazione dipendentemente dalla loro parcella, partendo dalla più alta. Restituire matricola, cognome, specializzazione, parcella e posizione in classifica.
```sql
SELECT M.Matricola, M.Cognome, M.Specializzazione,
		RANK() OVER( PARTITION BY M.Specializzazione ORDER BY M.Parcella DESC)
FROM Medico M;
```
Qua è necessario partizionare perchè la classifica delle parcelle è in base alla specializzazione.
Ordinamento è decrescente --> **DESC** 


volendo c'è una sintassi alternativa con WINDOW
```sql
SELECT M.Matricola, M.Cognome, M.Specializzazione,
		RANK() OVER w
FROM Medico M;

WINDOWS w AS ( PARTITION BY M.Specializzazione ORDER BY M.Parcella DESC)
```

**FUNZIONE DENSE_RANK**
Funziona come la RANK() ma nel caso di parimerito poi non salta le posizioni 

**RANK MULTIPLI**
- Stilare una classifica dei medici in base al numero di visite effettuate. Restituire cognome, specializzazione, numero di viste effettuate, posizione nella classifica generale, e posizione nella classifica per specializzazione.
```sql
WITH visite AS(
	SELECT V.Medico, M.Cognome, M.Specializzazione, COUNT(*) AS Visite
	FROM Visita V
		INNER JOIN
		Medico M ON V.Medico = M.Matricola
	GROUP BY V.Medico
) -- abbiamo usato una CTE

SELECT VV.Cognome, 
		VV.Specializzazione,
		VV.Visite,
		RANK() OVER(ORDER BY VV.Visite DESC) AS GlobalRank,
		RANK() OVER(PARTITION BY VV.Specializzazione ORDER BY VV.Visite DESC) AS SpecRank
FROM visite VV;
```

---
**FUNZIONI LEAD E LAG**
Ricavano il valore di un attributo di una row posta k posizioni prima (lag) o dopo (lead) la current row
-la LAG permette di affiancare a ogni record 'i' il valore di un record 'j' posizionato k posizioni prima del record 'i' all'interno della partition associata a i --> fa un balzo indietro di k rows, il valore di k deve essere passato come secondo argomento della funzione, mentre il primo argomento è il nome dell'attributo del record 'j' da restituire in uscita

- Considerare le visite otorinolaringoiatriche dal 2010 al 2019, restituire, per ciascuna, matricola del medico, codice fiscale del paziente, data, e data della visita precedente del paziente con un medico della stessa specializzazione
```sql
SELECT V.Medico, V.Paziente, V.Data, 
		LAG(V.Data, 1) OVER (PARTITION BY V.Paziente ORDER BY V.Data)
		-- LAG(V.'Data', 1) vuol dire he k=1
FROM Visita V
	INNER JOIN
	Medico M ON V.Medico = M.Matricola
WHERE M.Specializzazione = 'Otorinolaringoiatria' AND YEAR(V.Data) BETWEEN 2010 AND 2019;
```

-La funzione LEAD permette di affiancare a ogni record 'i' il valore di un record 'j' posizionato k posizioni dopo il record i all'interno della partition a i --> fa un balzo in avanti







```sql
SELECT 
FROM 
WHERE ;
```






