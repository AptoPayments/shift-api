Shift API
===================

Partners (companies and developers) integrate with the Shift API to create and manage branded debit cards and cardholder data. We've tried to make this documentation user-friendly and example-filled, but please drop us a line with any questions. If you're planning to use our API in production, you should take a look at Metropolitan Commercial Bank's [privacy policy](https://shiftpayments.com/privacy).

### Integration Options
When a cardholder uses his or her Shift Card to purchase goods or services, Shift's software must quickly confirm the cardholder has sufficient funds prior to authorizing the transaction. A cardholder's spendable balance can be exposed from a partner's API -or- mainly reside within Shift.

**Partner API provides spendable balance** acting as the infallible balance authority. In this case, Shift will request the user's current spendable balance from the partner's API (your software). Shift will perform basic performance testing of your API prior to launch. Please reach out to [Shift Support](mailto:support@shiftpayments.com) to discuss integration specifics.  

-OR-

**Shift stores a spendable balance** for each cardholder. Your service simply updates each cardholder spendable balance as it changes within your business logic by consuming Shift's [Cardholder Balance endpoint](#cardholder-balance).

### Lay of the land

The Shift API is architected around REST, using standard HTTP verbs to communicate and HTTP response codes to indicate status and errors. All responses come in standard JSON. The Shift API is served over HTTPS to ensure data privacy; HTTP is not supported. Every request must include your secret API key. Request data must be included in the body.

**API Endpoint**

    https://api.shiftpayments.com

**Authentication**

Authentication to the API occurs via HTTP Basic Auth. Provide your API publishable key as the basic auth username and the your API secret key as the password.

**Live mode and testing**

Every account is divided into two universes: one for testing, and one for running on your live website. All API requests exist in one of those two universes, and objects in one universe cannot be manipulated by objects in the other.

**Errors**

```
status code 401

{
  "error": {
    "type": "invalid_request_error",
    "message": "Authentication to the API occurs via HTTP Basic Auth"
  }
}
```

```
status code 400

{
  "error": {
    "type": "invalid_request_error",
    "message": "Invalid region provided (must be US two letter state)"
  }
}
```

----------

**Cardholders**
-
Use the `/cardholders` endpoint to create and manage Shift cardholders. Cardholder objects allow you to track application status and confirm basic customer info. You can retrieve individual cardholders as well as a list of all your cardholders.

**Create a cardholder**

    POST /cardholders
    
In test mode:
  * New users and cards are created but **not physically shipped**.
  * *kyc_passed* is returned as **true** when the last 4 digits of the provided SSN are '0000'.
  
In live mode:
  * 'Know Your Customer' validations are performed and debit cards are **physically shipped**.
  * If the provided email address has already ordered or has an active Shift Card, a new card will not be shipped.

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----:|
| first_name    | required      |
| last_name     | required      |
| email         | required      |
| phone_number  | required      | E.164 number formatting
| address       | required      | International address object
| date_of_birth | required      | ISO8601 Date format
| document      | required      | Identity document object
| access_token  | *optional*    | Partner / end user specific token (e.g. OAuth)
| refresh_token | *optional*    | Partner / end user specific refresh token
| card_design   | *optional*    | Shift card design key string

*Example Request Body*

```
{
   "first_name": "Mac",
   "last_name": "Demarco",
   "email": "mac.demarco@gmail.com",
   "phone_number": "+16157915987",
   "access_token": "f698691043e2f54b18b5f49725e675c8129dae95",
   "refresh_token": "98ushdh87g8e77g8gegf98uf3bbbn3ujjii8899",
   "date_of_birth": "1974-03-22",
   "address": {
      "street_one": "665 3rd St.",
      "street_two": "Suite 207",
      "locality": "San Francisco",
      "region": "CA",
      "postal_code": "94107",
      "country": "USA"
   },
   "document": {
      "type": "SSN",
      "value": "459489876"
   }
}
```
*Example Response:*

```
status code 200

{
  "cardholder": {
    "livemode": true,
    "email": "charles.hine@gmail.com",
    "created_at": "2016-02-05T20:11:00+00:00",
    "id": 432,
    "kyc_passed_at": "2016-02-05T20:11:00+00:00",
    "cards": [
      {
        "card": {
          "id": 273,
          "status": "ACTIVATED",
          "last_four": "2489",
          "activated_at": "2016-02-05T20:11:00+00:00",
          "design_key": "dark-slate"
        }
      }
    ]
  }
}
```

**Retrieve a cardholder**

    GET /cardholders/[cardholder_id]

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----:|
| cardholder    | required      | The identifier of the customer to be retrieved. |


*Example Response:*

```
status code 200

{
  "cardholder": {
    "livemode": true,
    "email": "charles.hine@gmail.com",
    "created_at": "2016-02-05T20:11:00+00:00",
    "id": 432,
    "kyc_passed_at": "2016-02-05T20:11:00+00:00",
    "cards": [
      {
        "card": {
          "id": 273,
          "status": "ACTIVATED",
          "last_four": "2489",
          "activated_at": "2016-02-05T20:11:00+00:00",
          "design_key": "dark-slate"
        }
      }
    ]
  }
}
```

**List all cardholders**

    GET /cardholders

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----|
| limit         | optional      | A limit on the number of objects to be returned. Limit can range between 1 and 100 items.|
| ending_before | optional | A cursor for use in pagination. An object ID that defines your place in the list. |
| starting_after| optional | A cursor for use in pagination. An object ID that defines your place in the list. |

*Example Response:*

```
status code 200

{
  "cardholders": [
    {
      "cardholder": {
        "livemode": true,
        "email": "jack.boling@icloud.com",
        "created_at": "2016-02-05T20:11:00+00:00",
        "id": 106,
        "kyc_passed_at": "2016-02-05T20:11:00+00:00",
        "cards": [
          {
            "card": {
              "id": 19418,
              "status": "DEACTIVATED",
              "last_four": "4559",
              "activated_at": "2016-02-05T20:11:00+00:00",
              "design_key": "blue"
            }
          }
        ]
      }
    },
    {...}
  ],
  "has_more": true
}
```

**Create a cardholder transaction**

    POST /cardholders/[cardholder_id]/transactions

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----:|
| cardholder    | required      | The identifier of the cardholder. |
| usd_amount    | required      | USD decimal representing transaction amount. |
   
*Example Request Body:*
```
{
  "usd_amount": 20.00
}
```

*Example Response:*

```
status code 200

{
  "transaction": {
    "id": 15,
    "created_at": "2016-02-05T20:11:00+00:00",
    "cardholder_id": 1,
    "transfers": [
      {
        "transfer": {
          "id": 14,
          "created_at": "2016-02-05T20:11:00+00:00",
          "usd_amount": 20.0,
          "issuer_transaction_id": u88gf543e
        }
      }
    ],
    "cardholder_email": "okokokok@test.com",
    "description": "HOME DEPOT",
    "usd_amount": 20.0
  }
}
```

**List all cardholder transactions**

    GET /cardholders/[cardholder_id]/transactions

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----|
| cardholder    | required      | The identifier of the customer to be retrieved. |
| limit         | optional      | A limit on the number of objects to be returned. Limit can range between 1 and 100 items.|
| ending_before | optional | A cursor for use in pagination. An object ID that defines your place in the list. |
| starting_after| optional | A cursor for use in pagination. An object ID that defines your place in the list. |

*Example Response:*

```
status code 200

{
  "transactions": [
    {
      "transaction": {
        "id": 17,
        "created_at": "2016-02-05T20:11:00+00:00",
        "cardholder_id": 2,
        "transfers": [
          {
            "transfer": {
              "id": 16,
              "created_at": "2016-02-05T20:11:00+00:00",
              "usd_amount": 20.00,
              "issuer_transaction_id": 87g649h
            }
          },
          {...}
        ],
        "cardholder_email": "ok@test.com",
        "description": "HOME DEPOT",
        "usd_amount": 20.00
      }
    },
    {...}
  ],
  "has_more": false
}
```

----------

**Cardholder Balances**
-
The ability to manage multiple spendable balances (buckets) for each cardholder.

**Retrieve a cardholder with balance information**

    GET /cardholders/[cardholder_id]/balances

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----:|
| cardholder    | required      | The identifier of the customer to be retrieved. |


*Example Response:*

```
status code 200

{
  "cardholder": {
    "livemode": true,
    "email": "charles.hine@gmail.com",
    "created_at": "2016-02-05T20:11:00+00:00",
    "id": 432,
    "kyc_passed_at": "2016-02-05T20:11:00+00:00",
    "cards": [
      {
        "card": {
          "id": 273,
          "status": "ACTIVATED",
          "last_four": "2489",
          "activated_at": "2016-02-05T20:11:00+00:00",
          "design_key": "dark-slate"
        }
      }
    ],
    "balances": [
      {
        "balance": {
          "key": "default",     
          "usd_balance": 100.00,
          "usd_spendable": 25.0,
          "usd_held": 75.00
        }
      }
    ]
  }
}
```

**Update a cardholder's spendable balance**

    PUT /cardholders/[cardholder_id]/balances

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----:|
| cardholder    | required      | The identifier of the customer to be retrieved. |
| usd_amount    | required      | USD amount the cardholder shall be able to spend. |
| balance_key   | optional      | Specific balance identifier or bucket string. |

*Example Request Body*

```
{
  "usd_amount": 300.00,
}
```


*Example Response:*

```
status code 200

{
  "cardholder": {
    "livemode": true,
    "email": "charles.hine@gmail.com",
    "created_at": "2016-02-05T20:11:00+00:00",
    "id": 432,
    "kyc_passed_at": "2016-02-05T20:11:00+00:00",
    "cards": [
      {
        "card": {
          "id": 273,
          "status": "ACTIVATED",
          "last_four": "2489",
          "activated_at": "2016-02-05T20:11:00+00:00",
          "design_key": "dark-slate"
        }
      }
    ],
    "balances": [
      {
        "balance": {
          "key": "default",     
          "usd_balance": 375.00,
          "usd_spendable": 300.0,
          "usd_held": 75.00
        }
      }
    ]
  }
}
```

----------
**Transactions**
-
Use the `/transactions` endpoint to retrieve Shift cardholder currency movement records.

**Retrieve a transaction**

    GET /transactions/[transaction_id]

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----:|
| transaction    | required      | The identifier of the transaction to be retrieved. |


*Example Response:*

```
status code 200

{
  "transaction": {
    "id": 15,
    "created_at": "2016-02-05T20:11:00+00:00",
    "cardholder_id": 1,
    "transfers": [
      {
        "transfer": {
          "id": 14,
          "created_at": "2016-02-05T20:11:00+00:00",
          "usd_amount": 20.0,
          "issuer_transaction_id": u88gf543e
        }
      }
    ],
    "cardholder_email": "okokokok@test.com",
    "description": "HOME DEPOT",
    "usd_amount": 20.0
  }
}
```

**List all transactions**

    GET /transactions

*Arguments*

| Parameter     | Type          | Details
| ------------- |:-------------:| -----|
| limit         | optional      | A limit on the number of objects to be returned. Limit can range between 1 and 100 items.|
| ending_before | optional | A cursor for use in pagination. An object ID that defines your place in the list. |
| starting_after| optional | A cursor for use in pagination. An object ID that defines your place in the list. |

*Example Response:*

```
status code 200

{
  "transactions": [
    {
      "transaction": {
        "id": 17,
        "created_at": "2016-02-05T20:11:00+00:00",
        "cardholder_id": 2,
        "transfers": [
          {
            "transfer": {
              "id": 16,
              "created_at": "2016-02-05T20:11:00+00:00",
              "usd_amount": 20.00,
              "issuer_transaction_id": 87g649h
            }
          }
        ],
        "cardholder_email": "ok@test.com",
        "description": "HOME DEPOT",
        "usd_amount": 20.00
      }
    }, 
    {...}
  ],
  "has_more": false
}
```
