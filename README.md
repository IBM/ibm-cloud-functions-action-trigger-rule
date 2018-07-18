# IBM Cloud Functions - Your first Action, Trigger, and Rule

*Read this in other languages: [한국어](README-ko.md).*

Simple demo showing Apache OpenWhisk actions, triggers, and rules on the IBM Cloud Functions platform. Created for the [Build a cloud native app with Apache OpenWhisk webinar](https://developer.ibm.com/tv/build-a-cloud-native-app-with-apache-openwhisk/). First, [you'll need an IBM Cloud account and the latest OpenWhisk command line tool.](/docs/OPENWHISK.md)

![High level diagram](demo-1.png)

## Action: `handler.js`

This simple JavaScript function (called an _action_ in OpenWhisk) accepts a `params` argument and writes information that can be retrieved from the Cloud Functions activation log and/or the IBM Cloud Functions monitoring console. It also returns a JSON object with the current date.

First, open a terminal window to start polling the activation log. The `console.log` statements in the action will be logged here, which you can stream with the following command:

```bash
bx wsk activation poll
```

In another terminal window, upload the action file as a Cloud Function using the following command:

```bash
bx wsk action create handler handler.js
```

Then invoke it manually. This will echo the resulting JSON message to the current window and log the activation in the other window.

```bash
bx wsk action invoke --blocking handler
```

## Trigger: `every-20-seconds`

This trigger uses the built-in alarm package feed to fire events every 20 seconds. This is specified through cron syntax in the `cron` parameter. The `maxTriggers` parameter ensures that it only fires for five minutes (15 times), rather than indefinitely.

Create it with the following command:

```bash
bx wsk trigger create every-20-seconds \
  --feed  /whisk.system/alarms/alarm \
  --param cron '*/20 * * * * *' \
  --param maxTriggers 15
```

## Rule: `invoke-periodically`

This rule shows how the `every-20-seconds` trigger can be declaratively mapped to the `handler.js` action. Notice that it's named somewhat abstractly so that if we wanted to use a different trigger - perhaps something that fires every minute instead - we could still keep the logical name.

Create the rule with the following command:

```bash
bx wsk rule create \
  invoke-periodically \
  every-20-seconds \
  handler
```

At this point you can check the activation log that you are tailing in the other window to confirm that the action is invoked by the trigger.

## Installation instructions

Cloud Functions developers will often automate the creation of actions, triggers, and rules as they iterate on their application. The convention that has arisen in many sample apps is to use a `deploy.sh` script, often in conjunction with a `local.env` file to externalize environment variables.

The script can be used to set up, tear down, and see the current configuration:

```bash
./deploy.sh --install
./deploy.sh --uninstall
./deploy.sh --env # Not used in this demo
```

> **Note**: `deploy.sh` will be replaced with [`wskdeploy`](https://github.com/openwhisk/openwhisk-wskdeploy) in the future. `wskdeploy` uses a manifest to deploy declared triggers, actions, and rules to OpenWhisk.

## Troubleshooting
Check for errors first in the activation log. Tail the log on the command line with `bx wsk activation poll` or drill into details visually with the [Cloud Functions monitoring console](https://console.ng.bluemix.net/openwhisk/dashboard).

If the error is not immediately obvious, make sure you have the [latest version of the `wsk` CLI installed](https://console.ng.bluemix.net/openwhisk/learn/cli). If it's older than a few weeks, download an update.
```bash
bx wsk property get --cliversion
```

## License
[Apache 2.0](LICENSE.txt)
