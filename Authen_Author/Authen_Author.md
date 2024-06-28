# Authentication and Authorization

![alt text](image.png)

### Authentication

Simply, ``authentication`` is the process of proving some information is true.

``Authentication`` is a process of proving a user's identity by providing their ``credentials`` which explains who you are to the system.

![alt text](1_jeDPVfFFHwlnPbOOKzUzSw.gif)

This is how ``authentication`` works.

![alt text](image-1.png)

### Authorization

``Authorization`` is the process of giving someone access to a system or information.

``Authorization`` is a security mechanism to determine access levels or user privileges related to system resources like files,  data, and application features. This is the process of granting or denying access to a network resource that allows the user access to various resources based on the user’s identity.

### Authentication and Authorization together

``Authentication`` is an important part of an application and with ``authorization``, we have more control over the users using the application. When users register in the application, they are informing the server that they are new members. The next time users provide the information (``credentials``), which provided is verified by the server and if the information is valid (``authentication``), they are allowed to enter the application. After entering the appli cation, the server can have control of what resources are accessible by the users, which handled by ``authorization``.

# Commonly used authentication types

### Single Factor Authentication

Single-Factor ``Authentication`` is the most common form of ``authentication``, it requires one factor to gain full access to the system. It could be a username and password or PIN.

### Two-Factor Authentication

Two-factor ``Authentication`` is the ``authentication`` method that is slowly gaining popularity in the world. After ``authentication`` (credentials), the client is asked to provide another information (``OTP``) for validating the client.

![alt text](image-2.png)

Factor 1: 

```python
if username == creds["username"] and password == creds["password"]:
    return redirect(url_for("login_2fa"))
```

Factor 2:

```python
@app.route("/login/2fa/", methods=["POST"])
def login_2fa_form():    secret = request.form.get("secret")
    otp = int(request.form.get("otp"))

    if pyotp.TOTP(secret).verify(otp):
        flash("The OTP is valid", "success")
        return redirect(url_for("login_2fa"))
    else:
        flash("Invalid OTP", "danger")
        return redirect(url_for("login_2fa"))
```

### SSO (Single-Sign-On)

![alt text](image-3.png)

# Commonly used authorization types

### API Keys

The API key is a long string that is typically included in the request URL or header. The API key is used to identify the person who is requesting the API call.

```
curl -X POST https://language.googleapis.com/v1/documents:analyzeEntities?key=API_KEY
```

### Basic Auth

In Basic Auth, the sender inserts encode username and password into the request header.

```
GET / HTTP/1.1 
Host: example.com 
Authorization: Basic ZGFuaWVsOnBhc3N3b3Jk
```

### OAuth

![alt text](image-4.png)

This technology to let users to exchange information about their accounts with third-party applications or websites. OAuth is a protocol that allows an authorization server to provide access tokens to third-party clients with the permission of the resource owner.

