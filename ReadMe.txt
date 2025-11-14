[How to Run Dedicated Server]
1. Edit DedicatedServerConfig_Sample.json
	
	<ServerName>
	This will be shown at server list
	
	<ServerMessage>
	This will be shown to player after connect. Use \n for a New Line
	
	<MaxPlayers>
	Set max player number. Recommend 20 for average system. Max 100.
	
	<bAllowPoliceVehicleByPlayerRole>
    Set 'true' to allow police vehicle by player role. If false, anyone can use police vehicles.

	<bAllowCorporation>
    Set 'false' to restrict corporation creation. (defaults to true)

	<bAllowCorporationAIDriver>
    Set 'false' to restrict corporation ai driver. (defaults to true)

	<Admins>
	Use this to give admin to players 
		To aquire UniqueNetId and Nickname
		A. Open Server using game client
		B. Add admin to players
		C. Open game config file(%appdata%..\Local\MotorTown\Saved\Config\Windows\GameUserSettings)
		D. Search 'Admins'

	<bAllowAdminToRemoveAdmin>
    Set 'true' to allow admin to remove admin. (defaults to false)
    
    <HostWebAPIDisabledCommands>
    List of Web API commands to disable. (defaults to empty)
    

2. Rename DedicatedServerConfig_Sample.json to DedicatedServerConfig.json
3. Launch Dedicated Server using RunDedicatedServer.bat or from Steam Library(Steam Login Required)


[Save Files Path]
Saved\SaveGames
(You can copy your Client Game Save files into this folder from '%appdata%..\Local\MotorTown\Saved\SaveGames')


[Host Web API Server]
You can enable Web API Server to get server status and player information

* DedicatedServerConfig.json Parameters

	<bEnableHostWebAPIServer>
	Set 'true' to enable Web API Server

	<HostWebAPIServerPassword>
	Set password for Web API Server

	<HostWebAPIServerPort>
	Set port for Web API Server (Default: 8080)
	This port should be opened in firewall

* Web API return values
	
	API returns Json string with following format
	<data>: contains return data
	<message>: contains simple message, or error message if API call failed
	<succeeded>: true if API call succeeded, false if failed

	{
		"data": {},
		"message": "",
		"succeeded": true,
	}

* Web API password
	You should set 'HostWebAPIServerPassword' in DedicatedServerConfig.json
	You should pass password as parameter in each API call

	API call example

		GET /player/count/?password=your_password
		POST /player/kick/?password=your_password&unique_id=player_unique_id

* Web API List

	POST /chat
	parameters
		message (required. chat message)
		type (optional. set chat type "announce" or "message". defaults to "announce".)
		color (optional. override text color(ex FF00FF) when type is "message". defaults to white.)

	GET /player/count
	return example
		"data": { "num_players": 2 }

	GET /player/list
	return example
		"data": {
			"0": {
				"name": "jake",
				"unique_id": "12345"
				"location": "X=2590.089 Y=1682.984 Z=98.927",
				"vehicle":
				{
					"name": "Scooty",
					"unique_id": 175114
				}

			},
			"1": {
				"name": "dooli",
				"unique_id": "12346"
				"location": "X=-108.632 Y=463.258 Z=188.150",
			}
		}

	POST /player/kick
	parameters
		unique_id (required. player's unique_id from player/list)

	GET /player/banlist
	return example
		"data": {
			"0": {
				"name": "jake",
				"unique_id": "12345"
				}
			}

	POST /player/ban
	parameters
		unique_id (required. player's unique_id from player/list)
		hours (optional. if not provided, the ban will be applied permanently.)
		reason (optional. reason for ban.)

	POST /player/unban
	parameters
		unique_id (required. player's unique_id from player/banlist)

	GET /player/role/list
	parameters
		role (required. set player role "admin" or "police".)
	return example
		"data":{
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
                },
            }
		}

	POST /player/role/add
	parameters
		role (required. set player role "admin" or "police".)
		unique_id (required. player's unique_id from player/list)

	POST /player/role/remove
	parameters
		role (required. set player role "admin" or "police".)
		unique_id (required. player's unique_id from player/list)

	GET /delivery/sites
	return example
		"data": {
			"17":
			{
				 "guid": "A65CAFE44ABD6FBBB130BC953C85E1BD",
				 "name": "Construction Site",
				 "location": "X=-3350.465 Y=2.171 Z=100.000",
				 "Deliveries":
				 {
					  "0":
					  {
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
				 "InputInventory":
				 {
					  "0":
					  {
							"amount": 0,
							"cargo":
							{
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
				 "OutputInventory":
				 {
					  "0":
					  {
							"amount": 1,
							"cargo":
							{
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
	
	GET /version
	return example
		"data": { "version": "0.7.13+CT4(B804)" }
	
	GET /housing/list
	return example
		"data":{
			"HousingTest_House0": {
				"owner_unique_id": "12345",
				"expire_time": "2025.04.11-03.04.57 (+6 days 23 hours)"
			}
		}
