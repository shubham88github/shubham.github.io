---
title: "Send system stats to slack"
excerpt_separator: "<!--more-->"
categories:
  - bash
tags:
  - bash
  - stats
  - slack
  - web hooks
---

I currently own few linux machines, be it a raspberry pi or a laptop. Id like to know whats the status on those machines at set intervals. Especially the public IP address, since its the only way to ssh into em. My headless servers which i use as vpn servers, and as well as my raspberrypis that could just be taking pictures to compile a timelapse would send me updates to my personal slack. It uses the incoming webhook intergration to send messages to my `#devices` channel.

To achieve this, i use a simple bash script to gather the required system stats. And then do a post request via curl to the webhook.

```bash
#!/bin/bash

SLACK_WEB_HOOK_URL="YOUR_SLACK_WEB_HOOK_URL"
# used to format the message
FORMAT='```'

# to store the netstat output
NETSTAT_LOG=netstat.log

# temporary destination for the message contents
MESSAGE_LOG=message.log

# delete the message contents of the previous run
if [ -e $MESSAGE_LOG ]
then
    rm $MESSAGE_LOG
fi

# fetch the public ip address from the url provided
IP="ip-address : $( curl -s http://whatismijnip.nl | cut -d " " -f 5 )"

echo $IP >> $MESSAGE_LOG

# all the ports and applications which are listening
netstat -l | grep "LISTEN " > $NETSTAT_LOG
NETSTAT=$( <$NETSTAT_LOG )

echo -e "\nnetstat : \n$NETSTAT" >> $MESSAGE_LOG

# system resource usage
TOP=$(top -bn3 | head -n 2 )

echo -e "\ntop:\n$TOP" >> $MESSAGE_LOG

# construct the message
MESSAGE=$FORMAT$(  <$MESSAGE_LOG )$FORMAT

# post the message to slack
curl -X POST --data-urlencode \
    "payload={\"channel\": \"#devices\", \"username\": \"thinkpad-of-death\", \"text\": \"$MESSAGE\", \"icon_emoji\": \":satellite_antenna:\"}" $SLACK_WEB_HOOK_URL

```

I used cron to call this script every 5 hours, and also on boot. So i will have a notification immedietly when the system boots.
below are some screenshots of the result.

The result from the code above.
![output of the code above](/assets/images/posts/update-slack-with-stats/screenshot-2.png){: .align-center }

Another example, but different format.
![different format](/assets/images/posts/update-slack-with-stats/screenshot-1.png){: .align-center }
