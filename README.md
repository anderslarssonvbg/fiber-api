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
			"type": "MDU" // "'MDU' = apartment building, 'SDU' = villa"
		},
		"realEstate": {
			"label": "PENGABINGEN 1",
			"municipality": "ANKEBORG"
		},
		"coordinate": {
			"latitude": 6581619.085,
			"longitude": 1628539.32,
			"projection": "WGS84"
		},
		"district": "GAMLA STAN",
		"suppliers": [ // "optional, should only be provided if not heavy to calculate"
			{
				"name": "STOKAB",
				"fiberStatus": "IN_REAL_ESTATE", // "'AT_ADDRESS', 'AT_SITE_BOUNDARY'"
				"statusValidationRequired": true // "indicates if the fiberStatus needs manual validation to assure availability"
			}
		],
 		"relatedPointIds": [ // "optional, list of other nearby points (addresses) which could be used instead of the searched point (address). should only be provided if not heavy to calculate"
			"CDE678",
			"CDE901"
		]
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
### Hämta detaljerad information och leverantörers fiberstatus på en specifik plats (adress), men kan också returnera mer än 1 plats om sökningen inte är tillräckligt specifik.

***Request:***

Sökning via adress

```http
GET /api/1.0/availability?city={city}&street_name={streetName}&street_number={streetNumber}&street_littera={streetLittera} HTTP/1.1
```

eller sökning på punkt-id

```http
GET /api/1.0/availability?pointId={pointId} HTTP/1.1
```

eller sökning på en koordinat och radie som returnerar de närmaste platserna som är inom angiven radie

```http
GET /api/1.0/availability?xCoordinate={xCoordinate}&yCoordinate={yCoordinate}&radius={radius} HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
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
			"type": "MDU" // "'MDU' = apartment building, 'SDU' = villa"
		},
		"realEstate": {
			"label": "PENGABINGEN 1",
			"municipality": "ANKEBORG"
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
	},
	...
]
```

# 3. Price Estimate API
### Hämta prisuppskattning på en förbindelse

***Request:***

```http
POST /api/1.0/priceEstimates HTTP/1.1
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
	"products": [
		"All" // "e.g. 'All', 'Point2Point', 'Star'"
	],
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
	"noOfSingleFibers": 1, // "number of wanted single fibers for single fiber products"
	"noOfFiberPairs": 1 // "number of wanted fiber pairs for fiber pairs products"
}
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
	{
		"supplier": "STOKAB",
		"products": [
			{
				"productId" : "8u3-3563-3635-365-ff",
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
				"subProducts": [
					{
						"productId": "51dceb13-8d25-4f5f-adb1-29169d6042e2",
						"name": "ResidentialNetwork",
						"price": {
							"oneTimeFee": 1100.0,
							"monthlyFee": 100.0
						}
					},
					[...]
				],
				"price": {
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
				"productId": "2b99bcfb-b563-4b04-8c8a-8fd636f56d67",
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
```

# 4. Offer Inquiry API
### Skicka en offert-förfrågan för en förbindelse av en specifik produkt från en specifik leverantör

***Request:***

```http
POST /api/1.0/inquiries HTTP/1.1
Content-Type: application/json

{
	"supplier": "STOKAB",
	"product": {
		"productId": "0d13c5e0-ce23-41a0-87b5-f480479fa71e",
		"name": "Point2Point"
	},
	"referenceId": "CH-12345", // "client own reference for this inquiry, could be empty"
	"from": {
		"pointId": "ABC123",
		"comment": "", // "if an additional comment for the from point could be useful for the supplier"
	},
	"to": { // "may be set to null if any product only requires one point (address)"
		"pointId": "ABC789", 
		"comment": "", // "if an additional comment for the to point could be useful for the supplier"
	},
	"comment": "Lorem ipsum", 
	"redundancy": { // "may be set to null if no redundancy is wanted"
		"type": "Full", // "'Normal', 'Full'"
		"toPointId": "CBA123"
	},
	"customerType": "Commercial", // "e.g. 'Commercial', 'Residential'"
	"serviceLevel": "Premium", // "e.g. 'Base', 'Gold', 'Premium'"
	"parameters": [
		{
			"name": "ConnectorType",
			"value": "SC/APC"
		},
		...
	],
	"subProducts": [
		{
			"productId": "51dceb13-8d25-4f5f-adb1-29169d6042e2",
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
	"noOfSingleFibers": 1, // "number of wanted single fibers for single fiber products"
	"noOfFiberPairs": 1 // "number of wanted fiber pairs for fiber pairs products"
	"asyncAnswerAllowed": true // "if asychronous answer is ok (might result in an extra charge if manual)"
}
```

***Response:***

```http
HTTP/1.1 201 CREATED
Last-Modified: Mon, 11 Jan 2015 12:03:28 GMT
Location: /api/1.0/inquiries/ec4bc754-6a30-11e2-a585-4fc569183061
Content-Type: application/json

{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"referenceId": "CH-12345",
	"status": {
		"state": "WAIT_ASYNC_ANSWER", // "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'"
		"message": "",
		"createDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", // "should be null if not yet done"
	},
	"supplier": "STOKAB",
	"offerValidUntilDate": "2016-01-31",
	"connectionId": "", // "may be set to the identifier for the connection if that is already generated when inquiry is answered"
	"deliveryDurationDays": 20, // "days from order to delivered connection"
	"product": {
		"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
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
		"subProducts": [
			{
				"productId": "51dceb13-8d25-4f5f-adb1-29169d6042e2",
				"name": "ResidentialNetwork",
				"price": {
					"oneTimeFee": 1100.0,
					"monthlyFee": 100.0
				}
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
	}
}
```

### Hämta uppdatering på en skickad offert-förfrågan

***Request:***

```http
GET /api/1.0/inquiries/ec4bc754-6a30-11e2-a585-4fc569183061 HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Mon, 11 Jan 2015 12:03:28 GMT
Content-Type: application/json

{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"referenceId": "CH-12345",
	"status": {
		"state": "DONE_ASYNC_ANSWER_SUCCESS",
		"message": "",
		"createDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", // "should be null if not yet done"
	},
	"supplier": "STOKAB",
	"offerValidUntilDate": "2016-01-31",
	"connectionId": "",
	"deliveryDurationDays": 20,
	"product": {
		"productId": "8u3-3563-3635-365-ff",
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
	}
}
```

### Hämta slutförda förfrågningar som krävde asynkront svar

Anropet skall returnera samtliga förfrågningar som skett sedan since. Om en förfrågan har exakt angiven tidpunkt som doneDateTime så skall den ordern inte ingå i svaret.

Listan är oföränderlig. Vid två tidpunkter, t1 och t2, där t2 inträffar efter t1, skall tjänstens svar vid t2 alltid omfatta hela t1. Svaret kan alltså enbart växa, inte förändras.

Av denna anledningen är det viktigt att tjänstens svar inte är en projektion på förfrågningar, som kan ändra status, utan endast innefattar sådana förfrågningar som har nått en slutstatus.

Listan över förfråningar skall vara stigande i kronologisk ordning (senaste förfrågan som har förändrats sist). Klienten kan då använda det sista lästa förfrågan som since-parameter i nästföljande anrop.

Enbart förfrågningar som har status DONE_ASYNC_ANSWER_SUCCESS eller DONE_ASYNC_ANSWER_FAILED får förekomma i svaret.

Om inte since skickas med så skall alla slutförda förfrågningar returneras.

För att hämta detaljerad information om respektive förfrågan, använd Hämta uppdatering på en skickad förfrågan.

***Request:***

```http
GET /api/1.0/inquiries?since=2016-08-22T15:01:02.000Z
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

# 5. Order API
### Lägg en beställning på en tidigare mottagen offert och komplettera med kund och fakturainformation.

***Request:***

```http
POST /api/1.0/orders HTTP/1.1
Content-Type: application/json

{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"responsiblePerson": {
		"firstName": "Anders",
		"lastName": "Larsson",
		"phoneNumber": "46701234567",
		"email": "anders.larsson@comhem.com"
	},
	"endCustomer": {
		"firstName": "Sven",
		"lastName": "Svensson",
		"phoneNumber": "46702233445",
		"email": "sven.svensson@company.com"
	},
	"invoiceGroup": "IK-12345"
}
```

***Response:***

```http
HTTP/1.1 201 CREATED
Last-Modified: Mon, 11 Jan 2015 12:05:28 GMT
Location: /api/1.0/orders/fc6cd754-6a30-11e2-a585-4fc569185689
Content-Type: application/json

{
	"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"supplier": "STOKAB",
	"product": {
		"productId": "8u3-3563-3635-365-ff",
		"name": "Point2Point"
	},
	"status": {
		"state": "ORDERED",
		"message": "",
		"orderDateTime": "2015-01-11T11:34:00.000Z"
	}
}
```

### Hämta uppdatering på en lagd order

***Request:***

```http
GET /api/1.0/orders/fc6cd754-6a30-11e2-a585-4fc569185689 HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Tue, 15 Feb 2015 15:12:53 GMT
Content-Type: application/json

{
	"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"supplier": "STOKAB",
	"product": {
		"productId": "8u3-3563-3635-365-ff",
		"name": "Point2Point"
	},
	"status": {
		"state": "DELIVERED", // "ORDERED", "DELIVERED", "REJECTED"
		"message": "",
		"orderDateTime": "2015-01-11T11:34:00.000Z",
		"doneDateTime": "2015-02-15T14:49:12.000Z"
	},
	"responsiblePerson": {
		"firstName": "Anders",
		"lastName": "Larsson",
		"phoneNumber": "46701234567",
		"email": "anders.larsson@comhem.com"
	},
	"endCustomer": {
		"firstName": "Sven",
		"lastName": "Svensson",
		"phoneNumber": "46702233445",
		"email": "sven.svensson@company.com"
	},
	"invoiceGroup": "IK-12345"
}
```

### Hämta slutförda ordrar

Anropet skall returnera samtliga ordrar som skett sedan since. Om en order har exakt angiven tidpunkt som doneDateTime så skall den ordern inte ingå i svaret.

Listan är oföränderlig. Vid två tidpunkter, t1 och t2, där t2 inträffar efter t1, skall tjänstens svar vid t2 alltid omfatta hela t1. Svaret kan alltså enbart växa, inte förändras.

Av denna anledningen är det viktigt att tjänstens svar inte är en projektion på ordrar, som kan ändra status, utan endast innefattar sådana ordrar som har nått en slutstatus.

Listan över ordrar skall vara stigande i kronologisk ordning (senaste order som har förändrats sist). Klienten kan då använda det sista lästa orderns  som since-parameter i nästföljande anrop.

Enbart ordrar som har status DELIVERED eller REJECTED får förekomma i svaret.

Om inte since skickas med så skall alla slutförda ordrar returneras.

För att hämta detaljerad information om respektive order, använd Hämta uppdatering på en lagd order.

***Request:***

```http
GET /api/1.0/orders?since=2016-08-22T10:09:23.000Z
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
