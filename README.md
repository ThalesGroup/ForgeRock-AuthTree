[Thales Logo](/images/Thales_Gemalto_logo.jpg)

# Thales Digital Banking IdCloud Nodes
Document version: 2.0 (November 2019)

The integration below targets our IdCloud solution and all its services. IdCloud is the short name for Thales Digital Banking Identity Cloud. Traditionally these security services were implemented and managed by banks in-house, but over the last few years a growing number of financial institutions have shown interest in moving some of their services and IT infrastructure to the Cloud, in search of greater agility when deploying their customer-oriented services, scalability, and cost efficiency.

## About Thales Digital Banking

The Thales Digital Banking offer enables banks to secure their Digital Banking use cases: new account opening, login, enrolment to additional services and transaction signing.

Our portfolio of solutions includes:
* **IdCloud** – our cloud-based offer orchestrating an extensive set of services:
  * **KYC** (Know Your Customer): Document Verification, Facial Recognition, AML checks, …
  * **Authentication**: Login, Transaction Signature, Secure Messaging, …
  * **Fraud Prevention**: Geolocation, device profiling, device reputation, cyber threat detection, ...
* **Mobile Authentication** solutions to secure all the digital banking channels: mobile, PC, ATM, kiosk, in-branch…
* **Authentication and Signing devices and enablers**: unconnected, USB/BLE-connectable and QR code readers and tokens

## How to install IdCloud nodes

These solutions are now integrated into ForgeRock Identity Platform using Intelligent Authentication trees from ForgeRock Access Management.

IdCloud is composed of a cloud backend and several client SDKs. For most use cases, both are used. This document will only discuss the backend integration, using IdCloud nodes in Access Management. In an actual project, the client SDKs also need to be taken into account and integrated into the solution. The client SDKs are mostly mobile SDKs (Android and iOS). Some browser SDKs are also available as JavaScript libraries. Please contact our representative for more details.

This guide is targeting Access Management (AM) 6.5 and above. Please view ForgeRock documentation on how to deploy and install it.

Access Management must be able to connect to IdCloud solution.


** Step 1 - Requesting an IdCloud account **
To start testing this ForgeRock integration, you will need an account in IdCloud. Please contact a Thales Digital Banking sales representative to obtain it. You will be provided with the nodes, an IdCloud account and a sample application to help you quickly build a working proof-of-concept (POC). You will also have access to a developer portal with more information on our solution.

** Step 2 - Install nodes **
1. Copy the jar file with the IdCloud nodes to your Access Management installation Example: <webserver>/openam/WEB-INF/lib/.
1. Restart Access Management web server.

** Step 3 - Add the IdCloud Service **
1. In Access Management console, access the Service panel.
1. Click Add Service and select "IdCloud Service" from the drop down list.
1. Configure the new service using the information found below in the "IdCloud Service" section.

** Step 4 - Create trees in Access Management **
The new nodes are available in the Intelligent Authentication UI. They are all prefixed by "IdCloud" and can be easily found in the left panel. You can now start including IdCloud nodes in new or existing trees.

You can use the examples shown below for several typical use of the nodes.

## IdCloud nodes

The IdCloud can be used to either improve an existing use case or create a brand new one. Here are some examples of what can be done:

* Integrate strong customer authentication to any banking use case, like login, transaction signature (dynamic linking), add beneficiary, approve batch transactions, and so on.
* Create an on boarding process using our KYC and Fraud connectors and automatically enrol the user's mobile device.
* Add fraud prevention to an existing use case like new account creation, loan approval, and so on.

### Main nodes

* **IdCloud KYC** - This node starts the KYC process in IdCloud. It sends all the data to IdCloud (document scans, "selfies", parameters) and starts the KYC process (ID verification, data extraction, AML checks, and so on). As this process can take some time, this nodes starts an asynchronous process in IdCloud. You need to poll the status to receive the result.
* **IdCloud KYC Status** - This node checks the status of the KYC process that was created by the "IdCloud KYC" node. While the operation is still in progress, this node returns "running". Once it has received the result, it will store the result in a shared variable and return "finished".
* **IdCloud KYC to IDM mapping** - This node converts the personal identification data (extracted from the ID document) into variables that can be used to create an account in the ForgeRock identity platform. It can only be used after IdCloud KYC Status has successfully received a result from IdCloud.
* **IdCloud Enroll Mobile** - This node initiates the mobile enrolment process in IdCloud. It is finished by the mobile application. This node returns 2 values: a registration code and the PIN of the mobile token. The registration code must be provided to the mobile application to complete the enrolment on the mobile. The PIN is used by the user to generate an OTP, change PIN or enable a biometric factor. For security reason, the registration code and the PIN must be provided via separate channels to the user.
* **IdCloud OOB Auth** - This node starts an out of band authentication on the mobile of the end user. IdCloud sends a notification to the end user's phone and he or she will need to approve the authentication request by verifying his or her PIN or using a biometric factor like a fingerprint, FaceID, TouchID, and so on. As this process can take some time, this nodes starts an asynchronous process in IdCloud. You need to poll the status to receive the result of the authentication.
* **IdCloud OOB Sign** - This node starts an out of band signature on the mobile of the end user. IdCloud sends a notification to the end user's phone. The user is shown the transaction details and he or she will need to approve the signature operation by verifying his or her PIN or using a biometric factor like a fingerprint, FaceID, TouchID, and so on. As this process can take some time,this nodes starts an asynchronous process in IdCloud. You need to poll the status to receive the result of the signature.
* **IdCloud OOB Status** - This node checks the status of an out of band operation, either an authentication or a signature. While the operation is still in progress, the node returns "waiting". Once it has received the result, it returns one of the two possible outcomes: "success" or "denied".

Notes: IdCloud supports several additional features that are not currently covered by these nodes, like other authentication and signature flows. Please contact a Thales Digital Banking sales representative for more information.

### Helper nodes

Most of the nodes below are used to set shared variables that are used by the main IdCloud nodes. They can be replaced by any nodes that set the same variables with the equivalent content type. The other nodes just display the result in a manner that is useful for demonstration purposes.

- **IdCloud Request Sign Data** - This node displays several input fields that need to be entered by the end user. They will be used during the signature process. In the case of PSD2 compliance, this can be used to link the OTP to the amount and beneficiary. This node can be replaced by any node that sets the transaction data as shared variables.
* **IdCloud File Upload** - This node is used to upload a "selfie", picture or scan of a document to the Access Manager before our KYC node sends it to IdCloud. It can be replaced by any node that can store an image in a shared variable.
* **IdCloud Display KYC result** - This node is for demonstration purposes only. It displays the result of the KYC verification such as firstname, lastname, and so on, scanned from the ID document. In a real project, personal data scanned from the ID document should be transferred to the appropriate services.
* **IdCloud Display Enroll Result** - This node displays the mobile enrolment result as a QR code that can be used by our sample mobile application. This method is convenient for demonstration or POC. In a real project, more robust solutions should be implemented. For security reasons, the registration code and PIN must be provided to the end user using two different channels.

### IdCloud Service

In order for the nodes to connect to your IdCloud tenant, the **IdCloud Service** must be added and configured in the Access Management Service section. This service requires three values:
* **IdCloud endpoint** - The URL where all IdCloud requests must be sent.
  * Example: https://hostname/scs/v1/scenarios
* **JWT** - The security token that proves our nodes are allowed to access IdCloud.
  * Example: eyJ0eXAiOiJKV1Qi...
* **Proxy** - If you are running in a environment that must use a proxy to connect to the outside, use this field to enter the URL of the proxy.
  * Example: https://myproxy.company.com:8443/


To be allowed to connect to IdCloud, either for a POC, pilot or production, please contact a Thales Digital Banking sales representative.

## Examples

Below are several examples of how the nodes can be used to build simple or complex trees.

** Mobile enrolment **
The user is able to enrol his or her mobile after authenticating using a existing authentication method, like username password.

![Mobile enrolment](/images/idcloudenroll.png)

** Out of band authentication **
When the user tries to access the bank's web portal, he or she enters the username. A notification is sent to his or her mobile device to confirm the login request by entering his or her PIN or using any biometric factor like FaceID, TouchID, and so on. This generates an OTP that is verified by IdCloud. If the complete process is successful, "success" is returned.

![Out of band authentication](/images/idcloudauth.png)

** Out of band transaction signature **
The user requests a banking transfer that requires a signature. A notification is sent to his or her mobile device to confirm the transaction details. The user signs the transaction by entering his or her PIN or using any biometric factor like FaceID, TouchID, and so on. This generates an OTP linked to the transaction details (PSD2 dynamic linking) that is verified by IdCloud. If the complete process is successful, "success" is returned.

![Out of band transaction signature](/images/idcloudsign.png)

** On boarding a new user **
The user is asked to enter a scan of an ID document and take a selfie. IdCloud verifies the validity of the ID document, verifies the user is the owner of the ID document, and performs any other additional check like AML checks. If successful, a new account is created in the ForgeRock identity platform and the mobile is enrolled so he or she can authenticate or sign transactions in the future.

![On boarding a new user](/images/idcloudkycenroll.png)

# Disclaimer
Thales hereby disclaims all warranties and conditions with regard to the information contained herein, including all implied warranties of merchantability, fitness for a particular purpose, title and non-infringement. In no event shall Thales be liable, whether in contract, tort or otherwise, for any indirect, special or consequential damages or any damages whatsoever including but not limited to damages resulting from loss of use, data, profits, revenues, or customers, arising out of or in connection with the use or performance of information contained in this document.
