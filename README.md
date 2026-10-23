UPI Offline Mesh – Demo Project

About the Project

This project is a demo of an offline payment system using a mesh network concept. The main idea is to allow a payment message to move between nearby phones even when there is no internet connection.

For example, imagine that two people are in a basement where there is no network. One person wants to send ₹500 to another person. The payment data is encrypted and passed between nearby devices. When one of the devices gets an internet connection, it sends the payment message to the backend for processing.

I have used Spring Boot to develop the backend and created a software-based mesh simulator to demonstrate the complete process on a single laptop. No actual Bluetooth hardware is required for this demo.

The project mainly focuses on:

- Sending encrypted payment data through multiple devices.
- Preventing the same payment from being processed more than once.
- Detecting modified or repeated payment messages.
- Updating account balances and maintaining a transaction record.

Note: This is a college/learning project to understand the concept. It is not a real UPI application and does not process actual bank payments.

---

1. What I Wanted to Demonstrate

While working on this project, I focused on three important problems.

1. Keeping payment information secure

The payment data is encrypted before it travels through the simulated network. Intermediate devices carry the encrypted message without directly reading its contents. RSA and AES-GCM are used for encryption and integrity checking.

2. Avoiding duplicate payments

The same payment message may reach the backend from more than one device. The backend checks whether it has already received that message, so duplicate requests can be rejected.

3. Detecting invalid messages

If the encrypted data is modified or an old payment message is replayed, the backend performs checks before allowing the payment to reach the settlement process.

These features can be demonstrated through the dashboard and the included tests, depending on the test results in the current code.

---

2. Technologies Used

The main technologies and concepts used in this project are:

- Java 17 – programming language.
- Spring Boot – backend development.
- Spring Data JPA – database operations.
- H2 Database – temporary database for the demo.
- RSA-OAEP – encryption of the AES key.
- AES-256-GCM – encryption and integrity verification of payment data.
- SHA-256 – generating a hash of the encrypted payment message.
- ConcurrentHashMap – checking duplicate requests in the running application.
- Maven Wrapper – building and running the project.
- HTML – dashboard interface.

The mesh network is simulated in software. The project does not establish actual Bluetooth connections between phones.

---

3. How to Run the Project

Requirements

Before running the application, make sure Java 17 or a newer compatible version is installed.

Check the Java version using:

java -version

The project includes a Maven Wrapper, so a separate Maven installation should not be necessary.

On Windows

Open Command Prompt or PowerShell in the project folder and run:

.\mvnw.cmd spring-boot:run

On Linux or macOS

Run:

./mvnw spring-boot:run

The first run may take longer because Maven needs to download the required dependencies.

Open the dashboard

After the application starts successfully, open the following address in your browser:

http://localhost:8080

The dashboard allows you to try the payment flow and view the simulated devices, account balances and transaction records.

Stop the application

Press "Ctrl + C" in the terminal.

Run the tests

On Windows:

.\mvnw.cmd test

On Linux or macOS:

./mvnw test

---

4. Working of the Project

The demo can be understood in four steps.

Step 1: Creating a payment

First, select the sender, receiver, amount and other required details on the dashboard. Click the Inject into Mesh button.

The backend creates a payment instruction containing details such as the sender, receiver, amount, nonce and timestamp. The payment data is encrypted and placed inside a mesh packet.

The packet is then added to the first virtual device.

Step 2: Passing the packet between devices

Click Run Gossip Round to simulate communication between devices.

During a gossip round, devices share the packets they hold with other devices in the simulated network. The packet's time-to-live (TTL) value is reduced as it moves through the network.

This represents the basic idea of passing a message between nearby phones. Actual Bluetooth communication is not implemented in this version.

Step 3: Sending the packet to the backend

Click Bridges Upload to Backend.

The virtual bridge device represents a phone that has regained internet connectivity. It sends the packet to the backend through the bridge ingestion API.

The backend then:

1. Calculates the hash of the ciphertext.
2. Checks whether the message has already been received.
3. Decrypts the payment data if the message is accepted.
4. Checks the timestamp.
5. Attempts to process the payment and update the account balances.
6. Records the transaction in the ledger if settlement succeeds.

The dashboard can be used to inspect the resulting account balances and transaction history.

Step 4: Checking duplicate requests

The same payment packet can be submitted more than once to check how the backend handles duplicates.

The project uses an atomic operation in "ConcurrentHashMap" to allow only one request to claim a particular ciphertext hash within the running application.

A concurrency test is also included to check whether three simultaneous requests for the same packet result in only one successful settlement.

---

5. Project Architecture

The application is divided into different parts to keep the code organised.

Dashboard and API layer

The dashboard is used to interact with the demo. The REST API receives requests and provides information about the accounts, transactions and mesh state.

Mesh simulation layer

The mesh simulator contains virtual devices and handles the exchange of packets between them.

Encryption layer

The encryption service handles the RSA and AES-GCM operations. The payment data remains encrypted while it is carried through the simulated network.

Duplicate-checking layer

The idempotency service checks whether a ciphertext hash has already been claimed.

Settlement layer

The settlement service handles the debit, credit and transaction record using a database transaction.

Simplified flow

Create Payment
      |
      v
Encrypt Payment Data
      |
      v
Create Mesh Packet
      |
      v
Virtual Devices Share Packet
      |
      v
Bridge Gets Internet
      |
      v
Backend Receives Packet
      |
      v
Check Duplicate + Decrypt
      |
      v
Validate Payment
      |
      v
Update Balances and Ledger

---

6. Important Problems and Their Solutions

Problem 1: Protecting payment data

When a payment message passes through other devices, the data should not be directly readable by those devices.

To address this, the project uses hybrid encryption. AES-256-GCM encrypts the payment information, while RSA-OAEP encrypts the AES key. The GCM authentication tag also helps detect changes to the encrypted data.

The encryption and decryption logic is handled in "HybridCryptoService.java".

Problem 2: Handling duplicate payments

If the same packet reaches the backend several times, processing every request could result in an incorrect balance update.

The "IdempotencyService.java" class uses "ConcurrentHashMap.putIfAbsent()" to check and claim a ciphertext hash atomically. Only the request that successfully claims the hash proceeds through the normal processing flow.

The project also uses a unique constraint on the transaction packet hash as an additional database-level check, if configured as described in the code.

For a real system running on several backend servers, a shared solution such as Redis and proper database constraints would be needed.

Problem 3: Preventing replay attacks

A replay attack involves sending a previously captured message again.

The backend checks the timestamp inside the decrypted payment instruction. The ciphertext hash is also used to identify repeated copies of the same packet.

These checks demonstrate basic replay protection. A production payment system would require additional safeguards, including secure sender authentication, key management and carefully designed transaction identifiers.

---

7. Project File Structure

The important files in the project are organised as follows:

upi-offline-mesh/
├── pom.xml
├── mvnw
├── mvnw.cmd
├── README.md
└── src/
    ├── main/
    │   ├── resources/
    │   │   ├── application.properties
    │   │   └── templates/
    │   │       └── dashboard.html
    │   └── java/com/demo/upimesh/
    │       ├── UpiMeshApplication.java
    │       ├── model/
    │       │   ├── Account.java
    │       │   ├── AccountRepository.java
    │       │   ├── Transaction.java
    │       │   ├── TransactionRepository.java
    │       │   ├── MeshPacket.java
    │       │   └── PaymentInstruction.java
    │       ├── crypto/
    │       │   ├── ServerKeyHolder.java
    │       │   └── HybridCryptoService.java
    │       ├── service/
    │       │   ├── DemoService.java
    │       │   ├── VirtualDevice.java
    │       │   ├── MeshSimulatorService.java
    │       │   ├── IdempotencyService.java
    │       │   ├── SettlementService.java
    │       │   └── BridgeIngestionService.java
    │       ├── controller/
    │       │   ├── ApiController.java
    │       │   └── DashboardController.java
    │       └── config/
    │           └── AppConfig.java
    └── test/
        └── java/com/demo/upimesh/
            └── IdempotencyConcurrencyTest.java

What the main files do

- "UpiMeshApplication.java" – starts the Spring Boot application.
- "Account.java" and "Transaction.java" – represent accounts and transaction records.
- "MeshPacket.java" – represents a packet travelling through the simulated network.
- "PaymentInstruction.java" – contains the payment details used by the backend.
- "HybridCryptoService.java" – handles encryption, decryption and ciphertext hashing.
- "MeshSimulatorService.java" – simulates communication between virtual devices.
- "IdempotencyService.java" – checks duplicate packet submissions.
- "BridgeIngestionService.java" – handles incoming packets and coordinates validation.
- "SettlementService.java" – processes account balance changes and transaction records.
- "ApiController.java" – provides the REST endpoints.
- "dashboard.html" – contains the interface used to run the demo.

---

8. API Endpoints

The following endpoints are available in the project, according to the current API design.

Method| Endpoint| Purpose
GET| "/"| Opens the dashboard
GET| "/api/server-key"| Returns the server's public key
GET| "/api/accounts"| Displays account information
GET| "/api/transactions"| Returns recent transactions
GET| "/api/mesh/state"| Shows the virtual device states
POST| "/api/demo/send"| Creates and injects a demo payment
POST| "/api/mesh/gossip"| Simulates a gossip round
POST| "/api/mesh/flush"| Uploads packets held by bridge devices
POST| "/api/mesh/reset"| Resets the simulated mesh
POST| "/api/bridge/ingest"| Receives a packet for backend processing
GET| "/h2-console"| Opens the H2 database console, if enabled

The exact request fields and responses depend on the controller and DTO definitions in the source code.

Example bridge request

The bridge ingestion endpoint accepts a JSON packet. An example request is shown below; use the actual field names and values supported by the application.

POST /api/bridge/ingest
Content-Type: application/json

{
  "packetId": "example-packet-id",
  "ttl": 2,
  "createdAt": 1730000000000,
  "ciphertext": "base64-encoded-encrypted-data"
}

The backend processes the request and returns an outcome based on the application's implementation. Possible outcomes in the documented design include "SETTLED", "DUPLICATE_DROPPED" and "INVALID".

---

9. Testing

The project includes tests for encryption and payment processing.

The main test cases are:

- Encryption and decryption: checks whether the original data can be recovered after encryption and decryption.
- Tampered ciphertext: checks whether modified encrypted data is rejected.
- Concurrent duplicate requests: submits the same packet from three threads and checks whether it is settled only once.

Run the tests with:

.\mvnw.cmd test

To run the concurrency test separately on Windows:

.\mvnw.cmd test -Dtest=IdempotencyConcurrencyTest#singlePacketDeliveredByThreeBridgesSettlesExactlyOnce

Test results should be checked in the terminal after execution.

---

10. Current Limitations and Future Improvements

This project is a software demo, so some parts would need to change before it could be used in a real payment system.

Current implementation| Possible future improvement
H2 in-memory database| Use a persistent production database
In-memory duplicate cache| Use shared Redis or equivalent storage
Server-generated RSA keys| Use secure, managed key storage
Virtual devices| Implement and test real Bluetooth communication
Simulated sender| Develop a mobile application
Demo account balances| Integrate with an authorised payment infrastructure
Basic bridge endpoint| Add bridge authentication and authorisation
Console logging| Add structured logs and monitoring

Other improvements could include better error handling, more extensive testing, rate limiting and secure management of payment keys.

---

11. Limitations of Offline Payments

There are some important challenges with the overall concept.

1. Checking available balance

When a phone is offline, the backend cannot immediately confirm the sender's current balance. A payment message alone does not guarantee that the payment will eventually succeed.

2. Double spending

A user might attempt to create multiple payments while offline using the same available balance. A real system needs a reliable way to prevent or manage this risk.

3. Real Bluetooth communication

This demo does not use real Bluetooth connections. Communication between devices is simulated in software. Implementing reliable background communication on Android and iOS would require additional development and testing.

4. Security and privacy

Even when the payment data is encrypted, devices carrying the packet may still observe some communication metadata. A production implementation would need a more complete security and privacy design.

These limitations are part of why the project should be treated as a learning prototype rather than a replacement for an actual UPI service.

---

12. Troubleshooting

Java is not recognised

Install JDK 17 or newer and check that Java is available through your system PATH.

Port 8080 is already in use

Change the "server.port" setting in "application.properties", then restart the application.

The first run takes a long time

Maven may need to download the build tools and project dependencies. Wait for the process to finish and check the terminal for any errors.

The Maven Wrapper command is not recognised in PowerShell

Use the ".\" prefix:

.\mvnw.cmd spring-boot:run

---

Conclusion

Through this project, I explored how encrypted payment messages could be transferred through a simulated offline mesh and processed later by a backend. I also worked with concepts such as hybrid encryption, duplicate-request handling, transaction processing and replay detection.

The current version focuses on demonstrating these concepts in a controlled environment. Actual offline payments, Bluetooth communication and integration with banking infrastructure would require further development.
