
**[ Deprecated: Chrome 43 supports MIDI natively, see this [post][1] ]**


[1) Introduction][4]
[2) Quick start guide][5]
[3) Documentation][6]
[4) About midi out][7]
[5) Earlier versions][8]
[6) What about Flash?][9]
[7) Known issues][10]
[8) Forthcoming features][11]

<a name="introduction"></a> **Introduction**

The midibridge is a Javascript API for interacting with the midi devices on your computer.

It provides methods for detecting the midi devices and for connecting these devices with each other.

The midibridge itself is considered a midi device as well, so it can be connected to the detected devices.

The midibridge can generate and send midi events to midi output devices, and receive midi events from midi input devices.

The midibridge can also filter and alter midi events.

A sequencer for playing back midi files is implemented as well, recording will be added in a later version.

A midi output device is a physical output port on your computer, a virtual output port or a software synthesizer. It can also be a sequencer, a file or a function.

A midi input device is a physical or virtual input port, a sequencer, a file or a function.

A midi device can be both in- and output. The midibridge itself for instance is both in- and output because it can send and receive midi events.

The actual interaction with the midi devices on your computer is done by a Java applet. The midibridge automatically adds the applet to your webpage.

The midibridge has no visual parts, it is 'headless' code. You could say the midibridge enables you to write a 'front-end' on top of the applet.

Midi Devices -> Java Applet -> Javascript Midibridge API -> a GUI in Javascript, Flash, SVG, C# (Silverlight)

Because the midibridge is written in native Javascript, you can use it conflict-free with any Javascript framework.

<a name="quick-start"></a> **Quick start guide**

**1)** Download the <a href="https://github.com/abudaan/midibridge-js/zipball/master" target="blank" title="abumarkub midibridge javascript" rel="abumarkub midibridge javascript">zip file</a> from Github.

**2)** In this zip file you'll find an index.html and 3 folders: /java contains a Java archive (jar) file, /lib contains a Javascript file (minified and non-minified) and /examples contains the code examples of this documentation.

On your webserver, put the jar file in a folder called "java" and the minified javascript file in the folder where you usually store your javascript libraries. Personally, i put the Javascript libraries that i use in a project in a "lib" folder, and the project specific javascript files in a folder "js".

**3)** Include the file midibridge-0.5.3.min.js in your webpage. If you use a Javascript framework (jQuery, YUI, Dojo, etc.), include it right after you've included the framework. See the index.html in the zip.

**4)** Start the midibridge. The page has to be fully loaded before you start the midibridge, so call the init function of the midibridge inside your onload handler:


    window.addEventListener('load', function() {
        midiBridge.init(function(midiEvent) {
            console.log(midiEvent);
        });
    }, false);



**5)** Done! If you have a midikeyboard connected to you computer, play some notes and you'll hear a piano sound. Also you will see that the midi events are printed to your console.

<a name="documentation"></a> **Documentation**

By adding the midibridge to your html page, a global variable `midiBridge` is created. Below follows a list of the methods that you can call on the `midiBridge` object. There is also a bunch of handy static members that you use in your code:

- [init()][12]   
- [sendMidiEvent(status, channel, data1, data2)][13]   
- [getDevices()][14]   
- [refreshDevices()][15]   
- [connectAllInputs()][16]   
- [connectFirstInput()][17]   
- [connectFirstOutput()][18]   
- [connectAllInputsToFirstOutput()][19]   
- [addConnection(input,output,filter)][20]   
- [removeConnection(input,output)][21]   
- [disconnectAll()][22]   
- [getNoteName(midiNoteNumber,mode)][23]   
- [getNoteNumber(noteName,octave)][24]   
- [getStatus(statusCode)][25]
- [loadBase64String(base64String)][26]   
- [playBase64String(base64String)][27]   
- [startSequencer()][28]   
- [pauseSequencer()][29]   
- [stopSequencer()][30]   
- [closeSequencer()][31]   
- [getSequencerPosition()][32]   
- [setSequencerPosition(microseconds)][33]   
- [getNiceTime(microseconds)][34]
- [getObject(id)][35]   
- [MidiMessage(jsonString)][36]   
- [List of static members][37]

<a name="init"></a> `init()`

The init method of the midibridge adds an extra div to the body of your html page, and in this div the Java applet gets loaded. After the applet has loaded, it detects all currently connected midi in- and outputs. Then it automatically connects all midi inputs to the midibridge. Also, the first found midi output gets connected to all midi inputs and to the midibridge.

This way you can start playing your midi keyboard directly after the midibridge is initialized. When you play, the callback method that you have passed as an argument to the init function will be called. In the example above, the incoming midi events are only printed to the console, but i'm sure you can come up with something more funky.

You can also call the init method with a configuration object as argument, this object may contain one or all of the following keys:

*   `ready` : [function] callback function that gets called as soon as the midibridge is ready/initialized
*   `error` : [function] callback function that gets called in case of an error, for instance the user does not have a Java plugin or an outdated version or the user is using a not-supported browser
*   `data` : [function] callback that gets called when midi data has arrived
*   `javaDir` : [string] the folder where you store the midiapplet.jar on your webserver, defaults to "java"
*   `connectFirstInput` : [true,false] the first found midi input device gets connected automatically, defaults to false
*   `connectAllInputs` : [true,false] all found midi input devices get connected automatically, defaults to false
*   `connectFirstOutput` : [true,false] the first found midi output device gets connected automatically, defaults to false
*   `connectAllInputsToFirstOutput` : [true,false] all found midi input devices will be automatically connected to the first found midi output device, defaults to true
*   `debug` : [true,false] when set to true, some debug information is printed to the console, eg: the filtered MIDI messages, see next bullet
*   `midiCommands` : [array] an array with the types of MIDI messages that the midibridge is passing to the data handler, defaults to: `[midiBridge.NOTE_OFF,
     midiBridge.NOTE_ON,
     midiBridge.CONTROL_CHANGE,
     midiBridge.PITCH_BEND,
     midiBridge.PROGRAM_CHANGE]` this setting overrules the filter setting on MIDI connections, see [addConnection()][20].

<!--
Only one of the 3 keys <code>connectAllInputsToFirstOutput</code>, <code>connectAllInputs</code> and <code>connectFirstOutput</code> can be true; if you set one of them to true, the other two will be set to false automatically.
--> In the example below all midi inputs get connected to the midibridge, but they will not be connected to the first midi output, i.e. you won't hear a sound when you play your keyboard(s).

Also, all messages from the midibridge are printed to a div in the html page.



    window.addEventListener('load', function() {

        var contentDiv = document.getElementById('content');
        contentDiv.innerHTML += 'midibridge says:<br/>';

        midiBridge.init({
            connectAllInputs: true,

            ready: function(msg){
                contentDiv.innerHTML += msg + '<br/>';
            },
            error: function(msg) {
                contentDiv.innerHTML += msg + '<br/>';
            },
            data: function(midiEvent) {
                contentDiv.innerHTML += midiEvent + '<br/>';
            }
        });


    }, false);



You can check this example <a href="../midibridge/examples/configobject.html" target="blank">here</a>.

<a name="sendMidiEvent"></a> `sendMidiEvent(status, channel, data1, data2)`

With this method you can send midi events to the applet. You can use this method for instance to make a vitual keyboard. Here is a code snippet to help you started:



    window.addEventListener('load', function() {

        midiBridge.init(function(midiEvent) {
            console.log(midiEvent);
        });

        var contentDiv = document.getElementById("content");
        contentDiv.innerHTML += "<div id='playAnote'>play a note</div>";
        var playNote = document.getElementById("playAnote");
        playNote.style.textDecoration = "underline";

        playNote.addEventListener("click",function(){
            midiBridge.sendMidiEvent(midiBridge.NOTE_ON,1,80,100);
        }, false);
    };




Now if you click on "play a note", you should hear a note.

You can check this example <a href="../midibridge/examples/playnote.html" target="blank">here</a>.

<a name="getDevices"></a> `getDevices()`

This method returns a JSON object that contains all midi devices that were detected on your computer when the midibridge was initialized.



    window.addEventListener('load', function() {

        var contentDiv = document.getElementById('content');

        midiBridge.init({
            connectAllInputsToFirstOutput: false,

            ready: function(msg){
                var devices = midiBridge.getDevices();
                for(var i = 0, max = devices.length; i < max; i++) {

                    var device  = devices[i];
                    var id      = device.id;
                    var type    = device.type;
                    var name    = device.name;
                    var descr   = device.descr;
                    var available = device.available;
                    contentDiv.innerHTML += id + ' : ' + type+ ' : ' + name+ ' : ' + descr+ ' : ' + available + '<br/>';
                }
            },
        });

    }, false);



The parameter `available` shows whether the device is currently in use or not. For instance if your midi keyboard was connected to a softsynth at the time you started the midibridge, that midi keyboard will be listed but the parameter available will be set to false.

You can check this example <a href="../midibridge/examples/getdevices.html" target="blank">here</a>.

<a name="refreshDevices"></a> `refreshDevices()`

You can use this method if your midi configuration has changed after you have started the midibridge. For instance if you connect a new midi device to your computer.

If you call this method, all current connections between your midi devices between your midi inputs and the midibridge will be disconnected.

<a name="connectAllInputs"></a> `connectAllInputs()`

By calling this method, all detected midi inputs will be connected to the midibridge. You can also achieve this if you call the `init()` method with a configuration object, and then set the key `connectAllInputs` to true.

<a name="connectFirstInput"></a> `connectFirstInput()`

By calling this method, the first detected midi input will be connected to the midibridge. You can also achieve this if you call the init() method with a configuration object, and then set the key `connectFirstInput` to true.

This method does not connect a midi output, so you can use this configuration for instance if your application only has a visual representation of the midi events.

NOTE: Sometimes the first midi input is not the device that you actually want to connect. In this case use `getDevices()` to check what id has been assigned to the midi input that you want to connect to the midibridge, and establish the connection with `addConnection(midiInputId,midiOutputId)`. This also applies to `connectFirstOutput`.

<a name="connectFirstOutput"></a> `connectFirstOutput()`

By calling this method, the first detected midi output will be connected to the midibridge. You can also achieve this if you call the init() method with a configuration object, and then set the key `connectFirstOutput` to true.

This method does not connect a midi input, so you can use this configuration for instance if your application has a virtual keyboard, or if you attach the regular keyboard to the midibridge.



    window.addEventListener('load', function() {

        midiBridge.init({
            connectFirstOutput : true
            ready : function(msg) {
                connectKeyboard();
            },
            error : function(msg) {
                console.log(msg);
            }
        });

        var noteNumbers = {
            //white keys
            65 : 48,     //key a -> note c
            83 : 50,     //key s -> note d
            68 : 52,     //key d -> note e
            70 : 53,     //key f -> note f
            71 : 55,     //key g -> note g
            72 : 57,     //key h -> note a
            74 : 59,     //key j -> note b
            75 : 60,     //key k -> note c
            76 : 62,     //key l -> note d
            186 : 64,    //key ; -> note e
            222 : 65,    //key : -> note f
            //black keys
            87 : 49,     //key w -> note c#/d♭
            69 : 51,     //key e -> note d#/e♭
            84 : 54,     //key t -> note f#/g♭
            89 : 56,     //key y -> note g#/a♭
            85 : 58,     //key u -> note a#/b♭
            79 : 61,     //key o -> note c#/d♭
            80 : 63      //key p -> note d#/e♭
        }

        var keysPressed = {};

        var connectKeyboard = function(){

            document.addEventListener('keydown', function(e) {
                if(e.which === 32) {
                    midiBridge.sendMidiEvent(midiBridge.CONTROL_CHANGE, 1, 64, 127);
                } else if(noteNumbers[e.which] &amp;&amp; !keysPressed[e.which]) {
                    midiBridge.sendMidiEvent(midiBridge.NOTE_ON, 1, noteNumbers[e.which], 100);
                    keysPressed[e.which] = true;
                }
            }, false);


            document.addEventListener('keyup', function(e) {
                if(e.which === 32) {
                    midiBridge.sendMidiEvent(midiBridge.CONTROL_CHANGE, 1, 64, 0);
                } else if(noteNumbers[e.which]) {
                    midiBridge.sendMidiEvent(midiBridge.NOTE_OFF, 1, noteNumbers[e.which], 0);
                    keysPressed[e.which] = false;
                }
            }, false);
        };


    }, false);




The spacebar is used for the sustainpedal. Pressing a sustainpedal is a control change event. The controller number of the sustainpedal is 64 and 127 means pedal down, 0 means pedal up.

You can check this example <a href="../midibridge/examples/regularkeyboard.html" target="blank">here</a>.

<a name="connectAllInputsToFirstOutput"></a> `connectAllInputsToFirstOutput()`

By calling this method, all detected midi inputs will be connected to the midibridge, and all inputs will also be connected to the first detected midi output.

This is the default configuration of the midibridge, so you can also achieve this if you call the `init()` method with no arguments, or with a callback function as argument.

It is interesting to know that the applet duplicates all midi events that arrive from your midi inputs (e.g. your midi keyboard). One event travels on to the midibridge and thus is available in Javascript. The other event travels to a midi out device if a midi output is connected to that midi input.

<a name="addConnection"></a>

`addConnection(input,output,filter)` You can set up a connection between any midi input and midi output device by passing the ids of the devices as arguments to this method. You can lookup the id of an device in the JSON object that is returned when you call `getDevices()` or `refreshDevices()`.

The id of the midi input and the midi output can also be set to `-1`. If you for instance only want to capture midi events in your application without connecting to an output device, you can set the id of the midi output to `-1`. If you set both the midi input and the midi outport to `-1`, nothing will happen.

The `filter` argument is an optional argument that you can use to pass an array with midi message types (status codes) that you are not interested in. If a midi message of one of the specified types is generated by an external midi input, the message will not be passed on to the midibridge. You can specify the midi message types with a decimal number (224), a hexadecimal number (0xF0) or with a human readable midibridge constant (midiBridge.PITCH_BEND). In the example below pitch bend midi messages will be filtered.

The code below shows you how you could implement functionality that allows the users to set up midi connections:



    window.addEventListener('load', function() {

        var contentDiv = document.getElementById('content');
        var currentMidiInputId = -1;
        var currentMidiOutputId = -1;

        //create a dropdown box for the midi inputs
        var midiInputs = document.createElement('select');
        midiInputs.setAttribute('id', 'midi-in');
        contentDiv.appendChild(midiInputs);
        midiInputs.addEventListener('change', function(e) {
            var device = midiInputs.options[midiInputs.selectedIndex];
            currentMidiInputId = device.id;
            var result = midiBridge.addConnection(currentMidiInputId, currentMidiOutputId, [midiBridge.PITCH_BEND]);
            parseResult(result);
        }, false);

        //create a dropdown box for the midi outputs
        var midiOutputs = document.createElement('select');
        midiOutputs.setAttribute('id', 'midi-out');
        contentDiv.appendChild(midiOutputs);
        midiOutputs.addEventListener('change', function(e) {
            var device = midiOutputs.options[midiOutputs.selectedIndex];
            currentMidiOutputId = device.id;
            var result = midiBridge.addConnection(currentMidiInputId, currentMidiOutputId, [midiBridge.PITCH_BEND]);
            parseResult(result);
        }, false);

        var devices;

        midiBridge.init({
            connectAllInputsToFirstOutput : false,

            ready : function(msg) {
                devices = midiBridge.getDevices();
                populateDropDownMenus()
            },

            error : function(msg) {
                contentDiv.innerHTML += msg + '<br/>';
            },

            data : function(midiEvent) {
                contentDiv.innerHTML += midiEvent + '<br/>';
            }

        });

        var populateDropDownMenus = function() {

            midiInputs.appendChild(createOption('-1', 'choose a midi input'));
            midiOutputs.appendChild(createOption('-1', 'choose a midi output'));

            for(var deviceId in devices) {
                var device = devices[deviceId];
                if(device.type === 'input' &amp;&amp; device.available === 'true') {
                    midiInputs.appendChild(createOption(device.id, device.name))
                } else if(device.type === 'output' &amp;&amp; device.available === 'true') {
                    midiOutputs.appendChild(createOption(device.id, device.name))
                }
            }
        }

        var createOption = function(id, label) {
            var option = document.createElement('option');
            option.setAttribute('id', id);
            option.innerHTML = label;
            return option;
        }

        var parseResult = function(data){
            contentDiv.innerHTML += data.msg + ' : ' + currentMidiInputId + ' : ' + currentMidiOutputId + '</br>';
        }


    }, false);


You can check this example <a href="../midibridge/examples/patchpanel.html" target="blank">here</a>.

<a name="removeConnection"></a> `removeConnection(input,output)`

You can remove a connection between a midi input and a midi output by passing their respective ids as arguments to this method.

<a name="disconnectAll"></a> `disconnectAll()`

This method does exactly what you would expect: it disconnects all current connections between midi in- and output and between your midi inputs and the midibridge

<a name="getNoteName"></a> `getNoteName(midiNoteNumber,mode)`

Returns the name of the note based on the midi notenumber. Midi notenumbers are part of the midi standard and range between 0 and 127. The central C (261.626 Hz in 440 pitch) has notenumber 60.

The parameter `mode` can be one of the following values:

*   `midiBridge.NOTE_NAMES_SHARP` : the black keys will be named sharps, so notenumber 61 will be called C#
*   `midiBridge.NOTE_NAMES_FLAT` : the black keys will be named flats, so notenumber 61 will be called D♭
*   `midiBridge.NOTE_NAMES_ENHARMONIC_SHARP` : all keys will be named sharps, so notenumber 60 will be called B#, notenumber 62 will be called C## (double sharp)
*   `midiBridge.NOTE_NAMES_ENHARMONIC_FLAT` : all keys will be named flats, notenumber 64 will be called F♭, notenumber 62 will be called E♭♭ (double flat)

<a name="getNoteNumber"></a> `getNoteNumber(midiName,octave)`

Returns the midi notenumber based on the name of the note and the octave. Octave -1 is the lowest possible octave number, octave 9 the highest possible. A regular 88 key keyboard ranges between A-1 and C7.

<a name="getStatus"></a> `getStatus(statusCode)`

Returns a string representation of the status byte of the midi message. For instance `midiBridge.getStatus(midiBridge.NOTE_ON)` returns "NOTE ON".

<a name="loadBase64String"></a> `loadBase64String(base64String)`

Loads a base64 encoded MIDI file in the sequencer and return an object containing data about the loaded MIDI file.

*   `microseconds`: duration of the MIDI file in microseconds
*   `ticklength`: duration of the MIDI file in ticks
*   `ticks`: number of ticks in the MIDI file

There are several ways of getting a MIDI file as base64 string into your application.

1) By using html5 drag and drop functionality with the File API, <a href='http://abumarkub.net/midibridge/examples/base64.html' title='abumarkub midibridge base64 example' target='blank'>see this example</a>.

2) By using the File API, <a href='http://abumarkub.net/midibridge/examples/playmidifile.html' title='abumarkub midibridge play MIDI file example' target='blank'>see this example</a>.

3) Store the MIDI file as a base64 string in a Javascript variable, <a href='http://abumarkub.net/midibridge/examples/playmidifile.html' title='abumarkub midibridge play MIDI file example' target='blank'>see also this example</a>. The Javascript file <a href='http://abumarkub.net/midibridge/examples/lib/chopin_opus18.mid.js' target='blank' title='abumarkub midibridge chopin midi file base64'>chopin_opus18.mid.js</a> contains a global variable chopin_opus18 that contains the MIDI file as base64 string.

4) Since global variables are bad code practise, loading base64 MIDI files from a webserver is preferred over option 3. Encoding MIDI files on the server can be done with a single line of php code:



    <?php

    $tmpFile = "tmp.mid";

    //store the uploaded MIDI file in a temporarily file file_put_contents(tmpFile, file_get_contents("php://input",r));

    //read the temp file, base64 encode it and echo it back echo base64_encode(file_get_contents($tmpFile));

    ?>



<a name="playBase64String"></a> `playBase64String(base64String)`

Same as `loadBase64String(base64String)` but the file starts playing immediately.

<a name="startSequencer"></a> `startSequencer()`

If a MIDI file is loaded the sequencer starts playing the file at the current position.

<a name="pauseSequencer"></a> `pauseSequencer()`

Pauses the sequencer. Note: pause does not toggle, so you have to call `startSequencer()` to unpause the sequencer.

<a name="stopSequencer"></a> `stopSequencer()`

Stops the sequencer and rewinds sets the position of the file back to the start.

<a name="closeSequencer"></a> `closeSequencer()`

Stops the sequencers and then closes it. You have to call `loadBase64String(base64String)` or `playBase64String(base64String)` to open a sequencer again.

<a name="getSequencerPosition"></a> `getSequencerPosition()`

Gets the position of the MIDI file in the sequencer in microseconds.

<a name="setSequencerPosition"></a> `setSequencerPosition(microseconds)`

Sets the position of the MIDI file in the sequencer in microseconds.

<a name="getNiceTime"></a> `getNiceTime(microseconds)`

Converts microseconds to the time format m:ss:SSS.

Typically used to display the sequencer position of the MIDI file.

<a name="getObject"></a> `getObject(id)`

Returns a reference to the object in the html page whose id is specified. It is used for getting a reference to the applet, but you can also use it for getting a reference to any other type of object. For instance for getting the swf object if your application is built with Flash, see [What about Flash?][9]

<a name="MidiMessage"></a> `MidiMessage(jsonString)`

An internal class of the midibrigde that is used for storage and easy handling of the midi events that arrive from the applet. The applet sends midi events as JSON strings to the midibridge. The midibridge uses the native `JSON.parse()` method to parse this string into a JSON object.

The MidiMessage class has 2 useful methods that you might need in your code `toString()` and `toJSONString()`. The first method is handy for printing the midi incoming events to a log, and the latter can be used if you want to send the midi event to Flash or another technology. See the offical <a href="http://www.json.org/index.html" target="blank" title="abumarkub midibridge js JSON" rel="abumarkub midibridge js JSON">JSON website</a> for more information. Flash programmers might be interested in the <a href="https://github.com/mikechambers/as3corelib" target="blank" title="abumarkub midibridge js JSON" rel="abumarkub midibridge js JSON">JSON library of Mike Chambers</a>.

<a name="statics"></a> **Static members** Besides methods, there are also a bunch of very handy static members that you can use:

*   `midiBridge.version` : version of the midibridge as string
*   `midiBridge.ready` : set to true if the midiBridge has been initialized successfully, otherwise set to false
*   `midiBridge.NOTE_NAMES_SHARP` : "sharp" see [getNoteNumber()][24]
*   `midiBridge.NOTE_NAMES_FLAT` : "flat" see [getNoteNumber()][24]
*   `midiBridge.NOTE_NAMES_ENHARMONIC_SHARP` : "enh-sharp" see [getNoteNumber(][24])
*   `midiBridge.NOTE_NAMES_ENHARMONIC_FLAT` : "enh-flat" see [getNoteNumber()][24]
*   `midiBridge.NOTE_OFF` : 0x80 (128)
*   `midiBridge.NOTE_ON` : 0x90 (144)
*   `midiBridge.POLY_PRESSURE` : 0xA0 (160)
*   `midiBridge.CONTROL_CHANGE` : 0xB0 (176)
*   `midiBridge.PROGRAM_CHANGE` : 0xC0 (192)
*   `midiBridge.CHANNEL_PRESSURE` : 0xD0 (208)
*   `midiBridge.PITCH_BEND` : 0xE0 (224)
*   `midiBridge.SYSTEM_EXCLUSIVE` : 0xF0 (240)

<a name="midi-out"> </a>**About midi out**

If you're on Windows or Linux, the latency of the Java Sound Synthesizer makes it almost impossible to play. On Windows you can also choose the Microsoft GS Wavetable Synth and with some soundcards you may get a decent latency (i was told the Realtek AC97 perfoms pretty well).

On a Mac you can just select a default midi synthesizer (Java Sound Synthesizer) and start playing with no noticeable latency.

Latency is caused by both the drivers of your soundcard and the way your synthesizer works. Most modern softsynths hardly cause any latency, but even with the latest M-Audio pro cards you'll experience latency when using the Java Sound Synthesizer or the Microsoft GS Wavetable Synth.

So we need to be able to connect to some real softsynths like <a href="http://www.pianoteq.com/" target="_blank" rel="abumarkub midibridge java applet command line flash actionscript" title="abumarkub midibridge Pianoteq">Pianoteq</a> or <a href="http://www.applied-acoustics.com/products/" target="_blank" rel="abumarkub midibridge java applet command line flash actionscript" title="abumarkub midibridge Lounge Lizard">Lounge Lizard</a> and for this we need a virtual midi driver.

If you're on a Mac, you're lucky because such a thing is already installed on your machine. It is called IAC Driver and you'll find it if you open the Audio MIDI Setup in your Applications folder.

If you are on Windows you can download LoopBe1 from <a href="http://www.nerds.de" target="_blank">nerds.de</a> and Linux users can check <a href="http://alsa.opensrc.org/index.php/VirMidi" target="_blank">VirMidi</a>

Below i'll give a brief explanation for every driver.

- [LoopBe][38]   
- [IAC][39]   
- [VirMidi][40]

<a name="loopbe"> </a>**LoopBe1**

Download it from <a href="http://nerds.de/en/download.html" target="_blank" rel="abumarkub midibridge java applet command line flash actionscript" title="abumarkub midibridge LoopBe">nerds.de</a> and run the installer. After the installation has finished LoopBe is up and running and will automatically start with Windows (if you don't want this, run msconfig and remove the LoopBe startup service).

Now LoopBe Internal MIDI will be listed as both a midi input as well as a midi output when you call `getDevices()`. You can setup a connection betweein your favorite keyboard as input device and LoopBe Internal MIDI as output with `addConnection()`.

Now open your favorite softsynth and go to the midi settings and set your synth's midi input to LoopBe Internal MIDI. Here is a screendump of what this looks like in Pianteq:

<img src="http://www.abumarkub.net/abublog/wp-content/uploads/2010/02/Pianoteq-LoopBe-Abumarkub-midibridge.jpg" alt="Pianoteq LoopBe Abumarkub midibridge" title="Pianoteq LoopBe Abumarkub midibridge" width="597" height="680" class="alignnone size-full wp-image-193" />

You should now be able to play your softsynth while midi data is passing thru the midibridge, and dependent on your soundcard's driver, with very low latency.

Please notice that LoopBe1 is only free for non-commercial use. For commercial use you need to acquire a license after a 30-day evolution period. But for only € 11,90 inc VAT it's really a bargain. If you are willing to spend an extra 5 euro on top, i would recommend to buy LoopBe1 bigger brother LoopBe30, which gives you up to 30 virtual midi ports! Check <a href="http://nerds.de/en/order.html" target="_blank">here</a>.

<a name="iac"></a> **IAC**

Open your Finder, go to menu Go -> Applications and scroll down till you've found a folder named Utilities. Open the folder Utilities and double click on Audio MIDI Setup. If you only see a window with Audio Devices, go to Window -> Show MIDI Window.

In the window that subsequently opens, you should see an icon named IAC Driver. IAC stands for Inter-Application Communication, and that is exactly what it does.

If the icon is greyed out double click it and check the box “Device is online” in the popup that appears. Now you should have a window like:

<img src="http://www.abumarkub.net/abublog/wp-content/uploads/2010/02/Picture-4.png" alt="IAC-Driver-Abumarkub-midibridge" title="IAC-Driver-Abumarkub-midibridge" width="504" height="534" class="alignnone size-full wp-image-194" />

Don't worry if looks a little different on your machine. You should see at least 1 port in the “Ports” part of the screen. If not, simply click the plus sign to add a port. I recommend to add a least 2 ports to the IAC Driver.

Close this popup and the Audio MIDI Setup. Now "IAC Driver IAC Bus 1" (or something alike) will be listed as midi input when you call `getDevices()`.

Set up a connection between your favorite keyboard as input device and "IAC Driver IAC Bus 1" as output with `addConnection()`. Open your favorite softsynth and go to the midi settings and set your synth's midi input to "IAC Driver IAC Bus 1". Here is a screendump of what this looks like in Lounge Lizard:

<img src="http://www.abumarkub.net/abublog/wp-content/uploads/2010/02/Picture-7.png" alt="Lounge Lizard IAC input Abumarkub midibridge" title="Lounge Lizard IAC input Abumarkub midibridge" width="866" height="482" class="alignnone size-full wp-image-195" />

Now you can play your softsynth while midi data is passing thru the midibridge.

<a name="virmidi"></a> **VirMidi**

If you are using Ubuntu or Kubuntu, there is a thread about VirMidi on the <a href="http://ubuntuforums.org/showthread.php?p=6616182" target="blank">Ubuntu forum</a>

Because snd-virmidi is a kernel module, you can simply load this module by typing `sudo modprobe snd-virmidi` on the command line.

Now if you call `getDevices()`, you should see at least 4 additional devices listed.

Set up a connection between your keyboard as input device and one of the virtual midi ports as output with `addConnection()`. Connect this output to the input of your favorite softsynth, for instance in Pianoteq this would look like:

<img src="http://www.abumarkub.net/abublog/wp-content/uploads/2010/11/VirMidi-Kubuntu-Pianoteq-midibridge-abumarkub.jpg" alt="VirMidi Kubuntu Pianoteq midibridge abumarkub" title="VirMidi Kubuntu Pianoteq midibridge abumarkub" width="599" height="677" class="alignnone size-full wp-image-429" />

Now you can play your softsynth while midi data is passing thru the midibridge.

Pianoteq is available for both 32 and 64 bits Linux, so if you want to try it yourself you can download a demo version <a href="http://pianoteq.com/try" target="_blank">over here</a>.

<a name="earlier-versions"> </a>**Earlier versions**

I started this project in the summer of 2008. Since then i have released 5 versions:

*   <a href="http://abumarkub.net/midibridge/v1/fp10/" target="blank">proof of concept</a> : With dynamical sound generation, a chord finder, a color organ, midi learn functionality, a virtual keyboard and an adjustable pitch bend controller that acts upon the generated sound
*   <a href="http://abumarkub.net/midibridge/v2/fp10/" target="blank">version 2</a> : With all features of the proof of concept, but with an extra swf that sits between the applet and the application swf. This extra swf connects on one site via ExternalInternface to the applet, and on the other side via LocalConnection to the application swf.
*   <a href="http://abumarkub.net/midibridge/v3/" target="blank">version 3</a> : The in-between swf removed again, dynamical sound generation replaced by midi out. A very simple GUI: virtual keyboard, chord finder and pich bend controller removed, simple animation added in.
*   <a href="http://abumarkub.net/midibridge/v4/" target="blank">version 4</a> : FluidSynth softsynth added that allows you to use Soundfont files at choice. Also the midibridge is able to generate midi events itself. Virtual keyboard added again
*   <a href="http://abumarkub.net/abublog/?p=381" target="blank">version 5</a> : Same as version 4 but you can also export the code to a standalone AIR version.

In my blogposts you can still find information about the earlier versions. This might be confusing and therefor i have started this page where you can find only up to date information about the latest, and thus featured release. Some of this information can also be found in various posts, but if it appears here as well it is still valid for the current release.

With this version, the development of all earlier versions of the midibridge will be frozen. The reason for this is that with this new version i have redefined what the midibridge exactly is.

In former version of the midibridge i have added too much GUI functionality. As explained in the [introduction][4], the new midibridge provides only a compact set of methods that allows you to interact transparently with the midi devices on your computer, but leaves the GUI totally up to you.

So therefor the new version has no control panel, virtual keyboard, midi learn functionality and so on. I have provided code examples for the basic features of the midibridge and i will add some more examples for more advanced features like sound generation soon. Also i might develop some configurable UI plugins for the midibridge (alike jQuery UI) somewhere in the future.

Another reason for stripping down the midibridge to its core is that former versions were too much tied to Flash and Actionscript. In modern browsers Flash is no longer the only reasonable choice for creating compelling animations. This applies to dynamically generating sound as well.

What's more, the latency of <a href="https://wiki.mozilla.org/Audio_Data_API" target="blank" title="abumarkub midibridge mozilla audio data API" rel="abumarkub midibridge mozilla audio data API">Mozilla's Audio Data API</a> allows you to set the minimal audio buffer as low as 512 samples, which results in a latency of only 512/44100 ≈ 11.6 ms(!). In Flash the minimal buffer size is 2048 (recommended) which results in an almost un-playable latency of 2048/44100 ≈ 46ms.

To summarize the benefits of the new approach:

*   the midibridge now only takes 5K of code
*   it makes it much easier to add the midibridge to your code
*   it gives you more control over what your website/application looks like
*   it does not impose a specific client side language on you

The only downside is that the new version is not compatible with the earlier versions. However, you can still use it; the code is fully functional and remains available at GitHub and Google Code. You are encouraged to switch to the new version though.

Code at GitHub (version 5):

<a href="http://github.com/abudaan/javamidi" rel="abumarkub midibridge java applet command line flash" title="abumarkub midibridge Java classes" target="blank">http://github.com/abudaan/javamidi</a>

<a href="http://github.com/abudaan/flashmidi" rel="abumarkub midibridge java applet command line flash" title="abumarkub midibridge Actionscript classes" target="blank">http://github.com/abudaan/flashmidi</a>

Code at Google Code (version 5):

<a href="http://code.google.com/p/miditoflash/downloads/" rel="abumarkub midibridge java applet command line flash" title="abumarkub midibridge Actionscript and Java classes" target="blank">http://code.google.com/p/miditoflash/downloads/</a>

As i mentioned above, the earlier version had both a web and an AIR version. The Air version uses <a href="http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/desktop/NativeProcess.html" target="blank" rel="abumarkub midibridge actionscript java air" title="Air 2.0 NativeProcess">NativeProcess</a> to start a Java program on the commandline, and communicates with this program via its standard input and standard output.

I am not yet sure what to do with the AIR version. You can use both Actionscript and Javascript in an AIR app, but i am actually looking for another way of creating a browser-less application. Any suggestions are welcome.

<a name="flash"></a> **What about Flash?**

The midibridge is fully accessible from Actionscript 3.0 if you use the <a href="http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/external/ExternalInterface.html" target="blank" title="abumarkub midibridge javascript flash" rel="abumarkub midibridge javascript flash">ExternalInterface</a>.

You probably want to load Flash before you initialize the midibridge so you can show some loading animation. In Actionscript you have to test when the page has fully loaded and then call the `init()` function of the midibridge.

First a Timer is set up to check if the midibridge object is `null`. As soon as the midibridge object exists, Flash creates callback handlers for Javascript: these are methods that can be called directly from Javascript.

Flash also calls the global Javascript method `callFromFlash()`. You can name it anything you like btw. The first parameter determines what action of the midibridge is requested:



    package {

        import flash.display.Sprite;
        import flash.events.TimerEvent;
        import flash.external.ExternalInterface;
        import flash.utils.Timer;


        public class Main extends Sprite {
            private var _readyTimer:Timer = new Timer(1, 1);

            public function Main() {
                if(ExternalInterface.available) {
                    _readyTimer.addEventListener(TimerEvent.TIMER, check);
                    _readyTimer.start();
                } else {
                    trace('ExternalInterface not avaible in this browser');
                }
            }

            public function midibridgeReady(msg:String):void {
                var jsonString:String = ExternalInterface.call('callFromFlash', 'getDevices');
                trace(jsonString);
            }

            public function midibridgeError(msg:String):void {
                trace(msg);
            }

            public function midibridgeData(msg:String):void {
                trace(msg);
            }

            private function check(e:TimerEvent = null):void {
                try {
                    if(ExternalInterface.call('midiBridge') !== null) {
                        ExternalInterface.addCallback('midibridgeReady', midibridgeReady);
                        ExternalInterface.addCallback('midibridgeError', midibridgeError);
                        ExternalInterface.addCallback('midibridgeData', midibridgeData);
                        ExternalInterface.call('callFromFlash', 'start');
                        _readyTimer.stop();
                    } else {
                        _readyTimer = new Timer(100, 1);
                        _readyTimer.addEventListener(TimerEvent.TIMER, check);
                        _readyTimer.start();
                    }
                } catch(err1:SecurityError) {
                    trace(err1.message);
                } catch(err2:Error) {
                    trace(err2.message);
                }
            }
        }


    }



Now back in Javascript, the call to `callFromFlash()` gets processed. The first parameter was "start" so the midibridge gets initialized. As you can see, the callback handlers of the midibridge get directly connected to the Javascript callback handlers of Flash. This way data and messages are routed from the midibridge to Flash:




    /**
     * You can only call global functions from Flash, therefor we declare a single global function that is used for all
     * communication with the Flashplayer.
     *
     */

     function callFromFlash() {

        /**
         * getObject is utility function of midiBridge, it is used to get the Applet object,
         * but it can also be used to get the swf object
         *
         * flashapp is the id of the swf object in the html page.
         */
        var flashObject = midiBridge.getObject('flashapp');

        /**
         * convert the arguments to a array, the first argument is the msgId.
         * the msgId is used to determine what the Flashplayer wants from the midibridge
         */
        var args = Array.prototype.slice.call(arguments);
        var msgId = args[0];


        switch(msgId) {

            case 'start':
                midiBridge.init({
                    connectAllInputsToFirstOutput : true,

                    ready : function(msg) {
                        flashObject.midibridgeReady(msg);
                        return msg;
                    },

                    error : function(msg) {
                        flashObject.midibridgeError(msg);
                    },

                    data : function(midiEvent) {
                        flashObject.midibridgeData(midiEvent.toString());
                    }

                });
                break;

            case 'getDevices':
                return midiBridge.getDevices();
                break;
        }


    };





You can check this example <a href="../midibridge/examples/flash.html" target="blank">here</a>.

<a name="known-issues"></a> **Known issues**

Currently the midibridge does not work with the Icedtea Java plugin on Linux.

<a name="roadmap"></a> **Forthcoming features**

In a future release i will add more functionality to the sequencer. The following methods will be implemented.

getSequencerTickPosition(); setSequencerTickPosition(ticks); recordMidi(file); setTempo(tempo);

For more feature requests or other suggestions, please drop me a line!

 [1]: http://abumarkub.net/abublog/?p=963
 [2]: http://www.abumarkub.net/abublog/?p=840
 [3]: http://jazz-soft.net/
 [4]: #introduction
 [5]: #quick-start
 [6]: #documentation
 [7]: #midi-out
 [8]: #earlier-versions
 [9]: #flash
 [10]: #known-issues
 [11]: #roadmap
 [12]: #init
 [13]: #sendMidiEvent
 [14]: #getDevices
 [15]: #refreshDevices
 [16]: #connectAllInputs
 [17]: #connectFirstInput
 [18]: #connectFirstOutput
 [19]: #connectAllInputsToFirstOutput
 [20]: #addConnection
 [21]: #removeConnection
 [22]: #disconnectAll
 [23]: #getNoteName
 [24]: #getNoteNumber
 [25]: #getStatus
 [26]: #loadBase64String
 [27]: #playBase64String
 [28]: #startSequencer
 [29]: #pauseSequencer
 [30]: #stopSequencer
 [31]: #closeSequencer
 [32]: #getSequencerPosition
 [33]: #setSequencerPosition
 [34]: #getNiceTime
 [35]: #getObject
 [36]: #MidiMessage
 [37]: #statics
 [38]: #loopbe
 [39]: #iac
 [40]: #virmidi