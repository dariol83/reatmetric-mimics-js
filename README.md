# ReatMetric Mimics JS Library

A Javascript library for the visualisation of SVG mimics in HTML pages, using declaration-oriented behaviour for SVG scripting, based on ReatMetric's mimics engine.
 
## Getting started

Getting started with ReatMetric mimics is very simple: just follow the steps below.

- Include the library in your HTML page

```
    <script type="text/javascript" src="rtmt-mimics.js"></script>
```
  
- Identify an element in your HTML page that will contain the SVG mimic that you want to use, 
e.g. a div element with id "div1". 

```
    <div id="div1"><!-- The SVG will go here --></div>
```

- Create a script block and a function, which will be triggered by some user interactions, e.g. on page load or on click,
e.g. with a button. The script block instantiates the mimics factory, and the function requests the 
loading of a SVG mimic.

```
    <script type="text/javascript">
        var factoryObj = new RtmtMimics(document); // Create the mimics factory
        var mimicController = null; // This variable will contain the reference to the SVG controller object 
        
        async function loadSVG(){
        	if(mimicController === null) { // Create the mimic if not done yet		
        		mimicController = factoryObj.newMimic(
        			'https://127.0.0.1:8080/station.svg', // URL of the SVG file
        			document.getElementById("div1"), // Element that will contain the SVG
        			["raw", "eng", "alarm", "validity"]); // Object properties that are part of an expression (read below)
        		await mimicController.initialise(); // Initialise the mimic with the bindings
        		var bindings = mimicController.getBindings(); // Return the array of the binding objects defined in the mimics
        		bindings.forEach(a => console.log("Binding detected: " + a)); // Just print the bindings
        	}
        }   
        
        [...]
        
        
    </script>
``` 
 
 - Once the mimic is created, it can be updated providing the state of the declared bindings. You can retrieve such bindings using the 
 getBindings() function call. It is important to know which bindings are declared in a mimic, so that you can subscribe or query 
 for the corresponding object states. A state of a binding is a plain Javascript object, whose properties are access by the conditions 
 and expressions defined in the SVG file. While conditions can access all properties of the object, expressions will only access 
 the properties declared during the creation of the mimic. A real application would retrieve such updates from a server service 
 (e.g. as a JSON resource) and provide the result to the mimic controller.
An example of update is:
 
``` 
     var updates = {
 		"ID.OF.PARAM1" : {
 			"raw" : 0,
 			"eng" : "ON",
 			"alarm" : "NOMINAL",
 			"validity" : "VALID"
 		},
 		"ANOTHER.ID.OF.PARAM2" : {
 			"raw" : 240,
 			"eng" : 240,
 			"alarm" : "ALARM",
 			"validity" : "VALID"
 		}
     };
     mimicController.update(updates);
``` 

- If the mimic must be disposed, the dispose() function can be called on the mimic controller.

```
    async function disposeSVG(){
        if(mimicController != null) {
            mimicController.dispose();
            mimicController = null;
        }
    }
```



## FAQ

### How do I know which data-* attributes are supported in my SVG? 

The SVG tags are described in the ReatMetric UI module: https://github.com/dariol83/reatmetric/blob/master/eu.dariolucia.reatmetric.ui/Mimics.md

### My JSON object does not have eng, raw, alarm, validity as properties. Can I still use this library?

This Javascript framework is not limited to the reference names ($eng, $raw, $alarm, $validity) defined by ReatMetric: **any** Javascript object property
can be access by this library in the condition evaluation, and **any** property can be access by this library in the expression evaluation, as long 
as it is declared in the newMimic() function. For instance, if your JSON object has *text*, *name* and *value* as property names, you
can access them using $text, $name and $value references in conditions and expressions. Do not forget to create the mimic accordingly:

```
mimicController = factoryObj.newMimic(
    'https://...', // URL of the SVG file
    document.getElementById(...), 
    ["text", "name", "value"]); 
```

### In ReatMetric SVGs you can also define actions (reatmetric.exec, reatmetric.set, etc...). Is that supported here?

Since SVG objects are plainly attached to the provided HTML element, function calls declared in the SVG file will be 
routed to corresponding functions defined in the hosting web page. For instance, if your SVG contains a circle object defined
as

```
  <circle id="pw-spl-led" cx="14.299" cy="101.49" r="2.6162" fill="#b3b3b3" stroke="#000000" stroke-linejoin="round" stroke-width=".51065" style="paint-order:normal"
    onclick="controller.execute('Circle clicked!')" cursor="pointer"
  />
```

when pressing on the circle, the JS code "controller.execute(...)" will be executed. Therefore, if there is a Javascript variable *controller*
declared in your web page, and if this object has the "execute(obj)" function defined, then the function will be invoked. This is standard
Javascript behaviour and it is independent from this library. Therefore, from SVG code you can call your own Javascript code as usual.