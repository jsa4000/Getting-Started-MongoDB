# MongoDB

## Events Logs

### Indexes

- Events Log Indexes

```json
[
    {
        "v" : 2,
        "key" : {
            "_id" : 1
        },
        "name" : "_id_",
        "ns" : "events_database.activity_events"
    },
    {
        "v" : 2,
        "key" : {
            "networkId" : 1
        },
        "name" : "network_id_index",
        "ns" : "events_database.activity_events"
    },
    {
        "v" : 2,
        "key" : {
            "date" : 1
        },
        "name" : "date_index",
        "ns" : "events_database.activity_events",
        "expireAfterSeconds" : 5184000
    }
]
```

> The **primary key** for events Log's documents is simple.

### Document

- Activity Log Document (`SYSTEM_LOGIN`)

```json
{
    "_id" : ObjectId("5c07845f9698a7000109092d"),
    "_class" : "com.example.services.eventsLog.persistence.entity.ActivityEvent",
    "createdTimestamp" : ISODate("2018-12-05T07:55:11.095Z"),
    "updatedTimestamp" : ISODate("2018-12-05T07:55:11.095Z"),
    "date" : ISODate("2018-12-05T07:55:11.092Z"),
    "traceId" : "9075078d43bce969",
    "service" : "auth",
    "operation" : "SYSTEM_LOGIN",
    "origin" : {
        "originId" : "99c3693b-ecc1-40cd-8de4-3035b94d2f9b",
        "type" : "AGENT"
    },
    "data" : {
        "firstLogin" : "false"
    },
    "priority" : {
        "$ref" : "activity_events_operations",
        "$id" : "SYSTEM_LOGIN"
    }
}
```

- Activity Log Document (`DISCONNECTED`)

```json
{
    "_id" : ObjectId("5c0737ee9698a70001090927"),
    "_class" : "com.example.services.eventsLog.persistence.entity.ActivityEvent",
    "createdTimestamp" : ISODate("2018-12-05T02:29:02.404Z"),
    "updatedTimestamp" : ISODate("2018-12-05T02:29:02.404Z"),
    "date" : ISODate("2018-12-05T02:29:02.000Z"),
    "traceId" : "2572f237afe2672e",
    "networkId" : "AA-BB-CC-DD-EE-FF",
    "deviceId" : "AA-BB-CC-DD-EE-FF",
    "memberId" : "AA-BB-CC-DD-EE-FF",
    "service" : "home-hub",
    "operation" : "DISCONNECTED",
    "origin" : {
        "originId" : "AA-BB-CC-DD-EE-FF",
        "type" : "DEVICE"
    },
    "priority" : {
        "$ref" : "activity_events_operations",
        "$id" : "DISCONNECTED"
    }
}
```

### Queries

- Count **all** the documents in the collection

        db.getCollection('activity_events').count()

- Query `find all items sorted in reverse order`

        db.getCollection('activity_events').find().sort({"_id": -1})

- Count `find all items by NetworkId`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF"}).count()

- Query `find all items by NetworkId sorted in reverse order`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF"}).sort({"_id": -1})

- Profile `find all items by networkId` (indexed)

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF"}).explain()

    ```json
    {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "networkId" : {
                    "$eq" : "AA-BB-CC-DD-EE-FF"
                }
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "networkId" : 1
                    },
                    "indexName" : "network_id_index",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "networkId" : []
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "networkId" : [ 
                            "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                        ]
                    }
                }
            },
            "rejectedPlans" : []
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

- Profile `find all items by memberId` (not indexed)

        db.getCollection('activity_events').find({"memberId" : "AA-BB-CC-DD-EE-FF"}).explain()

    ```json
   {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "memberId" : {
                    "$eq" : "AA-BB-CC-DD-EE-FF"
                }
            },
            "winningPlan" : {
                "stage" : "COLLSCAN",
                "filter" : {
                    "memberId" : {
                        "$eq" : "AA-BB-CC-DD-EE-FF"
                    }
                },
                "direction" : "forward"
            },
            "rejectedPlans" : []
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

#### Filtered Queries (by networkId)

- Query `find all items by NetworkId and operation`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF" })

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF" }).count()
        19

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF" }).explain()

    ```json
    {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "$and" : [ 
                    {
                        "networkId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "operation" : {
                            "$eq" : "SWITCH_OFF"
                        }
                    }
                ]
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "filter" : {
                    "operation" : {
                        "$eq" : "SWITCH_OFF"
                    }
                },
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "networkId" : 1
                    },
                    "indexName" : "network_id_index",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "networkId" : []
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "networkId" : [ 
                            "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                        ]
                    }
                }
            },
            "rejectedPlans" : []
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

- Query `find all items by NetworkId, operation and date`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} })

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).count()
        8

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).explain()

    ```json
   {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "$and" : [ 
                    {
                        "networkId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "operation" : {
                            "$eq" : "SWITCH_OFF"
                        }
                    }, 
                    {
                        "date" : {
                            "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                        }
                    }
                ]
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "filter" : {
                    "$and" : [ 
                        {
                            "operation" : {
                                "$eq" : "SWITCH_OFF"
                            }
                        }, 
                        {
                            "date" : {
                                "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                            }
                        }
                    ]
                },
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "networkId" : 1
                    },
                    "indexName" : "network_id_index",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "networkId" : []
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "networkId" : [ 
                            "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                        ]
                    }
                }
            },
            "rejectedPlans" : [ 
                {
                    "stage" : "FETCH",
                    "filter" : {
                        "$and" : [ 
                            {
                                "networkId" : {
                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                }
                            }, 
                            {
                                "operation" : {
                                    "$eq" : "SWITCH_OFF"
                                }
                            }
                        ]
                    },
                    "inputStage" : {
                        "stage" : "IXSCAN",
                        "keyPattern" : {
                            "date" : 1
                        },
                        "indexName" : "date_index",
                        "isMultiKey" : false,
                        "multiKeyPaths" : {
                            "date" : []
                        },
                        "isUnique" : false,
                        "isSparse" : false,
                        "isPartial" : false,
                        "indexVersion" : 2,
                        "direction" : "forward",
                        "indexBounds" : {
                            "date" : [ 
                                "[new Date(1541030400000), new Date(9223372036854775807)]"
                            ]
                        }
                    }
                }
            ]
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

- Query `find all items by NetworkId, operation and date sorted by date asc`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1})

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1}).count()
        8

#### Filtered Queries (by networkId and memeberId)

- Query `find all items by NetworkId, memberId and operation`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF","operation" : "STATUS_CHANGED" })

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "operation" : "STATUS_CHANGED" }).count()
        71

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF",  "operation" : "STATUS_CHANGED" }).explain()

    ```json
   {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "$and" : [ 
                    {
                        "memberId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "networkId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "operation" : {
                            "$eq" : "STATUS_CHANGED"
                        }
                    }
                ]
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "filter" : {
                    "$and" : [ 
                        {
                            "memberId" : {
                                "$eq" : "AA-BB-CC-DD-EE-FF"
                            }
                        }, 
                        {
                            "operation" : {
                                "$eq" : "STATUS_CHANGED"
                            }
                        }
                    ]
                },
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "networkId" : 1
                    },
                    "indexName" : "network_id_index",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "networkId" : []
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "networkId" : [ 
                            "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                        ]
                    }
                }
            },
            "rejectedPlans" : []
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

- Query `find all items by NetworkId, memberId and operation and date`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "operation" : "STATUS_CHANGED", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} })

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "operation" : "STATUS_CHANGED", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).count()
        30

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF",  "operation" : "STATUS_CHANGED", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).explain()

    ```json
  {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "$and" : [ 
                    {
                        "memberId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "networkId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "operation" : {
                            "$eq" : "STATUS_CHANGED"
                        }
                    }, 
                    {
                        "date" : {
                            "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                        }
                    }
                ]
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "filter" : {
                    "$and" : [ 
                        {
                            "memberId" : {
                                "$eq" : "AA-BB-CC-DD-EE-FF"
                            }
                        }, 
                        {
                            "operation" : {
                                "$eq" : "STATUS_CHANGED"
                            }
                        }, 
                        {
                            "date" : {
                                "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                            }
                        }
                    ]
                },
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "networkId" : 1
                    },
                    "indexName" : "network_id_index",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "networkId" : []
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "networkId" : [ 
                            "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                        ]
                    }
                }
            },
            "rejectedPlans" : [ 
                {
                    "stage" : "FETCH",
                    "filter" : {
                        "$and" : [ 
                            {
                                "memberId" : {
                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                }
                            }, 
                            {
                                "networkId" : {
                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                }
                            }, 
                            {
                                "operation" : {
                                    "$eq" : "STATUS_CHANGED"
                                }
                            }
                        ]
                    },
                    "inputStage" : {
                        "stage" : "IXSCAN",
                        "keyPattern" : {
                            "date" : 1
                        },
                        "indexName" : "date_index",
                        "isMultiKey" : false,
                        "multiKeyPaths" : {
                            "date" : []
                        },
                        "isUnique" : false,
                        "isSparse" : false,
                        "isPartial" : false,
                        "indexVersion" : 2,
                        "direction" : "forward",
                        "indexBounds" : {
                            "date" : [ 
                                "[new Date(1541030400000), new Date(9223372036854775807)]"
                            ]
                        }
                    }
                }
            ]
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

- Query `find all items by NetworkId, memberId, operation and date sorted by date asc`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "operation" : "STATUS_CHANGED", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1})

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "operation" : "STATUS_CHANGED", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1}).count()
        30

#### Filtered Queries (by networkId, memeberId and operations)

- Query `find all items by NetworkId, memberId, multiple operations and date sorted`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "$or": [{"operation" : "CONNECTED"}, {"operation" : "DISCONNECTED" }], "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} })

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "$or": [{"operation" : "CONNECTED"}, {"operation" : "DISCONNECTED" }], "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).count()
        75

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "$or": [{"operation" : "CONNECTED"}, {"operation" : "DISCONNECTED" }], "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).explain()

    ```json
    {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "$and" : [ 
                    {
                        "$or" : [ 
                            {
                                "operation" : {
                                    "$eq" : "CONNECTED"
                                }
                            }, 
                            {
                                "operation" : {
                                    "$eq" : "DISCONNECTED"
                                }
                            }
                        ]
                    }, 
                    {
                        "memberId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "networkId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "date" : {
                            "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                        }
                    }
                ]
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "filter" : {
                    "$and" : [ 
                        {
                            "$or" : [ 
                                {
                                    "operation" : {
                                        "$eq" : "CONNECTED"
                                    }
                                }, 
                                {
                                    "operation" : {
                                        "$eq" : "DISCONNECTED"
                                    }
                                }
                            ]
                        }, 
                        {
                            "memberId" : {
                                "$eq" : "AA-BB-CC-DD-EE-FF"
                            }
                        }, 
                        {
                            "date" : {
                                "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                            }
                        }
                    ]
                },
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "networkId" : 1
                    },
                    "indexName" : "network_id_index",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "networkId" : []
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "networkId" : [ 
                            "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                        ]
                    }
                }
            },
            "rejectedPlans" : [ 
                {
                    "stage" : "FETCH",
                    "filter" : {
                        "$and" : [ 
                            {
                                "$or" : [ 
                                    {
                                        "operation" : {
                                            "$eq" : "CONNECTED"
                                        }
                                    }, 
                                    {
                                        "operation" : {
                                            "$eq" : "DISCONNECTED"
                                        }
                                    }
                                ]
                            }, 
                            {
                                "memberId" : {
                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                }
                            }, 
                            {
                                "networkId" : {
                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                }
                            }
                        ]
                    },
                    "inputStage" : {
                        "stage" : "IXSCAN",
                        "keyPattern" : {
                            "date" : 1
                        },
                        "indexName" : "date_index",
                        "isMultiKey" : false,
                        "multiKeyPaths" : {
                            "date" : []
                        },
                        "isUnique" : false,
                        "isSparse" : false,
                        "isPartial" : false,
                        "indexVersion" : 2,
                        "direction" : "forward",
                        "indexBounds" : {
                            "date" : [ 
                                "[new Date(1541030400000), new Date(9223372036854775807)]"
                            ]
                        }
                    }
                }
            ]
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    ```

- Query `find all items by NetworkId, memberId, multiple operations and date sorted by date asc`

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "$or": [{"operation" : "CONNECTED"}, {"operation" : "DISCONNECTED" }], "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1})

        db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "$or": [{"operation" : "CONNECTED"}, {"operation" : "DISCONNECTED" }], "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1}).count()
        75

### Aggregate queries

Aggregate Following queries

- Query `find all items by NetworkId, operation (SWITCH_OFF) and date sorted by date asc`

    db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "operation" : "SWITCH_OFF", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1}).count()
    8

- Query `find all items by NetworkId, memberId, operation (STATUS_CHANGED) and date sorted by date asc`

    db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "operation" : "STATUS_CHANGED", "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1}).count()
    30

- Query `find all items by NetworkId, memberId, multiple operations (CONNECTED and CONNECTED) and date sorted by date asc`

    db.getCollection('activity_events').find({"networkId" : "AA-BB-CC-DD-EE-FF", "memberId" : "AA-BB-CC-DD-EE-FF", "$or": [{"operation" : "CONNECTED"}, {"operation" : "DISCONNECTED" }], "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")} }).sort({"date" : 1}).count()
    75

> Total documents after the aggregation: 8 + 30 + 75 = 113

- Final Aggregated query (0.31sec)

    ```json
    db.getCollection('activity_events').find(
        {
            "networkId" : "AA-BB-CC-DD-EE-FF",
            "date" : { "$gte" : ISODate("2018-11-01T00:00:00.000Z")},
            "$or": [
                {
                    // find all items by NetworkId, operation (SWITCH_OFF) and date sorted by date asc
                    "operation" : "SWITCH_OFF"
                },
                {
                    // find all items by NetworkId, memberId, operation (STATUS_CHANGED) and date sorted by date asc
                    "memberId" : "AA-BB-CC-DD-EE-FF",
                    "operation" : "STATUS_CHANGED"
                },
                {
                    // find all items by NetworkId, memberId, multiple operations (CONNECTED and CONNECTED) and date sorted by date asc
                    "memberId" : "AA-BB-CC-DD-EE-FF",
                    "$or": [
                        {"operation" : "CONNECTED"},
                        {"operation" : "DISCONNECTED" }
                    ]
                }
                ]
        }
        ).sort({"date" : 1})
        .count()
    ```

    ```json
    {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "event_log_development.activity_events",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "$and" : [ 
                    {
                        "$or" : [ 
                            {
                                "$and" : [ 
                                    {
                                        "$or" : [ 
                                            {
                                                "operation" : {
                                                    "$eq" : "CONNECTED"
                                                }
                                            }, 
                                            {
                                                "operation" : {
                                                    "$eq" : "DISCONNECTED"
                                                }
                                            }
                                        ]
                                    }, 
                                    {
                                        "memberId" : {
                                            "$eq" : "AA-BB-CC-DD-EE-FF"
                                        }
                                    }
                                ]
                            }, 
                            {
                                "$and" : [ 
                                    {
                                        "memberId" : {
                                            "$eq" : "AA-BB-CC-DD-EE-FF"
                                        }
                                    }, 
                                    {
                                        "operation" : {
                                            "$eq" : "STATUS_CHANGED"
                                        }
                                    }
                                ]
                            }, 
                            {
                                "operation" : {
                                    "$eq" : "SWITCH_OFF"
                                }
                            }
                        ]
                    }, 
                    {
                        "networkId" : {
                            "$eq" : "AA-BB-CC-DD-EE-FF"
                        }
                    }, 
                    {
                        "date" : {
                            "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                        }
                    }
                ]
            },
            "winningPlan" : {
                "stage" : "SORT",
                "sortPattern" : {
                    "date" : 1.0
                },
                "inputStage" : {
                    "stage" : "SORT_KEY_GENERATOR",
                    "inputStage" : {
                        "stage" : "FETCH",
                        "filter" : {
                            "$and" : [ 
                                {
                                    "$or" : [ 
                                        {
                                            "$and" : [ 
                                                {
                                                    "$or" : [ 
                                                        {
                                                            "operation" : {
                                                                "$eq" : "CONNECTED"
                                                            }
                                                        }, 
                                                        {
                                                            "operation" : {
                                                                "$eq" : "DISCONNECTED"
                                                            }
                                                        }
                                                    ]
                                                }, 
                                                {
                                                    "memberId" : {
                                                        "$eq" : "AA-BB-CC-DD-EE-FF"
                                                    }
                                                }
                                            ]
                                        }, 
                                        {
                                            "$and" : [ 
                                                {
                                                    "memberId" : {
                                                        "$eq" : "AA-BB-CC-DD-EE-FF"
                                                    }
                                                }, 
                                                {
                                                    "operation" : {
                                                        "$eq" : "STATUS_CHANGED"
                                                    }
                                                }
                                            ]
                                        }, 
                                        {
                                            "operation" : {
                                                "$eq" : "SWITCH_OFF"
                                            }
                                        }
                                    ]
                                }, 
                                {
                                    "date" : {
                                        "$gte" : ISODate("2018-11-01T00:00:00.000Z")
                                    }
                                }
                            ]
                        },
                        "inputStage" : {
                            "stage" : "IXSCAN",
                            "keyPattern" : {
                                "networkId" : 1
                            },
                            "indexName" : "network_id_index",
                            "isMultiKey" : false,
                            "multiKeyPaths" : {
                                "networkId" : []
                            },
                            "isUnique" : false,
                            "isSparse" : false,
                            "isPartial" : false,
                            "indexVersion" : 2,
                            "direction" : "forward",
                            "indexBounds" : {
                                "networkId" : [ 
                                    "[\"AA-BB-CC-DD-EE-FF\", \"AA-BB-CC-DD-EE-FF\"]"
                                ]
                            }
                        }
                    }
                }
            },
            "rejectedPlans" : [ 
                {
                    "stage" : "FETCH",
                    "filter" : {
                        "$and" : [ 
                            {
                                "$or" : [ 
                                    {
                                        "$and" : [ 
                                            {
                                                "$or" : [ 
                                                    {
                                                        "operation" : {
                                                            "$eq" : "CONNECTED"
                                                        }
                                                    }, 
                                                    {
                                                        "operation" : {
                                                            "$eq" : "DISCONNECTED"
                                                        }
                                                    }
                                                ]
                                            }, 
                                            {
                                                "memberId" : {
                                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                                }
                                            }
                                        ]
                                    }, 
                                    {
                                        "$and" : [ 
                                            {
                                                "memberId" : {
                                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                                }
                                            }, 
                                            {
                                                "operation" : {
                                                    "$eq" : "STATUS_CHANGED"
                                                }
                                            }
                                        ]
                                    }, 
                                    {
                                        "operation" : {
                                            "$eq" : "SWITCH_OFF"
                                        }
                                    }
                                ]
                            }, 
                            {
                                "networkId" : {
                                    "$eq" : "AA-BB-CC-DD-EE-FF"
                                }
                            }
                        ]
                    },
                    "inputStage" : {
                        "stage" : "IXSCAN",
                        "keyPattern" : {
                            "date" : 1
                        },
                        "indexName" : "date_index",
                        "isMultiKey" : false,
                        "multiKeyPaths" : {
                            "date" : []
                        },
                        "isUnique" : false,
                        "isSparse" : false,
                        "isPartial" : false,
                        "indexVersion" : 2,
                        "direction" : "forward",
                        "indexBounds" : {
                            "date" : [ 
                                "[new Date(1541030400000), new Date(9223372036854775807)]"
                            ]
                        }
                    }
                }
            ]
        },
        "serverInfo" : {
            "host" : "dev-shard-00-01-ocbsk.mongodb.net",
            "port" : 27017,
            "version" : "3.4.18",
            "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1.0
    }
    '''