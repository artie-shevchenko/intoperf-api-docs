# Getting Started
---

## Before you begin

Make sure you have your unique **IntoPerf API key**. If you don't or you get authentication errors when using it please contact ```admin@intoperf.com``` to get a new API key.

## Using the API

* Don't use HTTP, use HTTPS instead.
* Each request to IntoPerf API should have a ```key``` query parameter. For example: ```https://api.intoperf.com/v1/players?key=YOUR_API_KEY```

The usage example below is a good introduction to IntoPerf API. To find more examples of what you can do with this API and how to use it, visit the reference section of this site. You can use the **Try this API** tool on the right side of every API reference page to generate a sample request.

**Note:**  IntoPerf Data Navigator and Reports will do player's performance assessment only if there are at least 3 matches imported for that player. Otherwise, that player will not even be listed in the Unit select box.

## Usage example

Let’s import a match between CSKA Moscow and FK Krasnodar (2013/06/23) with ```Submaximal speed portion``` data for FK Krasnodar (see [GET /v1/teams](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/teams/get) response to find the correct team IDs).

**Note:** to see pushed data at [https://apidemo.intoperf.com](https://apidemo.intoperf.com) don't use **Try this API** tool. Instead use curl, wget, [Postman](https://www.getpostman.com/apps) or any similar tool with your unique **IntoPerf API key**.

**[Prework]** Use [GET /v1/players](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/players/get) and [GET /v1/eventtypes](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/get) responses to create a map between IntoPerf IDs and your IDs.

**[Prework]** Make [GET /v1/eventtypes](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/get) request and remember ```reportPeriodMillis``` for every quantitative event type used (in this example it's 10000 milliseconds for ```Submaximal speed portion```):

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

1.  **Start a match.**

    Make [POST /v1/matches](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/post) request with:

    ```
    localTeamId=65
    visitingTeamId=149
    startTimestampMillis=1371945601000
    timezoneUtcOffsetHours=3
    visitingTeamLineup=156,153,174,163,159,165,168,172,175,176
    visitingTeamLineupShirtNumbers=17,5,3,7,19,4,2,9,11,10
    realTime=true
    key=YOUR_API_KEY
    ```
    
    Response (it may be a different value for you, **you will need it later on**):
    
    ```
    m24
    ```

1.  **Push several performance events**.

    Push `Submaximal speed portion` value for the 10000-millis-aligned interval [1371945620000, 1371945670000). Make [POST /v1/matches/m24/quantitativePerfEvents](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/quantitativePerfEvents/post) request with:

    ```
    eventTypeId=13
    playerId=156
    value=0.12
    intervalStartTimestampMillis=1371945620000
    intervalFinishTimestampMillis=1371945670000
    key=YOUR_API_KEY
    ```
    
    Actually let's use batch API instead to do more with less API requests. And let's use longer intervals (in production you never want to use such intervals). Make [POST /v1/matches/m24/quantitativePerfEvents:batchCreate](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/quantitativePerfEvents:batchCreate/post) request with ```application/x-www-form-urlencoded```:
    
    ```
    eventTypeIds=13,13,13,13,13,13
    playerIds=156,156,153,153,163,163
    values=0.2,0.3,0.4,0.4,0.5,0.6
    intervalStartTimestampsMillis=1371945620000,1371946000000,1371945620000,1371946000000,1371945620000,1371946000000
    intervalFinishTimestampsMillis=1371947000000,1371947500000,1371947000000,1371947500000,1371947000000,1371947500000
    ```
    
    Don't forget setting ```key=YOUR_API_KEY``` in these requests too. Make another [POST /v1/matches/m24/quantitativePerfEvents:batchCreate](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/quantitativePerfEvents:batchCreate/post) request with ```application/x-www-form-urlencoded```:
    
    ```
    eventTypeIds=13,13,13,13
    playerIds=172,172,175,175
    values=0.5,0.5,0.2,0.4
    intervalStartTimestampsMillis=1371945620000,1371947000000,1371945620000,1371947000000
    intervalFinishTimestampsMillis=1371947000000,1371947500000,1371947000000,1371947500000
    ```
    
1.  **Push a substitution.**

    Make [POST /v1/matches/m24/player_out](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:player_out/post) request with:
    
    ```
    playerId=156
    playerOutTimestampMillis=1371947400000
    key=YOUR_API_KEY
    ```
    
    Make [POST /v1/matches/m24/player_in](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:player_in/post) request with:
    
    ```
    teamId=149
    playerId=178
    playerInTimestampMillis=1371947400000
    shirtNumber=10
    key=YOUR_API_KEY
    
    ```

1.  **Finish the first half** (and start the break after the first half).
    
    Make [POST /v1/matches/m24:start_next_period](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:start_next_period/post) request with:
    
    ```
    nextPeriodCode=break-after-first
    periodStartTimestampMillis=1371948420000
    key=YOUR_API_KEY
    ```
    
    ...
    
1.  **Start the second half** (and finish the break after the first half).

    Make [POST /v1/matches/m24:start_next_period](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:start_next_period/post) request with:
    
    ```
    nextPeriodCode=second
    periodStartTimestampMillis=1371949260000
    key=YOUR_API_KEY
    ```
    
    ...
    
1.  **Finish the match.**

    Make [POST /v1/matches/m24:finish](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:finish/post) request with:
    
    ```
    finishTimestampMillis=1371952200000
    key=YOUR_API_KEY
    ```

That’s it! The data you just uploaded can now be analyzed at [https://apidemo.intoperf.com](https://apidemo.intoperf.com):

1.  Go to [https://apidemo.intoperf.com](https://apidemo.intoperf.com).
1.  Choose `Team in match` context type.
1.  Choose `FK Krasnodar in CSKA Moscow-FK Krasnodar (2013-06-23) match`.
1.  [Optional] Select `1st half` as a period.
1.  Click `Search`.
1.  Click `Search with no filters`.

That's a tiny amount of data we just uploaded so confidence level is ~0%. Next step is to upload data for 10-20 matches and take a look how IntoPerf's intelligence helps you organize and understand your data.

**Note:** For other teams/players you would need to upload at least 3 matches first for data to show up at [https://apidemo.intoperf.com](https://apidemo.intoperf.com).