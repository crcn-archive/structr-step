### Structr-step makes asyncronous function chainable


### Projects using structr-step

- [mongodblite](crcn/node-mongodblite)
	- see [this JS files](/crcn/node-mongodblite/blob/master/lib/collection.js)

### Turn code from this

```javascript

var structr = require("structr"),
fs = require("fs");

var Config = structr({

	/**
	 */

	"__construct": function(file) {
		this.configFile = file;

	},
	
	
	/**
	 * loads the config file
	 */

	"load": function(next) {

		var self = this;

		//make sure the confg file exists
		fs.stat(this.configFile, function(err, stat) {

			//if it doesn't exist, then set the config to an object, and
			//skip the rest
			if(err) {
				self._config = {};
				return next();
			}

			//otherwise load it
			fs.readFile(self.configFile, function(err, content) {

				//and set the config
				next(null, self._config = JSON.parse(content));
			})

		});
	},

	/**
	 * returns the value in a config. This is blocked until
	 * the config has been loaded
	 */

	"getValue": function(key, next) {
		return this._config[key];
	},

	/**
	 * sets the vaue in the config
	 */

	"setValue": function(key, value, next) {
		this._config[key] = value;
		this.save();
		next();
	},

	/**
	 * saves the config
	 */

	"step save": function(next) {
		fs.writeFile(this.configFile, JSON.stringify(this._config), next);
	}
});



var config = new Config(__dirname + "/cache.json");
config.load(function(err) {
	if(err) return;
	var value = config.getValue("name");
	//do stuff...
});
```

### Into This
```javascript

var structr = require("structr"),
fs = require("fs");

//mix into structr
structr.mixin(require("../"));



var Config = structr({

	/**
	 */

	"__construct": function(file) {
		this.configFile = file;

		//load the config before any methods can be
		//called
		this.load();
	},
	
	
	/**
	 * loads the config file
	 */

	"step load": function(nextFn) {
		this.step(

			//make sure the confg file exists
			function(next) {
				fs.stat(this.configFile, next);
			},

			//if it doesn't exist, then set the config to an object, and
			//skip the rest. Otherwise load it.
			function(err, stats, next) {

				if(err) {
					this._config = {};
					return nextFn(err);
				}

				fs.readFile(this.configFile, next);
			},

			//set the config, and finish
			function(err, content, next) {
				next(null, this._config = JSON.parse(content));
			},

			//continue onto any other function called in this config
			nextFn
		);
	},

	/**
	 * returns the value in a config. This is blocked until
	 * the config has been loaded
	 */

	"step getValue": function(key, next) {
		next(null, this._config[key]);
	},

	/**
	 * sets the vaue in the config
	 */

	"step setValue": function(key, value, next) {
		this._config[key] = value;
		this.save();
		next();
	},

	/**
	 * saves the config
	 */

	"step save": function(next) {
		fs.writeFile(this.configFile, JSON.stringify(this._config), next);
	}
});



var config = new Config(__dirname + "/cache.json");
config.getValue("name", function(err, value) {
	if(!value) {
		console.log("config value does not exist, saving!");
		config.setValue("name", "craig");
	} else {
		console.log("hello %s!", value);
	}
});

```