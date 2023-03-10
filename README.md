<h1 align="center">Game Jolt API for Godot</h1>
<p align="center">
  <img width="75%" alt="Game Jolt API for Godot" src="addons/gamejolt/example/screenshot.jpg">
</p>

Wrapper for the Game Jolt API running through HTTP requests. It contains all Game Jolt API endpoints and aims to simplify its use where it's possible. Compatible with **Godot 3.5.x**.

## Documentation
You can use the methods below on the singleton `GameJolt` when the plugin is enabled.

- [General](#general)
- [Users](#users)
- [Sessions](#sessions)
- [Scores](#scores)
- [Trophies](#trophies)
- [Data Storage](#data-storage)
- [Friends](#friends)
- [Time](#time)
- [Batch Calls](#batch-calls)

### Handling Request Responses
Each method emits a signal after a request is completed, be it successful or not.
You can connect specific signals to capture responses on method callbacks:

```gdscript
func _ready() -> void:
    GameJolt.connect("time_completed", self, "_on_GameJolt_time_completed")
    GameJolt.time()


func _on_GameJolt_time_completed(result: Dictionary) -> void:
    # Do something with the request result...
```

Or you can `yield` the method result in a variable:

```gdscript
func _onButtonTime_pressed() -> void:
    var result: Dictionary = yield(GameJolt.time(), "time_completed")
    # Do something with the request result...
```

### General
General methods to configure `GameJolt` singleton locally.

#### set_user_name(value)
Set the user name for auth and other user scope tasks.

- `value: String` -> The user name.

#### get_user_name() -> String
Get current user name.

#### set_user_token(value)
Set the user token for auth and other user scope tasks.

- `value: String` -> The user game token.

#### get_user_token() -> String
Get current user game token.

### Users
#### [users_fetch](https://gamejolt.com/game-api/doc/users/fetch)(user_name, user_ids) -> GameJolt
Returns a user's data.

- `user_name: String` (optional) -> The username of the user whose data you'd like to fetch.
- `user_ids: Array[String|int]` (optional) -> The IDs of the users whose data you'd like to fetch.

**Emits:** `users_fetch_completed`

**Note:** The parameters `user_name` and `user_ids` are mutually exclusive, you should use only one of them, or none.
If none were provided, will fetch from the current user name set in `GameJolt` singleton.

#### [users_auth](https://gamejolt.com/game-api/doc/users/auth)() -> GameJolt
Authenticates the user's information.
This should be done before you make any calls for the user, to make sure the user's credentials (username and token) are valid.
The user name and token must be set on `GameJolt` singleton for it to succeed.

### Sessions
#### [sessions_open](https://gamejolt.com/game-api/doc/sessions/open)() -> GameJolt
Opens a game session for a particular user and allows you to tell Game Jolt that a user is playing your game.
You must ping the session to keep it active and you must close it when you're done with it.

**Notes:**
- You can only have one open session for a user at a time. If you try to open a new session while one is running, the system will close out the current one before opening the new one.
- Requires user name and token to be set on `GameJolt` singleton.

#### [sessions_ping](https://gamejolt.com/game-api/doc/sessions/ping)(status) -> GameJolt
Pings an open session to tell the system that it's still active.
If the session hasn't been pinged within 120 seconds, the system will close the session and you will have to open another one.
It's recommended that you ping about every 30 seconds or so to keep the system from clearing out your session.
You can also let the system know whether the player is in an `"active"` or `"idle"` state within your game.

- `status: String` (optional) -> Sets the status of the session to either `"active"` or `"idle"`.

**Note:** Requires user name and token to be set on `GameJolt` singleton.

#### [sessions_check](https://gamejolt.com/game-api/doc/sessions/check)() -> GameJolt
Checks to see if there is an open session for the user.
Can be used to see if a particular user account is active in the game.

**Notes:**
- This endpoint returns `false` for the `"success"`` field when no open session exists. That behaviour is different from other endpoints which use this field to indicate an error state.
- Requires user name and token to be set on `GameJolt` singleton.

#### [sessions_close](https://gamejolt.com/game-api/doc/sessions/close)() -> GameJolt
Closes the active session.

**Note:** Requires user name and token to be set on `GameJolt` singleton.

### Scores
#### [scores_fetch](https://gamejolt.com/game-api/doc/scores/fetch)(limit, table_id, guest, better_than, worse_than, this_user) -> GameJolt
Returns a list of scores either for a user or globally for a game.

- `limit: String|int` (optional) -> The number of scores you'd like to return.
- `table_id: String|int` (optional) -> The ID of the score table.
- `guest: String` (optional) -> A guest's name.
- `better_than: String|int` (optional) -> Fetch only scores better than this score sort value.
- `worse_than: String|int` (optional) -> Fetch only scores worse than this score sort value.
- `this_user: bool` (optional) -> If `true`, fetch only scores of current user. Else, fetch scores of all users.

**Notes:**
- Requires user name and token to be set on `GameJolt` singleton if `this_user` is `true`.
- The default value for `limit` is `10` scores. The maximum amount of scores you can retrieve is `100`.
- If ``table_id`` is left blank, the scores from the primary score table will be returned.
- Only pass in `this_user` as `true` if you would like to retrieve scores for just the user set in the class constructor. Leave `this_user` as `true` and `guest` as `""` to retrieve all scores.
- `guest` allows you to fetch scores by a specific guest name. Only pass either the `this_user` as `true` or the `guest` (or none), never both.
- Scores are returned in the order of the score table's sorting direction. e.g. for descending tables the bigger scores are returned first.

#### [scores_tables](https://gamejolt.com/game-api/doc/scores/tables)() -> GameJolt
Returns a list of high score tables for a game.

#### [scores_add](https://gamejolt.com/game-api/doc/scores/add)(score, sort, table_id, guest, extra_data) -> GameJolt
Adds a score for a user or guest.

- `score: String` -> This is a string value associated with the score. Example: `"500 Points"`.
- `sort: String|int` -> This is a numerical sorting value associated with the score. All sorting will be based on this number. Example: `500`.
- `table_id: String|int` (optional) -> The ID of the score table to submit to.
- `guest: String` (optional) -> The guest's name. Overrides the `GameJolt` singleton's user name.
- `extra_data: String|int|Dictionary|Array` (optional) -> If there's any extra data you would like to store as a string, you can use this variable.

**Notes:**
- You can either store a score for a user or a guest. If you're storing for a user, you must set user name and toke on `GameJolt` singleton and leave `guest` as empty. If you're storing for a guest, you must pass in the `guest` parameter.
- The `extra_data` value is only retrievable through the API and your game's dashboard. It's never displayed publicly to users on the site. If there is other data associated with the score such as time played, coins collected, etc., you should definitely include it. It will be helpful in cases where you believe a gamer has illegitimately achieved a high score.
- If `table_id` is left blank, the score will be submitted to the primary high score table.

#### [scores_get_rank](https://gamejolt.com/game-api/doc/scores/get-rank)(sort, table_id) -> GameJolt
Returns the rank of a particular score on a score table.

- `sort: String|int` -> This is a numerical sorting value that is represented by a rank on the score table.
- `table_id: String|int` (optional) -> The ID of the score table from which you want to get the rank.

**Notes:**
- If `table_id` is left blank, the ranks from the primary high score table will be returned.
- If the score is not represented by any rank on the score table, the request will return the rank that is closest to the requested score.

### Trophies
TODO

### Data Storage
TODO

### Friends
#### [friends](https://gamejolt.com/game-api/doc/friends/fetch)() -> GameJolt
Returns the list of a user's friends.

**Note:** Requires user name and token to be set on `GameJolt` singleton.

### Time
#### [time](https://gamejolt.com/game-api/doc/time/fetch)() -> GameJolt
Returns the time of the Game Jolt server.

### Batch Calls
A batch request is a collection of sub-requests that enables developers to send multiple API calls with one HTTP request.
To use batch calls in your code, place your request calls between a `batch_begin` and `batch_end`. For example, use your methods in the following order:

```gdscript
# Begin to gather batch requests
GameJolt.batch_begin()

# Add the time request to the batch
GameJolt.time()

# Add the scores_tables request to the batch
GameJolt.scores_tables()

# Stop gathering batch requests
GameJolt.batch_end()

# Perform the batch call with the two requests above (time and score_tables)
var result: Dictionary = yield(GameJolt.batch(), "batch_completed")
```

#### [batch](https://gamejolt.com/game-api/doc/batch)(parallel, break_on_error) -> GameJolt
Perform the batch request after gathering requests with `batch_begin` and `batch end`.

- `parallel: bool` (optional) -> By default, each sub-request is processed on the servers sequentially. If this is set to `true`, then all sub-requests are processed at the same time, without waiting for the previous sub-request to finish before the next one is started.
- `break_on_error: bool` (optional) -> If this is set to `true`, one sub-request failure will cause the entire batch to stop processing subsequent sub-requests and return a value of `false` for success.

**Notes:**
- The maximum amount of sub requests in one batch request is 50.
- The `parallel` and `break_on_error` parameters are mutually exclusive and cannot be used in the same request.

#### batch_begin()
Begins to gather requests for batch. Methods **will not** return responses after this call.
Call `batch_end` to finish the batch request gathering process.

#### batch_end()
Stops gathering requests for batch. Methods **will** return responses again after this call.
Must be used after `batch_begin`.
