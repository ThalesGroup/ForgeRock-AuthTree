![Gemalto Logo](/images/logo-gemalto-340x120.gif)

# Gemalto Authentication Solution
Document version: 1.0 (July 2018)

Gemalto Digital Banking Mobile Suite delivers strong multi-factor authentication – including biometrics – with best-in-class security, while fulfilling banking requirements. It ensures appropriate security features, while also allowing banks to comply with regulations and to offer simple, innovative experiences that increase the digital touchpoints in online channels. Gemalto Digital Banking Mobile Suite is developed and maintained by Gemalto with banking-grade security vetted by internal and external audits.

Backed up by an large R&D team of engineers (developers, architects, testers, and security officers) who are highly skilled in mobile security, cryptography and biometric authentication, the Gemalto Digital Banking Mobile Suite benefits from a strong product development plan. It is the guarantee of a product offering which not only remains always up to date with the latest mobile evolutions and threats, but also strives to incorporate the latest innovations to deliver banks and their users with the strongest security and a smooth user experience.

These solutions are now integrated into ForgeRock Identity Platform using the **Authentication Tree** from ForgeRock **Access Management**.


![ForgeRock Gemalto integration](/images/gto_overview.png)


## Pre-requisites

### Gemalto solution
For this integration, the Gemalto federation interface needs to running. It is the component that exposes Gemalto solutions via SAMLv2, OAuth 2, and OpenID Connect.
A new OpenID Connect client needs to be created. The following inputs will be needed for this guide:
* **OpenID Discovery URL** - example: ```"https://gemalto.mybank.com/auth/realms/dbanking/.well-known/openid-configuration"```
* **Client ID** - entered when creating the client in Gemalto systems - example: ```"forgerock"``` 
* **Client Secret** - a randomly generated secret used during the OpenID Connect exchange: example: ```"2ggse304-wg34-9nwx-193j-3jud85jdwpzm"```

### ForgeRock Access Management
This guide is targeting Access Management (AM) version 6.0+. It can be deployed using this set of [instructions](https://backstage.forgerock.com/docs/am/6/quick-start-guide/).

For ForgeRock 5.5+, please use our other guide that uses [modules and chains technology](https://backstage.forgerock.com/marketplace/api/catalog/entries/AWSAkhOFvlTdzmfcNqZl).

Access Management must be able to connect to Gemalto's solution.

## Create a new Authentication Tree

### Authentication Tree Diagram
You can create many types of authentication tree to match your specific deployment. Below are 2 variants than can typically be used for simple setup. The integration of the Gemalto authentication solutions is done via the OAuth 2.0 node.

#### Quick Demo Setup
For a **quick demo setup**, build this flow. It uses **"Provision Dynamic Account"**, that creates temporary accounts for logged in users. Those accounts are deleted once the session expires.


![Tree with temporary accounts](/images/tree-temporary.png)

#### Persistent Accounts
For a more realistic deployment, we recommend to use this flow. It uses **"Provision IDM Account"** to **permanently persist user accounts** in an IDM. An IDM needs to be configured in Access Management.


![Tree with permanent accounts](/images/tree-permanent.png)


### Configure OAuth 2.0 node
In the authentication tree, select the OAuth 2.0 node. A form tab will open on the right. The table below shows a typcial setup.

Properties | Values
---------- | -----------
Node Name | ```Gemalto Auth``` (Any name can be chosen)
Client ID | ```<See pre-requisites>```
Client Secret | ```<See pre-requisites>```
Authentication Endpoint URL | Copy value from the entry **authorization_endpoint** in the **OpenID Discovery URL** web page (see the pre-requisites)
Access Token Endpoint URL | Copy value from the entry **token_endpoint** in the **OpenID Discovery URL** web page (see the pre-requisites)
User Profile Service URL | Copy value from the entry **userinfo_endpoint** in the **OpenID Discovery URL** web page (see the pre-requisites)
OAuth Scope	| ```openid profile email```
Scope Delimiter | ``` ```  (enter the space character)
Redirect URL | Access Management redirect URL. Example: https://openam.mybank.com/openam/XUI/
Social Provider | ```Gemalto``` (Any name can be chosen)
Auth ID Key | ```preferred_username```
Use Basic Auth | ```enabled```
Account Provider | ```org.forgerock.openam.authentication.modules.common.mapping.DefaultAccountProvider```
Account Mapper | ```org.forgerock.openam.authentication.modules.common.mapping.JsonAttributeMapper```
Attribute Mapper | ```org.forgerock.openam.authentication.modules.common.mapping.JsonAttributeMapper```
Account Mapper Configuration | ```preferred_username=uid```
Attribute Mapper Configuration | Enter the following entries:
Attribute Mapper Configuration | ```given_name=givenName```
Attribute Mapper Configuration | ```preferred_username=uid```
Attribute Mapper Configuration | ```family_name=sn```
Attribute Mapper Configuration | ```name=cn```
Attribute Mapper Configuration | ```email=mail```
Save Attributes in the Session | ```enabled```
OAuth 2.0 Mix-Up Mitigation Enabled	| ```disabled```
Token Issuer | Copy value from the entry **issuer** in the **OpenID Discovery URL** web page (see the pre-requisites)


## Testing your configuration
Access Management provides a quick way to test your setup at this stage.
1. Trigger the chain using a browser by using its service name
Example: https://openam.mybank.com/openam/XUI/#login&service=GemaltoAuth

2. This should automatically redirect you to the Gemalto login page via an OpenID Connect redirection


![Demo login page](/images/demo-loginpage.png)


3. Login to the Gemalto authentication solution using one of the configured authentication method.

You should be redirected to the Access Management user portal


![ForgeRock User Portal](/images/demo-userpage.png)


4. Logout of the User Portal using the top right menu.


## Disclaimer
Gemalto hereby disclaims all warranties and conditions with regard to the information contained herein, including all implied warranties of merchantability, fitness for a particular purpose, title and non-infringement. In no event shall Gemalto be liable, whether in contract, tort or otherwise, for any indirect, special or consequential damages or any damages whatsoever including but not limited to damages resulting from loss of use, data, profits, revenues, or customers, arising out of or in connection with the use or performance of information contained in this document.
