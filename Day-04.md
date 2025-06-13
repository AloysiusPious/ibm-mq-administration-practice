# 📘 Day 4 - IBM MQ Practice

## 🎯 Objective
Enhance the security of your IBM MQ environment by configuring Transport Layer Security (TLS) to encrypt communication between two queue managers (QM1 and QM2).

## 🛠️ Prerequisites
IBM MQ 9.4.0.10 installed on CentOS

User with mqm group privileges

Environment variables configured using setmqenv

Queue managers QM1 and QM2 created and running

OpenSSL installed for certificate generation

## ✅ Steps
1. Create Key Databases for Each Queue Manager
Create a key database (.kdb) and stash file for each queue manager:
```bash
# For QM1
runmqakm -keydb -create -db QM1.kdb -pw password -type cms -stash
```
```bash
# For QM2
runmqakm -keydb -create -db QM2.kdb -pw password -type cms -stash
```
Place the .kdb files in the appropriate directories, typically under /var/mqm/qmgrs/QM1/ssl/ and /var/mqm/qmgrs/QM2/ssl/.

2. Generate Self-Signed Certificates
Generate a self-signed certificate for each queue manager:

```bash
# For QM1
runmqakm -cert -create -db QM1.kdb -pw password -label ibmwebspheremqQM1 -dn "CN=QM1, OU=MQ, O=YourOrg, C=IN"
```
```bash
# For QM2
runmqakm -cert -create -db QM2.kdb -pw password -label ibmwebspheremqQM2 -dn "CN=QM2, OU=MQ, O=YourOrg, C=IN"
```
3. Extract and Exchange Public Certificates
Extract the public certificate from each key database:

```bash
# For QM1
runmqakm -cert -extract -db QM1.kdb -pw password -label ibmwebspheremqQM1 -target QM1.arm -format ascii
```bash
# For QM2
runmqakm -cert -extract -db QM2.kdb -pw password -label ibmwebspheremqQM2 -target QM2.arm -format ascii
```
Then, import the other's certificate into each key database:

```bash
# On QM1, import QM2's certificate
runmqakm -cert -add -db QM1.kdb -pw password -label ibmwebspheremqQM2 -file QM2.arm -format ascii
```
```bash
# On QM2, import QM1's certificate
runmqakm -cert -add -db QM2.kdb -pw password -label ibmwebspheremqQM1 -file QM1.arm -format ascii
```
4. Configure SSL/TLS on Queue Managers
Set the SSL key repository location for each queue manager:

```bash
# For QM1
runmqsc QM1
ALTER QMGR SSLKEYR('/var/mqm/qmgrs/QM1/ssl/QM1')
END
```
```bash
# For QM2
runmqsc QM2
ALTER QMGR SSLKEYR('/var/mqm/qmgrs/QM2/ssl/QM2')
END
```
Note: Do not include the .kdb extension in the SSLKEYR path.

5. Define Secure Channels
Define sender and receiver channels with SSL configuration:

```bash
# On QM1
runmqsc QM1
DEFINE CHANNEL('QM1.TO.QM2') CHLTYPE(SDR) TRPTYPE(TCP) CONNAME('localhost(1415)') XMITQ('QM2') SSLCIPH('TLS_RSA_WITH_AES_128_CBC_SHA')
END
```
```bash
# On QM2
runmqsc QM2
DEFINE CHANNEL('QM1.TO.QM2') CHLTYPE(RCVR) TRPTYPE(TCP) SSLCIPH('TLS_RSA_WITH_AES_128_CBC_SHA')
END
```
Ensure that the SSLCIPH value matches on both ends.

6. Start the Listener on QM2
Start the listener to accept incoming connections:

```bash
runmqsc QM2
START LISTENER('QM2.LISTENER')
END
```
7. Start the Sender Channel on QM1
Initiate the sender channel to establish a secure connection:
```bash
runmqsc QM1
START CHANNEL('QM1.TO.QM2')
END
```
8. Verify the Secure Connection
Check the status of the channels to ensure they are running:
```bash
# On QM1
runmqsc QM1
DISPLAY CHSTATUS('QM1.TO.QM2')
END
```
```bash
# On QM2
runmqsc QM2
DISPLAY CHSTATUS('QM1.TO.QM2')
END
```
The STATUS should indicate RUNNING, confirming a successful TLS-secured connection.

📘 Explanation
Key Database (.kdb): Stores the queue manager's private keys and certificates.

Self-Signed Certificate: Used for encrypting communication; in production, certificates should be signed by a trusted Certificate Authority (CA).

SSLKEYR: Specifies the location of the key repository for the queue manager.

SSLCIPH: Defines the cipher specification for SSL/TLS encryption; must match on both ends of the channel. 
[https://www.ibm.com/support/pages/how-auto-start-and-stop-listener-mq-60-and-later/stub?utm_source=chatgpt.com]

[https://community.broadcom.com/discussion/lisaibmmqrecvmqmdreplytoqueuemanager-this-property-set-to-listener-queue-manager-instead-of-respond-queue-manager?utm_source=chatgpt.com]

[https://www.experts-exchange.com/questions/24396700/stop-start-mq-environment.html?utm_source=chatgpt.com]

[https://www.ibm.com/docs/en/ibm-mq/9.2.x?topic=types-listeners&utm_source=chatgpt.com]


## 📚 Additional Resources
(IBM MQ Security Overview) [https://www.ibm.com/docs/en/ibm-mq/9.2?topic=overview-security]

(Configuring SSL/TLS on IBM MQ Channels) [https://www.ibm.com/docs/en/ibm-mq/9.2?topic=channels-configuring-ssl-tls]

(IBM MQ Cipher Specifications) [https://www.ibm.com/docs/en/ibm-mq/9.2?topic=reference-cipher-specifications]
