## Protobuf
	- At the cosmos level, the Cosmos SDK knows about the owner
	- :
	- agoric wallet send Lo sign and broadcast
	  ```
	  ("body": "#[\"method!": V'executeoffer|", \"offer\": (l"id\":\"openVault-1738170493577\*, \"invita
	  LonSpec\": (\"callpipe) :[[\"getcollateralManager),I\"Se.Alleged: BoardRemoteATOM brand\"!
	  "makeVaultInvitation\"]],\*instancePath\":[\"VaultFactory\"],\"source) ":\"agoricContract\1
	  •\"proposal)": (\"gtve\":(\"Collateral\": (\"brand\": \"SO\", \"value\":\"+1000000000\"}}, \"want)
	  ': (\"Minted\": (\"brand\":\"$1.Alleged: BoardRemoteIST brand\", \"value\":\"+4000000000\"))]}}"
	  "slots": ["board05557", "boarde257" ]}
	  ```
	  /projects/agoric-sdk
	  11:08 connolly@bldbox$ jq. /tmp/want4k. Json|
	- ```
	  11:14 connolly@bldbox$ ja '.body | «[1:] | fromjson' /tmp/want4k. json|
	  { 
	  "method": "executeoffer", 
	  "offer": {
	  "1d": "openVault-1738170493577,
	    "invitationSpec": (
	      "callPipe": [
	         [ 
	          "getCollateralManager",
	             [
	                "$0.Alleged: BoardRemoteATOM brand"
	             ],
	             [
	               "makeVaultInvitation"
	              ]
	            ],
	           "instancePath": [
	             "VaultFactory"
	            ],
	            "source": "agoriccontract"
	  	},
	      "proposal": {
	        "give": { "Collateral": {
	          "brand": "S0"
	          "value" "+1000000000"
	          }
	         },
	          "want": {
	          "Minted": {
	          "brand": "$1.Alleged: BoardRemoteIST brand",
	          "value": "+4000Đ00ĐĐĐ"
	          	}
	          }
	        }
	      }
	    }    
	  ```
	- ```
	  [\"makeVaultInvitation\"]],\"instancePath\": [\"VaultFactory\"], \"source)": \"agoricContract\
	  ,\"proposal\": (\"give\": (\"Collateral\": (\"brand\":\"So\", \"value\":\"+1000000000\"3) ,\"war
	  t\": (\"Minted\": [\"brand\":\"$1.Alleged: BoardRemoteIST brand\", \"value\":\"+4000000000\*]]})
	  'slots": [
	  "board05557",
	  "board0257"
	  
	  ```
- ### CapData
	- #### Slots
		- Reciever has to know how to transform CapData into an object.
		- in the context of our offer, the receiver should know the proper way to construct the offer. Specificially, the receiver will need to map the capabilities in our offer to the correct values to that they can be sent over the wire.
		- When working with offers in agoric, the **smart wallet** takes on the receiver role.
		-
	- `"$0.Alleged: BoardRemoteATOM brand"`
- [[deploy script support]]
-
- tags:: [[Office Hours]], [[agoric sdk]]
-