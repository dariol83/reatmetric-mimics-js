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
        var mimicController = null; // This variable will  
        
        async function loadSVG(){
        	if(mimicController === null) { // Create the mimic if not done yet		
        		mimicController = factoryObj.newMimic(
        			'https://127.0.0.1:8080/station.svg', // URL of the SVG file
        			document.getElementById("div1"), // Element to attach the SVG to
        			["raw", "eng", "alarm", "validity"]); // Object properties that are part of an expression (read below)
        		await mimicController.initialise(); // Initialise the mimic with the bindings
        		var bindings = mimicController.getBindings(); // Return the array of the binding objects defined in the mimics
        		bindings.forEach(a => console.log("Binding detected: " + a));
        	}
        }   
        
        [...]
        
        
    </script>
``` 
 
 - Once the mimic is created, it can be updated providing the state of the declared bindings. A state is a plain Javascript object,
 whose properties are access by the conditions and expressions defined in the SVG file. While conditions can access all properties of the
 object, expressions will only access the properties declared during the creation of the mimic. A real application would retrieve such updates from a server service (e.g. as a JSON resource) and provide the result to the mimic controller.
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

##FAQ
###How do I know which data-* attributes are supported in my SVG? 
The SVG tags are described in the ReatMetric UI module: https://github.com/dariol83/reatmetric/blob/master/eu.dariolucia.reatmetric.ui/Mimics.md

###My JSON object does not have eng, raw, alarm, validity as properties. Can I still use this library?
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

