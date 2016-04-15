# 1. Feasibility API
### Hämta alla levererbara platser

***Request:***

```http
GET /api/1.0/feasibility HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Sun, 10 Jan 2016 12:03:28 GMT
Content-Type: application/json

[
	{
		"pointId": "ABC123",
		"address": {
			"street": "Testvägen",
			"number": "100",
			"littera": "",
			"postalCode": "10000",
			"city": "Ankeborg",
			"countryCode": "SE"
		},
		"building": {
			"distinguisher": "", // "set if something is needed to distinguish a specific building on the address"
			"type": "MDU", // "'MDU' = apartment building, 'SDU' = villa"
		},
		"realEstate": {
			"label": "PENGABINGEN 1",
			"municipality": "ANKEBORG",
		},
		"coordinate": {
			"latitude": 6581619.085,
			"longitude": 1628539.32,
			"projection": "WGS84"
		},
		"district": "GAMLA STAN"
	},
	...
]
```

## Begränsningsmekanism

För att bara få inkrementella uppdateringar av adresser så används If-Modified-Since. På det viset kan anropet ske ofta men fortfarande vara billigt.

Vid första anropet sker ingen begränsning, utan då frågar man om det fullständiga resultatet. Vid påföljande anrop används If-Modified-Since. Värdet för headern är föregående svars värde på Last-Modified.

Av den anledningen är "Last-Modified" obligatoriskt vid HTTP Status 200.
Begränsningsmekanism - Exempel

Exempel på anropssekvens:

***Request:***

```http
GET /api/1.0/feasibility HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Mon, 11 Jan 2016 12:03:28 GMT
Content-Type: application/json

...
```

Vid påföljande anrop skickas en "If-Modified-Since"-header för att bara be om uppdaterade poster.

***Request:***

```http
GET /api/1.0/feasibility HTTP/1.1
If-Modified-Since: Mon, 11 Jan 2016 12:03:28 GMT

...
```

Om det finns uppdaterade poster kan svaret se ut så här:

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Tue, 12 Jan 2016 09:54:55 GMT
Content-Type: application/json

...
```

Om det inte finns uppdaterade poster ser svaret istället ut så här:

***Response:***

```http
HTTP/1.1 304 Not Modified
```

# 2. Availability API
### Hämta detaljerad information och leverantörers fiberstatus på en specifik plats (adress)

***Request:***

Sökning via adress

```http
GET /api/1.0/availability?city={city}&street_name={streetName}&street_mumber={streetNumber}&street_littera={streetLittera} HTTP/1.1
```

eller sökning på punkt-id

```http
GET /api/1.0/availability?pointId={pointId} HTTP/1.1
```

eller sökning på en koordinat som returnerar närmaste kända plats

```http
GET /api/1.0/availability?xCoordinate={xCoordinate}&yCoordinate={yCoordinate} HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
	"pointId": "ABC123",
	"address": {
		"street": "Testvägen",
		"number": "100",
		"littera": "",
		"postalCode": "10000",
		"city": "Ankeborg",
		"countryCode": "SE"
	},
	"building": {
		"distinguisher": "", // "set if something is needed to distinguish a specific building on the address"
		"type": "MDU", // "'MDU' = apartment building, 'SDU' = villa"
	},
	"realEstate": {
		"label": "PENGABINGEN 1",
		"municipality": "ANKEBORG",
	},
	"coordinate": {
		"latitude": 6581619.085,
		"longitude": 1628539.32,
		"projection": "WGS84"
	},
	"district": "GAMLA STAN",
	"suppliers": [
		{
			"name": "STOKAB",
			"fiberStatus": "IN_REAL_ESTATE", // "'AT_ADDRESS', 'AT_SITE_BOUNDARY'"
			"statusValidationRequired": true // "indicates if the fiberStatus needs manual validation to assure availability"
		},
		...
	],
 	"relatedPointIds": [ // "list of other nearby points (addresses) which could be used instead of the searched point (address)"
		"CDE678",
		"CDE901"
	]
}
```

# 3. Inquiry API
### Skicka en förfrågan på en förbindelse

***Request:***

```http
POST /api/1.0/inquiry HTTP/1.1
Content-Type: application/json

{
	"from": {
		"pointId": "ABC123",
		"comment": "", // "if an additional comment for the from point could be useful for the supplier"
	},
	"to": { // "may be set to null if any product only requires one point (address)"
		"pointId": "ABC789", 
		"comment": "", // "if an additional comment for the to point could be useful for the supplier"
	},
	"redundancy": { // "may be set to null if no redundancy is wanted"
		"type": "Full", // "'Normal', 'Full'"
		"toPointId": "CBA123"
	},
	"customerType": "Commercial", // "e.g. 'Commercial', 'Residential'"
	"serviceLevel": "Premium", // "e.g. 'Base', 'Gold', 'Premium'"
	"product": "All", // "e.g. 'All', 'Point2Point', 'Star'"
	"parameters": [
		{
			"name": "ConnectorType",
			"value": "SC/APC"
		},
		...
	],
	"subProducts": [
		{
			"name": "ResidentialNetwork",
			"parameters": [
				{
					"name": "NoOfRooms",
					"value": "10"
				},
				...
			]
		},
		...
	],
	"contractPeriodMonths": 12,
	"noOfFibers": 1, // "number of wanted fiber pairs (or single fibers depending on product)"
	"asyncAnswerAllowed": true // "if asychronous answer is ok (might result in an extra charge if manual)"
}
```

***Response:***

```http
HTTP/1.1 201 CREATED
Last-Modified: Mon, 11 Jan 2015 12:03:28 GMT
Location: /api/1.0/inquiry/ec4bc754-6a30-11e2-a585-4fc569183061
Content-Type: application/json

{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"state": "WAIT_ASYNC_ANSWER", // "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'"
	"message": "",
	"offers": [
		{
			"supplier": "STOKAB",
			"offerValidUntilDate": "2016-01-31",
			"connectionId": "", // "may be set to the identifier for the connection if that is already generated when inquiry is answered"
			"deliveryDurationDays": 20, // "days from order to delivered connection"
			"products": [
				{
					"name": "Point2Point",
					"status": "AVAILABLE",
					"comment": "",
					"items": [
						{
							"name": "distance",
							"value": "987"
						},
						...
					],
					"price": {
						"status": "ESTIMATED", // "'ESTIMATED', 'OFFER'. Estimated price can be delivered in synchronous answer and then be overridden by an offer in an asynchronous answer"
						"oneTimeFee": 15100.0,
						"monthlyFee": 1200.0,
						"items": [
							{
								"name": "Connection based on distance",
								"parameters": [
									{
										"name": "distance",
										"value": "987"
									},
									...
								],
								"oneTimeFee": 5000.0,
								"monthlyFee": 1000.0
							},
							{
								"name": "Service level Premium",
								"parameters": [...],
								"oneTimeFee": 0.0,
								"monthlyFee": 200.0
							},
							{
								"name": "Connector",
								"parameters": [...],
								"oneTimeFee": 100.0,
								"monthlyFee": 0.0
							},
							{
								"name": "ResidentialNetwork",
								"parameters": [
									{
										"name": "NoOfRooms",
										"value": "10"
									}
								],
								"oneTimeFee": 10000.0,
								"monthlyFee": 0.0
							},
							...
						]
					}
				},
				{
					"name": "Star",
					"status": "POSSIBLE_WITH_CONDITIONS", // "'NOT_AVAILABLE', 'AVAILABLE'"
					"comment": "Connection to anslutningsnod is necessary for address to be available",
					"items": [],
					"price": null
				},
				...
			]
		},
		...
	]
}
```

### Hämta uppdatering på en skickad förfrågan

***Request:***

```http
GET /api/1.0/inquiry/ec4bc754-6a30-11e2-a585-4fc569183061 HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Mon, 11 Jan 2015 12:03:28 GMT
Content-Type: application/json

{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"state": "DONE_ASYNC_ANSWER_SUCCESS",
	"message": "",
	"offers": [
		{
			"supplier": "STOKAB",
			"offerValidUntilDate": "2016-01-31",
			"connectionId": "",
			"deliveryDurationDays": 20,
			"products": [
				{
					"name": "Point2Point",
					"status": "AVAILABLE",
					"comment": "",
					"items": [
						{
							"name": "distance",
							"value": "1046"
						},
						...
					],
					"price": {
						"status": "OFFER",
						"oneTimeFee": 15100.0,
						"monthlyFee": 1500.0,
						"items": [
							{
								"name": "Connection based on distance",
								"parameters": [
									{
										"name": "distance",
										"value": "1046"
									},
									...
								],
								"oneTimeFee": 5000.0,
								"monthlyFee": 1300.0
							},
							{
								"name": "Service level Premium",
								"parameters": [...],
								"oneTimeFee": 0.0,
								"monthlyFee": 200.0
							},
							{
								"name": "Connector",
								"parameters": [...],
								"oneTimeFee": 100.0,
								"monthlyFee": 0.0
							},
							{
								"name": "ResidentialNetwork",
								"parameters": [
									{
										"name": "NoOfRooms",
										"value": "10"
									}
								],
								"oneTimeFee": 10000.0,
								"monthlyFee": 0.0
							},
							...
						]
					}
				},
				{
					"name": "Star",
					"status": "NOT_AVAILABLE",
					"comment": "",
					"items": [],
					"price": null
				},
				...
			]
		},
		...
	]
}
```

### Hämta slutförda förfrågningar som krävde asynkront svar

Anropet skall returnera samtliga förfrågningar som skett sedan since. Förfrågan som since avser skall inte ingå i svaret.

Listan är oföränderlig. Vid två tidpunkter, t1 och t2, där t2 inträffar efter t1, skall tjänstens svar vid t2 alltid omfatta hela t1. Svaret kan alltså enbart växa, inte förändras.

Av denna anledningen är det viktigt att tjänstens svar inte är en projektion på förfrågningar, som kan ändra status, utan endast innefattar sådana förfrågningar som har nått en slutstatus.

Listan över förfråningar skall vara stigande i kronologisk ordning (senaste förfrågan som har förändrats sist). Klienten kan då använda det sista lästa förfrågan som since-parameter i nästföljande anrop.

Enbart förfrågningar som har status DONE_ASYNC_ANSWER_SUCCESS eller DONE_ASYNC_ANSWER_FAILED får förekomma i svaret.

Om inte since skickas med så skall alla slutförda förfrågningar returneras.

För att hämta detaljerad information om respektive förfrågan, använd Hämta uppdatering på en skickad förfrågan.

***Request:***

```http
GET /api/1.0/inquiries/?since=ca4bc754-6a30-11e2-a585-4fc569183569
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
	{
		"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
		"state": "DONE_ASYNC_ANSWER_SUCCESS",
		"message": ""
	},
	{
		"inquiryId": "fc4bc754-6a30-11e2-a585-4fc569183432",
		"state": "DONE_ASYNC_ANSWER_FAILED",
		"message": "An error occured"
	},
	...
]
```

# 4. Order API
### Lägg en beställning på en tidigare gjord förfrågan

***Request:***

```http
POST /api/1.0/order HTTP/1.1
Content-Type: application/json

{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"offer": {
		"supplier": "STOKAB",
		"product": "Point2Point"
	}
}
```

***Response:***

```http
HTTP/1.1 201 CREATED
Last-Modified: Mon, 11 Jan 2015 12:05:28 GMT
Location: /api/1.0/order/fc6cd754-6a30-11e2-a585-4fc569185689
Content-Type: application/json

{
	"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"offer": {
		"supplier": "STOKAB",
		"product": "Point2Point"
	},
	"state": "ORDERED",
	"message": ""
}
```

### Hämta uppdatering på en lagd order

***Request:***

```http
GET /api/1.0/order/fc6cd754-6a30-11e2-a585-4fc569185689 HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Tue, 15 Feb 2015 15:12:53 GMT
Content-Type: application/json

{
	"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"offer": {
		"supplier": "STOKAB",
		"product": "Point2Point"
	},
	"state": "DELIVERED", // "ORDERED", "DELIVERED", "REJECTED"
	"message": "",
	"orderDate": "2015-01-11",
	"deliveryDate": "2015-02-15"
}
```

### Hämta slutförda ordrar

Anropet skall returnera samtliga ordrar som skett sedan since. Order som since avser skall inte ingå i svaret.

Listan är oföränderlig. Vid två tidpunkter, t1 och t2, där t2 inträffar efter t1, skall tjänstens svar vid t2 alltid omfatta hela t1. Svaret kan alltså enbart växa, inte förändras.

Av denna anledningen är det viktigt att tjänstens svar inte är en projektion på ordrar, som kan ändra status, utan endast innefattar sådana ordrar som har nått en slutstatus.

Listan över ordrar skall vara stigande i kronologisk ordning (senaste order som har förändrats sist). Klienten kan då använda det sista lästa ordern som since-parameter i nästföljande anrop.

Enbart ordrar som har status DELIVERED eller REJECTED får förekomma i svaret.

Om inte since skickas med så skall alla slutförda ordrar returneras.

För att hämta detaljerad information om respektive order, använd Hämta uppdatering på en lagd order.

***Request:***

```http
GET /api/1.0/orders/?since=fc6cd754-6a30-11e2-a585-4fc569185689
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
	{
		"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
		"state": "DELIVERED",
		"message": ""
	},
	{
		"orderId": "ab6cd754-6a30-11e2-a585-4fc569185643",
		"state": "REJECTED",
		"message": "The order couldn't be delivered because of no available fibers"
	},
	...
]
```
