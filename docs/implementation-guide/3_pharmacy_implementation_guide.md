# 3. Implementation Guide

This document contains the REQUIRED and RECOMMENDED standard functionality that must be implemented by any DHP Service Provider Platform a.k.a BPPs and DHP Consumer Platform a.k.a BAPs.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

## 3.1 Discovery of Pharmacy Products

### 3.1.1 Recommendations for BPPs
The following recommendations need to be considered when implementing discovery functionality for a DHP BPP

- REQUIRED. The BPP MUST implement the `search` endpoint to receive an `Intent` object sent by BAPs
- REQUIRED. The BPP MUST return a catalog of DHP products on the `on_search` callback endpoint specified in the `context.bap_uri` field of the `search` request body.
- REQUIRED. The BPP MUST map its pharmacy items (medicines) to the `Item` schema.
- REQUIRED. Any DHP service provider-related information like name, logo, short description must be mapped to the `Provider.descriptor` schema
- REQUIRED. Any form that must be filled before receiving a quotation must be mapped to the `XInput` schema
- REQUIRED. If the platform wants to group its products under a specific category, it must map each category to the `Category` schema
- REQUIRED. Any service fulfillment-related information MUST be mapped to the `Fulfillment` schema.
- REQUIRED. If the BPP does not want to respond to a search request, it MUST return a `ack.status` value equal to `NACK`
- RECOMMENDED. Upon receiving a `search` request, the BPP SHOULD return a catalog that best matches the intent. This can be done by indexing the catalog against the various probable paths in the `Intent` schema relevant to typical DHP use cases

### 3.1.2 Recommendations for BAPs
- REQUIRED. The BAP MUST call the `search` endpoint of the BG to discover multiple BPPs on a network
- REQUIRED. The BAP MUST implement the `on_search` endpoint to consume the `Catalog` objects containing DHP Products sent by BPPs.
- REQUIRED. The BAP MUST expect multiple catalogs sent by the respective DHP Providers on the network
- REQUIRED. The DHP products can be found in the `Catalog.providers[].items[]` array in the `on_search` request
- REQUIRED. If the `catalog.providers[].items[].xinput` object is present, then the BAP MUST redirect the user to, or natively render the form present on the link specified on the `items[].xinput.form.url` field.
- REQUIRED. If the `catalog.providers[].items[].xinput.required` field is set to `"true"` , then the BAP MUST NOT fire a `select`, `init` or `confirm` call until the form is submitted and a successful response is received
- RECOMMENDED. If the `catalog.providers[].items[].xinput.required` field is set to `"false"` , then the BAP SHOULD allow the user to skip filling the form


### Example
A search request for a pharmacy item may look like this
```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "search",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "intent": {
        "category": {
          "id": "pharmacy"
        },
        "location": {
          "circle" : {
            "gps": "12.423423,77.325647",
            "radius": {
              "type": "CONSTANT",
              "value": "5",
              "unit": "km"
            }
          }
        },
        "item": {
          "descriptor" :{
            "name": "paracetamol"
          }
        }
      }
    }
}
```

An example catalog of pharmacy product may look like this
```
{
  "context": {
    "domain": "dhp:pharmacy:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "catalog": {
      "descriptor": {
        "name": "Available Pharmacy Stores"
      },
      "providers": [
        {
          "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
          "descriptor": {
            "name": "QuickPharma",
            "short_desc": "QuickPharma Pvt Ltd",
            "images": [
              {
                "url": "https://QuickPharma.in/images/logo.png"
              }
            ]
          },
          "categories": [
            {
              "id": "cat-01",
              "descriptor": {
                "code": "pharmacy",
                "name": "Pharmacy"
              }
            }
          ],
          "locations": [
            {
              "id": "phr-loc-01",
              "gps": "12.9164682,77.6089985"
            },
            {
              "id": "phr-loc-02",
              "gps": "12.91671,77.6092983"
            }
          ],
          "fulfillments": [
            {
              "id": "ful-01",
              "type": "IN-STORE-PICKUP",
              "stops": [
                {
                  "type": "start",
                  "time": {
                    "timestamp": "2023-09-15T08:00:00Z"
                  }
                },
                {
                  "type": "end",
                  "time": {
                    "timestamp": "2023-09-15T18:30:00Z"
                  }
                }
              ]
            },
            {
              "id": "ful-02",
              "type": "HOME-DELIVERY",
              "stops": [
                {
                  "type": "start",
                  "time": {
                    "timestamp": "2023-09-15T08:00:00Z"
                  }
                },
                {
                  "type": "end",
                  "time": {
                    "timestamp": "2023-09-15T18:30:00Z"
                  }
                }
              ]
            }
          ],
          "items": [
            {
              "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
              "descriptor": {
                "code": "benadryl",
                "name": "Benadryl cough syrup"
              },
              "category_ids": [
                "cat-01"
              ],
              "fulfillment_ids": [
                "ful-02",
                "ful-01"
              ],
              "location_ids": [
                "phr-loc-01"
              ],
              "price": {
                "value": "100",
                "currency": "INR"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "1000",
                    "unit": "units"
                  }
                },
                "maximum": {
                  "measure": {
                    "value": "5",
                    "unit": "units"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": {
                    "code": "symptoms",
                    "name": "Works against these symptoms"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "symptom-list"
                      },
                      "value": "cough, code, nausea, headache, phelgm"
                    }
                  ]
                }
              ]
            },
            {
              "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
              "descriptor": {
                "code": "hycodan",
                "name": "Hycodan"
              },
              "category_ids": [
                "cat-01"
              ],
              "fulfillment_ids": [
                "ful-02",
                "ful-01"
              ],
              "location_ids": [
                "phr-loc-01"
              ],
              "price": {
                "value": "150",
                "currency": "INR"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "1000",
                    "unit": "units"
                  }
                },
                "maximum": {
                  "measure": {
                    "value": "5",
                    "unit": "units"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": {
                    "code": "symptoms",
                    "name": "Works against these symptoms"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "symptom-list"
                      },
                      "value": "cough, code, nausea, headache, phelgm"
                    }
                  ]
                }
              ]
            },
            {
              "id": "5cbc6e9d-28f4-42c7-b81e-41a336ac96ea",
              "descriptor": {
                "code": "mucinex-dm",
                "name": "Mucinex DM"
              },
              "category_ids": [
                "cat-01"
              ],
              "fulfillment_ids": [
                "ful-02",
                "ful-01"
              ],
              "location_ids": [
                "phr-loc-01"
              ],
              "price": {
                "value": "180",
                "currency": "INR"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "1000",
                    "unit": "units"
                  }
                },
                "maximum": {
                  "measure": {
                    "value": "5",
                    "unit": "units"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": {
                    "code": "symptoms",
                    "name": "Works against these symptoms"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "symptom-list"
                      },
                      "value": "cough, code, nausea, headache, phelgm"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  }
}
```

## 3.2 Ordering for DHP(Pharmacy) 
This section provides recommendations for implementing the APIs related to ordering a pharmacy product.

### 3.2.1 Recommendations for BPPs

#### 3.2.1.1 Selecting a medicine (pharmacy product) from the catalog
- REQUIRED. The BPP MUST implement the `select` endpoint on the url specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry.
- REQUIRED. If the DHP provider has returned a successful acknowledgement to a `select` request, it MUST send the offer encapsulated in an `Order` object

#### 3.2.1.2 Initializing Order for the medicine (pharmacy product)
- REQUIRED. The BPP MUST implement the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry.

#### 3.2.1.3 Confirming Order for the medicine (pharmacy product)
- REQUIRED. The BPP MUST implement the `confirm` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 3.2.2 Recommendations for BAPs

#### 3.2.2.1 Selecting a medicine (pharmacy product) from the catalog
- REQUIRED. The BAP MUST implement the `on_select` endpoint on the url specified in the `context.bap_uri` field sent during `select`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.2.2.2  Initializing Order for the medicine (pharmacy product)
- REQUIRED. The BAP MUST hit the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_init` endpoint on the url specified in  the `context.bap_uri` field sent during `init`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry


#### 3.2.2.3 Confirming the Order for the medicine (pharmacy product)
- REQUIRED. The BAP MUST hit the `confirm` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_confirm` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `confirm`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 3.2.3 Example Workflow

### 3.2.3 Example Requests

Below is an example of a `select` request
```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "select",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "provider": {
          "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c"
        },
        "items": [
          {
            "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
            "quantity": {
              "selected": {
                "measure": {
                  "value": "2",
                  "unit": "units"
                }
              }
            }
          },
          {
            "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
            "quantity": {
              "selected": {
                "measure": {
                  "value": "3",
                  "unit": "units"
                }
              }
            }
          }
        ],
        "fulfillments": [
          {
            "id": "ful-02",
            "stops": [
              {
                "location": {
                  "gps": "1.3806217468119772, 103.74636438437074",
                  "area_code": "680230"
                }
              }
            ]
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
    "domain": "dhp:pharmacy:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
        "descriptor": {
          "name": "QuickPharma",
          "short_desc": "QuickPharma Pvt Ltd",
          "images": [
            {
              "url": "https://QuickPharma.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
          "descriptor": {
            "code": "benadryl",
            "name": "Benadryl cough syrup"
          },
          "price": {
            "value": "100",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "2",
                "unit": "units"
              }
            }
          }
        },
        {
          "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
          "descriptor": {
            "code": "hycodan",
            "name": "Hycodan"
          },
          "price": {
            "value": "150",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "3",
                "unit": "units"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-02",
          "type": "HOME-DELIVERY",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-09-15T08:00:00Z"
              }
            },
            {
              "type": "end",
              "time": {
                "timestamp": "2023-09-15T18:30:00Z"
              }
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "650",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Benadryl 2 units",
            "price": {
              "currency": "INR",
              "value": "200"
            }
          },
          {
            "title": "hycodan 3 units",
            "price": {
              "currency": "INR",
              "value": "450"
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
      "domain": "dhp:pharmacy:0.1.0",
      "action": "init",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "provider": {
          "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c"
        },
        "items": [
          {
            "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
            "quantity": {
              "selected": {
                "measure": {
                  "value": "2",
                  "unit": "units"
                }
              }
            }
          },
          {
            "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
            "quantity": {
              "selected": {
                "measure": {
                  "value": "3",
                  "unit": "units"
                }
              }
            }
          }
        ],
        "fulfillments": [
          {
            "id": "ful-02",
            "customer": {
              "contact": {
                "email": "fox.judie@abc.org",
                "phone": "+91-9999999999"
              },
              "person": {
                "name": "Judie Fox"
              }
            },
            "stops": [
              {
                "location": {
                  "gps": "1.3806217468119772, 103.74636438437074",
                  "address": "My House #, My building",
                  "city": {
                    "name": "Jurong East"
                  },
                  "country": {
                    "code": "SGP"
                  },
                  "area_code": "680230",
                  "state": {
                    "name": "bayern"
                  }
                },
                "contact": {
                  "phone": "9886098860"
                }
              }
            ]
          }
        ],
        "billing": {
          "name": "Alice Smith",
          "address": "Apt 303, Maple Towers, Richmond Road, 560001",
          "state": {
            "name": "Jurong East"
          },
          "city": {
            "name": "Jurong East"
          },
          "email": "alice.smith@example.com",
          "phone": "9886098860"
        },
        "docs": [
          {
            "descriptor": {
              "code": "prescription",
              "name": "Medicine prescription",
              "short_desc": "Prescription uploaded here"
            },
            "mime_type": "application/pdf",
            "url": "https://quickpharma.in/prescription/04389d8c-6a50-4664-9c08-4ee45fef44e8.pdf"
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
    "domain": "dhp:pharmacy:0.1.0",
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
        "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
        "descriptor": {
          "name": "QuickPharma",
          "short_desc": "QuickPharma Pvt Ltd",
          "images": [
            {
              "url": "https://quickPharma.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
          "descriptor": {
            "code": "benadryl",
            "name": "Benadryl cough syrup"
          },
          "price": {
            "value": "100",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "2",
                "unit": "units"
              }
            }
          }
        },
        {
          "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
          "descriptor": {
            "code": "hycodan",
            "name": "Hycodan"
          },
          "price": {
            "value": "150",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "3",
                "unit": "units"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-02",
          "type": "HOME-DELIVERY",
          "customer": {
            "contact": {
              "email": "fox.judie@abc.org",
              "phone": "+91-9999999999"
            },
            "person": {
              "name": "Judie Fox"
            }
          },
          "stops": [
            {
              "location": {
                "gps": "1.3806217468119772, 103.74636438437074",
                "address": "My House #, My building",
                "city": {
                  "name": "Jurong East"
                },
                "country": {
                  "code": "SGP"
                },
                "area_code": "680230",
                "state": {
                  "name": "bayern"
                }
              },
              "contact": {
                "phone": "9886098860"
              }
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "700",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Benadryl 2 units",
            "price": {
              "currency": "INR",
              "value": "200"
            }
          },
          {
            "title": "hycodan 3 units",
            "price": {
              "currency": "INR",
              "value": "450"
            }
          },
          {
            "title": "delivery charge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "billing": {
        "name": "Alice Smith",
        "address": "Apt 303, Maple Towers, Richmond Road, 560001",
        "state": {
          "name": "Jurong East"
        },
        "city": {
          "name": "Jurong East"
        },
        "email": "alice.smith@example.com",
        "phone": "9886098860"
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "700",
            "currency": "INR",
            "bank_account_number": "1234002341",
            "bank_code": "INB0004321",
            "bank_account_name": "Strapi BPP Limited"
          },
          "status": "NOT-PAID",
          "type": "PRE-ORDER"
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
            "url": "https://quickPharma.in/charge/tnc.html"
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
      "domain": "dhp:pharmacy:0.1.0",
      "action": "confirm",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order": {
        "provider": {
          "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c"
        },
        "items": [
          {
            "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
            "quantity": {
              "selected": {
                "measure": {
                  "value": "2",
                  "unit": "units"
                }
              }
            }
          },
          {
            "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
            "quantity": {
              "selected": {
                "measure": {
                  "value": "3",
                  "unit": "units"
                }
              }
            }
          }
        ],
        "fulfillments": [
          {
            "id": "ful-02",
            "customer": {
              "contact": {
                "email": "fox.judie@abc.org",
                "phone": "+91-9999999999"
              },
              "person": {
                "name": "Judie Fox"
              }
            },
            "stops": [
              {
                "location": {
                  "gps": "1.3806217468119772, 103.74636438437074",
                  "address": "My House #, My building",
                  "city": {
                    "name": "Jurong East"
                  },
                  "country": {
                    "code": "SGP"
                  },
                  "area_code": "680230",
                  "state": {
                    "name": "bayern"
                  }
                },
                "contact": {
                  "phone": "9886098860"
                }
              }
            ]
          }
        ],
        "billing": {
          "name": "Alice Smith",
          "address": "Apt 303, Maple Towers, Richmond Road, 560001",
          "state": {
            "name": "Jurong East"
          },
          "city": {
            "name": "Jurong East"
          },
          "email": "alice.smith@example.com",
          "phone": "9886098860"
        },
        "payments": [
          {
            "collected_by": "BPP",
            "params": {
              "amount": "700",
              "currency": "INR",
              "bank_account_number": "1234002341",
              "bank_code": "INB0004321",
              "bank_account_name": "Strapi BPP Limited"
            },
            "status": "PAID",
            "type": "PRE-ORDER",
            "transaction_id": "a35b56cf-e5cf-41f1-9b5d-fa99d8d5ac8c"
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
    "domain": "dhp:pharmacy:0.1.0",
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
      "id": "853c7593-f4bf-4557-8832-118a591787ba",
      "provider": {
        "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
        "descriptor": {
          "name": "QuickPharma",
          "short_desc": "QuickPharma Pvt Ltd",
          "images": [
            {
              "url": "https://quickPharma.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
          "descriptor": {
            "code": "benadryl",
            "name": "Benadryl cough syrup"
          },
          "price": {
            "value": "100",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "2",
                "unit": "units"
              }
            }
          }
        },
        {
          "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
          "descriptor": {
            "code": "hycodan",
            "name": "Hycodan"
          },
          "price": {
            "value": "150",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "3",
                "unit": "units"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-02",
          "type": "HOME-DELIVERY",
          "customer": {
            "contact": {
              "email": "fox.judie@abc.org",
              "phone": "+91-9999999999"
            },
            "person": {
              "name": "Judie Fox"
            }
          },
          "stops": [
            {
              "location": {
                "gps": "1.3806217468119772, 103.74636438437074",
                "address": "My House #, My building",
                "city": {
                  "name": "Jurong East"
                },
                "country": {
                  "code": "SGP"
                },
                "area_code": "680230",
                "state": {
                  "name": "bayern"
                }
              },
              "contact": {
                "phone": "9886098860"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "order-confirm",
              "name": "Order has been confirmed"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "700",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Benadryl 2 units",
            "price": {
              "currency": "INR",
              "value": "200"
            }
          },
          {
            "title": "hycodan 3 units",
            "price": {
              "currency": "INR",
              "value": "450"
            }
          },
          {
            "title": "delivery charge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "billing": {
        "name": "Alice Smith",
        "address": "Apt 303, Maple Towers, Richmond Road, 560001",
        "state": {
          "name": "Jurong East"
        },
        "city": {
          "name": "Jurong East"
        },
        "email": "alice.smith@example.com",
        "phone": "9886098860"
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "700",
            "currency": "INR",
            "bank_account_number": "1234002341",
            "bank_code": "INB0004321",
            "bank_account_name": "Strapi BPP Limited"
          },
          "status": "PAID",
          "type": "PRE-ORDER",
          "transaction_id": "a35b56cf-e5cf-41f1-9b5d-fa99d8d5ac8c"
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
            "url": "https://quickPharma.in/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

## 3.3 Fulfillment of medicine (pharmacy product)
This section contains recommendations for implementing the APIs related to fulfilling medicine (pharmacy product)

### 3.3.1 Recommendations for BPPs

#### 3.3.1.1 Sending status updates
- REQUIRED. The BPP MUST implement the `status` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.3.1.2 Updating an order
- REQUIRED. The BPP MUST implement the `update` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.3.1.3 Cancelling an order
- REQUIRED. The BPP MUST implement the `cancel` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_cancellation_reasons` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.3.1.4 Real-time tracking
- REQUIRED. The BPP MUST implement the `track` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 3.3.2 Recommendations for BAPs

#### 3.3.2.1 Receiving status updates
- REQUIRED. The BAP MUST implement the `on_status` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `status`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.3.2.2 Updating an order
- REQUIRED. The BAP MUST implement the `on_update` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `update`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.3.2.3 Cancelling an order
- REQUIRED. The BAP MUST implement the `on_cancel` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `cancel`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `cancellation_reasons` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_cancellation_reasons`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.3.2.4 Real-time tracking
- REQUIRED. The BAP MUST implement the `on_track` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `track`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 3.3.3 Example Workflow

### 3.3.4 Example Requests

Below is an example of a `status` request
```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "status",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order_id": "853c7593-f4bf-4557-8832-118a591787ba"
    }
  }
```

Below is an example of an `on_status` callback
```
{
  "context": {
    "domain": "dhp:pharmacy:0.1.0",
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
      "id": "853c7593-f4bf-4557-8832-118a591787ba",
      "provider": {
        "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
        "descriptor": {
          "name": "QuickPharma",
          "short_desc": "QuickPharma Pvt Ltd",
          "images": [
            {
              "url": "https://quickPharma.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
          "descriptor": {
            "code": "benadryl",
            "name": "Benadryl cough syrup"
          },
          "price": {
            "value": "100",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "2",
                "unit": "units"
              }
            }
          }
        },
        {
          "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
          "descriptor": {
            "code": "hycodan",
            "name": "Hycodan"
          },
          "price": {
            "value": "150",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "3",
                "unit": "units"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-02",
          "type": "HOME-DELIVERY",
          "customer": {
            "contact": {
              "email": "fox.judie@abc.org",
              "phone": "+91-9999999999"
            },
            "person": {
              "name": "Judie Fox"
            }
          },
          "stops": [
            {
              "location": {
                "gps": "1.3806217468119772, 103.74636438437074",
                "address": "My House #, My building",
                "city": {
                  "name": "Jurong East"
                },
                "country": {
                  "code": "SGP"
                },
                "area_code": "680230",
                "state": {
                  "name": "bayern"
                }
              },
              "contact": {
                "phone": "9886098860"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "order-packed",
              "name": "Order has been packed and can be picked up from the store"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "700",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Benadryl 2 units",
            "price": {
              "currency": "INR",
              "value": "200"
            }
          },
          {
            "title": "hycodan 3 units",
            "price": {
              "currency": "INR",
              "value": "450"
            }
          },
          {
            "title": "delivery charge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "billing": {
        "name": "Alice Smith",
        "address": "Apt 303, Maple Towers, Richmond Road, 560001",
        "state": {
          "name": "Jurong East"
        },
        "city": {
          "name": "Jurong East"
        },
        "email": "alice.smith@example.com",
        "phone": "9886098860"
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "700",
            "currency": "INR",
            "bank_account_number": "1234002341",
            "bank_code": "INB0004321",
            "bank_account_name": "Strapi BPP Limited"
          },
          "status": "PAID",
          "type": "PRE-ORDER",
          "transaction_id": "a35b56cf-e5cf-41f1-9b5d-fa99d8d5ac8c"
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
            "url": "https://quickPharma.in/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

Below is an example of a `cancel` request

```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "cancel",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "order_id": "853c7593-f4bf-4557-8832-118a591787ba",
      "cancellation_reason_id": "4",
      "descriptor": {
        "short_desc": "Prescription Changed",
        "long_desc": "The patient's prescription has been changed, and the medication order is no longer valid."
      }
    }
  }
```
Below is an example of an `on_cancel` callback
```
{
  "context": {
    "domain": "dhp:pharmacy:0.1.0",
    "action": "on_cancel",
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
      "id": "853c7593-f4bf-4557-8832-118a591787ba",
      "provider": {
        "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
        "descriptor": {
          "name": "QuickPharma",
          "short_desc": "QuickPharma Pvt Ltd",
          "images": [
            {
              "url": "https://quickPharma.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
          "descriptor": {
            "code": "benadryl",
            "name": "Benadryl cough syrup"
          },
          "price": {
            "value": "100",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "2",
                "unit": "units"
              }
            }
          }
        },
        {
          "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
          "descriptor": {
            "code": "hycodan",
            "name": "Hycodan"
          },
          "price": {
            "value": "150",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "3",
                "unit": "units"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-02",
          "type": "HOME-DELIVERY",
          "customer": {
            "contact": {
              "email": "fox.judie@abc.org",
              "phone": "+91-9999999999"
            },
            "person": {
              "name": "Judie Fox"
            }
          },
          "stops": [
            {
              "location": {
                "gps": "1.3806217468119772, 103.74636438437074",
                "address": "My House #, My building",
                "city": {
                  "name": "Jurong East"
                },
                "country": {
                  "code": "SGP"
                },
                "area_code": "680230",
                "state": {
                  "name": "bayern"
                }
              },
              "contact": {
                "phone": "9886098860"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "order-cancelled",
              "name": "Order has been cancelled"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "700",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Benadryl 2 units",
            "price": {
              "currency": "INR",
              "value": "200"
            }
          },
          {
            "title": "hycodan 3 units",
            "price": {
              "currency": "INR",
              "value": "450"
            }
          },
          {
            "title": "delivery charge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          },
          {
            "title": "cancellation charge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "billing": {
        "name": "Alice Smith",
        "address": "Apt 303, Maple Towers, Richmond Road, 560001",
        "state": {
          "name": "Jurong East"
        },
        "city": {
          "name": "Jurong East"
        },
        "email": "alice.smith@example.com",
        "phone": "9886098860"
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "700",
            "currency": "INR",
            "bank_account_number": "1234002341",
            "bank_code": "INB0004321",
            "bank_account_name": "Strapi BPP Limited"
          },
          "status": "PAID",
          "type": "PRE-ORDER",
          "transaction_id": "a35b56cf-e5cf-41f1-9b5d-fa99d8d5ac8c"
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
            "url": "https://quickPharma.in/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```
Below is an example of a `update` request

```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "update",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "update_target": "order.fulfillments[0].stops[0]",
      "order": {
        "id": "853c7593-f4bf-4557-8832-118a591787ba",
        "fulfillments": [
          {
            "stops": [
              {
                "location": {
                  "gps": "1.3806217468119772, 103.74636438437074",
                  "address": "My House #, My building",
                  "city": {
                    "name": "Jurong East"
                  },
                  "country": {
                    "code": "SGP"
                  },
                  "area_code": "680230",
                  "state": {
                    "name": "bayern"
                  }
                },
                "contact": {
                  "phone": "9886098860"
                }
              }
            ]
          }
        ]
      }
    }
  }
```
Below is an example of an `on_update` callback
```
{
  "context": {
    "domain": "dhp:pharmacy:0.1.0",
    "action": "on_update",
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
      "id": "853c7593-f4bf-4557-8832-118a591787ba",
      "provider": {
        "id": "e92e7a6e-1f20-4a8d-b343-956e0d45a48c",
        "descriptor": {
          "name": "QuickPharma",
          "short_desc": "QuickPharma Pvt Ltd",
          "images": [
            {
              "url": "https://quickPharma.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "fb0999b7-7755-46d6-a2ed-b286b7c98436",
          "descriptor": {
            "code": "benadryl",
            "name": "Benadryl cough syrup"
          },
          "price": {
            "value": "100",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "2",
                "unit": "units"
              }
            }
          }
        },
        {
          "id": "1cef39d8-72d0-46f7-99ca-3f18f4bda8e3",
          "descriptor": {
            "code": "hycodan",
            "name": "Hycodan"
          },
          "price": {
            "value": "150",
            "currency": "INR"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "3",
                "unit": "units"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-02",
          "type": "HOME-DELIVERY",
          "customer": {
            "contact": {
              "email": "fox.judie@abc.org",
              "phone": "+91-9999999999"
            },
            "person": {
              "name": "Judie Fox"
            }
          },
          "stops": [
            {
              "location": {
                "gps": "1.3806217468119772, 103.74636438437074",
                "address": "My House #, My building",
                "city": {
                  "name": "Jurong East"
                },
                "country": {
                  "code": "SGP"
                },
                "area_code": "680230",
                "state": {
                  "name": "bayern"
                }
              },
              "contact": {
                "phone": "9886098860"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "drop-location-changed",
              "name": "The drop location for the order has changed"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "700",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Benadryl 2 units",
            "price": {
              "currency": "INR",
              "value": "200"
            }
          },
          {
            "title": "hycodan 3 units",
            "price": {
              "currency": "INR",
              "value": "450"
            }
          },
          {
            "title": "delivery charge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "billing": {
        "name": "Alice Smith",
        "address": "Apt 303, Maple Towers, Richmond Road, 560001",
        "state": {
          "name": "Jurong East"
        },
        "city": {
          "name": "Jurong East"
        },
        "email": "alice.smith@example.com",
        "phone": "9886098860"
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "700",
            "currency": "INR",
            "bank_account_number": "1234002341",
            "bank_code": "INB0004321",
            "bank_account_name": "Strapi BPP Limited"
          },
          "status": "PAID",
          "type": "PRE-ORDER",
          "transaction_id": "a35b56cf-e5cf-41f1-9b5d-fa99d8d5ac8c"
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
            "url": "https://quickPharma.in/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

## 3.4 Post-fulfillment of medicine (pharmacy product)
This section contains recommendations for implementing the APIs after fulfilling a medicine (pharmacy product).

### 3.4.1 Recommendations for BPPs

#### 3.4.1.1 Rating and Feedback
- REQUIRED. The BPP MUST implement the `rating` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_rating_categories` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.4.1.2 Providing Support
- REQUIRED. The BPP MUST implement the `support` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 3.4.2 Recommendations for BAPs

#### 3.4.2.1 Rating and Feedback
- REQUIRED. The BAP MUST implement the `on_rating` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `rating`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `rating_categories` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_rating_categories`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 3.4.2.2 Providing Support
- REQUIRED. The BAP MUST implement the `on_support` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `support`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 3.4.3 Example Workflow

### 3.4.4 Example Requests

Below is an example of a `rating` request

```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "rating",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "ratings": [
        {
          "id": "853c7593-f4bf-4557-8832-118a591787ba",
          "rating_category": "Item",
          "value": "5"
        }
      ]
    }
  }
```
Below is an example of an `on_rating` callback
```
{
  "context": {
    "domain": "dhp:pharmacy:0.1.0",
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
    "feedback_form": {
      "form": {
        "url": "url of the feedback form",
        "mime_type": "text/html"
      },
      "required": true
    }
  }
}
```

Below is an example of a `support` request
```
{
  "context": {
    "domain": "dhp:pharmacy:0.1.0",
    "action": "support",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "{{bap_id}}",
    "bap_uri": "{{bap_uri}}",
    "bpp_id": "{{bpp_id}}",
    "bpp_uri": "{{bpp_uri}}",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "support": {
      "order_id": "853c7593-f4bf-4557-8832-118a591787ba",
      "callback_phone": "+91-8858150053"
    }
  }
}
```

Below is an example of an `on_support` callback
```
{
    "context": {
      "domain": "dhp:pharmacy:0.1.0",
      "action": "on_support",
      "location": {
        "country": {
          "name": "India",
          "code": "IND"
        }
      },
      "city": "std:080",
      "version": "1.1.0",
      "bap_id": "{{bap_id}}",
      "bap_uri": "{{bap_uri}}",
      "bpp_id": "{{bpp_id}}",
      "bpp_uri": "{{bpp_uri}}",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
      "support": {
        "ref_id": "853c7593-f4bf-4557-8832-118a591787ba",
        "order_id": "853c7593-f4bf-4557-8832-118a591787ba",
        "callback_phone": "+91-8858150053",
        "email": "support@ekstep.com",
        "phone": "+91-965676879",
        "url": "chat-url-for-support",
        "docs": [
          {
            "descriptor": {
              "name": "FAQs",
              "short_desc": "Frequently asked questions and common issues"
            },
            "url": "https://link-to-the-document.com",
            "mime_type": "application/pdf"
          }
        ]
      }
    }
}
```