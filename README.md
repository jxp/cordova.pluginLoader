cordova.pluginLoader
====================

Suggested change to plugin module definition for cordova.

The current implementation of plugin module definitions is:

    cordova.define("cordova/plugin/myplugin", function(require, exports, module) {
      var exec = require('cordova/exec');
    
    	var MyPlugin = function() {};
    
      // Define plugin behaviour here e.g.
    	MyPlugin.prototype.foo = function(successCallback,failureCallback) {
    	    exec(successCallback, failureCallback, 'MyPlugin', 'foo', []);
    	}
      /*
      
      ...
      
      */ 
    
    	var myplugin = new MyPlugin();
    	module.exports = myplugin;
    });
    

This definition is a mix of plugin definition and module loading implementation.

Having the two together like this makes customizing the loading implementation difficult.
The module definition and loading implementation should be separated to allow greater flexibility.

New module definition
---------------------

The suggested alternative is to make the plugin module simpler as follows:

    cordova.pluginLoader("myplugin", function(exec) {
      var MyPlugin = function() {};
    
      // Define plugin behaviour here e.g.
    	MyPlugin.prototype.foo = function(successCallback,failureCallback) {
    	    exec(successCallback, failureCallback, 'MyPlugin', 'foo', []);
    	}
      /*
      
      ...
      
      */ 
      return new MyPlugin();  
    });
    
This definition only contains the implementation of the plugin module along with its name.

Loader Option 1 - Standard definition
-------------------------------------

The plugin loader method can then reproduce the boiler-plate code from the previous module definition;

    cordova.pluginLoader = function(pluginName, factory) {
      cordova.define("cordova/plugin/" + pluginName, function(require, exports, module) {
        var exec = cordova.require('cordova/exec');
        module.exports = factory(exec);
      });
    };
    
This approach would allow developers to change the implementation of pluginLoader to suit their requirements 
e.g. A classic pluginLoader would be;

    cordova.pluginLoader = function(pluginName, factory) {
      if (!window.plugins) {
        window.plugins = {};
      }
    
      window.plugins[pluginName] = factory(cordova.exec);
    };
    
This approach would keep cordova's implementation clean (and identical to the current one).
    

Loader Option 2 - Flexible definition
-------------------------------------

Greater flexibility could be offered in the default cordova library without (as much) need for developer customisation.
e.g.

    cordova.pluginModes = { STANDARD: 0, CLASSIC: 1, AMD: 2 }
    
    // cordova initialization - determine pluginMode
    if (typeof define === 'function' && define.amd) {
      cordova.pluginMode = cordova.pluginModes.AMD;
    } else {
      cordova.pluginMode = cordova.pluginModes.STANDARD;
    }
    
    // Define switchable pluginLoader
    cordova.pluginLoader = function(pluginName, factory) {
      switch (cordova.pluginMode)
      {
        case cordova.pluginModes.CLASSIC: 
          if (!window.plugins) {
            window.plugins = {};
          }
        
          window.plugins[pluginName] = factory(cordova.exec);
          break;
        case cordova.pluginModes.AMD:
          define(function() {
            var exec = cordova.require('cordova/exec');
            return factory(exec);
          });
          break;
        default:
          cordova.define("cordova/plugin/" + pluginName, function(require, exports, module) {
            var exec = cordova.require('cordova/exec');
            module.exports = factory(exec);
          });
      }
    };
    
    
This approach would prevent developers needing to customize the pluginLoader.
It would also allow individual plugins to be loaded in different modes
(by changing the value of cordova.pluginMode before loading it).

