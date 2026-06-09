# 📝 Rezumat Schimbări - Sistem Dinamic de Selectare Bilete și Prețuri

## 🎯 Obiectiv
Implementarea unui sistem complet de selectare bilete cu:
- ✅ Selectare dinamică a numărului de bilete
- ✅ Hartă interactivă de locuri disponibile
- ✅ Prețuri dinamice bazate pe ocupare
- ✅ Deducere automată a locurilor după rezervare

---

## 📋 Fișiere Modificate

### 1. **BookingRequest.java** - DTO Actualizat
**Locație:** `src/main/java/com/app/zboruri/dto/BookingRequest.java`

**Adăugiri:**
```java
private int numberOfTickets;           // Numărul de bilete de cumpărat
private List<String> selectedSeats;    // Lista de locuri selectate (ex: ["1A", "2B", "3C"])
```

**Getters & Setters:** Adăugate pentru ambele câmpuri noi

---

### 2. **Flight.java** - Model cu Prețuri Dinamice
**Locație:** `src/main/java/com/app/zboruri/model/Flight.java`

**Schimbări Principale:**
- `price` → `basePrice`: Prețul de bază înainte de ajustări
- Adăugat: `int totalSeats` - capacitatea totală
- Adăugat: `int bookedSeats` - locuri rezervate
- `seatsAvailable` - actualizat din booked seats

**Metoda crucială - Prețuri Dinamice:**
```java
public double calculateDynamicPrice() {
    double occupancyRate = (double) bookedSeats / totalSeats;
    
    if (occupancyRate > 0.80) {
        return basePrice * 1.50;  // +50% dacă 80% ocupare
    } else if (occupancyRate > 0.50) {
        return basePrice * 1.25;  // +25% dacă 50-80% ocupare
    } else {
        return basePrice;         // Preț standard dacă <50% ocupare
    }
}
```

**Metoda de rezervare:**
```java
public boolean bookSeats(int numberOfSeats) {
    if (numberOfSeats > seatsAvailable) return false;
    this.bookedSeats += numberOfSeats;
    this.seatsAvailable -= numberOfSeats;
    return true;
}
```

---

### 3. **BookingService.java** - Logică de Deducere Locuri
**Locație:** `src/main/java/com/app/zboruri/service/BookingService.java`

**Adăugiri:**
```java
@Autowired
private FlightRepository flightRepository;

// Metoda pentru calculul prețului cu bilete multiple
public double calculateTicketPrice(Flight flight, int numberOfTickets) {
    double pricePerTicket = flight.calculateDynamicPrice();
    return pricePerTicket * numberOfTickets;
}

// Verificare disponibilitate locuri
public boolean isSeatsAvailable(Flight flight, int numberOfTickets) {
    return flight.getSeatsAvailable() >= numberOfTickets;
}
```

**Modificare metodă book():**
- Acum deducere automată de locuri din zbor
- Validare disponibilitate înainte de salvare
- Actualizare Flight în baza de date

---

### 4. **BookingController.java** - Endpoint Nou & Logică Îmbunătățită
**Locație:** `src/main/java/com/app/zboruri/controller/BookingController.java`

**Endpoint NOU - Pricing & Availability:**
```
GET /booking/flight/{flightId}/pricing?numberOfTickets=N
```

**Răspuns:**
```json
{
    "success": true,
    "seatsAvailable": 30,
    "totalSeats": 150,
    "bookedSeats": 120,
    "pricePerTicket": 134.99,
    "occupancyRate": 80.0,
    "canBook": true,
    "totalPrice": 404.97
}
```

**Modificări submitBooking():**
- Acceptă `numberOfTickets` și `selectedSeats` din request
- Validare disponibilitate locuri
- Calculă preț dinamic × numărul de bilete
- Adaugă taxe și servicii per bilet

---

### 5. **flight-details.html** - UI Complet Redesenat
**Locație:** `src/main/resources/static/flight-details.html`

#### **Noi Componente:**

##### A) **Selector Numărul de Bilete**
```html
<div class="ticket-counter">
    <button onclick="decreaseTickets()">−</button>
    <input type="number" id="numberOfTickets" value="1" min="1" max="9" readonly>
    <button onclick="increaseTickets()">+</button>
</div>
```

##### B) **Hartă Interactivă de Locuri**
- Grid 10×8 cu 80 de locuri
- Locuri disponibile: albastru (clickabil)
- Locuri ocupate: roșu (dezactivat)
- Locuri selectate: gradient albastru
- Legendă cu coduri de culori

```javascript
function generateSeatMap() {
    // Generează 10 rânduri × 8 locuri
    // Locurile ocupate: ['3C', '5A', '7F', '2B', '8D']
    // Click selectează loc dacă nu este plin
}
```

##### C) **Indicator de Ocupare în Timp Real**
```javascript
// Bară de progres cu culori:
// Verde (0-50%) → Portocaliu (50-80%) → Roșu (80%+)
const occupancyRate = (bookedSeats / totalSeats) * 100;
```

##### D) **Recalculare Prețuri Dinamice**
- Se apelează `/booking/flight/{flightId}/pricing` la fiecare schimbare
- Actualizează:
  - Preț per bilet
  - Subtotal bilete
  - Total cu taxe și servicii
  - Indicator culoare ocupare

#### **Flow JavaScript:**
1. `loadFlightDetails()` - încarcă zbor și date utilizator
2. `updatePricingAndSeats()` - cere API pricing actual
3. `generateSeatMap()` - creează grid locuri
4. `increaseTickets() / decreaseTickets()` - ajustează cantitate
5. `toggleSeat(seatId)` - selectează/deselectează loc
6. `updateSummary()` - recalculează total
7. `submitBooking()` - trimite date serverului

---

## 🔄 Flow Rezervare Completă

### Step 1: Utilizator accesează flight-details.html
```
1. Server: GET /flights/{id} → Flight + details
2. Client: GET /booking/flight/{id}/pricing → Preț + ocupare
3. UI: Afișează preț actual și hartă locuri
```

### Step 2: Selectează bilete
```
1. User: Apasă + pentru a crește bilete (1→2→3)
2. JS: Regenerează hartă locuri (schimbă limita selectare)
3. JS: Recalculează preț dinamic
4. UI: Afișează "Selectează 3 locuri pentru 3 bilete"
```

### Step 3: Selectează locuri
```
1. User: Click pe locuri disponibile
2. JS: Validează (trebuie egal cu numberOfTickets)
3. UI: Highlight selectate în albastru
```

### Step 4: Completează date & apasă finalizare
```
POST /booking/submit
{
    "flightId": 1,
    "numberOfTickets": 3,
    "selectedSeats": ["1A", "1B", "2D"],
    "passengerName": "Ion Popescu",
    "passengerEmail": "ion@gmail.com",
    "passengerPhone": "0712345678",
    "services": ["bagaj", "asigurare"],
    "paymentMethod": "card"
}
```

### Step 5: Server
```java
1. Verifică locuri disponibile
2. Calculează preț dinamic × 3
3. Adaugă taxe (10€ × 3)
4. Adaugă servicii (20€ + 10€ per bilet)
5. Deducere locuri: flight.bookSeats(3)
6. Salvează Booking
7. Trimite email confirmare
8. Returnează referință ZBR-XXXXX
```

---

## 📊 Exemplu Preț Dinamic

### Scenariu: Zbor cu 150 locuri

| Ocupare | Preț per bilet | 3 bilete | Taxe (3×10€) | Servicii (3×10€) | **TOTAL** |
|---------|---|---|---|---|---|
| 20% (30 ocupate) | 89,99€ | 269,97€ | 30€ | 30€ | **329,97€** |
| 50% (75 ocupate) | 89,99€ | 269,97€ | 30€ | 30€ | **329,97€** |
| 60% (90 ocupate) | 112,49€ (+25%) | 337,47€ | 30€ | 30€ | **397,47€** |
| 85% (127 ocupate) | 134,99€ (+50%) | 404,97€ | 30€ | 30€ | **464,97€** |
| 100% (150 ocupate) | ❌ Nu se pot vinde! | — | — | — | — |

---

## 🔧 Testare

### Test Local:
```bash
1. mvn clean compile
2. mvn spring-boot:run
3. Accesează: http://localhost:8080/flight-details.html?id=1
4. Selectează bilete și locuri
5. Finalizează rezervare
```

### Verificări:
- ✅ Bilete multiple selectează mai multe locuri
- ✅ Preț se actualizează după ocupare
- ✅ Nu poti selecta mai multe locuri decât bilete
- ✅ Locurile ocupate sunt dezactivate
- ✅ După rezervare, locurile sunt deduse automat
- ✅ Email de confirmare trimis

---

## 📦 Dependențe Necesare

Toate sunt deja în `pom.xml`:
- Spring Data JPA
- Spring Web
- MySQL/H2 Driver
- Lombok (optional)

---

## 🚀 Deployment

```bash
./mvnw clean package
java -jar target/airline-0.0.1-SNAPSHOT.jar
```

---

## 📝 Note de Dezvoltare

### Posibile Extensii:
1. **Locuri cu Preț Special:** Locuri lângă geam +15€
2. **Upgrade Clasă:** Economy → Business
3. **Combinații Speciale:** Familie (4 bilete = -10%)
4. **Vânzare Last-Minute:** Ceas cu numer ore până plecare
5. **Notificări:** SMS/Email când preț scade

### Limitări Curente:
- Hartă locuri statică (hard-coded 10×8)
- Locuri ocupate hard-coded în JavaScript
- Fără persistență pentru locuri ocupate în BD

### Îmbunătățiri Viitoare:
- Tabel `SeatBooking` în BD pentru persistență
- Locuri cu poziție și preț diferit
- Real-time seat updates cu WebSockets
- Payment gateway integration

---

## 📞 Suport

Pentru întrebări sau probleme, verifica:
1. Console browser (F12)
2. Server logs (terminal cu spring-boot:run)
3. Baza de date (flights table)

