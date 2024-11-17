# 1. ezmobile

Đầu tiên dùng ``jadx`` để decompile đọc src code của file apk.

![alt text](1_paBklCJGI0Txe5QnaDUWsw.webp)

Ở đây tại hàm main thì ta thấy có các hàm khác, đáng chú ý ở đây chính là hàm ``FlagCheck``. Hàm này sẽ so sánh chuỗi nhập vào của mình có giống với flag sau khi ``decode`` hay không, sau đó sẽ xuất ra đoạn text ``Congrats.............``.

![alt text](1_QJPGf0HeIrRe4l2eTiluuw.webp)

Flag sẽ được gọi từ ``strings.xml`` nằm ở thư mục ``resources``.

![alt text](1_6HXn74ZhPYWE8kOG3dgqtA.webp)

Decode base64 thì sẽ thu được đoạn flag.

```
FLAG: PWNSEC{w3lp_n07h!ng_Sp3Ci4l_Just_4_Fl4g_!n_7h3_s7r!ng5_xml_f!l3}
```

# 2. FireInTheHole

Thử thách này dựa vào các kĩ thuật ``firebase misconfiguration và ES decryption`` đối với file trong thư mục ``assets``, database không có giới hạn nào và có thể dễ dàng truy cập vào đó.

Bắt đầu decompile app với jadx và tiến hành phân tích ở file ``Manifest``.

![alt text](1_4zfPRdtr2cVraTt-YS5Ayw.webp)

Từ file ``Manifest`` ta có thể thấy được có 2 ``activity``. Tuy nhiên ở ``MainActivity`` không có gì đặc sắc nên ta sẽ tập trung vào ``Decrypter``.

![alt text](1_vkBPJ2xnTh1le5Ecr_d7hQ.webp)

Activity này có hàm ``getAssets`` để mở file ``one.txt`` ở thư mục ``assets``, nó cũng thực thi việc lấy ``key`` và ``IV``. Sau khi decrypt nội dung file ``one.txt`` thì tiếp tục sẽ decode ``key`` và ``IV``.

![alt text](1_C_Ol6KPgqxUl5PwEtm5IGw.webp)

Hàm ``decrypt`` thực hiện thuật toán mã hóa ``AES`` đối với file ``one.txt``. Tuy nhiên vấn đề là mình không tiếp thấy ``key`` và ``IV`` trong file apk, kiểm tra lại trong ``strings.xml``.

![alt text](1_nFUFiM9V8RRjwD2wY9PjMg.webp)

Tại đây mình thấy có 1 đường dẫn đi đến ``firebase database``, kiểm tra xem bên trong đó có gì.

![alt text](1_0gfztQ_9ct-PBkW1jgbHtg.webp)

Trong ``firebase`` có một số ``misconfiguration`` phổ biến bằng cách truy cập vào path ``/.json``, tại đây mình đã tìm thấy ``key`` và ``IV`` phục vụ cho thuật toán ``AES`` đã nêu ở phía trên.

![alt text](1_PCfJFS_809uU7Ei8TZ8jGg.webp)

```
FLAG: PWNSEC{Y0uR_F!r3_L4ck5_d!sciplin3}
```

# 3. FireStorm

Ở challenge này thì mình sẽ dùng ``frida`` hook vào hàm để lấy mật khẩu, sau đó dùng mật khẩu này cùng với email ở ``strings.xml`` để đăng nhập vào ``firebase``.

![alt text](1_kvC_hTa7kLyoKyTYF5NkOA.webp)

Hàm ``Password`` sẽ lấy các chuỗi từ ``strings.xml`` và từ các chuỗi này sẽ được chuyển vào ``generateRandomStrings``, hàm này được gọi lên từ native library ``firestorm``. Tuy nhiên hàm ``Password`` chưa được gọi, vì vậy mình cần dùng frida hook vào hàm để lấy giá trị của ``password``.

``` java
Java.perform(function() {
    function getPassword() {
        Java.choose('com.pwnsec.firestorm.MainActivity', {
            onMatch: function(instance) {
                console.log("MainActivity instance found: " + instance);
                try {
                    var pass = instance.Password();
                    console.log("FireBase Password: " + pass);
                } catch (e) {
                    console.log("Error occurred: " + e);
                }
            },
            onComplete: function() {
                console.log("Search completed. Exiting script.");

            }
        });
    }

    // Delay execution to ensure the app is fully started
    setTimeout(getPassword, 4000); // Adjust the delay as needed (4000 ms = 4 seconds)
});
```

Khi chạy script trên, nó sẽ hook vào hàm ``Password`` và thực thi sau đó xuất ra kết quả là chuỗi ``password``.

![alt text](1_FDEjZCugY42-MYXpdwOmaA.webp)

Bây giờ đã có ``password``, tiếp theo mình sẽ dùng các thông tin cấu hình ``firebase`` có trong ``strings.xml`` để đăng nhập ``firebase database``.

![alt text](1_tM0Lvpx9QSUxnp07H87x_Q.webp)

Dựa vào đây là đã có đủ thông tin về ``firebase``, cuối cùng chỉ cần đăng nhập và lấy flag.

``` python
import pyrebase
import requests


config = {
  "apiKey": "AIzaSyAXsK0qsx4RuLSA9C8IPSWd0eQ67HVHuJY",
  "authDomain": "firestorm-9d3db.firebaseapp.com",
  "databaseURL": "https://firestorm-9d3db-default-rtdb.firebaseio.com",
  "storageBucket": "firestorm-9d3db.appspot.com",
  "projectId": "firestorm-9d3db"
}


firebase = pyrebase.initialize_app(config)


auth = firebase.auth()
email = "TK757567@pwnsec.xyz"
password = "C7_dotpsC7t7f_._In_i.IdttpaofoaIIdIdnndIfC"
user = auth.sign_in_with_email_and_password(email, password)

db = firebase.database()

#FLAG will be printed 
print(db.get(user['idToken']).val()) 
```

```
FLAG: PWNSEC{C0ngr4ts_Th4t_w45_4N_345y_P4$$w0rd_t0_G3t!!!_0R_!5_!t???}
```

# 4. Snake

Challenge này thì sử dụng nhiều cơ chế bảo vệ hơn như ``rootCheck`` và ``fridaCheck``. Tuy nhiên có 1 ý tưởng khá đột phá của anh bạn pháp sư Trung Hoa là dùng ``frida`` để bypass ``fridaCheck`` =)))))))))). Challenge này là về ``Deserialization attack`` bằng cách dùng file ``yml`` mà ứng dụng đọc từ ``external storage`` để gọi và thực thi class mà gen ra flag.

![alt text](1_ESJmNxVkl8xmgxhYtncVZg.webp)

Dùng jadx để decompile thì thấy được là ứng dụng có cơ chế kiểm tra ``root``. Ở đây thì mình dùng các script bypass root có sẵn ở ``codeshare`` hay ``patch`` app lại đều được.

![alt text](1_lRFaMr8-VypzzDxSPgW0Qw.webp)

Sau khi bypass root thì ứng dụng sẽ hỏi về quyền truy cập external storage.

![alt text](1_hnQXl5Hv5T6vVl0tFrvS0w.webp)

Phân tích ứng dụng thì ta thấy ở đây có định nghĩa 1 ``intent`` và dùng nó để nhận 1 ``extra strings`` là ``SNAKE`` với giá trị là ``BigBoss``. Nếu điều kiện hợp lệ thì sẽ tìm trong thư mục ``snake`` và kiểm tra xem ``Skull_Face.yml`` có tồn tại ở ``external storage`` hay không, thì nó sẽ load nội dung của file và xuất ra ``logs`` của thiết bị.

![alt text](1_HZfSHWAVqeSGd_PTdSeCQg.webp)

Tìm kiếm trong file apk thì mình thấy có 1 class là ``BigBoss``, tại đây nó load đến thư viện ``snake`` và gọi đến hàm ``stringFromJNI``. Trong class đó còn có 1 hàm ``BigBoss`` sẽ nhận vào 1 chuỗi và chuyển chuỗi đó vào hàm ```stringFromJNI```. Nếu chuỗi string vào là ``Snaaaaaaaaaaaaaake`` thì nó sẽ xuất ra flag ở trong ``logs``.

Nhưng vấn đề chính ở đây là làm sao để gọi được hàm này, do có cơ chế ``fridaCheck``.

Ta nhìn vào file yaml, nó được load bởi thư viện ``snakeyaml`` và nó sử dụng phiên bản có chưa lỗ hổng ``Deserialization``.

[CVE-2022-1471](https://www.greynoise.io/blog/cve-2022-1471-snakeyaml-deserialization-deep-dive?source=post_page-----1b373affd69b--------------------------------)

Do đó mình cần tạo payload trong ``skull_Face.yml``, nó sẽ gọi đến class ``BigBoss`` và chuyển chuỗi ``Snaaaaaaaaaaaaaake`` vào.

Tiếp theo mình cần tạo thư mục ``snake`` ở ``external storage`` với file ``skull_Face.yml`` và payload bên trong nó là:

```
!!com.pwnsec.snake.BigBoss ["Snaaaaaaaaaaaaaake"]
```

Cuối cùng dùng adb để gọi ``MainActivity`` với extra string ``SNAKE`` có giá trị ``BigBoss``.

```
adb shell am start -n com.pwnsec.snake/.MainActivity -e SNAKE BigBoss
```

Kiểm tra ``logcat`` vì như đã nói flag sẽ được xuất ra ở trong phần ``logs`` của thiết bị.

![alt text](1_nozT8wiZpa2XFcNdOFUTKg.webp)

```
FLAG: PWNSEC{W3'r3_N0t_T00l5_0f_The_g0v3rnm3n7_0R_4ny0n3_3ls3}
```