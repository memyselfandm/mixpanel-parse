# Mixpanel for Parse Cloud Code

The existing Node module for [mixpanel-node](https://github.com/mixpanel/mixpanel-node)
did not work with Parse Cloud Code so I adjusted that code base to work as a
dependency when deployed and to also return promises for all functions so that
they could be used like other Parse functions.

The `http` module was also replaced with `Parse.Cloud.httpRequest` to make it more
compatible with the Parse environment.

## Usage

Simply copy `mixpanel.js` from this Git repository into your `cloud` folder for Parse.
To reference it use the require statement shown in the example below. You will need to
set a valid token to work with your Mixpanel profiles.

When tracking events which are related to a user use `distinct_id` to match up
those events with properties set with people with the same value for `distinctId`.

## Caveat

Typically with Node.js you can use `npm` to install and use modules. A directory named
`node_modules` is created for the dependencies. It does not work this way with Parse
so you must reference dependencies differently as described above and shown below.

## Sample Code

```javascript
var Mixpanel = require("cloud/mixpanel");

var mixpanelToken = "SET_VALID_TOKEN";

var mixpanel = Mixpanel.init(mixpanelToken);
var distinctId = "user123";

// track an event related to a user with their distinctId

mixpanel.track(eventName, properties).then(function() {
	console.log("event was tracked");
}, function(error) {
	console.log(error);
});

// set properties for a user using their distinctId

var properties = {"lastUseDate" : new Date()};

mixpanel.people.set(distinctId, properties).then(function(error) {
	console.log("person properties were set");
}, function(error) {
	console.log(error);
});
```

## Sample Jobs

```javascript
var Mixpanel = require("cloud/mixpanel");

var mixpanelToken = "SET_VALID_TOKEN";

Parse.Cloud.job("updateLastUseDate", function(request, status) {
    var mixpanel = Mixpanel.init(mixpanelToken);
    var distinctId = "user123";
    var properties = {
        "lastUseDate": new Date()
    };

    mixpanel.people.set(distinctId, properties).then(function(error) {
        console.log("person properties were set");
        status.success("OK");
    }, function(error) {
        console.log(error);
        status.error(error);
    });
});

Parse.Cloud.job("trackActivity", function(request, status) {
    var mixpanel = Mixpanel.init(mixpanelToken);
    var distinctId = "user123";
    var properties = {
        'distinct_id' : distinctId,
        'testing' : true,
        'now' : new Date()
    };

    mixpanel.track('activity', properties).then(function() {
        console.log("activity tracked");
        status.success("OK");
    }, function(error) {
        console.log(error);
        status.error(error);
    });

    mixpanel.people.set(distinctId, properties)
});
```

## License

MIT

## Author

Brennan Stehling (@smallsharptools)
