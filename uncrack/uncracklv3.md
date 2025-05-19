```javascript
Java.perform(function () {
    // Hook RootDetection
    let RootDetection = Java.use("sg.vantagepoint.util.RootDetection");

    RootDetection["checkRoot1"].implementation = function () {
        console.log(`RootDetection.checkRoot1 is called`);
        return false;
    };

    RootDetection["checkRoot2"].implementation = function () {
        console.log(`RootDetection.checkRoot2 is called`);
        return false;
    };

    RootDetection["checkRoot3"].implementation = function () {
        console.log(`RootDetection.checkRoot3 is called`);
        return false;
    };
});

var frida_bypass = Module.findExportByName(null, "strstr");
if (frida_bypass) {
    Interceptor.attach(frida_bypass, {
        onEnter: function (args) {
            try {
                var str = Memory.readCString(args[0]);
                if (str && str.indexOf("frida") >= 0) {
                    console.log("[+] Frida detected, bypassing...");
                }
            } catch (e) {
                console.log("[-] Error reading string in strstr: " + e);
            }
        },
        onLeave: function (retval) {
            retval.replace(ptr(0));
        }
    });
    console.log("[+] Successfully hooked strstr");
} else {
    console.log("[-] strstr not found!");
}
```
