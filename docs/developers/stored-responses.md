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

Choose an ID to reference your stored request data. Throughout this section, replace {id} with the ID you've chosen.

Add the file `stored_responses/data/by_id/{id}.json` and populate it with some [Seatbid](https://www.iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf#page=29) data, which could be:

```json
[
  {
    "bid": [
      {
        "id": "2615127768151731669",
        "price": 0.010000,
        "adid": "107987536",
        "adomain": [
            "bidder1.com"
        ]
      }
    ],
    "seat": "bidder1",
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
      "id": "imp_1",
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
            "bidder": "bidder1",
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
          "impid": "imp_1",
          "price": 0.010000,
          "adid": "107987536",
          "adomain": [
              "bidder1.com"
          ],
          "ext": {
            "prebid": {
              "type": "banner"
            }
          }
        }
      ],
      "seat": "bidder1",
      "group": 0
    }
  ],
  "cur": "USD"
}
```

In the above transaction, no call to the concerned adapter will be made, and only the response from the specific {id} file will be returned as long as it is applicable for any given request (which happens when the value of the incoming request.ext.prebid.storedauctionresponse.id matches with a stored auction file named {id}.json).

## Multiple Stored Bid Response

For an impression, this feature allows an auction to take place, where a live bid can be received from one/some bidder, while at the same time a stored-response can also be returned for the same response. 

Lets say we have two adapters/bidders, named bidder1, and bidder2. For each, choose unique id's, called  {id1} and {id2}, which are normal ascii strings ( and used as filenames). Create files `{id1}.json` and `{id2}.json` in directory `stored_responses/data/by_id/`. Populate the files with seatbid data as specified [here](https://www.iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf#page=29):


Example File `{id1}.json`
```json
[
  {
    "bid": [
      {
        "id": "2615127768151731635",
        "price": 0.010000,
        "adid": "107987535",
        "adomain": [
            "bidder1.com"
        ]
      }
    ],
    "seat": "bidder1",
    "group": 0
  }
]
```

Example File `{id2}.json`
```json
[
  {
    "bid": [
      {
        "id": "2615127768151731640",
        "price": 0.010000,
        "adid": "1079875440",
        "adomain": [
            "bidder2.com"
        ]
      }
    ],
    "seat": "bidder2",
    "group": 0
  }
]
```

### Stored Responses for Both Bidders

Lets first take a scenario where both bidders return stored response values. 

In this case `POST` to [`/openrtb2/auction`](../endpoints/openrtb2/auction.md) the following:

```json
{
  "id": "tid",
  "site": {
      "page": "prebid.org"
  },  
  "imp": [
    {
      "id": "imp1",
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
          "storedbidresponse": [
            {
              "bidder": "bidder1",
              "id": "{id1}"
            },
            {
              "bidder": "bidder1",
              "id": "{id2}"
            }
          ]
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
          "id": "2615127768151731635",
          "impid": "imp1",
          "price": 0.010000,
          "adid": "107987535",
          "adomain": [
              "bidder1.com"
          ],
          "ext": {
            "prebid": {
              "type": "banner"
            }
          }
        }
      ],
      "seat": "bidder1",
      "group": 0
    },
    {
      "bid": [
        {
          "id": "2615127768151731640",
          "impid": "imp1",
          "price": 0.010000,
          "adid": "1079875440",
          "adomain": [
              "bidder2.com"
          ],
          "ext": {
            "prebid": {
              "type": "banner"
            }
          }
        }
      ],
      "seat": "bidder2",
      "group": 0
    }

  ],
  "cur": "USD",
}
```

### Real Auction By One Bidder and Stored Response for Another

In this scenario, bidder1 will take part in a real auction, whereas a stored-response will be returned for bidder2 ( for the same impression).

In this case `POST` to [`/openrtb2/auction`](../endpoints/openrtb2/auction.md) the following:

```json
{
  "id": "tid",
  "site": {
      "page": "prebid.org"
  },  
  "imp": [
    {
      "id": "imp1",
      "banner": {
        "format": [
          {
            "w": 300,
            "h": 600
          }
        ]
      },
      "ext": {
        "bidder1": {
          "placementId": 12883451
        },
        "prebid": {
          "storedbidresponse": [
            {
              "bidder": "bidder2",
              "id": "{id2}"
            }
          ]
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
          "id": "UUUUUUUUUU",
          "impid": "imp1",
          "price": 0.U00000,
          "adid": "UUUUUUUUUUU",
          "adomain": [
              "bidder1.com"
          ],
          "ext": {
            "prebid": {
              "type": "banner"
            }
          }
        }
      ],
      "seat": "bidder1",
      "group": 0
    },
    {
      "bid": [
        {
          "id": "2615127768151731640",
          "impid": "imp1",
          "price": 0.010000,
          "adid": "1079875440",
          "adomain": [
              "bidder2.com"
          ],
          "ext": {
            "prebid": {
              "type": "banner"
            }
          }
        }
      ],
      "seat": "bidder2",
      "group": 0
    }

  ],
  "cur": "USD",
}
```

Use of character `U` in above response represents that its some value that's returned by a real auction by bidder1.

