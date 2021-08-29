[<- Back to README.md](../../README.md)
# Intro

One simple use case for a duress password is to simply notify IT of the user being in duress. In this demonstration we'll create a script which sends a pushover notification with the username reporting the duress, the hostname, and the current IP.

## Requirements

This will use curl, so install it if not available on your system.

```bash
sudo apt-get install curl
```

## The Script

First we'll make a pushover script. The first step for that should be to generate an app in your pushover account for this purpose. Then you'll want to generate an configuration file:

```bash
# /root/.pushover_creds
APP_TOKEN=[YOUR APP TOKEN]
USER_KEY=[YOUR USER KEY]
```

```bash
#!/bin/sh
#/etc/duress.d/pushover.sh

# Grab the credentials
. /root/.pushover_creds

# Priority of the pushover message.
PRIORITY=2
# How long to persist notifications for priority 2.
EXPIRE=3600 # 1 Hour
# How often to resend notifications for priority 2.
RETRY=900 # 15 minutes

# Format a message.
MSG="""$PAMUSER has used the duress password on $HOSTNAME.
Local IP is $( hostname -I ).
External IP is $( curl https://ipinfo.io/ip 2>/dev/null )"""

# Send the message to the pushover service.
curl -s \
  --form-string "token=$APP_TOKEN" \
  --form-string "user=$USER_KEY" \
  --form-string "message=$MSG" \
  --form-string "priority=$PRIORITY" \
  --form-string "expire=$EXPIRE" \
  --form-string "retry=$RETRY" \
  https://api.pushover.net/1/messages.json 2>&1 1>/dev/null

# Unset variables containing pushover secrets.
unset APP_TOKEN
unset USER_KEY
```

Now we'll sign the script with a password:

```bash
sudo duress_sign /etc/duress.d/pushover.sh
Password: # For example HackThePlanet
Confirm: # Again, we type HackThePlanet
# Ensure it can't be modified later.
sudo chmod 500 /etc/duress.d/pushover.sh
sudo chmod 400 /etc/duress.d/pushover.sh.sha256
```

This will produce a pushover.sh.sha256 file. The has is not the direct hash of the file but rather the hash of the password salted with the sha256 hash of the file.

You can then distribute this global configuration's password to new users you can simply give them the password. Before local configurations are run a password provided to the duress module will be checked against the hashes of the scripts in /etc/duress.d. If the password matches the script wil be run with root privileges.

**SECURITY NOTE:** If a person who is granted one of these shared passwords leaves the group/organization/etc, you should immediately re-sign the script. This is in keeping with the best practices of using organizational duress-words; they must be kept secret and need-to-know and if someone leaves the group, they don't need to know anymore so change it.

[<- Back to README.md](../../README.md)