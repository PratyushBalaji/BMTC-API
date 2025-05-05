
# BMTC Public API Documentation

This document summarises the publicly accessible BMTC APIs reverse-engineered from analysing network traffic on [the BMTC website](https://nammabmtcapp.karnataka.gov.in/commuter/dashboard).

Made with the help of the [Bruno API Client](https://www.usebruno.com/) and [existing documentation](https://nimmbus.netlify.app/) (outdated) by seadeep42 and Vonter on GitHub

---

## 1. 🔍 Search Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/SearchRoute_v2`

**Headers:**
```http
Content-Type: application/json
lan: en
```

**Body:**
```json
{
  "routetext": "MF-23E"
}
```

**Description:** Finds bus routes whose route number contains the input text.

---

## 2. 🚏 Find Stations by Route

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/SearchByRouteDetails_v4`

**Headers:**
```http
lan: en
deviceType: WEB
```

**Body:**
```json
{
  "routeid": 1699,
  "servicetypeid": 0
}
```

**Description:** Returns stop information for a route based on internal ID.

---

## 3. 📍 Find Bus Stop by Name

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/FindNearByBusStop_v2`

**Headers:**
```http
Accept: text/plain
lan: en
Content-Type: application/json
```

**Body:**
```json
{
  "stationName": "indirana"
}
```

**Description:** Search for stops containing a substring.

---

## 4. 🏪 Facilities Around Stations

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/AroundBusStops_v2_Webportal`

**Body:**
```json
{
  "deviceType": "WEB",
  "lan": "en"
}
```

**Description:** Returns nearby facilities for stations.

---

## 5. 🧭 Route Points

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/RoutePoints`

**Body:**
```json
{
  "routeid": 1732
}
```

**Description:** Returns lat and long data in an array for stations on a bus route, in an order corresponding to `SearchByRouteDetails_v4`

---

## 6. 🚐 List Vehicles by Substring

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/ListVehicles`

**Headers:**
```http
lan: en
deviceType: WEB
Content-Type: application/json
Accept: text/plain
```

**Body:**
```json
{
  "vehicleRegNo": "KA57F2201",
  "deviceType": "WEB",
  "lan": "en"
}
```

**Description:** Find vehicles using part of the registration number.

---

## 7. 📡 Live Track Vehicle

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/VehicleTripDetails_v2`

**Body:**
```json
{
  "vehicleId": 19285
}
```

**Description:** Returns live vehicle tracking data.

---

## 8. 💰 Get Fare Routes

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetFareRoutes`

**Headers:**
```http
Accept: application/json, text/plain, */*
lan: English
Content-Type: application/json
```

**Body:**
```json
{
  "fromStationId": 38642,
  "toStationId": 38888,
  "lan": "English"
}
```

**Description:** Returns all fare routes between two station IDs.

---

## 9. 💸 Get Fare Data

**POST** `https://bmtcmobileapi.karnataka.gov.in/WebAPI/GetMobileFareData_v2`

**Headers:**
```http
lan: en
```

**Body:**
```json
{
  "routeno": "335-E",
  "routeid": 1701,
  "route_direction": "Down",
  "source_code": "KDB-1",
  "destination_code": "KBS3"
}
```

**Description:** Returns fare data and stage list for a given route and stop codes.
