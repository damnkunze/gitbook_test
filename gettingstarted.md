# GettingStarted

## Getting Started

* [Installation](broken-reference)
* [Setup](broken-reference)
* [A minimal application](broken-reference)

### Installation

Optionally create environment to install to:

`$ python -m venv venv`

`$ . venv/bin/activate`

Install the requirements and ubirch library using pip:

`$ pip install -r requirements.txt`

`$ pip install ubirch-protocol`

> The required version of the `ubirch-protocol` package to run the provided scripts is `3.1.0`. Currently this version can only be installed through a local install.

If you want to install from another source than pip, follow along here.

### Setup

Before anything, you will need to do/get a couple of things:

* Open up the [Ubirch Console](https://console.demo.ubirch.com) (`'demo'` stage)
  * For guidance, check out the [uBirch console documentation](https://developer.ubirch.com/console.html)
  * First register to get an account
  * Then Create a "Thing":
    * Using a UUID generated with a [UUID-Generator](https://www.uuidgenerator.net/)
  * You will be using the shown UUID (ID) and the generated Auth-Token (password) from now on
* Come up or [generate](https://www.random.org/passwords/) a password for the KeyStore, which is where public and private Keys will be stored locally

#### Now you should have the following at hand:

Our [Ubirch API](http://developer.ubirch.com/function\_documentation/ubirch-protocol-python/) authentication with an uuid and a password:

```python
from uuid import UUID

uuid = UUID(hex = "f5ded8a3-d462-41c4-a8dc-af3fd072a217")
auth            = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

And credentials for a [KeyStore](http://developer.ubirch.com/function\_documentation/ubirch-protocol-python/) to store your public and private key:

```python
keystore_name     = "devices.jks"
keystore_password = "XXXXXXXXXXX"
```

### A minimal application

The smallest uBirch application looks something like this.

_The code can be found in `examples/GettingStarted.py` as well._

_Run it from your command prompt using `$ python examples/GettingStarted.py` or copy-paste the codeblocks._

Let's say we have got some environment-sensor data like:

```python
import time

data = {
    "timestamp": int(time.time()),
    "temperature": 11.2,
    "humidity": 35.8,
    "status": "OK"
}
```

To send a hash of the data to the Ubirch backend run these few lines inside of `examples/`:

```python
import ubirch
from UbirchWrapper import UbirchWrapper

# (1) Initialize an UbirchWrapper instance and pass the credentials for a `KeyStore`
client = UbirchWrapper(uuid, auth, keystore_name, keystore_password)

# (2) Check if the public key is registered at the Ubirch key service and register it if necessary
client.checkRegisterPubkey()

# (3) Create a chained Ubirch protocol packet (UPP) that contains a hash of the data 
currentUPP = client.createUPP(data)

# (4) Send the UPP to the Ubirch backend using the API and handle the response
response = client.api.send(uuid, currentUPP)
client.handleMessageResponse(response)

# (5) Verify that the response came from the backend
client.verifyResponseSender(response)

# (6) Unpack the received UPP to get its previous signature 
previousSignatureInUPP = client.extractPreviousSignature(response)

# (7) Make sure it is the same as the UPP signature sent
client.assertSignatureCorrect(previousSignatureInUPP)

# (9) Persist signature: Save last signatures to a `.sig` file
client.protocol.persist(uuid)

print("Successfully sent the UPP and verified the response!")
```

_This example uses the example UbirchWrapper that helps to implement general repetitive tasks._

**Next: Take a look at the Step-by-step-example.**
