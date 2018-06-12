# TheThingsAPI #

This library wraps the [TheThings.IO](http://www.thethings.io) Internet of Things Cloud.

**To add this library to your project, add** `#require "TheThingsAPI.class.nut:1.2.0"` **to the top of your device code.**

## Class Usage ##

### Callbacks ###

All methods which make web requests to TheThings.IO (*activate()*, *write()*, *read()* and *subscribe()*) take an optional callback as a parameter. The callbacks require three parameters: *err*, *resp* and *data*:

| Parameter | Notes |
| --------- | ----- |
| *err*     | `null` on success, or a string indicating the error |
| *resp*    | The [HTTP Response table](https://developer.electricimp.com/api/httprequest/sendasync) from the request |
| *data*    | The decoded body of the request |

### Constructor: TheThingsAPI(*[thingToken]*) ###

Create a new thing by passing its existing token as an argument &mdash; or leave it empty to activate it later using the *activate()* method. You can use the method *getToken()* to retrieve the thing’s Thing Token later.

#### Example ####

```squirrel
#require "TheThingsAPI.class.nut:1.2.0"

// Instantiate a thing with an existing token
thing <- TheThingsAPI("<EXISTING_THING_TOKEN>");
```

## Class Methods ##

### activate(*activationCode[, callback]*) ###

This method activates a thing using a TheThings.IO Activation Code, which identifies you as the creator of the thing and binds the thing to your account. Activation generates a unique Thing Token that will identify your thing at thethings.IO.

#### Example ####

```squirrel
// Create a thing object
thing <- TheThingsAPI();

// Activate a thing with a token from your Dev Console
thing.activate("<ACTIVATION_TOKEN>", function(err, resp, data) {
  // If it failed - log an error message
  if (err) {
    server.error("Failed to activate Thing: " + err);
    return;
  }
  server.log("Success! Activated Thing!");
});
```

### addVar(*key, value[, metadata]*) ###

This method allows you to add a key/value pair that will be sent to TheThings.IO the next time *write()* is called. An optional third parameter, *metadata*, can be included to add additional metadata to the key/value pair. The *metadata* parameter should be a table with any of the following keys:

| Key       | Type    | Description |
| --------- | ------- | ----------- |
| *timestamp* | Integer | A unix timestamp (generated with [time()](https://developer.electricimp.com/squirrel/system/time)) or a datetime string (YYYYMMDDHHmmss) indicating when the sample was collected |
| *geo*       | Table   | A table with *lat* and *long* keys indicating the latitude and longitude where the sample was collected |

The *addVar()* method returns a reference to *this*, allowing you to chain multiple calls to *addVar()* together, or invoke *write()* immediatly after *addVar()*.

#### Example ####

```squirrel
// Add a simple sample:
thing.addVar("foo", "bar");

// Add a sample with metadata:
thing.addVar("foo1", "bar1", { "timestamp": time() - 3600,  // 1 hour ago
                               "geo": { "lat": 41.4121132,
                                        "long": 2.2199454 } }
});

// Add two samples at once:
thing.addVar("foo2", "bar2").addVar("foo3", "bar3");

// Send all four samples to TheThings.IO
thing.write(function(err, resp, data) {
  if (err) {
    server.error(err);
    return;
  }
  
  server.log("Success!");
});
```

### write(*[callback]*) ###

Sends the information collected with *addVar()* to TheThings.IO.

See *addVar()*, above, for sample usage.

### read(*key[, filters][, callback]*) ###

This method returns one or more samples for the specified *key*. An optional second parameter, *filters*, can be passed in to provide more information about the samples you are searching for. The filter table can contain any of the following keys:

| Key       | Type    | Description |
| --------- | ------- | ----------- |
| *limit*     | Integer | The maximum number of results to return (1-100) |
| *startDate* | Integer | A unix timestamp (generated with [**time()**](https://developer.electricimp.com/squirrel/system/time)) or a datetime string (YYYYMMDDHHmmss) |
| *endDate*   | Integer | A unix timestamp (generated with [**time()**](https://developer.electricimp.com/squirrel/system/time)) or a datetime string (YYYYMMDDHHmmss) |

**Note** If a limit is not supplied, the *read()* method will return the last sample for the specified key.

#### Example ####

```squirrel
// Get the last sample collected for 'foo'
thing.read("foo", null, function(err, resp, data) {
  if (err) {
    server.error(err);
    return;
  }
  
  // Get the sample
  local result = data[0];
  
  // Do something with the sample
  server.log(result.datetime + ": " + result.value);
});

// Get the last 100 samples collected in the last 24 hours for 'foo'
local yesterday = time() - 86400; // 86400 seconds in a day
thing.read("foo", { "limit": 100, "startDate": yesterday }, function(err, resp, data) {
  if (err) {
    server.error(err);
    return;
  }

  foreach(sample in data) {
    server.log(sample.datetime + ": " + sample.value);
  }
});
```

### subscribe(*[callback]*) ###

This method subscribes to the thing channel to get real time updates. Updates are passed to the callback when available.

#### Example ####

```squirrel
thing.subscribe(function(error, response, data) {
  if (error) {
    server.error(error);
    return;
  }

  // We received an updated value from the thing channel
  if (typeof content == "array") {
    foreach (item in content) {
      // Do something with the update
      server.log(item.key + ": " + item.value);
    }
  }

  // We got a ping or connection status update
  if (typeof content == "table") {
    // Log status and message content
    if ("status" in content) server.log("Request status: " + content.status);
    if ("message" in content) server.log("Message: " + content.message);
  }
});
```

### getToken() ###

Returns the thing’s Thing Token, which identifies your thing at thethings.IO.

```squirrel
// Save the Thing's token:
local data = server.load();
data.token <- thing.getToken();
server.save(data);
```

## License ##

The TheThingsIO library is licensed under the [MIT License](https://github.com/electricimp/thethingsapi/tree/master/LICENSE).
