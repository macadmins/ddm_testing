# ddm_testing

Follow the directions below for getting started with your MAOS-hosted NanoHUB test server.

> [!CAUTION]
> 🚨🚨🚨 This is an UNSECURE setup. Only enroll machines where you won't lament losing data 🚨🚨🚨

### Initial setup

Download and install the nanohubctl binary

 * Download nanohubctl - https://github.com/macadmins/nanohubctl/releases/tag/1.0.6
 * Extract it and install it:
 ```
  unzip nanohubctl-darwin-arm64-1.0.6.zip
  xattr -r -d com.apple.quarantine ./nanohubctl-darwin-arm64-1.0.6/nanohubctl-darwin-arm64
  sudo mv ./nanohubctl-darwin-arm64-1.0.6/nanohubctl-darwin-arm64 /usr/local/bin/nanohubctl
  rm -rf nanohubctl-darwin-arm64-1.0.6
 ```

For the initial setup, clone this repo, cd into it and clone the repo shown below

```bash
git clone https://github.com/macadmins/ddm_testing.git
cd ddm_testing
git clone https://github.com/macadmins/ddm_examples.git
```
Let's setup our environment as well

```bash
# This should be the base url to nanohub, Example: https://nanohub.example.com/
export NANOHUB_URL="https://$YOUR_SERVER_HERE"
export NANOHUB_API_KEY="$YOUR_API_KEY"
```

Put your assigned info where the placeholders are. For instance, if you have instance `ddm42`, your export line for BASE_URL would be:

```bash
export BASE_URL="https://nanohub42.macadmins.io"
```

Let's confirm our keys and BASE_URL are setup correctly by listing all declarations on the server:

```bash
nanohubctl ddm declarations
[] # This makes sense because we've not added any declarations yet
```

With this you should now have communication with the MAOS NanoHUB server!

#### Enroll a test vm into your MDM server

The easier way to get started with a vm is using tart, but feel free to use whatever you prefer. See [Tart Quickstart](tart_quickstart.md) for more info.

- Copy the provided enroll_$NANOHUB.mobileconfig onto your VM and install it.
- Confirm there is stuff happening using `sudo log stream --info --debug --predicate 'subsystem == "com.apple.ManagedClient"'`

Lastly, grab your test VM's MDM UDID as we will need this for all DDM opertaions. The rest of this guide will assume the `NANOHUB_CLIENT_ID` env var is set to this ID. If the UUID is incorrect at any point moving forward, it will cause things to look like they are not working. Be 💯 sure it is correct.

For NanoHUB, the MDM UDID is the same as the hardware UUID. A couple easy ways to get this UUID on macOS:

- grab it from the the server logs. Check the testing channel on Slack for directions on getting the server logs.
- On the Mac hold the Option key, click the Apple menu in the upper left of the screen, and choose the first menu item "System Information..." It'll be listed toward the bottom the screen that was just opened.

```bash
export NANOHUB_CLIENT_ID=$YOUR_VM_UUID_HERE

# Confirm the ID matches the server logs for the enrollment
echo $NANOHUB_CLIENT_ID
```

#### Using ddm_examples
Within your working directory, run the following command to apply all examples from ddm_examples:

```bash
nanohubctl ddm sync ./ddm_examples
```

This script will apply everything in the ddm_examples folder. Note that is uses the environment variables from above (which can also be supplied as args).

If you run it a second time, you'll see that everything has already been applied on the server side:

```bash
nanohubctl ddm sync ./ddm_examples
```

Now you can make incremental changes and sync them to the repo. Notably when changes occur, nanohub should also notify any devices which have the declarations and trigger them to apply the changes.

#### Put the test vm into a set

Sets are a kmfddm abstraction of the DDM protocol. Think of a set as a Munki manifest. For a client to get the declarations applied to it, you must configure the machine to be a part of a set.

Let's do that now.

Confirm the machine does not have anything assigned to it yet:

```bash
nanohubctl ddm device sets
null
```
Add your machine to the default set and confirm it now shows up:
```bash
nanohubctl ddm device add default
CA75397A-A557-59C8-B48C-59394256E22C has been added to default

nanohubctl ddm device sets
[
        "default"
]
```
Now that the machine has been assigned to an enrollment set, we can look at the debug server logs and see all the raw DDM status items coming back as well as current DDM config status. Check out https://dozzle.macadmins.io for the raw logs (ask [Nate](https://github.com/natewalck) for access)

Now that we know how to apply a machine to a set and confirm the set has been applied to the client, go crazy! Make new sets in ddm_examples by copying sets/set.template.txt and naming it set.YOURNAME.txt.

Create new declarations in ddm_examples and run the `nanohubctl ddm sync` command like above.

If you've found an interesting test that others might want to share, feel free and encouraged to submit that as PR to the [ddm_examples repo](https://github.com/macadmins/ddm_examples)!
