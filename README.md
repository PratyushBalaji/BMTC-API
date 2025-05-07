
# BMTC Public API Documentation

This is work-in-progress documentation for the publicly accessible BMTC APIs. I've tried to reverse-engineer it by analysing network traffic on [the BMTC website](https://nammabmtcapp.karnataka.gov.in/commuter/dashboard).

Made with the help of the [Bruno API Client](https://www.usebruno.com/) and existing documentation([Swagger](https://nimmbus.netlify.app/), [GitHub](https://github.com/Vonter/open-bmtc)) by seadeep42 and Vonter on GitHub

---

## API Details

**Base URL :** `https://bmtcmobileapi.karnataka.gov.in/WebAPI`

**API routes :** 
- **Static**
  - `/SearchRoute_v2`
  - `/ListVehicles`
  - `/FindNearByBusStop_v2`
  - `/AroundBusStops_v2_Webportal`
  - `/GetFareRoutes`
  - `/GetMobileFareData_v2`
  - `/GetTimetableByStation_v4`
  - `/GetTimetableByRouteid_v3`
  - `/GetMapConfig`
- **Live**
  - `/SearchByRouteDetails_v4`
  - `/VehicleTripDetails_v2`
  - `/TripPlannerMSMD`

---

## 1. 🔍 Search Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/SearchRoute_v2`

**Headers :**
```http
Content-Type: application/json
lan: en
```

**Body :**
```json
{
  "routetext": "<input text>"
}
```

**Description :** Finds bus routes whose route number contains the input text. Used for search autocomplete functionality.

**Response :** JSON containing `data` array with entries for bus routes with `routetext` as a substring.
```json
{
  "data": [
    {
      "union_rowno": <int>,
      "row": <int>,
      "routeno": "<string>",
      "responsecode": <int>,
      "routeparentid": <int>
    },
    ...
  ],
  "Message": "<string>",
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```

---

## 2. 🚏 Find Stations by Route

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/SearchByRouteDetails_v4`

**Headers :**
```http
lan: en
deviceType: WEB
```

**Body :**
```json
{
  "routeid": <int>, // routeid here is the routeparentid returned from a /SearchRoute_v2 request
  "servicetypeid": 0 // filter for AC or Non-AC/Ordinary bus routes (default 0 for no filter)
}
```

**Description :** Returns stop information for a particular `routeid`

**Response :** JSON containing live `data` array with entries for each stop on the up and down route, and `mapData` array with entries for buses plying on the up and down route

```json
{
  "up": {
    "data": [
      {
        "routeid": <int>, // NOT the same as the routeid passed in the body, this is a new unique identifier
        "stationid": <int>, // station id for each station on the route
        "stationname": "<string>",
        "from": "<string>", // bus route origin
        "to": "<string>", // bus route destination
        "routeno": "<string>", // routeno from /SearchRoute_v2
        "distance_on_station": <int>,
        "centerlat": <float>, // station latitude coordinate
        "centerlong": <float>, // station longitude coordinate
        "responsecode": <int>,
        "isnotify": <int>,
        "vehicleDetails": [ // details for all vehicles currently plying on the up route
          {
            "vehicleid": <int>, // vehicle identifier
            "vehiclenumber": "<string>", // vehicle license plate
            "servicetypeid": <int>, // number indicating service type of this vehicle
            "servicetype": "<string>", // string containing "AC" or "Non AC/Ordinary"
            "centerlat": <float>, // vehicle latitude coordinate
            "centerlong": <float>, // vehicle longitude coordinate
            "eta": "<string>", // vehicle eta (YYYY-MM-DD HH:MM:SS), empty string if vehicle has already passed the station
            "sch_arrivaltime": "<string>", // scheduled arrival time at the station (HH:MM)
            "sch_departuretime": "<string", // scheduled departure time from the station (HH:MM)
            "actual_arrivaltime": "", // actual arrival time to the station (HH:MM), empty string if vehicle hasn't reached the station
            "actual_departuretime": "", // actual departure time from the station (HH:MM), empty string if vehicle hasn't left the station
            "sch_tripstarttime": "<string>", // scheduled trip start time
            "sch_tripendtime": "<string>", // scheduled trip end time
            "lastlocationid": <int>,
            "currentlocationid": <int>, // station id for last departed station
            "nextlocationid": <int>,
            "currentstop": null,
            "nextstop": null,
            "laststop": null,
            "stopCoveredStatus": <int>,
            "heading": <int>, // current compass heading of vehicle
            "lastrefreshon": "<string>", // time data was collected (DD-MM-YYYY HH:MM:SS)
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
  "message": "<string>",
  "issuccess": <boolean>,
  "exception": null,
  "rowCount": <int>,
  "responsecode": <int>
}
```

---

## 3. 📍 Find Bus Stop by Name

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/FindNearByBusStop_v2`

**Headers :**
```http
Accept: text/plain
lan: en
Content-Type: application/json
```

**Body :**
```json
{
  "stationName": "<input text>"
}
```

**Description :** Search for stops containing a substring.

**Response :** JSON containing `data` array with entries for bus stops with `stationName` as a substring.

```json
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
  "Message": "<string>",
  "Issuccess": <boolean>,
  "exception": null,
  "RowCount": <int>,
  "responsecode": <int>
}
```
---

## 4. 🏪 Facilities Around Stations

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/AroundBusStops_v2_Webportal`

**Body :**
```json
{
  "deviceType": "WEB",
  "lan": "en"
}
```

**Description :** Returns nearby facilities for stations.

**Response :** 

---

## 5. 🧭 Route Points

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/RoutePoints`

**Body :**
```json
{
  "routeid": <int>
}
```

**Description :** Returns lat and long data in an array for stations on a bus route, in an order corresponding to `SearchByRouteDetails_v4`

---

## 6. 🚐 List Vehicles by Substring

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/ListVehicles`

**Headers :**
```http
lan: en
deviceType: WEB
Content-Type: application/json
Accept: text/plain
```

**Body :**
```json
{
  "vehicleRegNo" : "KA57F2201",
  "deviceType" : "WEB",
  "lan" : "en"
}
```

**Description :** Find vehicles using part of the registration number.

---

## 7. 📡 Live Track Vehicle

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/VehicleTripDetails_v2`

**Body :**
```json
{
  "vehicleId": 19285
}
```

**Description :** Returns live vehicle tracking data.

---

## 8. 💰 Get Fare Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetFareRoutes`

**Headers :**
```http
Accept: application/json, text/plain, */*
lan: English
Content-Type: application/json
```

**Body :**
```json
{
  "fromStationId": 38642,
  "toStationId": 38888,
  "lan": "English"
}
```

**Description :** Returns all fare routes between two station IDs.

---

## 9. 💸 Get Fare Data

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetMobileFareData_v2`

**Headers :**
```http
lan: en
```

**Body :**
```json
{
  "routeno": "335-E",
  "routeid": 1701,
  "route_direction": "Down",
  "source_code": "KDB-1",
  "destination_code": "KBS3"
}
```

**Description :** Returns fare amount and service type for a given route and stop codes.
