# Commercial Rating API

### Using the API

The base URL of the Merchants Commercial Rating API is:

`https://mbc-prod-commercial-rating.herokuapp.com`

The current version of the API is v3:

`/api/v3/...`

### API Key

An API key is required for each request. The API key should be included in an HTTP header named `AGENCY_API_KEY`. Use the key provided to you by Merchants Bonding, or contact support@merchantsbonding.com to receive a new API key.

Here's an example of including an HTTP header using `curl`.

```
curl -H AGENCY_API_KEY:123456 "https://mbc-prod-commercial-rating.herokuapp.com/api/v3/calculate-premium?agency_id=10213&obligee_name=&bond_amount=10000&effective_date=2014-05-02&license_plate=PNOTR&number_of_years=2&state=AZ
```

#### Calculating premium

Submit a GET request to `/api/v3/calculate-premium` - the list of parameters is below.  For questions about valid parameter values, contact us at support@merchantsbonding.com.

    state                :: required; one of the 51 US state codes
    license_plate        :: required; a string containing the "license plate", e.g. PNEOG
    agency_id            :: required; an integer agency id
    obligee_name         :: optional; a string containing an obligee name (for obligee-specific premiums)
    effective_date       :: required; the effective date of the bond formatted yyyy-mm-dd
    bond_amount          :: required; a number representing the bond amount in dollars and cents.
    number_of_years      :: required; an integer representing the number of years to be priced
    term_premium         :: optional; boolean (the strings 'true' or 'false')
    number_insured       :: optional; an integer representing the number of insured (premium * number of insured)

For example, this request for API version 3:

````
https://mbc-prod-commercial-rating.herokuapp.com/api/v3/calculate-premium?agency_id=10213&obligee_name=&bond_amount=15000&effective_date=2014-05-02&license_plate=LCEOT&number_of_years=2&state=GA&number_insured=5
````

returns an HTTP status of `200` and the following upon success:

````json
[
    {
        "premium": 394
    }
]
````

Validation errors produce an HTTP status of `400` and look like this:

````json
{
    "errors": {
        "number_of_years": [
            "is not a number"
        ],
        "effective_date": [
            "is not a valid date"
        ],
        "bond_amount": [
            "is not a number"
        ],
        "license_plate": [
            "can't be blank"
        ],
        "state": [
            "is not included in the list"
        ]
    }
}
````

Commercial rating errors produce an HTTP status of `404` look like this:

````json
{
    "errors": {
        "exception": [
            "Unable to rate this bond"
        ]
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
