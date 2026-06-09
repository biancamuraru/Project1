# 📧 Setup Emailuri Gmail - AIRline Booking System

## ✅ Status Curent

Sistemul este **configurat și gata** să trimită emailuri reale pe Gmail! 

### Ce a fost implementat:

1. ✅ **EmailService.java** - Actualizat cu:
   - Trimitere HTML email-uri cu design profesional
   - Suport SMTP Gmail cu TLS
   - Detalii complete de rezervare în email

2. ✅ **application.properties** - Configurări Gmail:
   ```properties
   spring.mail.host=smtp.gmail.com
   spring.mail.port=587
   spring.mail.username=bianca.mru1306@gmail.com
   spring.mail.password=biancamaria1425
   spring.mail.properties.mail.smtp.auth=true
   spring.mail.properties.mail.smtp.starttls.enable=true
   spring.mail.properties.mail.smtp.starttls.required=true
   ```

3. ✅ **BookingController.java** - Trimite email cu:
   - Referință bilet unic
   - Ruta calatoriei
   - Detalii pasager
   - Link direct la bilet

---

## 🚀 Cum Funcționează

### 1. **Utilizatorul face o rezervare**
   - Selectează bilete, locuri, servicii
   - Apasă "FINALIZEAZA REZERVAREA"

### 2. **Serverul procesează**
   ```java
   // BookingController.submitBooking()
   String bookingReference = "ZBR" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
   // ex: ZBR7F3K9A2M1
   
   emailService.sendBookingConfirmation(
       userEmail,                    // Email din sesiune
       passengerName,                // Numele pasagerului
       flight.getFromCity(),         // Origine
       flight.getToCity(),           // Destinație
       bookingReference              // Referință bilet
   );
   ```

### 3. **EmailService construiește email HTML**
   - Design profesional cu gradient
   - Ruta vizuală (Sibiu → Hamburg)
   - Referință bilet highlighted
   - Instrucțiuni check-in
   - Link "VĂD BILETUL MEU"

### 4. **Gmail trimite emailul**
   - SMTP conectare via TLS (port 587)
   - Email sosit în inbox utilizatorului în segundă!

---

## 📧 Exemplu Email Trimis

```
TO: ion.popescu@gmail.com

SUBJECT: ✈️ Confirmare Rezervare AIRline - ZBR7F3K9A2M1

BODY (HTML):
┌─────────────────────────────────────────────┐
│           ✈️ AIRline                         │
│  Biletul tău de avion a fost confirmat!     │
├─────────────────────────────────────────────┤
│                                             │
│  Bună Ion Popescu,                          │
│                                             │
│  ✓ CONFIRMAT                                │
│                                             │
│       SBZ    →    HAM                       │
│                                             │
│  📋 Detalii Rezervare:                      │
│  • Pasager: Ion Popescu                     │
│  • Email: ion@gmail.com                     │
│  • Ruta: Sibiu → Hamburg                    │
│                                             │
│  🎫 Referință Bilet: ZBR7F3K9A2M1           │
│                                             │
│  📌 Informații Importante:                  │
│  ✓ Prezintă această confirmație la aeroport │
│  ✓ Vino 2 ore înainte de plecare            │
│  ✓ Adeverința e disponibilă în cont         │
│                                             │
│  [VĂD BILETUL MEU]                          │
│                                             │
├─────────────────────────────────────────────┤
│  AIRline © 2026                             │
│  contact@airline.ro | 0800 123 456          │
└─────────────────────────────────────────────┘
```

---

## ⚙️ Troubleshooting

### Dacă emailurile NU se trimit:

#### 1️⃣ **Verifica Gmail Security Settings**
   - Accesează: https://myaccount.google.com/security
   - Scroll down la "App passwords"
   - Generează App Password pentru "Mail" pe "Windows Computer"
   - Copiază parola și actualizează în `application.properties`:
   ```properties
   spring.mail.password=YOUR_APP_PASSWORD_HERE
   ```

#### 2️⃣ **Verifica Logs**
   ```bash
   # Cauta în console output:
   ✅ EMAIL TRIMIS CU SUCCES PE GMAIL: ion@gmail.com
   # sau
   ❌ EROARE SMTP: ...
   ```

#### 3️⃣ **Verifica application.properties**
   ```properties
   # ✅ Correct:
   spring.mail.host=smtp.gmail.com
   spring.mail.port=587
   spring.mail.username=bianca.mru1306@gmail.com
   spring.mail.password=XXXX
   spring.mail.properties.mail.smtp.auth=true
   spring.mail.properties.mail.smtp.starttls.enable=true
   spring.mail.properties.mail.smtp.starttls.required=true
   ```

#### 4️⃣ **Permite "Less secure apps"** (dacă app password nu merge)
   - Accesează: https://myaccount.google.com/u/0/security
   - Scroll down la "Less secure app access"
   - Activează "Allow less secure apps"

---

## 🧪 Test Manual

### Testează emailul din terminal:

```bash
# 1. Start server
./mvnw spring-boot:run

# 2. Accesează aplicația
http://localhost:8080/dashboard.html

# 3. Selectează un zbor și completează rezervarea

# 4. Verifica:
a) Console output - cauta "✅ EMAIL TRIMIS CU SUCCES PE GMAIL"
b) Gmail inbox - ar trebui să primești email în 1-2 secunde

# 5. Email should appear in Gmail with:
   - Subject: ✈️ Confirmare Rezervare AIRline - ZBR...
   - From: bianca.mru1306@gmail.com
   - Content: HTML profesional cu detalii bilet
```

---

## 🔐 Securitate

### Parola Gmail este în application.properties (ATENTIE!)

**⚠️ NU COMITA PAROLA ÎN GIT!**

### Soluție: Folosește Environment Variables

```bash
# Setează variabila de mediu:
set SPRING_MAIL_PASSWORD=YOUR_APP_PASSWORD

# Sau în application.properties:
spring.mail.password=${SPRING_MAIL_PASSWORD}

# Sau creeaza application-prod.properties cu override
```

---

## 📝 Codul Cheie

### EmailService.java
```java
MimeMessage mimeMessage = mailSender.createMimeMessage();
MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true, "UTF-8");

helper.setFrom("bianca.mru1306@gmail.com");
helper.setTo(recipientEmail);
helper.setSubject("✈️ Confirmare Rezervare AIRline - " + bookingReference);
helper.setText(htmlBody, true); // true = HTML content

mailSender.send(mimeMessage);
```

### BookingController.java
```java
String bookingReference = "ZBR" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();

emailService.sendBookingConfirmation(
    userEmail,                 // From session
    request.getPassengerName(),
    flight.getFromCity(),
    flight.getToCity(),
    bookingReference
);

return new BookingResponse(
    bookingReference,
    "✅ Rezervare confirmată pentru " + numberOfTickets + " bilet(e)!",
    totalPrice
);
```

---

## ✨ Features Email

- ✅ **HTML Design**: Profesional cu gradient albastru
- ✅ **Responsive**: Arată bine pe desktop și mobil
- ✅ **Detalii Complete**: Pasager, rută, referință
- ✅ **Referință Unica**: Fiecare bilet cu UUID
- ✅ **Link Interactiv**: Button "VĂD BILETUL MEU"
- ✅ **Instrucțiuni**: Check-in info și reminder
- ✅ **Branding**: Logo AIRline și contact info

---

## 📞 Contact

**Probleme cu emailuri?**
1. Verifica console logs pentru erori SMTP
2. Asigură-te că App Password e corect
3. Verifica firewall/proxy settings
4. Verifica că SMTP port 587 e deschis

---

## 📚 Resurse

- [Gmail SMTP Settings](https://support.google.com/mail/answer/7126229)
- [Spring Mail Documentation](https://spring.io/guides/gs/sending-email/)
- [Gmail App Passwords](https://support.google.com/accounts/answer/185833)

