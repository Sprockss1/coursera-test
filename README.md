# node-requestmanagement

This is the requestmangement service for Landryt.

## Flow of the program

This request.service publishes to 'caseRequest' topic name with the request message json. Pubsub hosted on VM recieves the message and does processing on it. Using the pubsub webhook the processed data is recieved via /responses API in response.controller. This message is stored in the mongodb and sent to integromat for any further chnages. Integromat calls Docupilot API(from integromat account not from this code) and docoupilot report is recieved by webhook via docupilotReciever post url in docupilot-reciever.controller. This report is a pdf file which is stored in gcp using multer.

Input: 	
Post /requests
```json
{
  "ownerName": "string",
  "address": "string",
  "user": {},
  "notes": "string",
  "created_on": "2020-08-08T07:09:16.720Z",
  "featureType": 0,
  "status": 0,
  "district": "string",
  "state": "string",
  "taluk": "string",
  "hobli": "string",
  "village": "string",
  "surveyNumber": 0,
  "hissaNumber": 0
}
```
Output:
Recieved using pubsub webhook via post /responses url
Message is in ecrypted form and has to be decrypted from base64 to utf-8 in the below format
```json
{
  "id": "5f1c6fbe0e149031984d9fab",
  "error": true,
  "error_msg": ["Exception: Could not find combination when attributes are passed individually; Error: Error finding combination for district, taluk, hobli, village"],
  "user": {
  	"id": "5f1c6fbe0e149031984d9fab",
  	"name": "string",
  	},
  "client_address": "string",
  "case_count": 0,
  "doc_count": 0,
  "map_count": 0, 
  "enc_count": 0, 
  "title_details": {"title_current_owners": "", "title_survey_no": "", "title_village": "string", "title_hobli": "string", "title_district": "string", "title_state": "string", "title_extent": "", "title_land_use": ""},
  "lineage": [], 
  "pending": [],
  "dispute": {
          "revenue": [], "surveySettlement": [], "districtCourt": [], "highCourt": []},
  "encumbrance": [],
  "map": [],
  "reference": []
}
```


## Payment flow
Frontend calls /payment api with given json:
This is web integration process given by Razorpay: https://razorpay.com/docs/payment-gateway/web-integration/standard/
So when the user enters the amount and clicks for starting the payment process, frontend will call the backend API on /payment with the following data :
```json
{
  "userId": userid,
  "amount": 500    
}
```
Then razorpayfunctions.service will  make an order on Razorpay server from the backend and will send the frontend the following data
```json
{
    "id": "order_FDDJF8CaIb3HE0",
    "currency": "INR",
    "amount": 500
}
```
This data has to be used to make initiate checkout.js script in frontend.

Two webhooks are currently being used of Razorpay, i.e. for successful payment and failed payment.
Razorpay will call our /paymentverification api when the payment is successful. Backend will verify the payment and update in mongodb.
Razorpay will call our /failedpayment api when the payment is failed and updation in the mongodb will be done.

