# DataLayer sugli eventi di Acquisto
Per il tracciamento dei dati e  commerce nel nuovo sito di GardaTours, avremo bisogno di implementare alcuni eventi e  commerce all'interno dell'oggetto dataLayer. 

Alcuni eventi sono già stati implementati ed il processo è abbastanza simile, quindi non dovremmo avere particolari difficoltà nel farlo. 

## DataLayer Universal Analytics
Universal Analytics è un pelino più rognoso di GA4 e necessita di alcune accortezze di implementazione. Lo strumento va infatti a leggersi in autonomia le informazioni contenute all'interno del DataLayer e si aspetta di trovarle formattate in una certa maniera. 

[A questo link](https://developers.google.com/analytics/devguides/collection/ua/gtm/enhanced  ecommerce) è possibile consultare la documentazione ufficiale degli sviluppatori per Google Analytics. Al suo interno sono contenute tutte le informazioni necessarie per la formattazione. 

## DataLayer GA4
Il dataLayer di GA4 ha regole meno stringerti, in quanto le informazioni contenute al suo interno servono solo come strumento di passaggio per l'inoltro di eventi e parametri. Saremo quindi più liberi nella sua implementazione. 

## Come trattare i servizi
I vari servizi aggiuntivi come le bottigliette d'acqua, i cappellini o le partene da porti differenti, vanno trattati come singoli oggetti e  commerce, in modo da poter visualizzare le informazioni principali in modo separato. 

## Ordine di Priorità degli eventi
Essendo stretti sulle tempistiche, daremo priorità agli eventi di acquisto (Purchase su tutti) rispetto agli eventi più standard. In questo modo potremo almeno raccogliere informazioni sulle effettive prenotazioni. 

L'ordine di implementazione consigliato è più o meno il seguente:
1. Purchase
2. Contatto

## DataLayer Custom
Insieme ai dataLayer standard, implementeremo anche alcuni eventi custom che utilizzeremo per la raccolta di informazioni varie da inoltrare ad eventi personalizzati (com'è già stato fatto con la visualizzazione dei Tour). Stiamo infatti piegando le logiche di funzionamento di un e  commerce ad un sistema di Booking Engine. Una volta che le informazioni saranno presenti all'interno del DataLayer, riusciremo abbastanza agevolmente a gestire i dati raccolti con GTM. 

## Esempi implementazione DataLayer
In questo paragrafo andrò ad inserire alcuni DataLayer di esempio con le informazioni a nostra disposizione. Cercherò di inserire quanti più dati possibili. Naturalmente tutte le informazioni dinamiche dovranno essere gestite di conseguenza. Per ogni evento, proporrò 3 dataLayer diversi:
1. Quello di Universal Analytics
2. Quello di GA4
3. Quello custom che utilizzeremo per raccogliere informazioni accessorie

### Purchase
L’evento di acquisto del Tour. Deve scattare dopo l’effettivo pagamento del Tour o dell’evento. Se presente, è possibile farlo scattare all’interno di Thank You Page o su di una pagina di passaggio (anche se preferisco la prima soluzione). 

Per quanto riguarda Universal è importante che tutte le informazioni rispettino gli standard. Per i due eventi successivi è invece possibile essere più elastici nella loro gestione, in quanto saremo noi ad avere il controllo sulle informazioni inviate. 

#### Purchase    Universal Analytics 
Il dataLayer di acquisto ha una struttura molto semplice e immediata. L’oggetto **ecommerce** è quello che contiene tutti gli oggetti relativi all’evento. 

L’oggetto **Purchase** contiene invece informazioni sulla transazione, in particolare l’ID, che dovrebbe essere univoco. Rispetto al dataLayer standard ho aggiunto alcune informazioni accessrie:
* **event**: parametro contenente il nome dell’evento che vedremo nel dataLayer.
* **“userid”:** parametro contente l’id dell’utente. Può essere utilizzato qualsiasi valore univoco che identifichi sempre l’utente, ogni qualvolta questo risulti longato. In fase di acquisto dovremmo poter avere quei dati sull’utente. 

**NOTA:** nel caso dei servizi, la variabile “Price” fa riferimento al valore dell’oggetto. Quindi in caso di più bottigliette d’acqua non va indicato il totale ma il costo della singola bottiglietta d’acqua. 

```JS
dataLayer.push({ ecommerce: null });// Clear the previous ecommerce object.
dataLayer.push({
'event':'purchase  ua',
'user_id':'UTENTEN1',
'ecommerce': {
'purchase': {
'actionField': {
'id': 'T12345', //ID Univoco della transazione
'revenue': '35.43', // Total transaction value (incl. tax and shipping)
'coupon': 'SUMMER_SALE' //Da popolare solo nel caso in cui sia stato effettivamente utilizzato un coupon
},
'products': [{ //Array di oggetti contentente tutti i servizi, compreso il tour
'name': 'Penisola di Sirmione', // Name or ID is required.
'id': '12345', //ID Del Tour
'price': '70€',
'quantity': 1,
'coupon': ''// Optional fields may be omitted or set to empty string.
 },
 {
'name': "Bottiglietta d'acqua",
'id': '67890',
'price': '1',
'quantity': 3
 },
	 {
'name': 'Cappellini',
'id': '67890',
'price': '9.99',
'quantity': 2
 }]
}
}
});
```

#### Purchase    GA4
Rispetto al dataLayer di Universal, quello di GA4 è molto più semplice in quanto la sua univa funzione è traghettare informazioni dal sito web al container di GTM. Nonostante ciò, avere le informazioni ordinate ci agevola in fase di produzione dei Tag. 
```JS
dataLayer.push({
event: "purchase",
ecommerce: {
transaction_id: "T_12345",
value: 36.32,
currency: "EUR",
coupon: "SUMMER_SALE",
items: [
 {
item_id: "43432432432",
item_name: "Tour di SIrmione",
coupon: "SUMMER_FUN",
currency: "EUR",
item_list_name: "Tour",
price: 9.99,
quantity: 1
},
{
item_id: "SKU_12346",
item_name: "Acqua",
coupon: "SUMMER_FUN",
currency: "EUR",
price: 1.00,
promotion_id: "P_12345",
promotion_name: "Summer Sale",
quantity: 1
}]
}
});
```

#### Purchase    Custom
Non prevedo l’utilizzo di Eventi Custom per quanto riguarda il Purchase, in quanto momentaneamente dovrebbero bastarci le informazioni contenute all’interno del dataLayer e  commerce di GA4 e Universal. Al più procederei all’inoltro di un semplice evento di backup, come da esempio: 

```JS
dataLayer.push({event: "acquisto_avvenuto"
})
```

### Evento di Contatto
Oltre a poter procedere con l’evento di acquisto, l’utente ha la possibilità di contattare il cliente tramite modulo presente sulla pagina contact. In questo caso sarà sufficiente un dataLayer custom contenete alcune informazioni di contatto. 

```JS
dataLayer.push({
	'event':'contatto',
 	'prenotazione':'notelliggio, tour privato, ecc...', // le opzioni presenti
	'Nome':'Enrico',
	'Email':'chioloenrico@gmail.com'
}
```

Le informazioni sugli utenti sono essenziali per fare in modo di inoltrare enhanced conversion alle piattaforme. 

### Altri Eventi
Oltre agli eventi di acquisto e di contatto, sarebbero presenti molti altri touchpoint che bisognerebbe tracciare. Tuttavia, per agevolare le tempistiche per tutti quelli eventi, inoltrerei semplicemente un parametro ‘event’:’nomeEvento’, senza ulteriori specifiche. 

I touchpoint principali sono: 
1. Aggiunta al carrello
2. Checkout

Per I quali rispettivamente inverai i seguenti eventi: 
```JS
dataLayer.push({
	'event':'ATC',
 	'Prodotto':'Tour Privato, Programmato, Noleggio, ecc...',
}
```
```JS
dataLayer.push({
	'event':'InizioCheckout',
 	'Prodotto':'Tour Privato, Programmato, Noleggio, ecc...',
}
```