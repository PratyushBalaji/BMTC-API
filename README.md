
# BMTC Public API Documentation

This is work-in-progress documentation for the publicly accessible BMTC APIs. I've tried to reverse-engineer it by analysing network traffic on [the BMTC website](https://nammabmtcapp.karnataka.gov.in/commuter/dashboard).

Made with the help of the [Bruno API Client](https://www.usebruno.com/) and existing documentation([Swagger](https://nimmbus.netlify.app/), [GitHub](https://github.com/Vonter/open-bmtc)) by seadeep42 and Vonter on GitHub

---

## API Details

**Base URL :** `https ://bmtcmobileapi.karnataka.gov.in/WebAPI`

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
  "routetext" : "MF-23E"
}
```

**Description :** Finds bus routes whose route number contains the input text. Used for search autocomplete functionality.

Response is JSON containing an array `data` with entries for bus routes with `routetext` as a substring.

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
  "routeid" : 1699,
  "servicetypeid" : 0
}
```

**Description :** 

Returns stop information for a particular `routeid`. `routeid` here is NOT the `routeparentid` response field found from `/SearchRoute_v2`.

`servicetypeid` is a filter for AC or Non AC/Ordinary service (Default `0`).

Response is JSON containing 2 data arrays (`up`/`down`) and standard response information (`message`, `responsecode`, etc). 

`up` and `down` each contain `data` and `mapData`. `data` contains stop information for each stop on the respective route. Order is from route origin to destination. Important fields found are below :

Standard fields (Same for each entry)
- `routeid`
- `from`/`to`: Origin/Destination station of the route (reversed for up/down line)
- `routeno`: Route number for the `routeid`(For example : `routeno` MF-23E for `routeid` 12511 / `routeparentid` 6660)

Corresponding fields (Different for each entry)
- `stationid`: Identifier for the particular station
- `stationname`: Name of the particular station
- `distance_on_station`: Unsure, could be cumulative distance from origin? Some calls have all entries as 0, other times it is a stricly increasing series (likely cumulative distance covered in kms)
- `centerlat`: Station latitude coordinate
- `centerlong`: Station longitude coordinate
- `vehicleDetails`: Array containing details of live tracked buses on the route. Includes various details such as : 
  - `vehicleid`: Vehicle identifier
  - `vehiclenumber`: Vehicle license plate
  - `servicetypeid`: ID indicating AC/Non-AC
  - `servicetype`: String containing AC/Non-AC
  - `centerlat`,`centerlong`: Vehicle coordinates
  - `eta`: Empty string for stops the vehicle has crossed, otherwise estimated time of arrival (YYYY-MM-DD HH:MM:SS)
  - `sch_arrivaltime`,`sch_departuretime`: Scheduled arrival / departure time from the station (HH:MM)
  - `actual_arrivaltime`,`actual_departuretime`: Empty string for stops the vehicle hasn't crossed, otherwise actual arrival / departure time from the station (HH:MM)
  - `sch_tripstarttime`,`sch_tripendtime`: Scheduled trip start / end time (HH:MM)
  - `lastlocationid`
  - `currentlocationid`: `stationid` from currently stopped station
  - `nextlocationid`
  - `currentstop`,`nextstop`,`laststop`: Unsure, from testing, the fields' values are usually "null"
  - `stopCoveredStatus`
  - `heading`: Compass heading of the bus (degrees)
  - `lastrefreshon`: Time data was collected (DD-MM-YYYY HH:MM:SS)
  - `lastreceiveddatetimeflag`
  - `tripposition`

`mapData` is essentially just the `vehicleDetails` field from the terminus station.

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
  "stationName" : "indirana"
}
```

**Description :** Search for stops containing a substring.

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

---

## 5. 🧭 Route Points

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/RoutePoints`

**Body :**
```json
{
  "routeid": 1732
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
