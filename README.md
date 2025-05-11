
# BMTC Public API Documentation

This is work-in-progress documentation for the publicly accessible BMTC APIs. I've tried to reverse-engineer it by analysing network traffic on [the BMTC website](https://nammabmtcapp.karnataka.gov.in/commuter/dashboard).

Made with the help of the [Bruno API Client](https://www.usebruno.com/) and existing documentation ([Swagger](https://nimmbus.netlify.app/), [GitHub](https://github.com/Vonter/open-bmtc)) by seadeep42 and Vonter on GitHub

---

## API Details

**Base URL :** `https://bmtcmobileapi.karnataka.gov.in/WebAPI`

**API routes :** 
- [**Search Functionality**](#search-functionality)
  - `/SearchRoute_v2`
  - `/FindNearByBusStop_v2`
  - `/ListVehicles`
- [**Static**](#static)
  - `/RoutePoints`
  - `/AroundBusStops_v2_Webportal`
  - `/GetAllRouteList`
  - `/GetFareRoutes`
  - `/GetMobileFareData_v2`
  - `/GetTimetableByStation_v4`
  - `/GetTimetableByRouteid_v3`
  - `/getWaypoints_v1` (MISSING / UNTESTED)
  - `/GetPathDetails`
- [**Live**](#live)
  - `/SearchByRouteDetails_v4`
  - `/VehicleTripDetails_v2`
  - `/TripPlannerMSMD` (INCOMPLETE)
- [**Misc**](#misc-frontend)
  - `/GetMapConfig`
  - `/GetAllServiceTypes` (INCOMPLETE)
  - `/GetHelplineData` (INCOMPLETE)

---

## Search Functionality

API calls used mainly for search suggestions in textboxes when searching for routes, bus stops, or buses, to ensure data validity.

---

### 1. 🔍 Search Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/SearchRoute_v2`

**Headers :**
```http
Content-Type: application/json
lan: en
```

**Body :**
```
{
  "routetext": "<input text>"
}
```

**Description :** Finds bus routes whose route number contains the input text. Used for search autocomplete functionality.

**Response :** JSON containing `data` array with entries for bus routes with `routetext` as a substring.
```
{
  "data": [
    {
      "union_rowno": <int>,
      "row": <int>,
      "routeno": <string>,
      "responsecode": <int>,
      "routeparentid": <int>
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 2. 🔍 Search Bus Stops

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/FindNearByBusStop_v2`

**Headers :**
```http
Accept: text/plain
lan: en
Content-Type: application/json
```

**Body :**
```
{
  "stationName": "<input text>"
}
```

**Description :** Search for stops containing a substring.

**Response :** JSON containing `data` array with entries for bus stops with `stationName` as a substring.

```
{
  "data": [
    {
      "srno": <int>,
      "routeno": "",
      "routeid": <int>, // idk what this number even is
      "center_lat": 0,
      "center_lon": 0,
      "responsecode": <int>,
      "routetypeid": "2",
      "routename": "<string", // station name
      "route": ""
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 3. 🔍 Search Buses

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/ListVehicles`

**Headers :**
```http
lan: en
deviceType: WEB
Content-Type: application/json
Accept: text/plain
```

**Body :**
```
{
  "vehicleRegNo" : <input text>, // vehicle license plate
  "deviceType" : "WEB"
}
```

**Description :** Finds vehicles whose license plate contains the input text. Used for search autocomplete functionality.

**Response :** JSON containing `data` array with entries for buses with `vehicleRegNo` as a substring of their license plate.

```
{
  "data": [
    {
      "vehicleid": <int>, // vehicle identifier
      "vehicleregno": <string>, // full license plate
      "responsecode": <int>
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

## Static

API calls that return static / infrequently changing data such as bus routes, station coordinates, facilities, fares, timetables, etc.

---

### 1. 🧭 Route Points

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/RoutePoints`

**Body :**
```
{
  "routeid": <int> // routeid here is the routeparentid returned from a /SearchRoute_v2 request
}
```

**Description :** Returns lat and long data for stations on a bus route, in an order corresponding to `SearchByRouteDetails_v4`. `routeid` is unique for the up and down line, so this data is unambiguous.

**Response :** 

```
{
  "data": [
    {
      "latitude": <string>,
      "longitude": <string>,
      "responsecode": <int>
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 2. 🏪 Facilities Around Stations

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/AroundBusStops_v2_Webportal`

**Body :**
```
{
  "deviceType": "WEB",
  "lan": <string> // one of "en" or "kd"
}
```

**Description :** Returns nearby facilities for stations.

**Response :** JSON containing

```
{
  "data": [
    {
      "stationname": <string>,
      "distance": <string>,
      "Arounds": [ // facilities around particular station
        {
          "type": <string>, // ATM, Hotel, Parking, etc
          "typeid": <string>, // identifier for the facility type
          "icon": <string>, // icon from https://bmtcmobileapi.karnataka.gov.in/StaticFiles/
          "list": [ // list of facilities of this type near particular station
            {
              "name": <string>,
              "latitude": <string>,
              "longitude": <string>,
              "distance": <string>
            }
          ]
        },
        ...
      ]
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 3. 🚌 Get All Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetAllRouteList`

**Headers :**
```http
Accept: application/json, text/plain, */*
```

**Description :** Returns basic information about all 11,000+ operational routes (up / down lines are considered distinct)

**Response :** 

```
{
  "data": [
    {
      "routeid": <int>, // new routeid from /SearchByRouteDetails_v4
      "routeno": <string>, // same routeno as other calls, except it specifies direction (Ex : "89-C UP")
      "routename": <string>,
      "fromstationid": <int>,
      "fromstation": <string>,
      "tostationid": <int>,
      "tostation": <string>,
      "responsecode": <int>
    },
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 4. 💰 Get Fare Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetFareRoutes`

**Headers :**
```http
Accept: application/json, text/plain, */*
Content-Type: application/json
```

**Body :**
```
{
  "fromStationId": <int>,
  "toStationId": <int>
}
```

**Description :** Returns all fare routes between two station IDs.

**Response :**

```
{
  "data": [
    {
      "id": <int>,
      "fromstationid": <int>, // id of this specific route, not from body
      "source_code": <string>,
      "from_displayname": <string>,
      "tostationid": <int>, // again id of this route, not from body
      "destination_code": <string>,
      "to_displayname": <string>,
      "fromdistance": <float>,
      "todistance": <float>,
      "routeid": <int>, // this is the new id from /SearchByRouteDetails_v4
      "routeno": <string>,
      "routename": <string>, // origin and terminus shortcode (Ex : KIA-KDG)
      "route_direction": <string>, // one of "Up" or "Down"
      "fromstationname": <string>,
      "tostationname": <string>
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 5. 💸 Get Fare Data

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetMobileFareData_v2`

**Headers :**
```http
lan: en
```

**Body :**
```
{
  "routeno": <string>,
  "routeid": <int>, // new routeid from /SearchByRouteDetails_v4
  "route_direction": <string>, // "Up" or "Down" (Must match routeid)
  "source_code": <string>,
  "destination_code": <string>
}
```

**Description :** Returns fare amount and service type for a given route and stop codes.

**Response :**

```
{
  "data": [
    {
      "servicetype": <string>, // NOT AC/Non AC, but instead the route type (Ex : "Bengaluru Sarige")
      "fare": <string> // fare amount in rupees
    }
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 6. 🕓 Get Timetable Between Two Stops

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetTimetableByStation_v4`

**Headers :**

```http
lan: en
Accept: application/json, text/plain, */*
Content-Type: application/json
```

**Body :**

```
{
  "fromStationId": <int>, // source station ID
  "toStationId": <int>, // destination station ID
  "p_startdate": "<string>", // start date-time (YYYY-MM-DD HH:MM)
  "p_enddate": "<string>", // end date-time (YYYY-MM-DD HH:MM)
  "p_isshortesttime": 2,
  "p_routeid": "",
  "p_date": "<string>" // usually same as p_startdate, but can be anything
}
```

**Description :** Returns scheduled bus trip times between two stations over a given time interval on the same day. Each entry corresponds to a unique route between the two stops.

**Response :**

```
{
  "data": [
    {
      "routeid": <int>, // route identifier
      "id": <int>,
      "fromstationid": <int>, // source station ID
      "tostationid": <int>, // destination station ID
      "routeno": <string>, // route number (e.g., "335-E")
      "routename": <string>, // route name (e.g., "KDG-KBS")
      "fromstationname": <string>, // source station name
      "tostationname": <string>, // destination station name
      "traveltime": <string>, // total travel time (HH:MM:SS)
      "distance": <float>, // total trip distance (km)
      "apptime": <string>, // approximate time (usually same as traveltime)
      "apptimesecs": <string>, // travel time in seconds (as string)
      "starttime": <string>, // departure time from source stop (HH:MM:SS)
      "platformname": null,
      "platformnumber": null,
      "baynumber": null
    },
    ...
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 7. 📅 Get Timetable by Route ID

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetTimetableByRouteId_v3`

**Headers :**

```http
lan: en
Accept: application/json, text/plain, */*
Content-Type: application/json
```

**Body :**

```
{
  "current_date": "<string>", // ISO 8601 date-time (e.g., "2025-05-05T18:30:00.000Z")
  "routeid": <int>, // route ID from /SearchByRouteDetails_v4
  "fromStationId": <int>, // source station ID
  "toStationId": <int>, // destination station ID
  "starttime": "<string>", // search start time (YYYY-MM-DD HH:MM)
  "endtime": "<string>" // search end time (YYYY-MM-DD HH:MM)
}
```

**Description :** Returns all scheduled trip times for a given route between a specified source and destination stop, within a specific time window.

**Response :**

```
{
  "data": [
    {
      "fromstationname": <string>, // source stop name
      "tostationname": <string>, // destination stop name
      "fromstationid": <string>, // source stop ID (as string)
      "tostationid": <string>, // destination stop ID (as string)
      "apptime": <string>, // approximate travel time (HH:MM:SS)
      "distance": <string>, // total trip distance (km)
      "platformname": null,
      "platformnumber": null,
      "baynumber": null,
      "tripdetails": [ // list of scheduled trip times between starttime and endtime in body
        {
          "starttime": <string>, // departure time from source stop (HH:MM)
          "endtime": <string> // arrival time at destination stop (HH:MM)
        },
        ...
      ]
    }
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 8. `/getWaypoints_v1`

---

### 9. 📍 Get Path Details Between Two Stations

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetPathDetails`

**Headers :**

```http
Content-Type: application/json
lan: en
deviceType: WEB
```

**Body :**

```
{
  "data": [
    {
      "fromStationId": <int>, // station ID from trip
      "toStationId": <int>,   // station ID from trip
      "tripId": <int>         // trip ID from /VehicleTripDetails_v2
    }
  ]
}
```

**Description :** Returns all intermediate stops, lat/lon, and scheduled times between two stations along a live bus trip. Used to retrieve the path segment of an ongoing trip.

**Response :**

```
{
  "data": [
    {
      "actual_arrivaltime": <string>,
      "actual_departuretime": <string>,
      "distance": <float>,
      "duration": null,
      "eta": <string>,
      "isTransfer": <boolean>, // always false for direct segments
      "latitude": <float>,
      "longitude": <float>,
      "routeId": <int>, // route ID of this trip segment
      "routeNo": <string>, // route number with direction (e.g. "600-F BSK-D32")
      "sch_arrivaltime": <string>, // scheduled arrival time (DD/MM/YYYY HH:MM:SS)
      "sch_departuretime": <string>, // scheduled departure time (DD/MM/YYYY HH:MM:SS)
      "stationId": <int>,
      "stationName": <string>,
      "tripId": <int>
    },
    ...
  ],
  "exception": null,
  "issuccess": <boolean>,
  "message": <string>,
  "responsecode": <int>,
  "rowCount": <int>
}
```

---

## Live

API calls that return live, frequently changing data, such as live coordinates for individual buses / all buses currently plying on a route.

---

### 1. 🚏 Find Stations by Route

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/SearchByRouteDetails_v4`

**Headers :**
```http
lan: en
deviceType: WEB
```

**Body :**
```
{
  "routeid": <int>, // routeid here is the routeparentid returned from a /SearchRoute_v2 request
  "servicetypeid": 0 // filter for AC or Non-AC/Ordinary bus routes (default 0 for no filter)
}
```

**Description :** Returns stop information for a particular `routeid`

**Response :** JSON containing live `data` array with entries for each stop on the up and down route, and `mapData` array with entries for buses plying on the up and down route

```
{
  "up": {
    "data": [
      {
        "routeid": <int>, // NOT the same as the routeid passed in the body, this is a new unique identifier
        "stationid": <int>, // station id for each station on the route
        "stationname": <string>,
        "from": <string>, // bus route origin
        "to": <string>, // bus route destination
        "routeno": <string>, // routeno from /SearchRoute_v2
        "distance_on_station": <int>,
        "centerlat": <float>, // station latitude coordinate
        "centerlong": <float>, // station longitude coordinate
        "responsecode": <int>,
        "isnotify": <int>,
        "vehicleDetails": [ // details for all vehicles currently plying on the up route
          {
            "vehicleid": <int>, // vehicle identifier
            "vehiclenumber": <string>, // vehicle license plate
            "servicetypeid": <int>, // number indicating service type of this vehicle
            "servicetype": <string>, // string containing "AC" or "Non AC/Ordinary"
            "centerlat": <float>, // vehicle latitude coordinate
            "centerlong": <float>, // vehicle longitude coordinate
            "eta": <string>, // vehicle eta (YYYY-MM-DD HH:MM:SS), empty string if vehicle has already passed the station
            "sch_arrivaltime": <string>, // scheduled arrival time at the station (HH:MM)
            "sch_departuretime": "<string", // scheduled departure time from the station (HH:MM)
            "actual_arrivaltime": "", // actual arrival time to the station (HH:MM), empty string if vehicle hasn't reached the station
            "actual_departuretime": "", // actual departure time from the station (HH:MM), empty string if vehicle hasn't left the station
            "sch_tripstarttime": <string>, // scheduled trip start time
            "sch_tripendtime": <string>, // scheduled trip end time
            "lastlocationid": <int>,
            "currentlocationid": <int>, // station id for last departed station
            "nextlocationid": <int>,
            "currentstop": null,
            "nextstop": null,
            "laststop": null,
            "stopCoveredStatus": <int>,
            "heading": <int>, // current compass heading of vehicle
            "lastrefreshon": <string>, // time data was collected (DD-MM-YYYY HH:MM:SS)
            "lastreceiveddatetimeflag": <int>,
            "tripposition": <int>
          }
        ]
      },
      ...
    ],
    "mapData": [ // essentially the vehicleDetails array from the terminus / destination station of the route
      {
        ...
      },
      ...
    ]
  },
  "down": { // same data but for the down route
    "data": [...],
    "mapData": [...]
  },
  "message": <string>,
  "issuccess": <boolean>,
  "exception": null,
  "rowCount": <int>,
  "responsecode": <int>
}
```

---

### 2. 📡 Live Track Vehicle

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/VehicleTripDetails_v2`

**Body :**
```
{
  "vehicleId": <int> // vehicleId from /ListVehicles
}
```

**Description :** Returns live vehicle tracking data.

**Response :** JSON containing 

```
{
  "RouteDetails": [ // lists info for each stop on the route
    {
      "rowid": <int>,
      "tripid": <int>, // new identifier for this particular trip
      "routeno": <string>,
      "routename": <string>, // origin and terminus shortcode (Ex : KIA-KDG)
      "busno": <string>, // vehicleregno from /LiveVehicles
      "tripstatus": <string>, // usually "Running"
      "tripstatusid": <string>,
      "sourcestation": <string>,
      "destinationstation": <string>,
      "servicetype": <string>,
      "webservicetype": <string>,
      "servicetypeid": <int>,
      "lastupdatedat": <string>, // query response timestamp (DD-MM-YYYY HH:MM:SS)
      "stationname": <string>,
      "stationid": <int>,
      "actual_arrivaltime": <string>, // (HH:MM)
      "etastatus": <string>, // original eta for stop (HH:MM)
      "etastatusmapview": <string>, // eta to display on website (HH:MM)
      "latitude": <float>,
      "longitude": <float>,
      "currentstop": <string>, // empty string if not currently stopped
      "laststop": <string>,
      "weblaststop": <string>, // stop name to display on website
      "nextstop": <string>,
      "currlatitude": <float>,
      "currlongitude": <float>,
      "sch_arrivaltime": <string>, // (HH:MM)
      "sch_departuretime": <string>, // (HH:MM)
      "eta": "", // eta (HH:MM), empty string is stop was passed
      "actual_arrivaltime1": <string>, // (HH:MM) duplicate
      "actual_departudetime": <string>, // (HH:MM) actually mispelled for some reason
      "tripstarttime": <string>, // (HH:MM)
      "tripendtime": <string>, // (HH:MM)
      "routeid": <int>,
      "vehicleid": <int>, // same as passed in body
      "responsecode": <int>,
      "lastreceiveddatetimeflag": <int>,
      "srno": <int>,
      "tripposition": <int>,
      "stopstatus": <int>,
      "stopstatus_distance": <int>, // always seems to return 999
      "lastetaupdated": null,
      "minstopstatus_distance": <float>
    },
    ...
  ],
  "LiveLocation": [
    {
      "latitude": <float>,
      "longitude": <float>,
      "location": <string>, // full address of vehicle current location
      "lastrefreshon": <string>, // same as lastupdatedat (DD-MM-YYYY HH:MM:SS)
      "nextstop": <string>,
      "previousstop": <string>,
      "vehicleid": <int>,
      "vehiclenumber": <string>, // vehicleregno from /ListVehicles
      "routeno": <string>,
      "servicetypeid": <int>,
      "servicetype": <string>,
      "heading": <int>, // compass heading
      "responsecode": <int>,
      "trip_status": <int>,
      "lastreceiveddatetimeflag": <int>
    }
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 3. `/TripPlannerMSMD`

---

## Misc (Frontend)

Miscellaneous data used mainly for frontend information, such as Google Maps API keys, up-to-date service types, or phone numbers. 

---

### 1. 🗺️ Get Map Config

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetMapConfig`

**Headers :**

```http
lan: en
Accept: application/json, text/plain, */*
```

**Body :** None required

**Description :** Returns configuration data for maps used by the BMTC web/Android applications. This includes the public Google Maps API key used by the frontend.

**Response :**

```
{
  "data": [
    {
      "api_key": <string> // Google Maps API key used by BMTC apps
    }
  ],
  "Message": <string>,
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

### 2. `/GetAllServiceTypes`

---

### 3. `/GetHelplineData`

---
