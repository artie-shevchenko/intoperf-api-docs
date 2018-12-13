# Getting Started
---

## Before you begin

Make sure you have your unique **IntoPerf API key**. If you don't or you get authentication errors when using it please contact ```admin@intoperf.com``` and we will provide you with a new API key.

## Using the API

* Don't use HTTP, use HTTPS instead.
* Each request to IntoPerf API should have a ```key``` parameter. For example: ```https://api.intoperf.com/v1/players?key=YOUR_API_KEY```

The sample below is a good overview of the API. To find more examples of what you can do with this API and how to use it, visit the reference section of this site. You can use the **Try this API** tool on the right side of every API method page to generate a sample request.

**Note:**  IntoPerf Data Navigator and Reports will only do a player's performance assessment if there are at least 3 matches imported for that player. Otherwise, that player will not even be listed in select boxes.

## Sample

Let’s import a match between FK Krasnodar and CSKA (2012/06/26) with ```Submaximal speed portion``` data for FK Krasnodar (see ```GET /v1/teams``` response to find the correct team IDs).

**Note:** 

**[Prework]** Use [GET /v1/players](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/players/get) and [GET /v1/eventtypes](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/get) responses to create a map between IntoPerf and your IDs.

**[Prework]** Make [GET /v1/eventtypes](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/get) request and remember ```reportPeriodMillis``` for every periodic event type used (in this example it's 10000 milliseconds for ```Submaximal speed portion```):

Response:

```
[
{
    "id": 12,
    "name": "Shot",
    "description": "Shots accuracy.",
    "type": "quality",
    "reportPeriodMillis": 0
  },
  {
    "id": 13,
    "name": "Submaximal speed portion [fake]",
    "description": "Submaximal running speed portion.",
    "type": "quantity",
    "reportPeriodMillis": 10000
  },
]
```

1.  Start a match.

    Make [POST /v1/matches](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/post) request with:

    ```
    localTeamId=65
    visitingTeamId=149
    startTimestampMillis=1340670001000
    timezoneUtcOffsetHours=3
    visitingTeamLineup=156,153,174,163,159,165,168,172,175,176
    visitingTeamLineupShirtNumbers=17,5,3,7,19,4,2,9,11,10
    realTime=true
    key=YOUR_API_KEY
    ```
    
    Response (it will be a different value for you, **you will need it later on**):
    
    ```
    m24
    ```

1. Push a `Submaximal speed portion` value for the first complete aligned 10-second period [1340670020000, 1542670030000).

    Make [POST /v1/matches/m24/perfevents](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/perfevents/post) request with:

    ```
    eventTypeId=13
    matchId=m24
    playerId=156
    value=0.24
    eventTimestampMillis=1542670030000
    key=YOUR_API_KEY
    ```

    ... Push more events ...

1. Push a substitution.

    Make [POST /v1/matches/m24/player_out](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:player_out/post) request with:
    
    ```
    matchId=m24
    playerId=156
    playerOutTimestampMillis=1542671034250
    key=YOUR_API_KEY
    ```
    
    Make [POST /v1/matches/m24/player_in](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:player_in/post) request with:
    
    ```
    matchId=m24
    playerId=178
    playerInTimestampMillis=1542671034250
    shirtNumber=10
    key=YOUR_API_KEY
    
    ```

1.  Finish the first half (and start the break after the first half).
    
    Make [POST /v1/matches/m24:start_next_period](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:start_next_period/post) request with:
    
    ```
    matchId=m24
    nextPeriodCode=break-after-first
    periodStartTimestampMillis=1542673030000
    key=YOUR_API_KEY
    ```
    
    ...
    
1.  Start the second half (and finish the break after the first half).

    Make [POST /v1/matches/m24:start_next_period](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:start_next_period/post) request with:
    
    ```
    matchId=m24
    nextPeriodCode=second
    periodStartTimestampMillis=1542674000000
    key=YOUR_API_KEY
    ```
    
    ...
    
1.  Finish the match.

    Make [POST /v1/matches/m24:finish](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:finish/post) request with:
    
    ```
    matchId=m24
    finishTimestampMillis=1542677000000
    key=YOUR_API_KEY
    ```

That’s it! The `Submaximal speed portion` data you just uploaded can be analyzed now at [https://apidemo.intoperf.com](https://apidemo.intoperf.com).

**Note:** For other teams/players you would need to upload 3 matches first for data to show up at [https://apidemo.intoperf.com](https://apidemo.intoperf.com).