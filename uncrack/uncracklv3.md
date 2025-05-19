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

![image](https://github.com/user-attachments/assets/29427e90-37ce-4cf5-95fc-a081fdd8dc95)
Tuy nhiên sau khi bypass rootcheck thì ứng dụng vẫn bị crash, kiểm tra logcat thì thấy có hàm ``goodbye()`` gì đó được gọi từ ``libfoo.so``.
Vậy thì phải mở IDA lên decompile lib đó về để đọc thôi.

![image](https://github.com/user-attachments/assets/4e31a200-cbcf-47ef-b76d-218653006d0b)
Hàm nó chỉ đơn giản là thoát ra khỏi ứng dụng thôi, vậy thì điều kiện gì để nó được gọi hoặc nó được gọi từ 1 hàm nào khác.
Do vậy nên mình sẽ mò mẫm xung quanh xem nó được gọi từ đâu và cần điều kiện gì.

![image](https://github.com/user-attachments/assets/213965df-c4e6-47e8-a4d1-eb6e0c358e75)
Hàm này sẽ được gọi khi khởi động ứng dụng, đại loại là nó sẽ check xem chúng ta có sử dụng ``frida`` không, nếu có thì sẽ crash ngay.
Vậy thì bypass nó bằng cách nào??

![image](https://github.com/user-attachments/assets/c0a4f84b-5ce7-41d7-b1a0-ea80d3431473)
Ở đây mình thấy nó dùng hàm ``strstr`` để so sánh chuỗi với nhau, và nếu không thỏa mãn thì nó sẽ crash ứng dụng.
Do đó mình sẽ dùng frida để hook vào đó và return mọi kết quả trả về là true
```javascript
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
Khi khởi động ứng dụng thì frida sẽ hook vào hàm ``strstr``, ``onEnter`` là khi khởi động ứng dụng, nó sẽ check xem có chuỗi ``frida`` hay không.
Cuối cùng là ``onLeave`` sau khi hàm ``strstr`` kết thúc, nó sẽ sửa đổi giá trị kết quả của hàm sao cho kết quả trả về luôn đúng.

![image](https://github.com/user-attachments/assets/9b796d84-4707-42de-847a-badcfd7c7eb5)
