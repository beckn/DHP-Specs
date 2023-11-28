# 1. Implementation Guide

This document contains the REQUIRED and RECOMMENDED standard functionality that must be implemented by any DHP Service Provider Platform a.k.a BPPs and DHP Consumer Platform a.k.a BAPs.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

## 1.1 Discovery of Consultation & OPD Services

### 1.1.1 Recommendations for BPPs
The following recommendations need to be considered when implementing discovery functionality for a DHP BPP

- REQUIRED. The BPP MUST implement the `search` endpoint to receive an `Intent` object sent by BAPs
- REQUIRED. The BPP MUST return a catalog of DHP products on the `on_search` callback endpoint specified in the `context.bap_uri` field of the `search` request body.
- REQUIRED. The BPP MUST map its OPD & consultation service to the `Item` schema.
- REQUIRED. Any DHP service provider-related information like name, logo, short description must be mapped to the `Provider.descriptor` schema
- REQUIRED. Any form that must be filled before receiving a quotation must be mapped to the `XInput` schema
- REQUIRED. If the platform wants to group its products under a specific category, it must map each category to the `Category` schema
- REQUIRED. Any service fulfillment-related information MUST be mapped to the `Fulfillment` schema.
- REQUIRED. If the BPP does not want to respond to a search request, it MUST return a `ack.status` value equal to `NACK`
- RECOMMENDED. Upon receiving a `search` request, the BPP SHOULD return a catalog that best matches the intent. This can be done by indexing the catalog against the various probable paths in the `Intent` schema relevant to typical DHP use cases

### 1.1.2 Recommendations for BAPs
- REQUIRED. The BAP MUST call the `search` endpoint of the BG to discover multiple BPPs on a network
- REQUIRED. The BAP MUST implement the `on_search` endpoint to consume the `Catalog` objects containing DHP Products sent by BPPs.
- REQUIRED. The BAP MUST expect multiple catalogs sent by the respective DHP Providers on the network
- REQUIRED. The DHP products can be found in the `Catalog.providers[].items[]` array in the `on_search` request
- REQUIRED. If the `catalog.providers[].items[].xinput` object is present, then the BAP MUST redirect the user to, or natively render the form present on the link specified on the `items[].xinput.form.url` field.
- REQUIRED. If the `catalog.providers[].items[].xinput.required` field is set to `"true"` , then the BAP MUST NOT fire a `select`, `init` or `confirm` call until the form is submitted and a successful response is received
- RECOMMENDED. If the `catalog.providers[].items[].xinput.required` field is set to `"false"` , then the BAP SHOULD allow the user to skip filling the form


### Example
A search request for a consultation/OPD service may look like this
```
{
    "context": {
        "domain": "dhp:consultation:0.1.0",
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
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "intent": {
            "item": {
                "descriptor": {
                    "name": "Orthopedaedic Surgeon"
                }
            }
        }
    }
}
```

An example catalog of consultation/OPD service may look like this
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
            "name": "Example Hospitals",
            "short_desc": "Example Hospitals Pvt Ltd",
            "images": [
              {
                "url": "https://ExampleHospitals.in/images/logo.png"
              }
            ]
          },
          "categories": [
            {
              "id": "cat-02",
              "descriptor": {
                "code": "orthopaedic",
                "name": "Orthopaedic"
              }
            }
          ],
          "fulfillments": [
            {
              "id": "ful-01",
              "type": "OPD",
              "agent": {
                "person": {
                  "id": "237402938409485039850935",
                  "name": "Dr Asthana",
                  "gender": "male",
                  "creds": [
                    {
                      "id": "237402938409485039850935",
                      "type": "VerifiableCredential",
                      "url": "https://www.aiims.com/honours/237402938409485039850935"
                    }
                  ],
                  "languages": [
                    {
                      "code": "hindi",
                      "name": "Hindi"
                    }
                  ],
                  "skills": [
                    {
                      "code": "mbbs",
                      "name": "MBBS"
                    }
                  ],
                  "tags": [
                    {
                      "display": true,
                      "descriptor": {
                        "code": "about",
                        "name": "About the Doctor"
                      },
                      "list": [
                        {
                          "value": "With over 10 years of experience in Orthopaedics"
                        }
                      ]
                    }
                  ]
                },
                "rating": "4.5"
              },
              "stops": [
                {
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
          "items": [
            {
              "id": "cons-01",
              "descriptor": {
                "code": "consultation",
                "name": "Orthopedaedic Surgeon",
                "short_desc": "Orthopaedic Surgeon",
                "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
              },
              "price": {
                "value": "300",
                "currency": "INR"
              },
              "category_ids": [
                "cat-02"
              ],
              "fulfillment_ids": [
                "ful-01"
              ],
              "time": {
                "duration": "PT30M"
              }
            }
          ]
        }
      ]
    }
  }
}
```

## 1.2 Application for DHP(Consultation) 
This section provides recommendations for implementing the APIs related to booking a consultation/OPD service.

### 1.2.1 Recommendations for BPPs

#### 1.2.1.1 Selecting a DHP service from the catalog
- REQUIRED. The BPP MUST implement the `select` endpoint on the url specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry.
- REQUIRED. If the DHP provider has returned a successful acknowledgement to a `select` request, it MUST send the offer encapsulated in an `Order` object

#### 1.2.1.2 Initializing Appointmnet for the consultation
- REQUIRED. The BPP MUST implement the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. If the DHP provider has returned a successful acknowledgement to the first `init` request, it MUST send the Xinput form in `on_init`.
- REQUIRED. The BPP MUST check for a form submission at the URL specified on the `xinput.form.url` before initiating 2nd `on_init` request.


#### 1.2.1.3 Confirming the Appointmnet for the consultation
- REQUIRED. The BPP MUST implement the `confirm` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.2.2 Recommendations for BAPs

#### 1.2.2.1 Selecting a DHP service from the catalog
- REQUIRED. The BAP MUST implement the `on_select` endpoint on the url specified in the `context.bap_uri` field sent during `select`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.2.2  Initializing Appointmnet for the consultation
- REQUIRED. The BAP MUST hit the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_init` endpoint on the url specified in  the `context.bap_uri` field sent during `init`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP user MUST submit the form on the url received from  `on_init`  under `xinput.form.url`.


#### 1.2.2.3 Confirming the Appointmnet for the consultation
- REQUIRED. The BAP MUST hit the `confirm` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_confirm` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `confirm`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.2.3 Example Workflow

### 1.2.3 Example Requests

Below is an example of a `select` request
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
      "fulfillments": [
        {
          "id": "ful-01"
        }
      ],
      "items": [
        {
          "id": "cons-01"
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
    "domain": "dhp:consultation:0.1.0",
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
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male",
              "creds": [
                {
                  "id": "237402938409485039850935",
                  "type": "VerifiableCredential",
                  "url": "https://www.aiims.com/honours/237402938409485039850935"
                }
              ],
              "languages": [
                {
                  "code": "hindi",
                  "name": "Hindi"
                }
              ],
              "skills": [
                {
                  "code": "mbbs",
                  "name": "MBBS"
                }
              ],
              "tags": [
                {
                  "display": true,
                  "descriptor": {
                    "code": "about",
                    "name": "About the Doctor"
                  },
                  "list": [
                    {
                      "value": "With over 10 years of experience in Orthopaedics"
                    }
                  ]
                }
              ]
            },
            "rating": "4.5"
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Consultancy Slots",
                "duration": "PT30M",
                "range": {
                  "start": "10:00",
                  "end": "18:00"
                },
                "days": "1,2,3,4,5,6",
                "schedule": {
                  "holidays": [
                    "2023-11-11",
                    "2023-11-12",
                    "2023-11-13"
                  ]
                }
              }
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "300",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
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
    "domain": "dhp:consultation:0.1.0",
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
      "fulfillments": [
        {
          "id": "ful-01",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "stops": [
            {
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "items": [
        {
          "id": "cons-01"
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      }
    }
  }
}
```

Below is an example of first `on_init` callback
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          },
          "xinput": {
            "required": true,
            "head": {
              "descriptor": {
                "name": "Medical Information"
              },
              "index": {
                "min": 0,
                "cur": 0,
                "max": 0
              },
              "headings": [
                "Medical Information"
              ]
            },
            "form": {
              "mime_type": "text/html",
              "url": "https://6vs8xnx5i7.ExampleHospitals.co.in/medical-details/xinput/formid/a23f2fdfbbb8ac402bfd54f",
              "resubmit": false,
              "auth": {
                "descriptor": {
                  "code": "jwt"
                },
                "value": "eyJhbGciOiJIUzI.eyJzdWIiOiIxMjM0NTY3O.SflKxwRJSMeKKF2QT4"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male",
              "creds": [
                {
                  "id": "237402938409485039850935",
                  "type": "VerifiableCredential",
                  "url": "https://www.aiims.com/honours/237402938409485039850935"
                }
              ],
              "languages": [
                {
                  "code": "hindi",
                  "name": "Hindi"
                }
              ],
              "skills": [
                {
                  "code": "mbbs",
                  "name": "MBBS"
                }
              ],
              "tags": [
                {
                  "display": true,
                  "descriptor": {
                    "code": "about",
                    "name": "About the Doctor"
                  },
                  "list": [
                    {
                      "value": "With over 10 years of experience in Orthopaedics"
                    }
                  ]
                }
              ]
            },
            "rating": "4.5"
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "quote": {
        "price": {
          "value": "350",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
            }
          },
          {
            "title": "Slot surcharge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
          "reason_required": true,
          "cancellation_fee": {
            "amount": {
              "currency": "EUR",
              "value": "100"
            }
          }
        }
      ]
    }
  }
}
```
Below is an example of Second `on_init` callback

```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male",
              "creds": [
                {
                  "id": "237402938409485039850935",
                  "type": "VerifiableCredential",
                  "url": "https://www.aiims.com/honours/237402938409485039850935"
                }
              ],
              "languages": [
                {
                  "code": "hindi",
                  "name": "Hindi"
                }
              ],
              "skills": [
                {
                  "code": "mbbs",
                  "name": "MBBS"
                }
              ],
              "tags": [
                {
                  "display": true,
                  "descriptor": {
                    "code": "about",
                    "name": "About the Doctor"
                  },
                  "list": [
                    {
                      "value": "With over 10 years of experience in Orthopaedics"
                    }
                  ]
                }
              ]
            },
            "rating": "4.5"
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "quote": {
        "price": {
          "value": "350",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
            }
          },
          {
            "title": "Slot surcharge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
          "reason_required": true,
          "cancellation_fee": {
            "amount": {
              "currency": "EUR",
              "value": "100"
            }
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
    "domain": "dhp:consultation:0.1.0",
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
      "fulfillments": [
        {
          "id": "ful-01",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "stops": [
            {
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "items": [
        {
          "id": "cons-01"
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
    "domain": "dhp:consultation:0.1.0",
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
      "id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "provider": {
        "id": "289edce4-d002-4962-b311-4c025e22b4f6",
        "descriptor": {
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "state": {
            "descriptor": {
              "code": "slot-booked",
              "name": "Slot has been Booked"
            }
          },
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male"
            }
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "quote": {
        "price": {
          "value": "350",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
            }
          },
          {
            "title": "Slot surcharge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
          "reason_required": true,
          "cancellation_fee": {
            "amount": {
              "currency": "EUR",
              "value": "100"
            }
          }
        }
      ]
    }
  }
}
```

## 1.3 Fulfillment of DHP Services
This section contains recommendations for implementing the APIs related to fulfilling a DHP service

### 1.3.1 Recommendations for BPPs

#### 1.3.1.1 Sending status updates
- REQUIRED. The BPP MUST implement the `status` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.2 Updating an appointment
- REQUIRED. The BPP MUST implement the `update` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.3 Cancelling an appointment
- REQUIRED. The BPP MUST implement the `cancel` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_cancellation_reasons` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.4 Real-time tracking
- REQUIRED. The BPP MUST implement the `track` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.3.2 Recommendations for BAPs

#### 1.3.2.1 Receiving status updates
- REQUIRED. The BAP MUST implement the `on_status` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `status`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.2 Updating an appointment
- REQUIRED. The BAP MUST implement the `on_update` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `update`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.3 Cancelling an appointment
- REQUIRED. The BAP MUST implement the `on_cancel` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `cancel`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `cancellation_reasons` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_cancellation_reasons`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.4 Real-time tracking
- REQUIRED. The BAP MUST implement the `on_track` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `track`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.3.3 Example Workflow

### 1.3.4 Example Requests

Below is an example of a `status` request
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io/",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order_id": "759b905c-5a30-4d18-a6af-0decbac9282f"
  }
}
```

Below is an example of an `on_status` callback
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
      "id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "provider": {
        "id": "289edce4-d002-4962-b311-4c025e22b4f6",
        "descriptor": {
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "state": {
            "descriptor": {
              "code": "slot-confirmed",
              "name": "Slot has been confirmed"
            }
          },
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male"
            }
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "quote": {
        "price": {
          "value": "350",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
            }
          },
          {
            "title": "Slot surcharge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
          "reason_required": true,
          "cancellation_fee": {
            "amount": {
              "currency": "EUR",
              "value": "100"
            }
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
    "domain": "dhp:consultation:0.1.0",
    "action": "cancel",
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
    "order_id": "759b905c-5a30-4d18-a6af-0decbac9282f",
    "cancellation_reason_id": "4",
    "descriptor": {
      "short_desc": "Not Needed"
    }
  }
}
```
Below is an example of an `on_cancel` callback
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
      "id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "provider": {
        "id": "289edce4-d002-4962-b311-4c025e22b4f6",
        "descriptor": {
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "state": {
            "descriptor": {
              "code": "slot-cancelled",
              "name": "Slot has been Cancelled"
            }
          },
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male"
            }
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "quote": {
        "price": {
          "value": "350",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
            }
          },
          {
            "title": "Slot surcharge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
          "reason_required": true,
          "cancellation_fee": {
            "amount": {
              "currency": "EUR",
              "value": "100"
            }
          }
        }
      ],
      "cancellation": {
        "time": "2023-07-16T04:41:16Z",
        "cancelled_by": "CONSUMER"
      }
    }
  }
}
```
Below is an example of a `update` request

```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io/",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "fulfillments": [
        {
          "stops": [
            {
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ]
    },
    "update_target": "order.fulfillments[0].stops[0].time"
  }
}
```
Below is an example of an `on_update` callback
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
      "id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "provider": {
        "id": "289edce4-d002-4962-b311-4c025e22b4f6",
        "descriptor": {
          "name": "Example Hospitals",
          "short_desc": "Example Hospitals Pvt Ltd",
          "images": [
            {
              "url": "https://ExampleHospitals.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "cons-01",
          "descriptor": {
            "code": "consultation",
            "name": "Orthopedaedic Surgeon",
            "short_desc": "Orthopaedic Surgeon",
            "long_desc": "Skilled orthopaedic surgeon specializing in diagnosing & treating musculoskeletal conditions using advanced surgical techniques & compassionate care"
          },
          "price": {
            "value": "300",
            "currency": "INR"
          },
          "category_ids": [
            "cat-02"
          ],
          "fulfillment_ids": [
            "ful-01"
          ],
          "time": {
            "duration": "PT30M"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "ful-01",
          "type": "OPD",
          "customer": {
            "person": {
              "name": "Jane Doe",
              "age": "13",
              "gender": "female",
              "dob": "1995-09-11"
            },
            "contact": {
              "phone": "+91-9663088848",
              "email": "jane.doe@example.com"
            }
          },
          "state": {
            "descriptor": {
              "code": "patient-details-updated",
              "name": "Patient Detail is Updated"
            }
          },
          "agent": {
            "person": {
              "id": "237402938409485039850935",
              "name": "Dr Asthana",
              "gender": "male"
            }
          },
          "stops": [
            {
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
              },
              "time": {
                "label": "Booking Slots",
                "range": {
                  "start": "2023-11-04 10:00",
                  "end": "2023-11-04 10:30"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Rajesh Kumar",
        "address": "Villa 5, Green Valley, Malleshwaram, 560012",
        "state": {
          "name": "Madhya Pradesh"
        },
        "city": {
          "name": "Bhopal"
        },
        "email": "rajesh.kumar@example.com",
        "phone": "+91-9999999999"
      },
      "quote": {
        "price": {
          "value": "350",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "General Physician Consultation",
            "price": {
              "currency": "INR",
              "value": "300"
            }
          },
          {
            "title": "Slot surcharge",
            "price": {
              "currency": "INR",
              "value": "50"
            }
          }
        ]
      },
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "350",
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
          "reason_required": true,
          "cancellation_fee": {
            "amount": {
              "currency": "EUR",
              "value": "100"
            }
          }
        }
      ]
    }
  }
}
```

## 1.4 Post-fulfillment of DHP Services
This section contains recommendations for implementing the APIs after fulfilling a DHP service

### 1.4.1 Recommendations for BPPs

#### 1.4.1.1 Rating and Feedback
- REQUIRED. The BPP MUST implement the `rating` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_rating_categories` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.4.1.2 Providing Support
- REQUIRED. The BPP MUST implement the `support` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.4.2 Recommendations for BAPs

#### 1.4.2.1 Rating and Feedback
- REQUIRED. The BAP MUST implement the `on_rating` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `rating`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `rating_categories` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_rating_categories`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.4.2.2 Providing Support
- REQUIRED. The BAP MUST implement the `on_support` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `support`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.4.3 Example Workflow

### 1.4.4 Example Requests

Below is an example of a `rating` request

```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io/",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "rating": {
      "rating_category": "Agent",
      "id": "FULAGE123",
      "value": "5"
    }
  }
}
```
Below is an example of an `on_rating` callback
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io/",
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
    "domain": "dhp:consultation:0.1.0",
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
    "bap_url": "https://ps-bap-client.becknprotocol.io/",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "support": {
      "order_id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "callback_phone": "+91-8858150053"
    }
  }
}
```

Below is an example of an `on_support` callback
```
{
  "context": {
    "domain": "dhp:consultation:0.1.0",
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
      "ref_id": "759b905c-5a30-4d18-a6af-0decbac9282f",
      "order_id": "759b905c-5a30-4d18-a6af-0decbac9282f",
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