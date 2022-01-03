# Thingsboard TLS Demo
This is an example demo of both tls and non-tls nodes communicating to tb-server (v3.3.2) simultaneously.

## Server Side Settings
A community-edition of tbv3.3.2 is installed in a local machine (IP: 192.168.1.110). Port 8099 is configured for unsecured mqtt. Port 8883 is configured for secured mqtt over tls.
Server side self-signed certificate and key pair was generated using openssl. Server public key server.pem and server private key key.pem is pointed correctly in the thingsboard.conf file.

Refer https://thingsboard.io/docs/pe/user-guide/mqtt-over-ssl/ for details about generating server side keys using openssl and configuring the tb-server for SSL

## Client Side Settings (TLS node)
Client side certificate and key pair is generated using openssl
Create a device (named as dev-02-tls) in the above said local tb-server instance. Application port for tb-server is 8080
Select the device-credential-type as x.509 certificate.
Copy the contents of the client side public key in cert.pem file and paste it as device credential in tb-server.

Now open node-red and simulate a node with secured mqtt connection.
No need to provide mqtt username and password.
In the TLS Configuration, provide the path to client-public-key (cert.pem), client-private-key (key.pem) and server-public-key (server.pem). These keys are available in the "../keys" folder in this project
Uncheck the option "verify server certificate" since this server-public-key is self-signed

Start node-red.
The simulated device should connect to the server via mqtt over tls.
The handshake and the tls communication can be observed in the wireshark-log-files available in the "../wireshark-logs" folder.

Refer https://thingsboard.io/docs/pe/user-guide/certificates/ for generating client/device side certificate using openssl and configuring it in tb-server.

## Client Side Settings (non-TLS node)
Another device (named as dev-01-tls) is created in the above said local tb-server instance.
For this device, accessToken is selected as device-credential-type
A random accessToken is generated for the device

Now open node-red and simulate a node with normal mqtt connection.
Provide mqtt username as accessToken of the device created in te-server. Password is left blank.

Start node-red.
The simulated device should connect to the server via mqtt.
The messages sent as telemetry shall be seen in the wireshark-log-files available in the "../wireshark-logs" folder.

## Observations
1. The TLS messages have a frame size of 177 bytes for a simple telemetry example of
{"ts":1641204464295,"values":{"volt":249,"current":10}}
<img src="https://github.com/schnell-rnd/tb-tls-demo/blob/ea9a53e1f4fa3fd7524697595f0cc71c29ac5d71/screenshots/screenshot-tls-communication.png" alt="screenshot-TLS" width="256">
2. The non-TLS message have a frame size of 148 bytes for a simple telemetry example of
{"ts":1641204443676,"values":{"volt":242,"current":14}}
<img src="https://github.com/schnell-rnd/tb-tls-demo/blob/ea9a53e1f4fa3fd7524697595f0cc71c29ac5d71/screenshots/screenshot-noTLS-communication.png" alt="screenshot-noTLS" width="256">
3. It can be seen that the telemetry messages from the TLS node are encrypted and send to port 8883
4. Also it can be seen that the telemetry messages from the non-TLS node are not encrypted and the data
is visible in the log file as clear text. The data is send to the sever port 8099
Screenshots with the above details are availble in the "../screenshots" folder.
5. All the three files cert.pem, key.pem and server.pem files are needed by node-red mqtt node to establish an mqtt connection over tls.
6. The option "verify server certificate" has to be unchecked if the server-public-key is self-signed or self-generated using openssl. If a
  a signed certificate is available, then this option can be checked.
7. While copying the contents of the cert.pem file to the device-credential, leave out the BEGIN-CERTIFICATE and END-CERTIFICATE lines.
  Copy only the contents in between these lines
