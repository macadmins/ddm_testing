# ddm_testing
Follow the directions below for getting started with your MAOS-hosted ddm test server

> 🚨🚨🚨 This is NOT a secure setup. Only use machines where you won't lament losing data 🚨🚨🚨

### Initial setup
For the initial setup, create a working directory in your favorite location and clone the repos shown below

```bash
git clone https://github.com/jessepeterson/kmfddm.git
git clone https://github.com/macadmins/ddm_examples.git
```

Also install jq, it'll make everything easier to read:
```
brew install jq
```

Let's setup our environment as well
```bash
export API_KEY="$YOUR_DDM_API_KEY"
export BASE_URL="https://$YOUR_DDM_INSTANCE.macadmins.io"
```
Put your assigned info where the placeholders are. For instance, if you have instance `ddm42`, your export line for BASE_URL would be:
```bash
export BASE_URL="https://ddm42.macadmins.io"
```
Let's confirm our keys and BASE_URL are setup correctly by listing all declarations on the server:
```bash
./kmfddm/tools/api-declarations-get.sh
[] # This makes sense because we've not added any declarations yet
```
You are all set up!

#### Enroll a test vm into your MDM server
The easier way to get started with a vm is using tart, but feel free to use whatever you prefer. See [Tart Quickstart](tart_quickstart.md) for more info.

- Copy the enroll_$MDM.mobileconfig onto your VM and install it.
- Confirm there is stuff happening using `sudo log stream --info --debug --predicate 'subsystem == "com.apple.ManagedClient"'`

Lastly, grab your test VM's hardware UUID as we will need this for all ddm opertaions. The rest of this guide will assume the `ID` env var is set to your test vm's UUID. If the UUID is incorrect at any point moving forward, it will cause things to look like they are not working. Be 💯 sure it is correct.

The easiest way to get the UUID is to grab it from the the server logs. Check the testing channel on Slack for directions on getting the server logs.

```bash
export ID=$YOUR_VM_UUID_HERE

# Confirm the ID matches the server logs for the enrollment
echo $ID
```

#### Using ddm_examples
For a deep dive into the various functionality in kmfddm, check out the [kmfddm quickstart](https://github.com/jessepeterson/kmfddm/blob/main/docs/quickstart.md#basic-setup).

If you do not have python3 installed and setup already, do the following:
- Install [MacAdmins Python](https://github.com/macadmins/python/releases).
- Run `sudo ln -s /usr/local/bin/managed_python3 /usr/local/bin/python3`

Within your working directory, run the following command to apply all examples from ddm_examples:

```bash
./kmfddm/tools/syncdir.py ddm_examples
```
This script will apply everything in the ddm_examples folder.

If you run it a second time, you'll see that everything has already been applied on the server side:

```bash
./kmfddm/tools/syncdir.py ddm_examples
unchanged declarations: 27
unchanged sets: 2
no changed declarations or sets
```

#### Put the test vm into a set
Sets are a kmfddm abstraction of the ddm protocol. Think of a set as a Munki manifest. For a client to get the declarations applied to it, you must configure the machine with a set.

Let's do that right now.

Confirm the machine does not have anything assigned to it yet:
```bash
./kmfddm/tools/api-enrollment-sets-get.sh $ID
null
```
Add your machine to the default set and confirm it now shows up:
```bash
./kmfddm/tools/api-enrollment-sets-put.sh $ID default
Response HTTP Code: 204 # This means it was successful. 304 means it is already set.

./kmfddm/tools/api-enrollment-sets-get.sh $ID
```

Now that we know those declarations in the default set hass been applied, we can check the status of the declarations:

```bash
./kmfddm/tools/ddm-declaration-items.sh $ID | jq .
{
  "Declarations": {
    "Activations": [
      {
        "Identifier": "io.macadmins.activation.test",
        "ServerToken": "3f23b3e0259807a0"
      },
      {
        "Identifier": "io.macadmins.activation.subs",
        "ServerToken": "992c1937368d9d13"
      }
    ],
    "Assets": [],
    "Configurations": [
      {
        "Identifier": "io.macadmins.config.test",
        "ServerToken": "d7985a151f76f968"
      },
      {
        "Identifier": "io.macadmins.config.subs",
        "ServerToken": "d5c3ccfed4141094"
      }
    ],
    "Management": [
      {
        "Identifier": "io.macadmins.management.props",
        "ServerToken": "06f04438a1855837"
      }
    ]
  },
  "DeclarationsToken": "0bcddb213411ff39"
}
```
As we can see, a few basic declarations have been assigned to the test vm.

If you want to see what the server thinks a declaration contains for the given machine, you can run the following command to get the config details:

```bash
./kmfddm/tools/ddm-declaration.sh $ID configuration/io.macadmins.config.test | jq .

{
  "Identifier": "io.macadmins.config.test",
  "Payload": {
    "Echo": "KMFDDM"
  },
  "ServerToken": "d7985a151f76f968",
  "Type": "com.apple.configuration.management.test"
}
```
Note: You must pass in configuration/$declaration_identifier to this command to see the contents.

We can also see the status for a all declarations on a given client:

```bash
./kmfddm/tools/api-status-declaration-get.sh $ID | jq .
{
  "C2D94C2D-4EE1-5A3E-A53E-FE50B202055D": [
    {
      "identifier": "io.macadmins.config.test",
      "active": true,
      "valid": "valid",
      "server-token": "bd52fda3126b205f",
      "ManifestType": "configurations",
      "current": true,
      "status_received": "2024-06-07T20:51:00.056637877Z",
      "reasons": [
        {
          "code": "KMFDDM"
        }
      ]
    },
    {
      "identifier": "io.macadmins.config.subs",
      "active": true,
      "valid": "valid",
      "server-token": "ec3806409783c225",
      "ManifestType": "configurations",
      "current": true,
      "status_received": "2024-06-07T20:51:00.056637877Z"
    },
    {
      "identifier": "io.macadmins.management.props",
      "active": false,
      "valid": "valid",
      "server-token": "144c3da1ca3d400c",
      "ManifestType": "management",
      "current": true,
      "status_received": "2024-06-07T20:51:00.056637877Z"
    },
    {
      "identifier": "io.macadmins.activation.test",
      "active": true,
      "valid": "valid",
      "server-token": "7391c82ea697f3a7",
      "ManifestType": "activations",
      "current": true,
      "status_received": "2024-06-07T20:51:00.056637877Z"
    },
    {
      "identifier": "io.macadmins.activation.subs",
      "active": true,
      "valid": "valid",
      "server-token": "86ce345b44a5bce4",
      "ManifestType": "activations",
      "current": true,
      "status_received": "2024-06-07T20:51:00.056637877Z"
    }
  ]
}

```

Now that we know how to apply a machine to a set and confirm the set has been applied to the client, go crazy! Make new sets in ddm_examples by copying sets/set.template.txt and naming it set.YOURNAME.txt.

Create new declarations in ddm_examples and run the syncdir.py command like above.

