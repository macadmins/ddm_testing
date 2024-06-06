# ddm_testing
Follow the directions below for getting started with your MAOS-hosted ddm test server

### Initial setup
For the initial setup, create a working directory in your favorite location and clone the repos shown below

```
git clone https://github.com/jessepeterson/kmfddm.git
git clone https://github.com/macadmins/ddm_examples.git
```

Let's setup our environment as well
```
export API_KEY="$YOUR_DDM_API_KEY"
export BASE_URL="https://$YOURINSTANCE.macadmins.io"
```
Put your assigned info where the placeholders are. For instance, if you have instance `mdm42`, your export line for BASE_URL would be:
```
export BASE_URL="https://mdm42.macadmins.io"
```
Let's confirm our keys and BASE_URL are setup correctly by listing all declarations on the server:
```
cd kmfddm
./tools/api-declarations-get.sh
[] # This makes sense because we've not added any declarations yet
```
You are all set up!

### Using ddm_examples
