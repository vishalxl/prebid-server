# Stored Responses

This document gives a technical overview of the Stored Responses feature.

Stored responses are of two types: Single Stored Auction Response, and Multiple Stored Bid Response.

## Quickstart

Configure your server to read stored responses from the filesystem by adding the following to your config file ( normally named 'pbs.yaml'):

```yaml
stored_responses:
  filesystem: true
```

And then start your server:

```bash
go build .
./prebid-server
```

## Single Stored Auction Response

Choose an ID to reference your stored request data. Throughout this doc, replace {id} with the ID you've chosen.

Add the file `stored_responses/data/by_id/{id}.json` and populate it with some [Seatbid](https://www.iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf#page=29) data, which could be:

```json
[
  {
    "bid": [
      {
        "id": "2615127768151731669",
        "impid": "appnexus_imp_id1",
		"price": 0.010000,
		"adid": "107987536",
		"adomain": [
			"appnexus.com"
		]
      }
    ],
    "seat": "appnexus",
    "group": 0
  }
]
```

And then `POST` to [`/openrtb2/auction`](../endpoints/openrtb2/auction.md) with your chosen ID.

```json
{
  "id": "tid",
  "site": {
      "page": "prebid.org"
  },  
  "imp": [
    {
      "id": "appnexus_imp_id1",
      "banner": {
        "format": [
          {
            "w": 300,
            "h": 600
          }
        ]
      },
      "ext": {
        "prebid": {
          "storedauctionresponse": {
            "bidder": "appnexus",
            "id": "{id}"
          }
        }
      }
    }
  ]
}
```

Expected Response ( as body of response):

```json
{
    "id": "tid",
    "seatbid": [
        {
            "bid": [
                {
                    "id": "2615127768151731669",
                    "impid": "appnexus_imp_id1",
                    "price": 0.010000,
                    "adid": "107987536",
                    "adomain": [
                        "appnexus.com"
                    ],
                    "ext": {
                        "prebid": {
                            "type": "banner"
                        }
                    }
                }
            ],
            "seat": "appnexus",
            "group": 0
        }
    ],
    "cur": "USD",
    "ext": {
        "responsetimemillis": {
            "appnexus": 0
        },
        "tmaxrequest": 900,
        "auctiontimestamp": 1588751121655
    }
}
```

In the above transaction, no call to the concerned adapter will be made, and only the response from the specific {id} file will be returned as long as it is applicable for any given request ( which happens when the impid matches between the {id} file and incoming request).

## Multiple Stored Bid Response

