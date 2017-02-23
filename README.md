# OpenWhisk 101 - Your first Action, Trigger, and Rule

Simple demo showing OpenWhisk actions, triggers, and rules. Created for the [Build a cloud native app with Apache OpenWhisk webinar](https://developer.ibm.com/tv/build-a-cloud-native-app-with-apache-openwhisk/).

![High level diagram](demo-1.png)

## Action: `handler.js`

This simple JavaScript function (called an _action_ in OpenWhisk parlance) accepts a `params` argument and writes information that can be retrieved from OpenWhisk logs and/or the IBM Bluemix monitoring console. It also returns a JSON object with the current date.

This action can be uploaded to OpenWhisk using the following command:

```bash
wsk action create handler handler.js
```

It can be manually invoked during testing in a synchronous manner. This will echo the resulting JSON message:

```bash
wsk action invoke --blocking handler
```

The `console.log` statements will be logged to the OpenWhisk activation log, which you can stream with the following command:

```bash
wsk activation poll
```

## Trigger: `every-20-seconds`

This trigger uses the built in alarm package feed to fire events every 20 seconds which is specified by a cron syntax in the `cron` parameter passed when it is created. The `maxTriggers` parameter ensures that it only fires for two minutes, rather than the default of 10,000 invocations.

The trigger can be created with the following command:

```bash
wsk trigger create every-20-seconds \
    --feed  /whisk.system/alarms/alarm \
    --param cron '*/20 * * * * *' \
    --param maxTriggers 6
```

## Rule: `invoke-periodically`

This rule shows how the `every-20-seconds` trigger can be declaratively mapped to the `handler.js` action. Notice that it's named somewhat abstractly so that if we wanted to use a different trigger, perhaps something that fires every minute instead, we could still keep the logical name.

The rule can be created with the following command:

```bash
wsk rule create \
    invoke-periodically \
    every-20-seconds \
    handler
```

At this point you can check the activation log to see if indeed the action is invoked by our trigger:

```bash
wsk activation poll
```

## Installation instructions

OpenWhisk developers will often automate the creation of actions, triggers, and rules as they iterate on their application. The convention that has arisen in many sample apps is to use a `deploy.sh` script, often in conjunction with a `local.env` file to externalize environment variables.

The script can be used to set up, tear down, and see the current configuration:

```bash
./deploy.sh --install
./deploy.sh --uninstall
./deploy.sh --env # Not used in this demo
```

### Install the `wsk` CLI from Bluemix

After registering for [Bluemix](http://bluemix.net/), navigate to the "[OpenWhisk](https://console.ng.bluemix.net/openwhisk/)" section. You'll find it in the left navigation, under the three horizontal bar (hamburger icon).

Click the "Download OpenWhisk CLI" button and place the `wsk` binary in your path, such as in `~/bin`. Open a terminal and set your namespace and authorization as shown in step 2\. Then create your first action in step 3.
