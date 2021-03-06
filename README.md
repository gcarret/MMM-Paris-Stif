# MMM-Paris-RATP-PG

MagicMirror MichMich module to display next buses and RERs for a configured list of stations/ destinations.

Forked from MMM-HH-LocalTransport see more detailed information on gerog90 [blog](https://lane6.de).

# Presentation
A module to display:
* the different buses, metros, rers & tramways, in order to avoid waiting too much for them when leaving home. 
* general traffic information for lines of metros, rers & tramways (only using v3 API)
* available docks & bikes in selected Velib stations, along with the trend over the last day (configurable).

# Screenshot
![screenshot](https://github.com/da4throux/MMM-Paris-RATP-PG/blob/history/MMM-Paris-RATP-PG%201.6.png)

# API

It is based on the open REST API from Pierre Grimaud https://github.com/pgrimaud/horaires-ratp-api, which does not require any configuration / registration. Immediate use. Support v2 & v3 (however the v2 seems to be deprecated, so you should rely on v3).
It also use the Paris Open Data for the velib: https://opendata.paris.fr/explore/dataset/stations-velib-disponibilites-en-temps-reel/ (use it to get the 5 digits stations you will need for the configuration).

# Install

1. Clone repository into `../modules/` inside your MagicMirror folder.
2. Run `npm install` inside `../modules/MMM-Paris-RATP-PG/` folder
3. Add the module to the MagicMirror config
```
		{
	        module: 'MMM-Paris-RATP-PG',
	        position: 'bottom_right',
	        header: 'Connections',
	        config: {
	        }
    	},
```

# specific configuration
* [name]: [default value], //information
* maximumEntries: 2, //if the APIs sends several results for the incoming transport how many should be displayed
* updateInterval: 60000, //time in ms between pulling request for new times (update request)
* converToWaitingTime: true, // messages received from API can be 'hh:mm' in that case convert it in the waiting time 'x mn'
* maxLeterrsForDestination: 22, //will limit the length of the destination string
* concatenateArrivals: true, //if for a transport there is the same destination and several times, they will be displayed on one line
* showSecondsToNextUpdate: true, // display a countdown to the next update pull (should I wait for a refresh before going ?)
* showLastUpdateTime: false, //display the time when the last pulled occured (taste & color...)
* oldUpdateOpacity: 0.5, //when a displayed time age has reached a threshold their display turns darker (i.e. less reliable)
* oldThreshold: 0.1, //if (1+x) of the updateInterval has passed since the last refresh... then the oldUpdateOpacity is applied
* debug: false, //console.log more things to help debugging
* busStations: [] // the list of stations/directions to monitor (bus and RERs, probably works also for Subways)
* busStations is an array of objects with different properties:
  - 'api': Optional: needs to be set to 'v3' if the v3 of the API is to be used for the pierre-grimaud interface. If missing, v2 is assumed for backward compatibility (ignore for velib). Mandatory 'v3' for 'traffic'
  - 'type': Mandatory: Possible value:['bus', 'rers', 'tramways', 'velib', 'traffic']
  - 'line': Mandatory for 'bus', 'rers' & 'tramways']: Value such as:[28, 'B'] -> typically the official name but you can check through: 
   . v2 - bus-metros-rers-tramways: https://api-ratp.pierre-grimaud.fr/v2/bus, https://api-ratp.pierre-grimaud.fr/v2/rers, https://api-ratp.pierre-grimaud.fr/v2/tramways, https://api-ratp.pierre-grimaud.fr/v2/metros
   . v3 - bus-metros-rers-tramways: https://api-ratp.pierre-grimaud.fr/v3/lines/bus, https://api-ratp.pierre-grimaud.fr/v3/lines/rers, https://api-ratp.pierre-grimaud.fr/v3/lines/tramways, https://api-ratp.pierre-grimaud.fr/v3/lines/metros
   . v3 - traffic: https://api-ratp.pierre-grimaud.fr/v3/traffic set the line as: [type, line], such as: ['metros', 6], ['rers', 'A']...
  - 'stations': Mandatory: [digits of the station in v2, name of the station in v3] ->
    . v2 for bus/rers/tramways/metros, the station id, look it up with the url, typically: https://api-ratp.pierre-grimaud.fr/v2/{type}/{line}
    . v3 for bus/rers/tramways/metros, https://api-ratp.pierre-grimaud.fr/v3/stations/{type}/{line}
    . for velib, you can search here: https://opendata.paris.fr/explore/dataset/stations-velib-disponibilites-en-temps-reel/
    . not required for traffic
  - 'destination': 
    . v2: Mandatory for 'metros', 'bus', 'rers' & 'tramways': [the destination id] (indicated in the same look up url)
    . v3: Mandatory for 'metros', 'bus', 'rers' & tramways: either 'A' or 'R'
    . Optional for 'velib': ['leaving', 'arriving', '']: indicate if only one value is needed //not in use yet
    . not required for traffic.
  - 'label': Optional: to rename the line differently if needed
* conversion: object of key/ values to convert traffic message. Those message can be very long, and it might worth to convert them in a simpler text. by default:
  - conversion: {"Trafic normal sur l'ensemble de la ligne." : 'Traffic normal'}
  - don't hesitate to add more when there's works on a specific line or others...

Example:
```javascript
busStations: [
	{type: 'bus', line: 38, stations: 2758, destination: 183, label: 'bus vers le Nord'},
	{api: 'v3', type: 'bus', line: 38, stations: 'observatoire+++port+royal', destination: 'A'},
        {type: 'rers', line: 'B', stations: 62, destination: 4},
	{api: 'v3', type: 'rers', line: 'B', stations: 'port+royal', destination: 'A'},
	{api: 'v3', type: 'traffic', line: ['rers', 'B']},
	{api: 'v3', type: 'traffic', line: ['tramways', 1], label: 'T1'}, //label to avoid confusion with metros line 1
	{type: 'tramways', line: '3a', stations: 464, destination: 41},	
	{api: 'v3', type: 'tramways', line: '3a', stations: 'georges+brassens', destination: 'R'},
	{type: 'metros', line: '6', stations: 145, destination: 17},	
	{api: 'v3', type: 'metros', line: '6', stations: 'raspail', destination: 'A'},
	{type: 'velib', stations: 05029, destination: 'leaving', label 'RER'}]
```
# v1.6
