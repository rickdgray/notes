# Vehicle API Endpoint Review & Gap Analysis

## ✅ Existing Vehicle-Related Endpoints

### 1. DealerTrack Integration (`POST /vehicleapi.asmx`)

- **Content Type:** `application/soap+xml`
- **Handles:**
  - `<VehicleLookup>` → `VehicleLookup.Request.Envelope`
  - `<VehicleAdd>` → `VehicleAdd.Request.Envelope`
- **Response:** Determined by Mediator pipeline
- **Notes:**
  - No detailed field/response documentation
  - XML SOAP-based; logic based on string matching in raw request body

---

### 2. Front-End Vehicle Management API (`/frontendapi/dealers/{dealerId}/vehicles`)

#### GET `/dealers/{dealerId}/vehicles`

- **Purpose:** Fetch all vehicles for a dealer
- **Params:** `dealerId` (GUID)
- **Returns:** `List<GetAllResponse>`
- **Status Codes:** `200 OK`, `400 Bad Request`, `500 Internal Server Error`

#### GET `/dealers/{dealerId}/vehicles/{vehicleId}`

- **Purpose:** Fetch a single vehicle
- **Params:** `dealerId`, `vehicleId` (GUIDs)
- **Returns:** `GetSingleResponse`

#### POST `/dealers/{dealerId}/vehicles`

- **Purpose:** Insert or update a vehicle
- **Body:** `UpsertCommand` (includes `DealerId`)
- **Returns:** `void | CommandResult`

#### DELETE `/dealers/{dealerId}/vehicles/{vehicelId}`

- **Purpose:** Delete a vehicle
- **Note:** Typo in route param `vehicelId` → should be `vehicleId`
- **Returns:** Result of `DeleteCommand`

---

### 3. CDK Pip Extract Integration (`POST /pip-extract/inventoryvehicleext/extract`)

- **Authentication:** Basic Auth (internal use via Postman)
- **Query Parameters:**
  - `dealerId` (optional)
  - `queryId` (optional)
  - `qparamInvCompany` (required)
  - `deltaDate` (optional)
- **Request Object:** `InventoryVehiclesQuery`
- **Returns:** XML output
- **Notes:** 
  - Lacks standard request body; uses query strings
  - No response schema documentation

---

### 4. DealerBuilt XML API (`POST /0.99a/Api.svc`)

- **Content Type:** `text/xml`
- **Routes all requests through a single XML handler**
- **String-based routing (examples):**
  - `<PullCustomers>`
  - `<PushProspects>`
  - `<PushCustomers>`
  - `<PushDeals>`
  - `<PushAppraisals>`
- **Vehicle operations** may be embedded in `<PushDeals>` or `<PushAppraisals>`

---

## 🚨 Identified Gaps & Inconsistencies

| Area                          | Issue                                        | Notes                                                |
|-------------------------------|----------------------------------------------|------------------------------------------------------|
| Route Typo                   | `vehicelId` should be `vehicleId`            | Minor fix required                                   |
| Missing Bulk Operations      | No bulk import/export endpoints              | Consider adding batch processing for vehicle data    |
| DealerBuilt Vehicle Handling | No explicit vehicle operations documented    | Vehicle data possibly embedded — needs confirmation  |
| Input/Output Contracts       | No schema or examples for SOAP endpoints     | Add sample requests/responses                        |
| API Style Inconsistency      | Mix of SOAP, REST, and query-based POST APIs | Consider unifying through a shared gateway strategy  |

---

## 🧠 Recommendations

1. **Fix Bugs**
   - Correct `vehicelId` route typo

2. **Enhance Documentation**
   - Add detailed request/response schemas
   - Provide example XML/JSON payloads
   - Add Swagger/OpenAPI for REST endpoints

3. **Clarify DealerBuilt Vehicle Operations**
   - Determine whether vehicle data is embedded in `<PushDeals>` or other nodes

4. **Implement Bulk Vehicle Operations**
   - Add endpoints like `POST /vehicles/bulk` or `POST /dealers/{id}/vehicles/bulk`
   - Allow batch import/export support

5. **Unify Integration Approach**
   - Create internal gateway abstraction around SOAP services
   - Normalize vehicle functionality under RESTful patterns where feasible

6. **Integration Support Matrix**
   - Create a matrix that shows which operations are supported by each provider
   - Helps quickly identify capability gaps

---

## ✅ Next Steps

- [ ] Fix route typo (`vehicelId`)
- [ ] Confirm DealerBuilt vehicle handling
- [ ] Design bulk import/export API contract
- [ ] Draft schema documentation for SOAP endpoints
- [ ] Propose unified gateway or API normalization strategy

