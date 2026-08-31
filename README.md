### Stage Breakdown & Progression

| Stage Name | Step / Action | Step Type | Description | Assigned Persona / Routing |
| :--- | :--- | :--- | :--- | :--- |
| **1. Submit Request** | `Submit Booking Request` | Form Step (Collect Information) | Captures customer contact details, selected movie, show date/time, show type, and ticket quantity. Computes total cost dynamically. | Customer / Author |
| **2. Availability** | `Check Show Availability` | User Action | Checks theater hall capacity, calculates remaining seats, and sets availability status. | Booking Agent |
| **3. Approval** | `Confirm Booking Request` | User Action / Review | Managerial review of consolidated booking details, ticket quantities, and computed pricing. | Manager / Approver |
| **4. Booking Execution** | `Allocate Seats & Process` | Form / Multi-step Assignment | Conditionally routes the case by `ShowType` (Premium vs Standard) and assigns physical seat numbers. | Work Queues (`AcctMgmt:CSR` / `AcctMgmt:Users`) |
| **5. Resolution** | `Notify Booking Confirmation` | Send Email (Correspondence) | Dispatches an automated confirmation email containing booking reference and ticket details to the customer. Resolves case to `Resolved-Completed`. | Automation (System) |

---

## 👥 Personas & Work Queues

### Personas
* **Customer / Author:** Initiates and submits the movie ticket reservation request.
* **Booking Agent:** Verifies theater hall capacity and ticket inventory availability.
* **CSR (Customer Service Representative):** Manages high-priority allocations and VIP/Premium show bookings.
* **Manager / Administrator:** Evaluates approval criteria and manages master movie/show catalogs.

### Work Queues & Routing Rules
* **`AcctMgmt:CSR`**: Receives booking assignments when `Show Type` equals `"Premium"`.
* **`AcctMgmt:Users`**: Receives booking assignments for all standard and general show types (`otherwise` route).

---

## 💾 Data Models & Data Objects

### 1. `Movie` Data Object
* **`MovieName`** (*Text*): Title of the movie.
* **`MovieID`** (*Text*): Unique identifier for the catalog record.
* **`Genre`** (*Text*): Action, Drama, Sci-Fi, Comedy, Thriller, etc.
* **`Duration`** (*Text / Integer*): Movie duration in minutes.
* **`Language`** (*Text*): Audio language of the screening.
* **`Description`** (*Text*): Brief synopsis and plot outline.

### 2. `Show` Data Object
* **`ShowID`** (*Text*): Primary identifier for the specific show schedule.
* **`MovieName`** (*Text*): Associated movie title.
* **`ShowDate`** (*Date*): Date of screening.
* **`ShowTime`** (*TimeOfDay*): Scheduled start time.
* **`ShowType`** (*Picklist*): `Premium`, `Standard`.
* **`VenueCapacity`** (*Integer*): Total capacity of the screening hall.
* **`AvailableSeatsCount`** (*Integer*): Remaining seats available for allocation.
* **`TicketPrice`** (*Currency*): Unit base price per ticket.

---

## ⚙️ Key Technical Implementations

### 1. Automated Total Cost Calculation
* **Rule Name:** `TotalCost`
* **Rule Type:** Calculated Field / Declare Expression
* **Target Property:** `.TotalCost` (Currency, Read-Only)
* **Formula:**
  $$\text{TotalCost} = \text{TicketPrice\_1} \times \text{NumberOfTickets}$$
  *(Pega Expression: `.TicketPrice_1 * .NumberOfTickets`)*

### 2. Service Level Agreement (SLA)
* **Goal:** `1 Day` (Urgency increment: `+10`)
* **Deadline:** `2 Days` (Urgency increment: `+20`)
* **Passed Deadline:** Automated escalation to supervising manager.

### 3. Business Logic Routing
* **Context:** Booking Execution step assignment routing.
* **Configuration:**
  
```pegascript
  IF .ShowType == "Premium"
      ROUTE TO WorkQueue "AcctMgmt:CSR"
  OTHERWISE
      ROUTE TO WorkQueue "AcctMgmt:Users"
