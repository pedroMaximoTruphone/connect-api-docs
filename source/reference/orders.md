# Orders

## Order Resource

SIM's and connectivity plans are purchased by creating orders. Order fulfilment is asynchronous due to the nature of SIM and network provisioning. Typical use cases include creating an order and polling for the status shortly after.

### Properties

|   Property Name   |                                                             Description                                                              |
| :---------------: | :----------------------------------------------------------------------------------------------------------------------------------: |
|    externalId     | Order identifier, generated by Truphone, for tracking the order.<br/>Users should store this value in order to poll for order status |
|      status       |                                            Order status: `COMPLETED`/`FAILED`/`FULFILING`                                            |
|       input       |                                    Data required to process the order. Depends on the order type                                     |
|      output       |                                                   Produced output after completed                                                    |
|   output.iccid    |                                                       eSIM Profile identifier                                                        |
| output.matchingId |                                                         eSIM Activation Code                                                         |
|  output.smdpUrl   |                                                         SM-DP+ Instance URL                                                          |

## Purchase a New eSIM

Customer does not own a Truphone SIM and wants to order one. This operation provisions/activates a SIM for installing on a given device and adds a data plan to it.

- Allowed roles: `RESELLER`, `ACCOUNT_MANAGER`
- URL: `v1/order`
- METHOD: `POST`

### Parameters

|         Parameter Name         |                   Description                   |   Required   |
| :----------------------------: | :---------------------------------------------: | :----------: |
|         operationType          |               Must be `NEW_ESIM`                |     yes      |
|          countryCode           |     Country where the order is being placed     | configurable |
|            customer            | End customer who is purchasing the subscription |     yes      |
|         customer.email         |       End customer's email or identifier        |     yes      |
|       customer.firstName       |            End customer's first name            | configurable |
|       customer.lastName        |            End customer's last name             | configurable |
|  customer.countryOfResidence   |       End customer's country of residence       | configurable |
|             device             | Device where the subscription will be installed |     yes      |
|          device.type           |            `ios`, `android` or `iot`            |      no      |
|           device.id            |        Unique identifier for the device         |     yes      |
|          device.model          |                  Device Model                   |      no      |
|          device.make           |                   Device Make                   |      no      |
|         subscriptions          |          List of products to purchase           |     yes      |
|   subscriptions[].product_id   |          Id of the product to purchase          |     yes      |
| subscriptions[].activationDate |     Date when the product can be activated      |      no      |
|     subscriptions[].price      |      Price for which the product was sold       |      no      |
|    subscriptions[].curremcy    |     Currency for which the product was sold     |      no      |

### Rules and validations

- If the customer does not exist it will be created
- `subscriptions.product_id` (preferred) or `subscriptions.allowanceId` (deprecated) - Needs to be an id that belongs to the product catalog of the customer
- `subscriptions.activationDate` - Can not be a date in the past
- `subscriptions.price` - If is sent, it must be sent in pair with `subscriptions.currency`
- `subscriptions.currency` - If is sent, it must be sent in pair with `subscriptions.price` and needs to be one of the currencies supported by the customer

### Example Request

```bash
curl -X POST \
  https://services.truphone.com/esim/v1/order \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'X-Correlation-ID: unique-id-from-requester-123' \
  -d '{
      	"input": {
            "operationType": "NEW_ESIM",
            "countryCode": "PT",
            "customer": {
                "email": "john@doe.com",
                "countryOfResidence": "US"
            },
            "device": {
                "id": "123456789",
                "model": "iPhone",
                "type": "ios"
            },
      	    "subscriptions": [{
      	        "product_id": "a3u3z000000PZBhAAO",
      	        "activationDate": "2019-09-13T12:00:00Z",
      	        "price": 10,
      	        "currency": "USD"
      	    }]
      	}
      }'
```

### Example Response

```json
{
  "externalId": "3a6acf89-1ccf-4611-9424-453930f57ef1",
  "status": "ACCEPTED"
}
```

## Purchase a Top-Up

The customer already has a Truphone SIM. Topping up will add connectivity plans to this SIM

- Allowed roles: `RESELLER`, `ACCOUNT_MANAGER`
- URL: `v1/order`
- METHOD: `POST`

### Parameters

|         Parameter Name         |                   Description                   |   Required   | Default |
| :----------------------------: | :---------------------------------------------: | :----------: | :-----: |
|         operationType          |                 Must be `TOPUP`                 |     yes      |         |
|          countryCode           |     Country where the order is being placed     | configurable |         |
|            customer            | End customer who is purchasing the subscription |     yes      |         |
|         customer.email         |       End customer's email or identifier        |     yes      |         |
|             device             | Device where the subscription will be installed |     yes      |         |
|          device.type           |            `ios`, `android` or `iot`            |      no      |         |
|           device.id            |        Unique identifier for the device         |     yes      |         |
|          device.model          |                  Device Model                   |      no      |         |
|          device.make           |                   Device Make                   |      no      |         |
|         subscriptions          |          List of products to purchase           |     yes      |         |
|   subscriptions[].product_id   |          Id of the product to purchase          |     yes      |         |
| subscriptions[].activationDate |     Date when the product can be activated      |      no      |         |
|     subscriptions[].price      |      Price for which the product was sold       |      no      |         |
|    subscriptions[].curremcy    |     Currency for which the product was sold     |      no      |         |
|              esim              |     The eSIM to which add the subscription      |     yes      |         |
|           esim.iccid           |    The iccid corresponding to the subscriber    |     yes      |         |

### Rules and validations

- The customer must already exist
- `operationType` - Needs to be `TOPUP` (preferred) or `TOPUP_ACTIVATION` (deprecated)
- `subscriptions.product_id` (preferred) or `subscriptions.allowanceId` (deprecated) - Needs to be an id that belongs to the product catalog of the customer
- `subscriptions.activationDate` - Can not be a date in the past
- `subscriptions.price` - If is sent, it must be sent in pair with `subscriptions.currency`
- `subscriptions.currency` - If is sent, it must be sent in pair with `subscriptions.price` and needs to be one of the currencies supported by the customer
- `device.type` - Must be one of the following values `ios` or `android`

In order to call this endpoint, both the `customer.email` and `esim.iccid` nodes are mandatory and the customer must be entitled to that esim, otherwise the topup won't be performed:

- `esim` and `esim.iccid` are mandatory
- `customer.email` - the email used to order the first eSIM;
- `esim.iccid` - the iccid obtained when the first eSIM was ordered.

### Example Request

```bash
curl -X POST \
  https://services.truphone.com/esim/v1/order \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'X-Correlation-ID: unique-id-from-requester-123' \
  -d '{
      	"input": {
            "operationType": "TOPUP_ACTIVATION",
            "countryCode": "PT",
            "customer": {
                "email": "john@doe.com",
                "countryOfResidence": "US"
            },
            "device": {
                "id": "123456789",
                "model": "iPhone",
                "type": "ios"
            },
            "subscriptions": [{
      	        "product_id": "a3u3z000000PZBhAAO",
      	        "activationDate": "2019-09-13T12:00:00Z",
      	        "price": 10,
      	        "currency": "USD"
      	    }],
      	    "esim": {
      	        "iccid": "123456789"
      	    }
      	}
      }'
```

### Example Response

```json
{
  "externalId": "3a6acf89-1ccf-4611-9424-453930f57ef1",
  "status": "ACCEPTED"
}
```

## Check an order's status

Order fulfilment is not synchronous, so after placing an order, the API user must use the order id to periodically check until the order is complete

- Allowed roles: `RESELLER`, `ACCOUNT_MANAGER`
- URL: `v1/order/{orderId}`
- METHOD: `GET`

### Example Request

```bash
curl -X GET \
  https://services.truphone.com/esim/v1/order/3a6acf89-1ccf-4611-9424-453930f57ef1 \
   -H "Authorization: Bearer $ACCESS_TOKEN" \
   -H 'Cache-Control: no-cache' \
   -H 'Content-Type: application/json' \
   -H 'X-Correlation-ID: unique-id-from-requester-123'
```

### Example Response

```json
{
  "externalId": "3a6acf89-1ccf-4611-9424-453930f57ef1",
  "status": "COMPLETED",
  "output": {
    "iccid": "1234567890",
    "matchingId": "M-123DSA-AS",
    "smdpUrl": "rsp.truphone.com"
  }
}
```