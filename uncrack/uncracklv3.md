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

# Bypass rootcheck

![image](https://github.com/user-attachments/assets/d41e3ef9-996c-4813-bcf6-07c3a48a65e6)
Tại ``MainActivity`` có dùng class để check root.
Truy cập class ``RootDetection`` để xem bên trong có gì.

![image](https://github.com/user-attachments/assets/9ae85e3d-ece5-4bea-909f-5f9cb542cd99)
Phần này ta thấy có 3 phương thức để checkroot đó là 1,2,3 việc bypass đơn giản chỉ là tự implement lại 3 phương thức này và ``return false`` là xong.
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
```
# Bypass fridacheck
