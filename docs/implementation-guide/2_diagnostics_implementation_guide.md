# 2. Implementation Guide

This document contains the REQUIRED and RECOMMENDED standard functionality that must be implemented by any DHP Service Provider Platform a.k.a BPPs and DHP Consumer Platform a.k.a BAPs.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

## 2.1 Discovery of Diagnostics Center/Labs

### 2.1.1 Recommendations for BPPs
The following recommendations need to be considered when implementing discovery functionality for a DHP BPP

- REQUIRED. The BPP MUST implement the `search` endpoint to receive an `Intent` object sent by BAPs
- REQUIRED. The BPP MUST return a catalog of DHP products on the `on_search` callback endpoint specified in the `context.bap_uri` field of the `search` request body.
- REQUIRED. The BPP MUST map its diagnostics service to the `Item` schema.
- REQUIRED. Any DHP service provider-related information like name, logo, short description must be mapped to the `Provider.descriptor` schema
- REQUIRED. Any form that must be filled before receiving a quotation must be mapped to the `XInput` schema
- REQUIRED. If the platform wants to group its products under a specific category, it must map each category to the `Category` schema
- REQUIRED. Any service fulfillment-related information MUST be mapped to the `Fulfillment` schema.
- REQUIRED. If the BPP does not want to respond to a search request, it MUST return a `ack.status` value equal to `NACK`
- RECOMMENDED. Upon receiving a `search` request, the BPP SHOULD return a catalog that best matches the intent. This can be done by indexing the catalog against the various probable paths in the `Intent` schema relevant to typical DHP use cases

### 2.1.2 Recommendations for BAPs
- REQUIRED. The BAP MUST call the `search` endpoint of the BG to discover multiple BPPs on a network
- REQUIRED. The BAP MUST implement the `on_search` endpoint to consume the `Catalog` objects containing DHP Products sent by BPPs.
- REQUIRED. The BAP MUST expect multiple catalogs sent by the respective DHP Providers on the network
- REQUIRED. The DHP products can be found in the `Catalog.providers[].items[]` array in the `on_search` request
- REQUIRED. If the `catalog.providers[].items[].xinput` object is present, then the BAP MUST redirect the user to, or natively render the form present on the link specified on the `items[].xinput.form.url` field.
- REQUIRED. If the `catalog.providers[].items[].xinput.required` field is set to `"true"` , then the BAP MUST NOT fire a `select`, `init` or `confirm` call until the form is submitted and a successful response is received
- RECOMMENDED. If the `catalog.providers[].items[].xinput.required` field is set to `"false"` , then the BAP SHOULD allow the user to skip filling the form


### Example
A search request for a diagnostic service may look like this
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "search",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_url": "https://ps-bap-client.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "intent": {
            "category": {
                "descriptor": {
                    "name": "cardiology"
                }
            }
    }
    }
}
```

An example catalog of diagnostic service may look like this
```
{
  "context": {
    "domain": "dhp:diagnostics:0.1.0",
    "action": "on_search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_url": "https://ps-bap-client.becknprotocol.io/",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "catalog": {
      "descriptor": {
        "name": "Diagnostic services"
      },
      "providers": [
        {
          "id": "289edce4-d002-4962-b311-4c025e22b4f6",
          "descriptor": {
            "name": "GioLabs Pvt Ltd",
            "short_desc": "X-rays, tests and more",
            "long_desc": "Advanced diagnostics & precise testing at XYZ Pathology Lab. Fast, accurate results for informed healthcare decisions.",
            "images": [
              {
                "url": "https://giolabs.in/images/logo.png"
              }
            ],
            "media": [
              {
                "mimetype": "application/pdf",
                "url": "https://www.aiims.com/honours/237402938409485039850935"
              }
            ]
          },
          "location": [
            {
              "gps": "12.9716° N, 77.5946° E",
              "address": "Akashya nagar B17/14",
              "state": {
                "name": "Madhya Pradesh"
              },
              "city": {
                "name": "Bhopal"
              },
              "area_code": "462001"
            }
          ],
          "rating": "4.5",
          "categories": [
            {
              "id": "cat-01",
              "descriptor": {
                "code": "cardiology",
                "name": "Cardiology"
              }
            }
          ],
          "fulfillments": [
            {
              "id": "ful-01",
              "type": "walk-in"
            },
            {
              "id": "ful-02",
              "type": "doorstep service"
            }
          ],
          "items": [
            {
              "id": "lab-test-01",
              "descriptor": {
                "code": "ecg",
                "name": "Electrocardiogram test"
              },
              "price": {
                "value": "3000",
                "currency": "INR"
              },
              "category_ids": [
                "cat-01"
              ],
              "fulfillment_ids": [
                "ful-01",
                "ful-02"
              ]
            }
          ]
        }
      ]
    }
  }
}
```

## 2.2 Application for DHP(Diagnostics) 
This section provides recommendations for implementing the APIs related to booking a diagnostics service.

### 2.2.1 Recommendations for BPPs

#### 2.2.1.1 Selecting a DHP service from the catalog
- REQUIRED. The BPP MUST implement the `select` endpoint on the url specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry.
- REQUIRED. If the DHP provider has returned a successful acknowledgement to a `select` request, it MUST send the offer encapsulated in an `Order` object

#### 2.2.1.2 Initializing the Booking for a diagnostic service
- REQUIRED. The BPP MUST implement the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.2.1.3 Confirming the Booking for a diagnostic service
- REQUIRED. The BPP MUST implement the `confirm` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 2.2.2 Recommendations for BAPs

#### 2.2.2.1 Selecting a DHP service from the catalog
- REQUIRED. The BAP MUST implement the `on_select` endpoint on the url specified in the `context.bap_uri` field sent during `select`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.2.2.2  Initializing the Booking for a diagnostic service
- REQUIRED. The BAP MUST hit the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_init` endpoint on the url specified in  the `context.bap_uri` field sent during `init`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.2.2.3 Confirming the Booking for a diagnostic service
- REQUIRED. The BAP MUST hit the `confirm` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_confirm` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `confirm`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 2.2.3 Example Workflow

### 2.2.3 Example Requests

Below is an example of a `select` request
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "select",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_url": "https://ps-bap-client.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "289edce4-d002-4962-b311-4c025e22b4f6"
            },
            "fulfillments" : [
                {
                    "id" : "ful-01"
                }
            ],
            "items": [
                {
                    "id": "lab-test-01"
                }
            ]
        }
    }
}
```

Below is an example of an `on_select` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_select",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io/",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "provider": {
            "id": "289edce4-d002-4962-b311-4c025e22b4f6",
            "descriptor": {
              "name": "GioLabs Pvt Ltd",
              "short_desc": "X-rays, tests and more",
              "long_desc" : "Advanced diagnostics & precise testing at XYZ Pathology Lab. Fast, accurate results for informed healthcare decisions.",
              "images": [
                {
                  "url": "https://giolabs.in/images/logo.png"
                }
              ],
              "media" : [
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/237402938409485039850935"
                },
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/2374029384094850394240935"
                }
              ]
            },
            "location" : [{
                "gps": "12.9716° N, 77.5946° E",
                "address": "Akashya nagar B17/14",
                "state": {
                    "name": "Madhya Pradesh"
                },
                "city": {
                    "name": "Bhopal"
                },
                "area_code": "462001"
            }],
            "rating": "4.5"
        },
        "items": [
          {
            "id": "lab-test-01",
            "descriptor": {
                "code": "ecg",
                "name": "Electrocardiogram test"
            },
            "price": {
                "value": "3000",
                "currency": "INR"
            },
            "category_ids": [
                "cat-01"
            ],
            "fulfillment_ids": [
                "ful-01",
                "ful-02"
            ]
          }
        ],
        "fulfillments": [
          {
            "id": "ful-01",
            "type": "walk-in",
            "stops": [
              {
                "type": "end",
                "time": {
                  "range" : {
                    "start" : "0000-00-00T10:00:00Z",
                    "end" : "0000-00-00T18:00:00Z"
                  },
                  "days": "1,2,3,4,5,6,7",
                  "schedule" : {
                    "holidays" : [
                        "2024-12-10",
                        "2024-12-11",
                        "2024-12-12"
                    ]
                  }
                },
                "location": {
                    "gps": "12.9716° N, 77.5946° E",
                    "address": "Akashya nagar B17/14",
                    "state": {
                        "name": "Madhya Pradesh"
                    },
                    "city": {
                        "name": "Bhopal"
                    },
                    "area_code": "462001"
                }
              }
            ]
          }
        ],
        "quote": {
          "price": {
            "value": "3000",
            "currency": "INR"
          },
          "breakup": [
            {
              "title": "ECG Procedure",
              "price": {
                "currency": "INR",
                "value": "3000"
              }
            }
          ]
        }
      }
    }
  }
```


Below is an example of a `init` request
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "init",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_url": "https://ps-bap-client.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "289edce4-d002-4962-b311-4c025e22b4f6"
            },
            "items": [
                {
                    "id": "lab-test-01"
                }
            ],
            "billing": {
                "name": "Rajesh Kumar",
                "address": "Villa 5, Green Valley, Malleshwaram, 560012",
                "email": "rajesh.kumar@example.com",
                "phone": "+91-9999999999"
            },
            "fulfillments": [
                {
                    "id": "ful-01",
                    "customer": {
                        "person": {
                            "name": "Rajesh Kumar",
                            "age": "26",
                            "gender": "M",
                            "dob": "1995-09-11"
                        },
                        "contact" :{
                            "phone" : "+91-9999999999",
                            "email" : "rajesh.kumar@example.com"
                        }
                    }
                }
            ]
        }
    }
}
```

Below is an example of `on_init` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_init",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io/",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "provider": {
            "id": "289edce4-d002-4962-b311-4c025e22b4f6",
            "descriptor": {
              "name": "GioLabs Pvt Ltd",
              "short_desc": "X-rays, tests and more",
              "long_desc" : "Advanced diagnostics & precise testing at XYZ Pathology Lab. Fast, accurate results for informed healthcare decisions.",
              "images": [
                {
                  "url": "https://giolabs.in/images/logo.png"
                }
              ],
              "media" : [
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/237402938409485039850935"
                },
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/2374029384094850394240935"
                }
              ]
            },
            "rating": "4.5"
        },
        "items": [
            {
                "id": "lab-test-01",
                "descriptor": {
                    "code": "ecg",
                    "name": "Electrocardiogram test"
                },
                "price": {
                    "value": "3000",
                    "currency": "INR"
                },
                "category_ids": [
                    "cat-01"
                ],
                "fulfillment_ids": [
                    "ful-01",
                    "ful-02"
                ]
            }
        ],
        "fulfillments": [
          {
            "id": "ful-01",
            "type": "walk-in",
            "stops": [
                {
                  "type": "end",
                  "time": {
                    "range" : {
                      "start" : "0000-00-00T10:00:00Z",
                      "end" : "0000-00-00T18:00:00Z"
                    },
                    "days": "1,2,3,4,5,6,7",
                    "schedule" : {
                      "holidays" : [
                          "2024-12-10",
                          "2024-12-11",
                          "2024-12-12"
                      ]
                    }
                  },
                  "location": {
                      "gps": "12.9716° N, 77.5946° E",
                      "address": "Akashya nagar B17/14",
                      "state": {
                          "name": "Madhya Pradesh"
                      },
                      "city": {
                          "name": "Bhopal"
                      },
                      "area_code": "462001"
                  }
                }
            ],
            "customer": {
                "person": {
                    "name": "Rajesh Kumar",
                    "age": "26",
                    "gender": "M",
                    "dob": "1995-09-11"
                },
                "contact" :{
                    "phone" : "+91-9999999999",
                    "email" : "rajesh.kumar@example.com"
                }
            },
            "agent": {
              "person": {
                "id": "237402938409485039850935",
                "name": "Mr. Shivam Arora",
                "gender": "male"
              },
              "contact" : {
                "phone" : "+91-7897898765",
                "email" : "shivam-labsamplecollector@gmail.com"
              }
            }
          }
        ],
        "quote": {
          "price": {
            "value": "3000",
            "currency": "INR"
          },
          "breakup": [
            {
              "title": "ECG Procedure",
              "price": {
                "currency": "INR",
                "value": "3000"
              }
            },
            {
              "title": "taxes",
              "price": {
                "currency": "INR",
                "value": "30"
              }
            }
          ]
        },
        "billing": {
            "name": "Rajesh Kumar",
            "address": "Villa 5, Green Valley, Malleshwaram, 560012",
            "email": "rajesh.kumar@example.com",
            "phone": "+91-9999999999"
        },
        "payments": [
          {
            "type": "PRE-FULFILLMENT",
            "collected_by": "BPP",
            "status": "NOT-PAID",
            "params": {
              "url": "payto://ban/98273982749428?amount=INR:3030&ifsc=SBIN0000575&message=payment",
              "amount": "3030",
              "currency": "INR"
            }
          }
        ],
        "cancellation_terms": [
          {
            "fulfillment_state": {
              "descriptor": {
                "code": "in-progress"
              }
            },
            "cancellation_fee": {
              "percentage": "30%"
            },
            "external_ref": {
              "mimetype": "text/html",
              "url": "https://giolabs.in/charge/tnc.html"
            }
          }
        ]
      }
    }
}
```

Below is an example of a `confirm` request
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "confirm",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_url": "https://ps-bap-client.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "289edce4-d002-4962-b311-4c025e22b4f6"
            },
            "items": [
                {
                    "id": "lab-test-01"
                }
            ],
            "billing": {
                "name": "Rajesh Kumar",
                "address": "Villa 5, Green Valley, Malleshwaram, 560012",
                "email": "rajesh.kumar@example.com",
                "phone": "+91-9999999999"
            },
            "fulfillments": [
                {
                    "id": "ful-01",
                    "customer": {
                        "person": {
                            "name": "Rajesh Kumar",
                            "age": "26",
                            "gender": "M",
                            "dob": "1995-09-11"
                        },
                        "contact" :{
                            "phone" : "+91-9999999999",
                            "email" : "rajesh.kumar@example.com"
                        }
                    }
                }
            ],
            "payments": [
                {
                  "type": "PRE-FULFILLMENT",
                  "collected_by": "BPP",
                  "status": "PAID",
                  "params": {
                    "url": "payto://ban/98273982749428?amount=INR:3030&ifsc=SBIN0000575&message=payment",
                    "transaction_id" : "ABCD2414143231",
                    "amount": "3030",
                    "currency": "INR"
                  }
                }
            ]
        }
    }
}
```
Below is an example of an `on_confirm` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_confirm",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io/",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "id": "be69bb43-8286-48b4-801f-a8c5b41ad450",
        "provider": {
            "id": "289edce4-d002-4962-b311-4c025e22b4f6",
            "descriptor": {
              "name": "GioLabs Pvt Ltd",
              "short_desc": "X-rays, tests and more",
              "long_desc" : "Advanced diagnostics & precise testing at XYZ Pathology Lab. Fast, accurate results for informed healthcare decisions.",
              "images": [
                {
                  "url": "https://giolabs.in/images/logo.png"
                }
              ],
              "media" : [
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/237402938409485039850935"
                },
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/2374029384094850394240935"
                }
              ]
            },
            "rating": "4.5"
        },
        "items": [
            {
                "id": "lab-test-01",
                "descriptor": {
                    "code": "ecg",
                    "name": "Electrocardiogram test"
                },
                "price": {
                    "value": "3000",
                    "currency": "INR"
                },
                "category_ids": [
                    "cat-01"
                ],
                "fulfillment_ids": [
                    "ful-01",
                    "ful-02"
                ]
            }
        ],
        "fulfillments": [
          {
            "id": "ful-01",
            "type": "walk-in",
            "stops": [
                {
                  "type": "end",
                  "time": {
                    "range" : {
                      "start" : "0000-00-00T10:00:00Z",
                      "end" : "0000-00-00T18:00:00Z"
                    },
                    "days": "1,2,3,4,5,6,7",
                    "schedule" : {
                      "holidays" : [
                          "2024-12-10",
                          "2024-12-11",
                          "2024-12-12"
                      ]
                    }
                  },
                  "location": {
                      "gps": "12.9716° N, 77.5946° E",
                      "address": "Akashya nagar B17/14",
                      "state": {
                          "name": "Madhya Pradesh"
                      },
                      "city": {
                          "name": "Bhopal"
                      },
                      "area_code": "462001"
                  }
                }
            ],
            "customer": {
                "person": {
                    "name": "Rajesh Kumar",
                    "age": "26",
                    "gender": "M",
                    "dob": "1995-09-11"
                },
                "contact" :{
                    "phone" : "+91-9999999999",
                    "email" : "rajesh.kumar@example.com"
                }
            },
            "agent": {
              "person": {
                "id": "237402938409485039850935",
                "name": "Mr. Shivam Arora",
                "gender": "male"
              },
              "contact" : {
                "phone" : "+91-7897898765",
                "email" : "shivam-labsamplecollector@gmail.com"
              }
            },
            "state": {
              "descriptor": {
                "code": "appointment-booked",
                "name": "Your appointment has been booked"
              }
            }
          }
        ],
        "quote": {
          "price": {
            "value": "3000",
            "currency": "INR"
          },
          "breakup": [
            {
              "title": "ECG Procedure",
              "price": {
                "currency": "INR",
                "value": "3000"
              }
            },
            {
              "title": "taxes",
              "price": {
                "currency": "INR",
                "value": "30"
              }
            }
          ]
        },
        "billing": {
            "name": "Rajesh Kumar",
            "address": "Villa 5, Green Valley, Malleshwaram, 560012",
            "email": "rajesh.kumar@example.com",
            "phone": "+91-9999999999"
        },
        "payments": [
          {
            "type": "PRE-FULFILLMENT",
            "collected_by": "BPP",
            "status": "PAID",
            "params": {
                "url": "payto://ban/98273982749428?amount=INR:3030&ifsc=SBIN0000575&message=payment",
                "transaction_id" : "ABCD2414143231",
                "amount": "3030",
                "currency": "INR"
            }
          }
        ],
        "cancellation_terms": [
          {
            "fulfillment_state": {
              "descriptor": {
                "code": "in-progress"
              }
            },
            "cancellation_fee": {
              "percentage": "30%"
            },
            "external_ref": {
              "mimetype": "text/html",
              "url": "https://giolabs.in/charge/tnc.html"
            }
          }
        ]
      }
    }
  }
```

## 2.3 Fulfillment of DHP Services (Diagnostics)
This section contains recommendations for implementing the APIs related to fulfilling a DHP service (Diagnostics).

### 2.3.1 Recommendations for BPPs

#### 2.3.1.1 Sending status updates
- REQUIRED. The BPP MUST implement the `status` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.3.1.2 Updating a Diagnostic booking
- REQUIRED. The BPP MUST implement the `update` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.3.1.3 Cancelling a Diagnostic booking
- REQUIRED. The BPP MUST implement the `cancel` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_cancellation_reasons` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.3.1.4 Real-time tracking
- REQUIRED. The BPP MUST implement the `track` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 2.3.2 Recommendations for BAPs

#### 2.3.2.1 Receiving status updates
- REQUIRED. The BAP MUST implement the `on_status` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `status`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.3.2.2 Updating a Diagnostic booking
- REQUIRED. The BAP MUST implement the `on_update` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `update`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.3.2.3 Cancelling a Diagnostic booking
- REQUIRED. The BAP MUST implement the `on_cancel` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `cancel`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `cancellation_reasons` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_cancellation_reasons`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.3.2.4 Real-time tracking
- REQUIRED. The BAP MUST implement the `on_track` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `track`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 2.3.3 Example Workflow

### 2.3.4 Example Requests

Below is an example of a `status` request
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "status",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_uri": "https://ps-bap-network.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "order_id": "be69bb43-8286-48b4-801f-a8c5b41ad450"
    }
}
```

Below is an example of an `on_status` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_status",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io/",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "id": "be69bb43-8286-48b4-801f-a8c5b41ad450",
        "provider": {
            "id": "289edce4-d002-4962-b311-4c025e22b4f6",
            "descriptor": {
              "name": "GioLabs Pvt Ltd",
              "short_desc": "X-rays, tests and more",
              "long_desc" : "Advanced diagnostics & precise testing at XYZ Pathology Lab. Fast, accurate results for informed healthcare decisions.",
              "images": [
                {
                  "url": "https://giolabs.in/images/logo.png"
                }
              ],
              "media" : [
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/237402938409485039850935"
                },
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/2374029384094850394240935"
                }
              ]
            },
            "rating": "4.5"
        },
        "items": [
            {
                "id": "lab-test-01",
                "descriptor": {
                    "code": "ecg",
                    "name": "Electrocardiogram test"
                },
                "price": {
                    "value": "3000",
                    "currency": "INR"
                },
                "category_ids": [
                    "cat-01"
                ],
                "fulfillment_ids": [
                    "ful-01",
                    "ful-02"
                ]
            }
        ],
        "fulfillments": [
          {
            "id": "ful-01",
            "type": "walk-in",
            "stops": [
                {
                  "type": "end",
                  "time": {
                    "range" : {
                      "start" : "0000-00-00T10:00:00Z",
                      "end" : "0000-00-00T18:00:00Z"
                    },
                    "days": "1,2,3,4,5,6,7",
                    "schedule" : {
                      "holidays" : [
                          "2024-12-10",
                          "2024-12-11",
                          "2024-12-12"
                      ]
                    }
                  },
                  "location": {
                      "gps": "12.9716° N, 77.5946° E",
                      "address": "Akashya nagar B17/14",
                      "state": {
                          "name": "Madhya Pradesh"
                      },
                      "city": {
                          "name": "Bhopal"
                      },
                      "area_code": "462001"
                  }
                }
            ],
            "customer": {
                "person": {
                    "name": "Rajesh Kumar",
                    "age": "26",
                    "gender": "M",
                    "dob": "1995-09-11"
                },
                "contact" :{
                    "phone" : "+91-9999999999",
                    "email" : "rajesh.kumar@example.com"
                }
            },
            "agent": {
              "person": {
                "id": "237402938409485039850935",
                "name": "Mr. Shivam Arora",
                "gender": "male"
              },
              "contact" : {
                "phone" : "+91-7897898765",
                "email" : "shivam-labsamplecollector@gmail.com"
              }
            },
            "state": {
              "descriptor": {
                "code": "test-completed",
                "name": "Your Test is completed and sample has been collected"
              }
            }
          }
        ],
        "quote": {
          "price": {
            "value": "3000",
            "currency": "INR"
          },
          "breakup": [
            {
              "title": "ECG Procedure",
              "price": {
                "currency": "INR",
                "value": "3000"
              }
            },
            {
              "title": "taxes",
              "price": {
                "currency": "INR",
                "value": "30"
              }
            }
          ]
        },
        "billing": {
            "name": "Rajesh Kumar",
            "address": "Villa 5, Green Valley, Malleshwaram, 560012",
            "email": "rajesh.kumar@example.com",
            "phone": "+91-9999999999"
        },
        "payments": [
          {
            "type": "PRE-FULFILLMENT",
            "collected_by": "BPP",
            "status": "PAID",
            "params": {
                "url": "payto://ban/98273982749428?amount=INR:3030&ifsc=SBIN0000575&message=payment",
                "transaction_id" : "ABCD2414143231",
                "amount": "3030",
                "currency": "INR"
            }
          }
        ],
        "cancellation_terms": [
          {
            "fulfillment_state": {
              "descriptor": {
                "code": "in-progress"
              }
            },
            "cancellation_fee": {
              "percentage": "30%"
            },
            "external_ref": {
              "mimetype": "text/html",
              "url": "https://giolabs.in/charge/tnc.html"
            }
          }
        ]
      }
    }
  }
```

Below is an example of a `track` request

```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "track",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_uri": "https://ps-bap-network.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "order_id": "be69bb43-8286-48b4-801f-a8c5b41ad450",
        "callback_url": "https://dhp-network-bap.becknprotocol.io/track/callback"
    }
}
```
Below is an example of an `on_track` callback
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "on_track",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_url": "https://ps-bap-client.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "tracking": {
            "id": "853c7593-f4bf-4557-8832-118a591787ba",
            "url": "https://quickdiagnostic-provider.com/track/3210fedcba98",
            "status": "active",
            "location": {
                "descriptor": {
                    "name": "Last location of the Lab agent. ( incase of doorstep service)"
                },
                "gps": "12.971598, 77.594566"
            }
        }
    }
}

```
Below is an example of a `update` request

```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "update",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_uri": "https://ps-bap-network.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "update_target": "order.billing",
        "order": {
            "id": "be69bb43-8286-48b4-801f-a8c5b41ad450",
            "billing": {
                "name": "Rajesh Kumar",
                "address": "Villa 5, Green Valley, Malleshwaram, 560012",
                "email": "rajesh.kumar@example.com",
                "phone": "+91-9999999999"
            }
        }
    }
}

```
Below is an example of an `on_update` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_status",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io/",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "id": "be69bb43-8286-48b4-801f-a8c5b41ad450",
        "provider": {
            "id": "289edce4-d002-4962-b311-4c025e22b4f6",
            "descriptor": {
              "name": "GioLabs Pvt Ltd",
              "short_desc": "X-rays, tests and more",
              "long_desc" : "Advanced diagnostics & precise testing at XYZ Pathology Lab. Fast, accurate results for informed healthcare decisions.",
              "images": [
                {
                  "url": "https://giolabs.in/images/logo.png"
                }
              ],
              "media" : [
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/237402938409485039850935"
                },
                {
                    "mimetype": "application/pdf",
                    "url": "https://www.aiims.com/honours/2374029384094850394240935"
                }
              ]
            },
            "rating": "4.5"
        },
        "items": [
            {
                "id": "lab-test-01",
                "descriptor": {
                    "code": "ecg",
                    "name": "Electrocardiogram test"
                },
                "price": {
                    "value": "3000",
                    "currency": "INR"
                },
                "category_ids": [
                    "cat-01"
                ],
                "fulfillment_ids": [
                    "ful-01",
                    "ful-02"
                ]
            }
        ],
        "fulfillments": [
          {
            "id": "ful-01",
            "type": "walk-in",
            "stops": [
                {
                  "type": "end",
                  "time": {
                    "range" : {
                      "start" : "0000-00-00T10:00:00Z",
                      "end" : "0000-00-00T18:00:00Z"
                    },
                    "days": "1,2,3,4,5,6,7",
                    "schedule" : {
                      "holidays" : [
                          "2024-12-10",
                          "2024-12-11",
                          "2024-12-12"
                      ]
                    }
                  },
                  "location": {
                      "gps": "12.9716° N, 77.5946° E",
                      "address": "Akashya nagar B17/14",
                      "state": {
                          "name": "Madhya Pradesh"
                      },
                      "city": {
                          "name": "Bhopal"
                      },
                      "area_code": "462001"
                  }
                }
            ],
            "customer": {
                "person": {
                    "name": "Rajesh Kumar",
                    "age": "26",
                    "gender": "M",
                    "dob": "1995-09-11"
                },
                "contact" :{
                    "phone" : "+91-9999999999",
                    "email" : "rajesh.kumar@example.com"
                }
            },
            "agent": {
              "person": {
                "id": "237402938409485039850935",
                "name": "Mr. Shivam Arora",
                "gender": "male"
              },
              "contact" : {
                "phone" : "+91-7897898765",
                "email" : "shivam-labsamplecollector@gmail.com"
              }
            },
            "state": {
              "descriptor": {
                "code": "billing-updated",
                "name": "Billing Address Updated"
              }
            }
          }
        ],
        "quote": {
          "price": {
            "value": "3000",
            "currency": "INR"
          },
          "breakup": [
            {
              "title": "ECG Procedure",
              "price": {
                "currency": "INR",
                "value": "3000"
              }
            },
            {
              "title": "taxes",
              "price": {
                "currency": "INR",
                "value": "30"
              }
            }
          ]
        },
        "billing": {
            "name": "Rajesh Kumar",
            "address": "Villa 5, Green Valley, Malleshwaram, 560012",
            "email": "rajesh.kumar@example.com",
            "phone": "+91-9999999999"
        },
        "payments": [
          {
            "type": "PRE-FULFILLMENT",
            "collected_by": "BPP",
            "status": "PAID",
            "params": {
                "url": "payto://ban/98273982749428?amount=INR:3030&ifsc=SBIN0000575&message=payment",
                "transaction_id" : "ABCD2414143231",
                "amount": "3030",
                "currency": "INR"
            }
          }
        ],
        "cancellation_terms": [
          {
            "fulfillment_state": {
              "descriptor": {
                "code": "in-progress"
              }
            },
            "cancellation_fee": {
              "percentage": "30%"
            },
            "external_ref": {
              "mimetype": "text/html",
              "url": "https://giolabs.in/charge/tnc.html"
            }
          }
        ]
      }
    }
  }
```

## 2.4 Post-fulfillment of DHP Services(Diagnostics)
This section contains recommendations for implementing the APIs after fulfilling a DHP service

### 2.4.1 Recommendations for BPPs

#### 2.4.1.1 Rating and Feedback
- REQUIRED. The BPP MUST implement the `rating` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_rating_categories` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.4.1.2 Providing Support
- REQUIRED. The BPP MUST implement the `support` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 2.4.2 Recommendations for BAPs

#### 2.4.2.1 Rating and Feedback
- REQUIRED. The BAP MUST implement the `on_rating` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `rating`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `rating_categories` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_rating_categories`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 2.4.2.2 Providing Support
- REQUIRED. The BAP MUST implement the `on_support` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `support`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 2.4.3 Example Workflow

### 2.4.4 Example Requests

Below is an example of a `rating` request

```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "rating",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_uri": "https://ps-bap-network.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "ratings": [
            {
                "id": "lab-test-01",
                "rating_category": "Item",
                "value" : "5"
            }
        ]
    }
}
```
Below is an example of an `on_rating` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_rating",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io//",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "feedback_form" : {
            "form" : {
                "url" : "url of the feedback form",
                "mime_type" :"text/html"
            },
            "required" : true
        } 
    }
}
```

Below is an example of a `support` request
```
{
    "context": {
        "domain": "dhp:diagnostics:0.1.0",
        "action": "support",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_uri": "https://ps-bap-network.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "support": {
            "order_id" : "be69bb43-8286-48b4-801f-a8c5b41ad450",
            "callback_phone" : "+91-8858150053"
        }
    }
}
```

Below is an example of an `on_support` callback
```
{
    "context": {
      "domain": "dhp:diagnostics:0.1.0",
      "action": "on_support",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "ps-bap-network.becknprotocol.io",
      "bap_url": "https://ps-bap-client.becknprotocol.io/",
      "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
      "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "support": {
        "ref_id" : "Abjhjh13773",
        "order_id" : "be69bb43-8286-48b4-801f-a8c5b41ad450",
        "callback_phone" : "+91-8858150053",
        "email" : "support@ekstep.com",
        "phone" : "+91-965676879",
        "url" : "chat-url-for-support",
        "docs": [
            {
                "descriptor":{
                    "name" :"FAQs",
                    "short_desc" : "Frequently asked questions and common issues"
                },
                "url" : "https://link-to-the-document.com",
                "mime_type" : "application/pdf"
            }
        ]
      }
    }
  }
```