---

# **Lab:1 Write-Up: Lab: Modifying serialized objects**  

## **Lab Summary**  
In this lab, we exploit **insecure deserialization** in PHP to escalate privileges and delete a user account. The session cookie contains a **serialized PHP object**, which we modify to gain admin access.  

---

## **Steps to Exploit the Vulnerability**  

### **Step 1: Log in and Identify the Session Cookie**  
1. Log in using your own credentials.  
2. Observe the `GET /my-account` request, which includes a **session cookie** in the request headers.  
3. The session cookie appears to be **Base64-encoded** and **URL-encoded**.  

---

### **Step 2: Decode the Session Cookie**  
1. Use **Burp Suite's Inspector panel** to **decode** the session cookie.  
2. Upon decoding, you will see that the cookie contains a **serialized PHP object** similar to:  
   ```
   O:4:"User":1:{s:5:"admin";b:0;}
   ```
   - `O:4:"User"` → This means an object of class `User` with 4 characters in the name.  
   - `s:5:"admin";b:0;` → This indicates the **admin attribute** is set to `false` (`b:0`).  
![Screenshot from 2025-02-04 18-57-00](https://github.com/user-attachments/assets/f1a0e751-201e-483f-b0da-a5987d873d21)

---

### **Step 3: Modify the Serialized Object to Gain Admin Access**  
1. Send the request to **Burp Repeater**.  
2. In Burp Repeater, open the **Inspector panel** and modify the `admin` attribute:  
   ```
   O:4:"User":1:{s:5:"admin";b:1;}
   ```
![Screenshot from 2025-02-04 18-58-58](https://github.com/user-attachments/assets/73093e7f-b9fe-488e-bced-7dec9daab663)

   - Changing `b:0` → `b:1` sets **admin privileges to true**.  
3. Click **Apply Changes**.  
   - Burp Suite will automatically **re-encode** the object into Base64 and URL encoding.  

---

### **Step 4: Send the Modified Request**  
1. Send the modified request with the **updated session cookie**.  
2. Check the response.  
   - If successful, you should now see a **link to the admin panel (`/admin`)**, confirming that you have admin access.  

---

### **Step 5: Access the Admin Panel**  
1. Change the **request path** to:  
   ```
   GET /admin HTTP/1.1
   ```
2. Send the request.  
3. Observe that the `/admin` page contains options to **delete user accounts**.  

---

### **Step 6: Delete the User "Carlos"**  
1. Change the request path to:  
   ```
   GET /admin/delete?username=carlos HTTP/1.1
   ```
2. Send the request.  
3. If successful, **Carlos' account is deleted**, and you have solved the lab!
![Screenshot from 2025-02-04 19-00-21](https://github.com/user-attachments/assets/0dde111d-b487-4bf9-96b2-c82fbc736bf0)

----
# **Lab:2 Modifying serialized data types**  

## **Lab Summary**  
This lab demonstrates how **insecure deserialization** in PHP can be exploited to gain **admin privileges** by modifying serialized session data. By changing a username field and altering the data type of an access token, we escalate our privileges and gain access to the admin panel.  

---

## **Steps to Exploit the Vulnerability**  

### **Step 1: Log in and Identify the Session Cookie**  
1. Log in using your own credentials.  
2. In **Burp Suite**, open the **GET /my-account** request after logging in.  
3. Locate the **session cookie** in the request headers.  
4. Use **Burp Inspector** to **decode** the session cookie.  

#### **Decoded PHP Serialized Object (Example Before Modification)**  
```
O:4:"User":2:{s:8:"username";s:4:"user";s:12:"access_token";s:10:"random1234";}
```
- `O:4:"User"` → Object of class `User`.  
- `s:8:"username";s:4:"user";` → Username field (4-character string "user").  
- `s:12:"access_token";s:10:"random1234";` → Access token (10-character string "random1234").  
![Screenshot from 2025-02-04 19-34-38](https://github.com/user-attachments/assets/9177b793-aeef-4a45-bfa8-f67ef6babba2)

---

### **Step 2: Modify the Session Cookie to Escalate Privileges**  
1. **Send the request to Burp Repeater.**  
2. **Modify the session cookie using Burp's Inspector panel:**  
   - **Change the username to `"administrator"`** and update its length to **13** characters.  
   - **Change the access token to an integer (`0`)** instead of a string:  
     - **Remove the double quotes** around the value.  
     - **Update the type label from `s` (string) to `i` (integer).**  

#### **Modified Serialized Object**  
```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```
- `s:13:"administrator";` → Username is now **"administrator"** (13-character string).  
- `s:12:"access_token";i:0;` → Access token is now **integer 0** (not a string).  
---

### **Step 3: Apply Changes and Send the Request**  
1. Click **"Apply changes"** in Burp.  
   - Burp will **re-encode** the object automatically.  
2. **Send the modified request.**  
3. Check the response.  
   - If successful, the response should now include a **link to the admin panel (`/admin`)**, confirming that you have **admin access**.  

---

### **Step 4: Access the Admin Panel**  
1. Change the **request path** to:  
   ```
   GET /admin HTTP/1.1
   ```
2. Send the request.  
3. The **admin panel** should now be accessible.  
4. The page contains **links to delete specific user accounts**.  

---

### **Step 5: Delete the User "Carlos"**  
1. Change the request path to:  
   ```
   GET /admin/delete?username=carlos HTTP/1.1
   ```
2. Send the request.  
3. If successful, **Carlos' account is deleted**, and the lab is solved!
![Screenshot from 2025-02-04 19-35-36](https://github.com/user-attachments/assets/6836b957-da88-4b8f-a295-4bf80ee974b0)

----

# **Lab:3  Write-Up: Using application functionality to exploit insecure deserialization**  

## **Lab Summary**  
This lab demonstrates how **insecure deserialization** can be exploited to delete **arbitrary files** on the server. The web application stores user session data as a **serialized PHP object**, which includes an **avatar_link** attribute. By modifying this attribute to point to a sensitive file (`/home/carlos/morale.txt`), we can **leverage the account deletion functionality** to delete the targeted file.  

---

## **Steps to Exploit the Vulnerability**  

### **Step 1: Log in and Identify the Session Cookie**  
1. **Log in** to your account.  
2. Navigate to the **"My Account"** page.  
3. Notice the option to **delete your account** by sending a `POST` request to:  
   ```
   POST /my-account/delete HTTP/1.1
   ```
4. **Send the request to Burp Repeater** for further analysis.  

---

### **Step 2: Identify and Modify the Serialized Object**  
1. In **Burp Suite**, examine the **session cookie** in the request headers.  
2. Use **Burp Inspector** to **decode the session cookie** and reveal a **PHP serialized object**.  
3. Locate the **`avatar_link` attribute**, which stores the **file path to your avatar**.  

#### **Example of Decoded Serialized Object Before Modification:**  
```
O:4:"User":2:{s:8:"username";s:4:"user";s:11:"avatar_link";s:20:"/avatars/user123.jpg";}
```
- `s:8:"username";s:4:"user";` → Username is `"user"`.  
- `s:11:"avatar_link";s:20:"/avatars/user123.jpg";` → Avatar stored at `/avatars/user123.jpg`.  
![Screenshot from 2025-02-05 10-08-50](https://github.com/user-attachments/assets/b005e598-b9bc-420c-806b-354cb5bac546)

---

### **Step 3: Modify the Serialized Data to Target Carlos’s File**  
1. **Change the `avatar_link` attribute** to point to **Carlos’s morale.txt file**:  
   ```
   s:11:"avatar_link";s:23:"/home/carlos/morale.txt";
   ```
   - `s:23` → Indicates the new file path is **23 characters long**.  
2. **Update the serialized object accordingly**.  

#### **Modified Serialized Object:**  
```
O:4:"User":2:{s:8:"username";s:4:"user";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```
![Screenshot from 2025-02-05 10-08-58](https://github.com/user-attachments/assets/10dce3e0-2d80-47e9-9d99-4d2e6979b967)

3. Click **"Apply changes"** in Burp.  
   - Burp will **re-encode** the object automatically.  

---

### **Step 4: Send the Modified Request to Delete the File**  
1. Change the **request method** to:  
   ```
   POST /my-account/delete HTTP/1.1
   ```
   ![Screenshot from 2025-02-05 11-00-17](https://github.com/user-attachments/assets/178bc4da-4eba-408b-80e1-4e9ba2b721fd)

2. **Send the request** with the modified session cookie.  
3. Since the web application **automatically deletes the file stored in `avatar_link`** when deleting a user, it will delete **Carlos's morale.txt file** instead of the avatar.  
4. The **lab is solved!**  
![Screenshot from 2025-02-05 10-59-58](https://github.com/user-attachments/assets/c8aceaf3-6bec-40c3-9ee4-d85ff27d5212)

---

