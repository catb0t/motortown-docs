# Motor Town Dedicated Server Web API Documentation

## Unofficial documentation for the server Web API.

#### Adapted by @catb0t (psybrcatTRL) from the officially distributed Dedi `ReadMe.txt`.

**Best interpretation of the official ReadMe, but it may contain mistakes.**

**Especially**, descriptions of exact field meanings/semantics and endpoint behaviour are inferred and written with my best knowledge of the game and testing the API, but may differ from the source code.

## Host Web API Server
You can enable Web API Server to get server status and player information.

### `DedicatedServerConfig.json` Parameters

- `<bEnableHostWebAPIServer>`

  Set 'true' to enable Web API Server

- `<HostWebAPIServerPassword>`

  Set password for Web API Server

- `<HostWebAPIServerPort>`

  Set TCP port for Web API Server (**Default**: 8080)
  <br>This TCP port can be opened in firewall if you want to use the API from a remote client.

  **The API and its password are not encrypted for transport, and appear in plaintext in web traffic.**

  **Exposing the Web API to the public internet is not recommended, as an attacker can intercept the password and make requests on their own.**

  **Until the API is HTTPS, best practise is to close the Dedi's own Web API port and make all API calls from the same local server machine that is running the Dedi.**

  **You can use a language like Python and an HTTPS/TLS server library to host an HTTPS forwarder on your server, which you can contact encrypted, and forward remote requests to the local machine's Web API.**

  **It's recommended to change the default port, and be aware that it's non-TLS HTTP.**

### Web API Return Values

API returns JSON string with following format
- `data`: `object` contains return data
- `message`: `string` contains simple message, or error message if API call failed
- `succeeded`: `bool` true if API call succeeded, false if failed
- `code`: `int` Undocumented. Assumed to be

```json
{
  "data": {},
  "message": "",
  "succeeded": true,
  "code": 0
}
```

### Web API password

You should set `"HostWebAPIServerPassword"` to a string in `DedicatedServerConfig.json`. *Currently, stored in plaintext, so be careful.*

You need to pass the password as a parameter in each API call.

**Note that the password is sent <u>unencrypted</u> because of current lack of Web API HTTPS/TLS.** <u>It's not recommended to send it over the public internet</u> unless you can use another layer of encryption.

**API call example**

```
GET  /player/count/
     ?password=your_password
```
```
POST /player/kick/
     ?password=your_password
     &unique_id=player_unique_id
     &reason=reason
```

## Web API Endpoint List

### GET

- **`GET /player/count`**

  Returns:

  `num_players`: `int`

  Example:
  ```json
  "data": { "num_players": 2 }
  ```

- **`GET /player/list`**

  Returns:

  - `"n"`: `object` (Player) with keys
    - `name`: `string` Nickname
    - `unique_id`: `int` Steam ID
    - `location`: `string` of coordinates `X`, `Y`, and `Z` float values
    - `vehicle`: `object` with keys

      - `name`: `string`
      - `unique_id`: `int`

  Example:

  ```json
  "data": {
    "0": {
      "name": "jake",
      "unique_id": "12345",
      "location": "X=2590.089 Y=1682.984 Z=98.927",
      "vehicle":
      {
        "name": "Scooty",
        "unique_id": 175114
      }

    },
    "1": {
      "name": "dooli",
      "unique_id": "12346",
      "location": "X=-108.632 Y=463.258 Z=188.150"
    }
  }
  ```

- **`GET /player/banlist`**

  Returns:

  - `"n"`: `object` (Player) with keys
    - `name`: `string` Nickname
    - `unique_id`: `string` Steam ID

  Example:

  ```json
    "data": {
      "0": {
        "name": "jake",
        "unique_id": "12345"
        }
      }
  ```

- **`GET /player/role/list`**

  Returns an object of 0-indexed player objects matching the input role type.

  Parameters:

  - `role`: `string` (**required**)
  <br>Values: `"admin"` or `"police"`

  Returns:

  - `role`: `string` (either `"admin"` or `"police"`) with keys
    - `"n"`: `object` (Player) with keys
      - `unique_id`: `string` Steam ID
      - `nickname`: `string`

  Example:

  ```json
  "data": {
    "admin":
    {
      "0":
      {
        "unique_id": "12345",
        "nickname": "dooli"
      },
      "1":
      {
        "unique_id": "67890",
        "nickname": "betty"
      }
    }
  }
  ```

- **`GET /delivery/sites`**

  Returns an object of 0-indexed keys of job sites in the world. A job site may have multiple in-game markers which refer to the same inventory and delivery jobs (contracts).

  Returns:

  - `"n"`: `object` (Site) with keys
    - `guid`: `string` Globally unique site ID
    - `name`: `string` In-game site name
    - `location`: `string` of coordinates `X`, `Y`, and `Z` float values
    - `Deliveries`: `object` 0-indexed (Delivery job contracts) with keys
      - `id`: `int` World delivery job ID/index

      - `cargo_type`: `string` Comma-separated list of cargo types (Sand, Box)
      - `num_cargos`: `string` Fraction of available cargo types to total cargo types for this contract
      - `sender_point`: `string` GUID of the pickup marker for this contract
      - `receiver_point`: `string` GUID of the delivery marker for this contract
      - `register_time`: `string` Delay before this contract becomes available again
      - `expire_time`: `string` Delay before this contract will expire and be refreshed
      - `base_payment`: `int` Base payment multiplier for the contract.
      - `timer_seconds`: `int` Current timer position before expiry.<br>`-1` if the contract will not currently expire.

    - `InputInventory`: `object` 0-indexed (Inventory items) with keys
      - `amount`: `int` Currently held amount of this cargo as input, for consumption by the industry

      - `cargo`: `object` (Cargo descriptor) with keys
        - `name`: `string`

        - `type`: `string`
        - `space_types`: `string` Comma-separated list of cargo space types needed
        - `volume_size`: `int` Cargo space units needed to hold one of these cargo
        - `weight_range`: `string` of `X` (lower) and `Y` (upper) values
        - `allow_stacking`: `string` Whether this cargo can be stacked in its space
        <br> Values: `"true"` or `"false"`
        - `use_damage`: `string` Whether damage is applied to this cargo in transport
        <br> Values: `"true"` or `"false"`
        - `spawn_probability`: `int` Chance that this cargo will spawn in a given refresh cycle (Range unknown)
        - `num_cargo_range`: `string` of `X` (lower) and `Y` (upper) limits of how many cargo units will spawn for each delivery target in the "Load Cargo" list
        - `base_payment`: `int` Base payment multiplier for this cargo
        <br> `0` indicates no multiplier
        - `payment_per1km`: `int` Coins payment per 1 km driven to deliver this cargo to its destination
        - `delivery_distance_range`: `string` of `X` (lower) and `Y` (upper) distance limits for delivery targets of this cargo
    - `OutputInventory`: `object` 0-indexed (Inventory items) with keys
      - `amount`: `int` Currently held amount of this cargo as output, able to be loaded by Player/AI for delivery elsewhere
      - `cargo`: `object` (Cargo descriptor) with keys **as above**


  Example:
  ```json
  "data": {
    "17": {
      "guid": "A65CAFE44ABD6FBBB130BC953C85E1BD",
      "name": "Construction Site",
      "location": "X=-3350.465 Y=2.171 Z=100.000",
      "Deliveries": {
        "0": {
          "id": 68,
          "cargo_type": "Sand",
          "num_cargos": "1/1",
          "sender_point": "A65CAFE44ABD6FBBB130BC953C85E1BD",
          "receiver_point": "2F3A8E4C4CB0A3A1273B8A8036FCDBCA",
          "register_time": 122.30276489257812,
          "expire_time": 482.48467864096165,
          "base_payment": 5,
          "timer_seconds": -1
        }
      },
      "InputInventory": {
        "0": {
          "amount": 0,
          "cargo": {
            "name": "Carrots",
            "type": "Box",
            "space_types": "Flatbed, Box",
            "volume_size": 1,
            "weight_range": "X=0.000 Y=0.000",
            "allow_stacking": "false",
            "use_damage": "false",
            "fragile": 0,
            "spawn_probability": 10,
            "num_cargo_range": "X=1 Y=2",
            "base_payment": 0,
            "payment_per1km": 220,
            "delivery_distance_range": "X=0 Y=0"
          }
        }
      },
      "OutputInventory": {
        "0": {
          "amount": 1,
          "cargo": {
            "name": "Sand",
            "type": "Sand",
            "space_types": "Dump",
            "volume_size": 1,
            "weight_range": "X=1600.000 Y=1600.000",
            "allow_stacking": "false",
            "use_damage": "false",
            "fragile": 0,
            "spawn_probability": 10,
            "num_cargo_range": "X=5 Y=20",
            "base_payment": 0,
            "payment_per1km": 380,
            "delivery_distance_range": "X=0 Y=0"
          }
        }
      }
    }
  }
  ```

- **`GET /version`**

  Get the current game version string of the server.
  <br>Version parts:
  - `0.7.13` Semantic major/minor/patch game version

  - `+CT4` Patch number of this test version, CT for Closed Test, T for Public Test, TD for Test Driver (unused)
  - `(B804)` Build number (unique, sequential)

  Returns:

  - `version`: `string` Version string as above

  Example:

  ```json
  "data": { "version": "0.7.13+CT4(B804)" }
  ```

- **`GET /housing/list`**

  Get all the player-ownable houses on the map, indexed by name.

  Returns:

  - `<name>`: `object` (House) called `<name>` with keys
    - `owner_unique_id`: `string` SteamID of player owner
    - `expire_time`: `string` Time descriptor of when this house rental will pay rent next or expire, including date and time delta
    <br> Format: `YYYY.MM.DD-HH.MM.SS (+/-N days NN hours)`

  ```json
  "data":{
    "HousingTest_House0": {
      "owner_unique_id": "12345",
      "expire_time": "2025.04.11-03.04.57 (+6 days 23 hours)"
    }
  }
  ```

### POST

**Post parameters are provided in the request URL query string**, not as JSON.

**The top-level `data` key will be empty.** Check the `message` string key for the message and the `succeeded` boolean key for whether the server thinks it did what you asked.

In general, for a successful request, the return object will have:

`code` will be `0`.
<br>
`succeeded` will be `true`.

<br>

- **`POST /chat`**

  Send an announcement or a message to the server. No formatting is applied to chat messages, i.e. does not automatically start with `[SERVER]` or similar.

  Parameters:
  - `message`: (**required**)
  <br>Message to display as an announcement or in chat.
  - `type`: (optional)
  <br> Values: `"announce"` (**default**) or `"message"`
  - `color`: (optional)
  <br>Override chat text color (ex FF00FF) when type is `"message"`.
  <br>Format: Hex without `0x`: `FFAA00`.
  <br>**Default**: `FFFFFF`

  Returns:

  *Success*:
    - `message` will be `message sent`.

- **`POST /player/kick`**

  Kick a player by Steam ID.

  Parameters:

  - `unique_id` (**required**)
  <br>Player's `unique_id` (Steam ID) from `/player/list`

  Returns:

  *Success*:
    - `message` will be `Player kicked`

  Example:

  ```json
  {
    "data": { },
    "message": "Player kicked",
    "succeeded": true,
    "code": 0
  }
  ```

- **`POST /player/ban`**

  Ban a player by their Steam ID. If the player is on the server, they will be kicked and shown the `reason` message in a dialog box. They'll be unable to rejoin the server until `hours` has passed, or permanently if not provided.

  If the player isn't on the server, the player will be added as a `BannedPlayers` entry to the `GameUserSettings.ini`, with the other attributes recorded. As above, they won't be able to join until `hours` pass or indefinitely.

  Parameters:

  - `unique_id` (**required**)
  <br>Player's `unique_id` (Steam ID) from `/player/list`
  - `hours`: (optional)
  <br>If missing, the ban will be applied permanently.
  - `reason` (optional)
  <br>Reason for the ban. Will be shown to the player and appear in the `GameUserSettings` with the `BannedPlayers` entry.

  Returns:

  *Success*:
    - `message` will be `Player banned`

  Example:

  ```json
  {
    "data": { },
    "message": "Player banned",
    "succeeded": true,
    "code": 0
  }
  ```


<p></p>

- **`POST /player/unban`**

  Parameters:

  - `unique_id` (**required**
  <br>Player's `unique_id` (Steam ID) from `player/banlist`

  Returns:

  *Success*:
    - `message` will be `Player unbanned`

  Example:

  ```json
  {
    "data": { },
    "message": "Player unbanned",
    "succeeded": true,
    "code": 0
  }
  ```

<p></p>

- **`POST /player/role/add`**

  Add the player by Steam ID to the named role. Results in an immediate role change for an online player, and updates `GameUserSettings.ini` with the entry.

  Because the `GameUserSettings` file is a flat data structure, each role applied to a player is its own line instead of a nested data structure either of Player objects with roles or Role objects with Players.

  If a player has both `"admin"` and `"police"`, two separate line entries will denote the player and each role.

  Parameters:

  - `role` (**required**)
  <br>Set player role
  <br>Values: `"admin"` or `"police"`
  - `unique_id` (**required**)
  <br>Player's `unique_id` (Steam ID) from `player/list` or `GameUserSettings`

  Returns:

  *Success*:
    - ?

  Example:

  ?

<p></p>

- **`POST /player/role/remove`**

  Remove the player by Steam ID to the named role. Results in an immediate role change for an online player, and updates `GameUserSettings.ini` by removing the entry.

  Because the `GameUserSettings` file is a flat data structure, each role applied to a player is its own line instead of a nested data structure either of Player objects with roles or Role objects with Players.

  If a player has both `"admin"` and `"police"`, two separate line entries will denote the player and each role.

  Parameters:

  - `role` (**required**)
  <br>Remove player role
  <br>Value: `"admin"` or `"police"`
  - `unique_id` (**required**)
  <br>Player's `unique_id` from `player/list` or `GameUserSettings`

  Returns:

  *Success*:
    - ?

  Example:

  ?
