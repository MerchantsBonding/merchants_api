# Commercial Bonds API

*All content is work in progress and subject to change*

## Table of Contents

* [High Level](#high-level)
* [Using the API](#using-the-api)
* [API Key](#api-key)
* [Obligations](#obligations)
* [Validating an Order](#validating-an-order)
* [Effective Date State Validations](#effective-date-state-validations)
* [Change Log](#change-log)

### High level
Use the obligations endpoint to retrieve two key pieces of information:
- license_plate
- obligation_id

Submit a validation request.


### Using the API

The base URL of the Merchants Commercial Obligations API is:

*Test*

`https://hub.mbctestweb.com`

*Production*

`https://hub.merchantsbonding.com`

The current version of the API is v4:

`/api/v4/...`

### API Key

An API key is required for each request. The API key should be included in an HTTP header named `API_KEY`.
Use the key provided to you by Merchants Bonding, or contact support@merchantsbonding.com to receive a new API key.

Here's an example of including an HTTP header using `curl`.

```
curl -H API_KEY:123456 "https://hub.mbctestweb.com/api/v4/...
```

### Obligations

Obligations are how we determine the correct form to use for the order.

Example:

```
curl -H API_KEY:123456 "https://hub.mbctestweb.com/api/v4/obligations"
```

returns an HTTP status of `200` and the following upon success:
````json
{
  "success":true,
  "obligations":
    [
      {
        "id":136880,
        "state":"TX",
        "name":"PRE-EXECUTED $10,000 Notary Public Bond",
        "license_plate":"PNOTR",
        "bond_amounts":[10000],
        "term_length":4
      },
      {
        "id":118489,
        "state":"AL",
        "name":"$25,000 Notary Public Bond",
        "license_plate":"PNOTR",
        "bond_amounts":[25000],
        "term_length":4
      },
      (content removed for brevity)
      {
        "id":136267,
        "state":"MN",
        "name":"$10,000 Notary Public Errors \u0026 Omissions Policy",
        "license_plate":"PNEO",
        "bond_amounts":[10000],
        "term_length":5
      },
      {
        "id":140334,
        "state":"CA",
        "name":"Notary Public Errors \u0026 Omissions Policy",
        "license_plate":"PNEO",
        "bond_amounts":[1000,3000,5000,10000,15000,20000,25000,30000,35000,50000,75000,100000],
        "term_length":4
      }
    ]
  }    
````

### Validating an Order

Submit a POST request to `/api/v4/orders/validate`.  For questions about valid parameter values, contact us at support@merchantsbonding.com.

    license_plate    :: required; a string containing the "license plate", e.g. PNEOG (obtained from obligations end point)
    obligation_id    :: required; a number identifying the obligation to be used (obtained from obligations endpoint)
    obligee_name     :: optional; a string containing an obligee name (for obligee-specific premiums) 
    bond_amount      :: required; a number representing the bond amount in dollars and cents.
   
    
    principal        :: required; a key containing principal related params
      name     :: required; a string containing a name
      address1 :: required; a string containing the first line of the address
      city     :: required; a string containing the city
      state    :: required; one of the 51 US state codes
      zip      :: required; a string containing the ZIP
    effective_date   :: required; the effective date of the bond, formatted yyyy-mm-dd
    expiration_date  :: required; the expiration date of the bond, formatted yyyy-mm-dd
    attorney_in_fact :: required; a string containing a name
    producer_name    :: required; a string containing a name
    state            :: required; one of the 51 US state codes
    
Current validations:
  * principal
    * name must be present
    * address1 must be present
    * city must be present
    * state must be present
    * zip must be present
  * dates
    * effective date 
      * must be present
      * must be in ISO 8601 format (YYYY-MM-DD)
      * [must be within range defined by state](#effective-date-state-validations)
    * expiration_date
      * must be present
      * must be in ISO 8601 format (YYYY-MM-DD)
      * [must be within range defined by state](#effective-date-state-validations)
      * must have year equivalent to effective date's year plus the term length specified by the obligation
  * other
    * attorney in fact must be present
    * producer name must be present
    * state must be present

For example, this request for API version 4:

````
curl -H API_KEY:123456 -H "Content-Type: application/json" -d '{"principal":{"name":"Santa Claus","address1":"Snowy Lane","state":"AK","city":"North Pole","zip":"12345"},"effective_date":"2017-10-15","attorney_in_fact":"Somebody","producer_name":"Someone Else","state":"IL","license_plate":"PNOTR"}' "https://hub.mbctestweb.com/api/v4/orders/validate"
````

returns an HTTP status of `200` and the following upon success:

````json
{
  "valid": true
}
````

Validation errors produce an HTTP status of `400` and look like this:

````json
{
  "valid":false,
  "errors":
    {
      "principal_address1":["can't be blank"],
      "principal_city":["can't be blank"],
      "principal_state":["can't be blank"],
      "principal_zip":["can't be blank"],
      "effective_date":["can't be blank","does not match ISO 8601 date format of \"YYYY-MM-DD\""],
      "attorney_in_fact":["can't be blank"],
      "producer_name":["can't be blank"],
      "state":["can't be blank"],
      "license_plate":["can't be blank"]
    }
}
````

General errors produce an HTTP status of `500` and look like this (provided the application is able to produce a response):

````json
{
    "errors": {
        "exception": [
            "Internal server error"
        ]
    }
}
````

### Effective Date State Validations
````
  AL
    days_before: 40
     days_after: 120
  
  AK
    days_before: 1
     days_after: 90
    
  AZ
    days_before: 367
     days_after: 60
    
  AR
    days_before: 1
     days_after: 60
    
  CA
    days_before: 30
     days_after: 30
    
  DC
    days_before: 60
     days_after: 60
    
  FL
    days_before: 367
     days_after: 120
    
  HI
    days_before: 367
     days_after: 60
    
  ID
    days_before: 1
     days_after: 90
    
  IL
    days_before: 180
     days_after: 180
    
  IN
    days_before: 1
     days_after: 60
    
  KS
    days_before: 1
     days_after: 60
  
  KY
    days_before: 60
     days_after: 45
  
  LA
    days_before: 30
     days_after: 90
  
  MI
    days_before: 180
     days_after: 60
  
  MS
    days_before: 180
     days_after: 91
  
  MO
    days_before: 90
     days_after: 60
  
  MT
    days_before: 30
     days_after: 30
  
  NE
    days_before: 1
     days_after: 60
  
  NV
    days_before: 180
     days_after: 180
  
  NM
    days_before: 1
     days_after: 90
  
  ND
    days_before: 1
     days_after: 90
  
  OK
    days_before: 367
     days_after: 45
  
  PA
    days_before: 45
     days_after: 90
  
  SD
    days_before: 1
     days_after: 90
  
  TN
    days_before: 367
     days_after: 120
  
  TX
    days_before: 367
     days_after: 180
  
  UT
    days_before: 1
     days_after: 60
  
  WA
    days_before: 180
     days_after: 180
  
  WI
    days_before: 1
     days_after: 90
  
  WY
    days_before: 60
     days_after: 60
````

### Change Log

- 2017-10-06 - Updated obligations example with results including bond_amounts and term_length
- 2017-10-09 - Validates state codes
