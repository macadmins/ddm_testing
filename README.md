# ddm_testing

Follow the directions below for getting started with your MAOS-hosted DDM test server.

> [!CAUTION]
> 🚨🚨🚨 This is an INSECURE setup. Only enroll machines where you won't lament losing data 🚨🚨🚨

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

If you don't like or don't want to use homebrew, the [jq project provides compiled binaries](https://jqlang.github.io/jq/download/). Be sure it put it in a place in your `$PATH`.

Let's setup our environment as well

```bash
export API_KEY="$YOUR_DDM_API_KEY"
export BASE_URL="https://$YOUR_DDM_INSTANCE.macadmins.io"
```

And these additional env vars are needed for `ddmctl`
```bash
export DDM_API_KEY="$YOUR_DDM_API_KEY"
export DDM_URL="https://$YOUR_DDM_INSTANCE.macadmins.io"
export DDM_CLIENT_ID="$YOUR_TEST_CLIENT_ENROLLMENT_UUID"
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
Or using `ddmctl`
```bash
ddmctl declaration get all
```

With this you should now have communication with the MAOS KMFDDM server!

#### Enroll a test vm into your MDM server

The easier way to get started with a vm is using tart, but feel free to use whatever you prefer. See [Tart Quickstart](tart_quickstart.md) for more info.

- Copy the enroll_$MDM.mobileconfig onto your VM and install it.
- Confirm there is stuff happening using `sudo log stream --info --debug --predicate 'subsystem == "com.apple.ManagedClient"'`

Lastly, grab your test VM's MDM UDID as we will need this for all DDM opertaions. The rest of this guide will assume the `ID` env var is set to this ID. If the UUID is incorrect at any point moving forward, it will cause things to look like they are not working. Be 💯 sure it is correct.

On macOS the MDM UDID is the same as the hardware UUID. A couple easy ways to get this UUID on macOS:

- grab it from the the server logs. Check the testing channel on Slack for directions on getting the server logs.
- On the Mac hold the Option key, click the Apple menu in the upper left of the screen, and choose the first menu item "System Information..." It'll be listed toward the bottom the screen that was just opened.

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
If you run it a second time, you'll see that everything has already been applied on the server side:

```bash
./kmfddm/tools/syncdir.py ddm_examples
unchanged declarations: 27
unchanged sets: 2
no changed declarations or sets
```

In this way you can make incremental changes and sync to to the repo. Notably when changes occur syncdir should also notify any devices of the changes.

#### Put the test vm into a set

Sets are a kmfddm abstraction of the DDM protocol. Think of a set as a Munki manifest. For a client to get the declarations applied to it, you must configure the machine to be a part of a set.

Let's do that now.

Confirm the machine does not have anything assigned to it yet:

```bash
./kmfddm/tools/api-enrollment-sets-get.sh $ID
null
```
or via `ddmctl`
```bash
ddmctl device sets
```

Add your machine to the default set and confirm it now shows up:
```bash
./kmfddm/tools/api-enrollment-sets-put.sh $ID default
Response HTTP Code: 204 # This means it was successful. 304 means it is already in the set.

./kmfddm/tools/api-enrollment-sets-get.sh $ID
```
or using `ddmctl`
```bash
ddmctl device add default
$YOUR_DEVICE_UID has been added to default

ddmctl device sets

```

Now that we know those the device has been assigned to the the default, we can check what the server thinks the the devices's declarations *should* be:

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

As we can see, the server thinks few basic declarations *should* be assigned to the test VM. I.e. what the server will try to configure the client to have, not necessarily what the client *is* configured for.

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

To retrive the actual declaration *status* for a client (that is, the status of each declaration as reported back to the client on the DDM status channel), you can do this:

If all the declarations have `current=true` and `valid=valid` attributes, then the client has applied all the current assigned declarations and is good to go!

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

If you've found an interesting test that others might want to share, feel free and encouraged to submit that as PR to the [ddm_examples repo](https://github.com/macadmins/ddm_examples)!
