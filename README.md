# Autominer API
Automatically rent miners based on the current cost to mine the currency. You should run the frontend and only make calls from an SSL connection if including the API key. If you do not use an SSL secure connection you will be risking exposing your API keys to anybody who might be sniffing your traffic.

## Installation
Install Node.js then run the following commands to download and install the autominer-api.
```
$ git clone https://github.com/dloa/autominer-api.git
$ cd autominer-api
$ npm install
```

Next, you will need to get your API credentials from Mining Rig Rentals. In order to do this, you will need to follow the following steps.
1. Login to your Mining Rig Rentals account. (If you have not yet created an account, you can follow the instructions on how to fully get your MRR account ready below.)
2. Navigate to the URL https://www.miningrigrentals.com/account/apikey
3. Press the "Add A Key" button. Make sure to give it permission to Withdraw, Rent Rigs, and Manage Rigs (for some reason the MRR API will not rent machines unless all three permissions are granted). After giving it the correct permissions, press the "Save" button.

Now that we have the API credentials, we can go ahead and start up the API and enter in our config details using the following command:
```
$ node app.js
```
Once you have started the application, it will walk you through a similar dialog to this:
```
$ node app.js
Welcome to the Alexandria Autominer!

It looks like you have not yet setup the Autominer yet, please follow the directions found here: bit.ly/2rAVVVi


Please enter your MiningRigRentals API Key: d448a54df68a8sd8f48as4d8f7e6ad48745ds52a1f234
Please enter your MiningRigRentals API Secret: 4574a1s6d84654as86d4fga8447d8s4ad8a4gf8a4s56
How much BTC would you like to spend each week?: 0.001
The "minimum margin" is the threashold at which it will begin mining. If this is set to 0, then it will always rent, however if you set it to anything higher, it will wait until the margin is met to begin mining.
Please enter your "minimum margin": 0
The RPI threashold is the minimum machine avaialbilty that will be accepted. An RPI threashold of 80 is standard.
Please enter your RPI threashold: 80
Please enter a password for this API: example-password
 ======== PROFILES ======== 
ID: 12345 | Name: Profile 1
ID: 23451 | Name: Profile 2
ID: 34512 | Name: Profile 3
 ========================== 
Please enter a PROFILE ID from the list above: 12345
autominer-api listening on port 3123!
```

## Mining Rig Rentals Account Setup
1. Create a Mining Rig Rentals account at: https://www.miningrigrentals.com/register
2. Deposit some starting funds to your MRR account (this will allow you to start mining sooner!)
3. Browse to the [MRR pools page](https://www.miningrigrentals.com/account/pools) and select "Add A Pool"
4. Enter in the details for your pool. If you would like to mine on the Alexandria pool, just enter the following details:
![](http://skylarostler.com/img/pG4kseCUx7.png)
5. Browse to the [MRR profiles page](https://www.miningrigrentals.com/account/profiles) and select "Create New Profile"
6. Give it a name, and select your target algorithm (If you want to mine Florincoin, this will be "Scrypt")
![](http://skylarostler.com/img/Z00I3kEnAg.png)
7. After creating your Profile, it should load a section that allows you to add a pool to the profile. Select the pool that you added in Step 4 and click "Add To Profile"
![](http://skylarostler.com/img/GyYtpbsU6D.png)
8. After you have successfully added your Profile, you will need to create an API key for the `autominer-api` application to use. You can do this by browsing to the [MRR API keys page](https://www.miningrigrentals.com/account/apikey).
9. Once you are on the API key page, select "Add A Key" and give it all permissions. 

# Usage
## Running the Application
To run the autominer-api application just run the following command and keep it running using something like screen.
```
$ node app.js
```

## API Endpoints:
`/info`: Responds with JSON that includes the following data:
```javascript
{
	'status': "Just rented a rig... Waiting for rental period to end...", // String containing human legible information about the autominer-api status
	'pool_hashrate': 978670933, 		// The current hashrate of the Alexandria Pool
	'networkhashps': 1974515030, 		// Current hashrate of all florincoin miners
	'MiningRigRentals_last10': 0.00014861, // Average cost per hash for an hour from the last 10 rented miners
	'fmd_weighted_btc': 0.00000325, 	// Value per FLO in BTC
	'fmd_weighted_usd': 0.00144, 		// Value per FLO in USD
	'flo_spotcost_btc': 0.00000543, 	// Cost to mine 1 FLO in BTC
	'flo_spotcost_usd': 0.00241, 		// Cost to mine 1 FLO in USD
	'pool_influence': 0.9828, 			// Amount of the entire Hashrate that the pool controls (1 = 50%)
	'pool_influence_code': 0, 			// 0: Pool influence below 50%, 1: Pool influence over 50%
	'pool_influence_multiplier': 1, 	// if pool_influence_code = 0 { pool_influence_multiplier = 1 } 
											//else if pool_infleunce_code = 1 { pool_influence_multiplier = 1 / ( pool_influence^2) }	
	'market_conditions': 1.0064, 		// (((((Pool_Max_Margin / 100) + 1) x flo_spotcost_btc) - fmd_weighted_btc) ÷ fmd_weighted_btc)	
	'market_conditions_code': 2,		// if market_conditions ≤ 0 { market_conditions_code = “0: Market conditions support Max Pool margin” }
											// else if market_conditions > 0 and ≤ 1 { market_conditions_code = “1: Max Pool margin too high for market conditions” }
											// else if market_conditions > 1 { market_conditions_code = “2: Any Pool margin too high for market conditions”}
	'market_conditions_multiplier': 0,	// if market_conditions_code=0, market_conditions_multiplier = 1
											// else if market_conditions_code=1, market_conditions_multiplier = 1-(market_conditions^.5)
											// else if market_conditions_code=2, market_conditions_multiplier = 0
	'pool_margin': 0, 					// Pool_Max_Margin x Pool_Influence_Multiplier x Market_Conditions_Multiplier
	'offer_btc': 0.00000543 			// Current BTC offer based on the cost for 1 FLO plus margins
}
```
`/status`: Requires the post of an API key, responds with JSON that includes the following data from the status of ongoing rentals:
```javascript
{
	'hash_rented': 191879347, 		// Current hashrate
	'week_spent_btc': 0.4872, 		// Amount of BTC spent in the current week
	'account_balance': 0.1874123, 	// Current account balance
	'hash_history': [				// History of the hashrate, contains up to 168 hourly records.
		{
			'time': 1467067237,
			'hashrate': 1762487,
			'machines_rented': [
				{
					'name': 'The Beast',
					'hash': 716274,
					'cost_per_hour': 0.0015
				}
			]
		}
	]
}
```
`/config`: Requires the post of an API key in order to get config info, you can post any variables below in order to change them. Responds with JSON that includes the following data from the current config:
```javascript
{
	'weekly_budget_btc': 1, // Maximum budget to spend per week in BTC
	'min_margin': 10,		// Margin that you wish to make by mining
	"max_difficulty": 1000, // The maximum difficulty of which you want to be mining
    "mrr_last_10_max_multiplier": 1.5, // This value is multiplied against the MRR last 10 average price. For example, if the MRR last 10 average is `0.00001` per hash, and you are ok paying up to 150% of that, then you would use a multiplier of 1.5. This would allow it to pay up to `0.00015` per hash for example. This prevents the API from renting machines charging 4-6x the average.
	'RPI_threshold': 80		// Minimum RPI allowed for renting devices
	'max_difficulty': 1000, // If the difficulty goes higher than this value, the miner will stop renting more rigs. Once it drops back down below, it will start renting rigs again.
	'api_key': '8uuijau898ue9823uj29iu8d',
	'MRR_API_key': 'd448a54df68a8sd8f48as4d8f7e6ad48745ds52a1f234',
	'MRR_API_secret': '4574a1s6d84654as86d4fga8447d8s4ad8a4gf8a4s56'
}
```
`/logs`: Requires the post of an API key in order to get logs, you can post a variable called 'amount' to specify how many logs you want. Default amount to return is 50 unless specified, -1 will return all. Responds with logs like this:
```javascript
{
  "amount": 2,
  "logs": [{
      "id": 2,
      "timestamp": 1469389445,
      "type": "info",
      "message": "Started up autominer-api on port 3000",
      "extrainfo": ""
    },
    {
      "id": 1,
      "timestamp": 1469389445,
      "type": "error",
      "message": "Error creating default settings file from settings.example.cfg",
      "extrainfo": "error"
    }
  ]
}
```

### Example Config Change
Change the config by posting to the config url:
```
$.post('127.0.0.1/config', JSON.stringify({
	key:"kjh87uyh9i3yhu98ayui0938u", 
	weekly_budget_btc: 0.5,
	min_margin: 25,
	RPI_threshold: 75
}));
```
