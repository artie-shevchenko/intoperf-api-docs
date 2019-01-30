# Getting Started
---

## Before you begin

Make sure you have your unique **IntoPerf API key**. If you don't or you get authentication errors when using it please contact ```admin@intoperf.com``` to get a new API key.

## Using the API

* Don't use HTTP, use HTTPS instead.
* Each request to IntoPerf API should have a ```key``` **query parameter**. For example: ```https://api.intoperf.com/v1/players?key=YOUR_API_KEY```

The example below is a good introduction to IntoPerf API. To find more examples of what you can do with this API and how to use it, visit the reference section of this site.

**Note:**  IntoPerf Data Navigator and Reports will do player/team's performance assessment only if there are at least 3 matches imported for that player/team. Otherwise, it will not even appear in the lists.

## Example

Let’s import a match between CSKA Moscow and FK Krasnodar (2013/06/23) with ```Submaximal speed portion``` data for FK Krasnodar.

**Note:** to see pushed data at [https://apidemo.intoperf.com](https://apidemo.intoperf.com) use curl, wget, [Postman](https://www.getpostman.com/apps) or any similar tool to send HTTP requests with your unique **IntoPerf API key**.

**[Prework]** Create a mapping between your IDs and IntoPerf IDs. Either reuse existing entities:
* [GET /v1/teams?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/teams/get)
* [GET /v1/players?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/players/get)
* [GET /v1/eventtypes?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/get).

Or create new:
* [POST /v1/teams?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/teams/post).
* [POST /v1/players?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/players/post).
* [POST /v1/eventtypes?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/post).

**[Prework]** Make [GET /v1/eventtypes?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/eventtypes/get) request and remember ```reportPeriodMillis``` for every quantitative event type used (in this example it's 10000 milliseconds for ```Submaximal speed portion```):

Response:

```
[
  ...
  
  {
    "id": 13,
    "name": "Submaximal speed portion [fake]",
    "description": "Submaximal running speed portion.",
    "type": "quantity",
    "reportPeriodMillis": 10000
  },
  
  ...
]
```

1.  **Start a match.**

    Make [POST /v1/matches?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/post) request with parameters:

    ```
    localTeamId=65
    visitingTeamId=149
    startTimestampMillis=1371945601000
    timezoneUtcOffsetHours=3
    visitingTeamLineup=156,153,174,163,159,165,168,172,175,176
    visitingTeamLineupShirtNumbers=17,5,3,7,19,4,2,9,11,10
    realTime=true
    ```
    
    Response:
    
    ```
    {
      "status" : "success",
      "data" : {
        "matchId": "sR1J5K"
      }
    }
    ```
    
    **Important**: Use the ```matchId``` returned to replace ```{matchId}``` in the following requests.

1.  **Push several performance events**.

    Push `Submaximal speed portion` value for the 10000-millis-aligned interval [1371945620000, 1371945670000). Make [POST /v1/matches/{matchId}/quantitativePerfEvents?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/quantitativePerfEvents/post) request with parameters:

    ```
    eventTypeId=13
    playerId=156
    value=0.12
    intervalStartTimestampMillis=1371945620000
    intervalFinishTimestampMillis=1371945670000
    ```
    
    Actually let's use batch API instead to do more with less API requests. And let's use longer intervals (in production you never want to use such intervals). Make [POST /v1/matches/{matchId}/quantitativePerfEvents:batchCreate?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/quantitativePerfEvents:batchCreate/post) request with parameters:
    
    ```
    eventTypeIds=13,13,13,13,13,13
    playerIds=156,156,153,153,163,163
    values=0.2,0.3,0.4,0.4,0.4,0.5
    intervalStartTimestampsMillis=1371945620000,1371946000000,1371945620000,1371946000000,1371945620000,1371946000000
    intervalFinishTimestampsMillis=1371947000000,1371947500000,1371947000000,1371947500000,1371947000000,1371947500000
    ```
    
    Make another [POST /v1/matches/{matchId}/quantitativePerfEvents:batchCreate?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D/quantitativePerfEvents:batchCreate/post) request with parameters:
    
    ```
    eventTypeIds=13,13,13,13
    playerIds=172,172,175,175
    values=0.5,0.5,0.2,0.4
    intervalStartTimestampsMillis=1371945620000,1371947000000,1371945620000,1371947000000
    intervalFinishTimestampsMillis=1371947000000,1371947500000,1371947000000,1371947500000
    ```
    
1.  **Push a substitution.**

    Make [POST /v1/matches/{matchId}:player_out?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:player_out/post) request with parameters:
    
    ```
    playerId=156
    playerOutTimestampMillis=1371947400000
    ```
    
    Make [POST /v1/matches/{matchId}:player_in?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:player_in/post) request with parameters:
    
    ```
    teamId=149
    playerId=178
    playerInTimestampMillis=1371947400000
    shirtNumber=10
    
    ```

1.  **Finish the first half** (and start the break after the first half).
    
    Make [POST /v1/matches/{matchId}:start_next_period?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:start_next_period/post) request with parameters:
    
    ```
    nextPeriodCode=break-after-first
    periodStartTimestampMillis=1371948420000
    ```
    
1.  **Start the second half** (and finish the break after the first half).

    Make [POST /v1/matches/{matchId}:start_next_period?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:start_next_period/post) request with parameters:
    
    ```
    nextPeriodCode=second
    periodStartTimestampMillis=1371949260000
    ```
    
1.  **Finish the match.**

    Make [POST /v1/matches/{matchId}:finish?key=YOUR_API_KEY](https://apidoc.intoperf.com/docs/intoperf-api.appspot.com/1/routes/v1/matches/%7BmatchId%7D:finish/post) request with parameter:
    
    ```
    finishTimestampMillis=1371952200000
    ```

That’s it! The data you just uploaded can now be analyzed at [https://apidemo.intoperf.com](https://apidemo.intoperf.com):

1.  Go to [https://apidemo.intoperf.com](https://apidemo.intoperf.com).
1.  Choose `Team in match` context type.
1.  Choose `FK Krasnodar in CSKA Moscow-FK Krasnodar (2013-06-23) match`.
1.  [Optional] Select `1st half` as a period.
1.  Click `Search`.
1.  Set filters to 0 to see more players.

Next step is to upload some real data for 10-20 matches and take a look how IntoPerf can help you organize and understand your data.

**Note:** For different teams/players you would need to upload at least 3 matches first for data to show up at [https://apidemo.intoperf.com](https://apidemo.intoperf.com).