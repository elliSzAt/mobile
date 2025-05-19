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

Quay lại với việc phân tích apk.
![image](https://github.com/user-attachments/assets/644135d5-fd19-4b4b-bb64-adcd7ceb567f)

Ngay đầu của ``MainActivity`` thì ta thấy được 1 ``xorkey`` được hardcode.
![image](https://github.com/user-attachments/assets/d0eb0304-81a3-42ed-bf78-42f294bd76fb)

Tiếp theo biến này được đưa vào trong hàm ``init`` của ``libfoo.so``.
![image](https://github.com/user-attachments/assets/d5a26c80-e958-4cd0-a9cb-7f6c2b8be2d3)

Key được đưa vào như biến ``*v5`` và sau đó là thực hiện cả đống phép tính gì đó rồi copy giá trị ``*v5`` vào ``qword_15038``.
Có thể nó để dành để làm việc gì đó sau này, nên tạm thời cứ để đó.
![image](https://github.com/user-attachments/assets/0fd26cf9-dee4-487a-a331-34f799675514)

Tiếp tục kiểm tra apk thì thấy có hàm ``check.code``.
![image](https://github.com/user-attachments/assets/57e7aad4-a52c-4192-b333-421286492fbf)

Vào xem thì tiếp tục nó lại gọi đến ``CodeCheck_bar`` trong thư viện C.
![image](https://github.com/user-attachments/assets/8662a01e-89be-48f0-ba85-b380fe07d7d4)

Tiếp theo nó sẽ đi đến hàm ``sub_10E0`` để kiểm tra gì đó.
![image](https://github.com/user-attachments/assets/dbaf48ad-f4d4-4c78-9034-f6158dca60ac)

Kéo xuống cuối cùng thì thấy có chuỗi hex khả nghi, nhưng các phép tính bên trên hình như không đầy đủ thì phải.
Nên mình tìm kiếm chuỗi ``0x14130817005A0E08`` để xem rõ hơn các kí tự còn lại.
![image](https://github.com/user-attachments/assets/1b5dbb46-216d-48ed-916c-6704bc2a58e7)

Và đây rồi, nó xuất hiện đầy đủ ở đây, bây giờ ta sẽ xor chuỗi hex này với key ban đầu.
![image](https://github.com/user-attachments/assets/7fbe3afe-41f7-4351-a37c-47f2a1685881)

Tìm được chuỗi ``making owasp great again``, và đây chính là flag hehe ~ ~
