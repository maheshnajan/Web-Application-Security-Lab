---
#  Lab:1 Unprotected admin functionality
## Write-up: Accessing the Admin Panel via robots.txt  

### Step 1: Discovering the Admin Panel Path  
1. Open the lab URL in your browser.  
2. Append `/robots.txt` to the URL (e.g., `https://lab-url.com/robots.txt`).  
3. The `Disallow` directive reveals a restricted path, which leads to the admin panel.  

### Step 2: Accessing the Admin Panel  
1. Replace `/robots.txt` in the URL with `/administrator-panel`.  
2. Press Enter to navigate to the admin panel.  

### Step 3: Deleting the User  
1. Locate the option to manage or delete users.  
2. Find the user "carlos" and delete their account.  
---
---
#  Lab:2 Unprotected admin functionality with unpredictable URL
## Write-up: Finding the Admin Panel URL in JavaScript  

### Step 1: Reviewing the Source Code  
1. Open the lab home page in your browser.  
2. Use **Burp Suite** (HTTP history) or your browser’s **Developer Tools** (`Ctrl + U` or `Ctrl + Shift + I` → Elements/Network tab).  
3. Look for JavaScript code that reveals the admin panel URL.  

### Step 2: Accessing the Admin Panel  
1. Copy the disclosed admin panel URL.  
2. Paste it into the browser’s address bar and load the page.  

### Step 3: Deleting the User  
1. Locate the user management section.  
2. Find and delete the user **carlos**.  

This completes the lab.

---
---
#  Lab:3 User role controlled by request parameter
## Write-up: Bypassing Admin Restriction via Cookie Manipulation  

### Step 1: Checking Admin Access  
1. Open the lab URL in your browser.  
2. Navigate to `/admin` and observe that access is restricted.  

### Step 2: Capturing and Modifying the Login Response  
1. Browse to the login page and open **Burp Suite**.  
2. In **Burp Proxy**, turn **interception on** and enable **response interception**.  
3. Enter any credentials and submit the login form.  
4. In Burp, forward the login request.  
5. Observe the server’s response, which sets a cookie:  
   ```
   Set-Cookie: Admin=false
   ```  
6. Modify the cookie value from `Admin=false` to `Admin=true`.  
7. Forward the modified response.  

### Step 3: Accessing the Admin Panel and Deleting Carlos  
1. Reload `/admin`, and you should now have access.  
2. Locate the user **carlos** and delete the account.  

This completes the lab.

---
---
#  Lab:4 User role can be modified in user profile
## Write-up: Privilege Escalation via Role ID Manipulation  

### Step 1: Logging In  
1. Use the provided credentials to log in to your account.  
2. Navigate to your **account settings** page.  

### Step 2: Identifying Role ID in the Response  
1. Use the feature to update your **email address**.  
2. Observe the server’s response, which contains your `roleid`.  

### Step 3: Modifying the Role ID  
1. Open **Burp Suite** and send the email update request to **Burp Repeater**.  
2. Modify the JSON request body to include:  
   ```json
   "roleid": 2
   ```  
3. Resend the request and check the response to confirm that `roleid` has changed to **2**.  

### Step 4: Accessing the Admin Panel and Deleting Carlos  
1. Browse to `/admin` and confirm admin access.  
2. Locate **carlos** in the user management section and delete the account.  

This completes the lab.

---
# Lab:5 URL-based access control can be circumvented
## Write-up: Bypassing Front-End Restrictions Using `X-Original-URL` Header  

### Step 1: Identifying the Front-End Block  
1. Navigate to `/admin` and observe that access is denied.  
2. Notice that the response is **plain**, suggesting the block occurs at a front-end system rather than the back-end.  

### Step 2: Testing for Backend URL Handling  
1. Send the `/admin` request to **Burp Repeater**.  
2. Modify the **request line** to `/` and add the following header:  
   ```
   X-Original-URL: /invalid
   ```  
3. Observe that the response now returns **"not found"**, indicating the backend is processing this header instead of the original request URL.  

### Step 3: Gaining Admin Access  
1. Change the value of `X-Original-URL` to:  
   ```
   X-Original-URL: /admin
   ```  
2. Resend the request and confirm that you can now access the **admin panel**.  

### Step 4: Deleting Carlos  
1. Modify the request as follows:  
   - Add `?username=carlos` to the real query string.  
   - Change `X-Original-URL` to:  
     ```
     X-Original-URL: /admin/delete
     ```  
2. Resend the request to delete **carlos**.  

This completes the lab.  

---

## Understanding the `X-Original-URL` Header  

The `X-Original-URL` header is used by **reverse proxies** (such as **NGINX, Apache, and certain load balancers**) to preserve the original request path before rewriting it. Some back-end applications trust this header to determine the actual URL being requested, which can lead to bypassing front-end access controls.  

### Similar Headers That Can Bypass Controls  

1. **`X-Rewrite-URL`**  
   - Used in **IIS (Internet Information Services)** for URL rewriting.  
   - If a front-end system blocks access but the backend trusts this header, modifying it may bypass restrictions.  
   - Example:  
     ```
     X-Rewrite-URL: /admin
     ```

2. **`X-Forwarded-For`**  
   - Commonly used for **IP spoofing** when access is restricted based on IP.  
   - Some systems trust this header to determine the real client IP.  
   - Example:  
     ```
     X-Forwarded-For: 127.0.0.1
     ```

3. **`X-Forwarded-Host`**  
   - Can be used to manipulate **host-based access controls**.  
   - Some applications use this header to determine routing or security decisions.  
   - Example:  
     ```
     X-Forwarded-Host: admin.target.com
     ```

4. **`X-Forwarded-Proto`**  
   - Indicates whether the request was originally **HTTP or HTTPS**.  
   - If an application enforces HTTPS but trusts this header, setting it to `https` may bypass restrictions.  
   - Example:  
     ```
     X-Forwarded-Proto: https
     ```

5. **`X-Original-Host`**  
   - Similar to `X-Forwarded-Host`, but some proxies use this header to determine the original host.  
   - Example:  
     ```
     X-Original-Host: admin.target.com
     ```
---
# Lab:6 Method-based access control can be circumvented
## Write-up: HTTP Method Manipulation for Privilege Escalation  

### Step 1: Logging in as an Admin  
1. Use the provided **admin credentials** to log in.  
2. Navigate to the **admin panel** and promote **carlos**.  
3. Capture the **promotion request** in **Burp Suite** and send it to **Burp Repeater**.  

### Step 2: Testing Unauthorized Access  
1. Open a **private/incognito window** and log in as a **non-admin user**.  
2. Copy the **non-admin session cookie** into the **Burp Repeater request**.  
3. Resend the request and observe the **"Unauthorized"** response.  

### Step 3: Manipulating the HTTP Method  
1. Change the **HTTP method** from `POST` to `POSTX`.  
2. Resend the request and observe the **"missing parameter"** error.  
   - This suggests the server is handling unknown methods differently.  

### Step 4: Bypassing Controls with GET Method  
1. Right-click in **Burp Repeater** and select **"Change request method"** to convert it to `GET`.  
2. Modify the `username` parameter to your **own username** instead of `carlos`.  
3. Resend the request and check if the privilege escalation succeeds.  

This completes the lab.  

---

