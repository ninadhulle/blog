Our team is building micro services using OSS stack. Below are some common conventions regarding https status code which were following:

•	Use nouns for api endpoints – why? – generally convention is to use verbs for method names. But for api endpoints, the http verbs (GET, POST, PUT, DELETE) are used, so we just need endpoints with nouns. Eg. /dogs or /cats and not /getDogs or not /getCats.
•	Below table details out the general convention for api endpoint naming. Single api endpoint will support CRUD operations (Create, Read, Update, Delete).

API   | Endpoint	POST (Create) Operation	  | GET (Read) Operation	  | PUT(Update) Operation	  | DELETE(Delete) Operation
------|-------------------------------------|-------------------------|-------------------------|-------------------------
/dogs | Create a new dog | Get list of dogs |	Update many dogs |	Delete all dogs
/dogs/1| Return Error	| Return dog with id 1 | Update dog#1 if it exists else return error | Delete dog#1

•	So the url pattern for api endpoints is:<HTTP verb> /parent-resource/identifier/child-resource
Eg. GET /owners/1/dogs or DELETE /owners/1/dogs/2
•	Feel free to use optional parameters generously
Eg. GET /dogs?breed=husky&color=black
•	Search:
o	Global search format should be GET /search?question=white+husky
o	Scoped search, endpoint should expose - GET /owners/1/dogs?breed=husky&color=black
•	By default your api’s should return json and if you want to return xml, doc or something, don’t change the endpoint, use content-type. 
•	For pagination use parameter like GET /search?question=white+husky&start=50&count=50. Don’t return huge amount of data if it is not required.
Below are common error codes and applicable scenarios
  
Error Code  | Http Code description   | Operations
------------|-------------------------|-----------
200	| Success – any success scenarios	| For below operations •	GET /search or GET /dogs •	PUT /dogs •	DELETE /dogs
201	| Created |	•	POST /dogs
304	| Not modified |	•	PUT /dogs
400	| Bad request	| For all PUT/DELETE/POST, if service validation fails. Return custom header with description as what input validation fails.
404	| Not found	| For below operations •	GET /search or GET /dogs •	PUT /dogs •	DELETE /dogs
304	| Not modified	| •	PUT /dogs
409	| Conflict |	Request is already locked, or is being edited •	PUT /dogs •	DELETE /dogs
412	| Precondition failed	| For below operations, if lock is not acquired •	PUT /dogs •	DELETE /dogs
500	| Internal Error | If api catches exception and throws 500, then it should return 500 with Custom Description in header
