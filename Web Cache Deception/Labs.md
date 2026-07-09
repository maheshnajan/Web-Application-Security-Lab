# Web Cache Deception Write-Up
## Lab: Exploiting path mapping for web cache deception
## **1. Identifying a Target Endpoint**

1. Log in to the application using the provided credentials:
   - **Username:** `wiener`
   - **Password:** `peter`

2. Observe that the response contains your **API key**. This indicates that the `/my-account` endpoint returns sensitive user-specific information.

---

## **2. Identifying a Path Mapping Discrepancy**

1. In **Burp Suite**, navigate to **Proxy > HTTP history**.
2. Locate the `GET /my-account` request, right-click, and select **Send to Repeater**.
3. In the **Repeater** tab, modify the request path by appending an arbitrary segment:
   ```
   GET /my-account/abc HTTP/1.1
   ```
4. Send the request and observe that the response still contains your API key. This suggests that the **origin server abstracts the URL path** and still maps it to `/my-account`.

5. Next, modify the request by adding a static file extension:
   ```
   GET /my-account/abc.js HTTP/1.1
   ```
6. Send the request and examine the response headers:
   - `X-Cache: miss` → Indicates the response was **not served from the cache**.
   - `Cache-Control: max-age=30` → Suggests that if cached, the response will be stored **for 30 seconds**.

7. Resend the request **within 30 seconds** and check the `X-Cache` header. Notice that it now changes to `hit`, confirming that the response was **served from the cache**.
8. This confirms that the caching mechanism **misinterprets** `/my-account/abc.js` as a static file and stores the response, making it a potential vector for exploitation.

---

## **3. Crafting the Exploit**

1. In **Burp Suite's browser**, click **Go to exploit server**.
2. In the **Body** section of the exploit, craft the following payload to force the victim (`carlos`) to visit the malicious cached URL:
   ```html
   <script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account/wcd.js"</script>
   ```
![Screenshot (133)](https://github.com/user-attachments/assets/1aa5844c-268f-45e0-ba61-80833626cf76)

3. Click **Deliver exploit to victim**.
4. When the victim (`carlos`) accesses the exploit, their response (including their **API key**) is **cached**.
![Screenshot (134)](https://github.com/user-attachments/assets/e2fd425f-9e3a-4778-be73-83c0222a2766)

---

## **4. Extracting the API Key and Submitting the Solution**

1. Manually visit the cached URL:
   ```
   https://YOUR-LAB-ID.web-security-academy.net/my-account/wcd.js
   ```
2. Observe that the response **now contains carlos' API key**.
3. Copy the API key. 
4. Click **Submit solution** and paste carlos’ API key to complete the challenge.
5. API Key for my lab 
```
ydYS2hO3IhusK9NEChqkMbqPlBUQSH
```
![Screenshot (132)](https://github.com/user-attachments/assets/4374f00c-a8f1-4cf9-b969-de8c2a1dd480)


# Lab2: Exploiting path delimiters for web cache deception

## 1. Identifying a Target Endpoint

1. Log in to the application using the provided credentials:
   - Username: wiener
   - Password: peter

2. Observe that the response contains your API key. This confirms that the `/my-account` endpoint returns user-specific data.

## 2. Identifying Path Delimiters Used by the Origin Server

1. In Burp Suite, navigate to Proxy > HTTP history.
2. Locate the `GET /my-account` request, right-click, and select Send to Repeater.
3. In the Repeater tab, modify the request path by appending an arbitrary segment:
   ```
   GET /my-account/abc HTTP/1.1
   ```
4. Send the request and observe the 404 Not Found response, confirming that the origin server does not abstract `/my-account`.

5. Modify the request by appending an arbitrary string without a delimiter:
   ```
   GET /my-accountabc HTTP/1.1
   ```
6. Send the request. The 404 Not Found response confirms that the server does not recognize this modification.

7. Right-click the request and select Send to Intruder.
8. In Intruder, ensure that Sniper attack mode is selected.
9. Add a payload position after `/my-account`:
   ```
   /my-account§§abc
   ```
10. In the Payloads side panel, under Payload configuration, add a list of possible delimiter characters.
11. Under Payload encoding, deselect URL encoding for these characters.
12. Click Start attack.

13. Once the attack completes, sort the results by Status code.
14. Observe that the `;` and `?` characters return a 200 OK response with the API key, while all others return 404 Not Found.

## 3. Investigating Path Delimiter Discrepancies

1. Go to the Repeater tab with the `/my-accountabc` request.
2. Add the `?` delimiter and append a static file extension:
   ```
   GET /my-account?abc.js HTTP/1.1
   ```
3. Send the request. No caching headers appear in the response, indicating that the cache also uses `?` as a delimiter.

4. Repeat the test using `;` instead of `?`:
   ```
   GET /my-account;abc.js HTTP/1.1
   ```
   ![Screenshot from 2025-02-09 09-49-11](https://github.com/user-attachments/assets/66685527-b07f-4f74-88a4-f460e758cbff)

5. Send the request. Observe that the response includes the X-Cache: miss header.
![Screenshot from 2025-02-09 09-50-24](https://github.com/user-attachments/assets/54d717f6-868a-494b-ac2e-05311db4449b)

6. Resend the request within the cache duration.
7. The X-Cache header changes to `hit`, confirming that:
   - The cache does not use `;` as a delimiter.
   - The cache has a rule based on the `.js` extension, allowing the response to be stored.

## 4. Crafting the Exploit

1. In Burp Suite's browser, go to the Exploit Server.
2. In the Body section, craft an exploit to redirect the victim (carlos) to the malicious cached URL:
   ```html
   <script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account;wcd.js"</script>
   ```
   ![Screenshot from 2025-02-09 09-49-22](https://github.com/user-attachments/assets/9c736b23-c603-4b11-a691-3e50cb760e0a)

3. Click Deliver exploit to victim.
4. When the victim views the exploit, their API key response is cached.

## 5. Extracting the API Key and Submitting the Solution

1. Manually visit the cached exploit URL:
   ```
   https://YOUR-LAB-ID.web-security-academy.net/my-account;wcd.js
   ```
2. Observe that the response contains carlos’ API key.
![Screenshot from 2025-02-09 09-48-36](https://github.com/user-attachments/assets/2132b995-cbaa-415a-808f-823c53148d67)
3. Copy the API key.
4. Click Submit solution and paste carlos' API key to complete the challenge.
5. API key for my lab was :
```
CKA6jfg6ybxTwPDnUJCpqGId5Ql7XB2f
```
![Screenshot from 2025-02-09 09-48-59](https://github.com/user-attachments/assets/5e6c9f45-a896-4286-8d51-da887921d647)



# Lab: Exploiting origin server normalization for web cache deception

## 1. Identifying a Target Endpoint

1. Log in to the application using the provided credentials:
   - Username: wiener
   - Password: peter

2. Observe that the response contains your API key, confirming that `/my-account` serves user-specific data.

## 2. Investigating Path Delimiter Discrepancies

1. In **Burp Suite**, navigate to **Proxy > HTTP history**.
2. Locate the `GET /my-account` request, right-click, and select **Send to Repeater**.
3. Modify the request path to `/my-account/abc`, then send the request.
   - Observe the **404 Not Found** response, indicating that the origin server does not abstract the path.
4. Modify the request path to `/my-accountabc`, then send the request.
   - Observe the **404 Not Found** response with no caching evidence.
5. Right-click the request and select **Send to Intruder**.
6. In **Intruder**, ensure **Sniper attack mode** is selected.
7. Add a **payload position** after `/my-account`:
   ```
   /my-account§§abc
   ```
8. Under **Payloads**, add a list of potential delimiter characters.
9. Under **Payload encoding**, **deselect URL encoding** for these characters.
10. Click **Start attack**.
11. Once the attack completes, sort the results by **Status code**.
12. Observe that **only the `?` character returns a 200 OK response with the API key**.
13. Since `?` is a universal delimiter, move on to investigate **normalization discrepancies**.

## 3. Investigating Normalization Discrepancies

1. In **Repeater**, remove the arbitrary `abc` string and add an **encoded dot-segment**:
   ```
   /aaa/..%2fmy-account
   ```
2. Send the request.
   - Observe that it receives a **200 response with your API key**, indicating that the origin server decodes and resolves dot-segments.
3. In **Proxy > HTTP history**, notice that static resources use the `/resources` prefix.
4. Responses with the `/resources` prefix **show caching evidence**.
5. Right-click a request with the `/resources` prefix and select **Send to Repeater**.
6. In **Repeater**, add an **encoded dot-segment**:
   ```
   /resources/..%2fYOUR-RESOURCE
   ```
7. Send the request.
   - Observe the **404 response with X-Cache: miss**.
8. Resend the request.
   - Observe that **X-Cache changes to hit**, suggesting that the cache does not resolve dot-segments and has a cache rule based on `/resources`.
9. Modify the URL path after `/resources`:
   ```
   /resources/aaa
   ```
   ![Screenshot from 2025-02-10 21-10-15](https://github.com/user-attachments/assets/39dbc3e4-2aac-4806-8e78-bc3117e390ab)

10. Send the request.
    - Observe the **404 response with X-Cache: miss**.
11. Resend the request.
    - Observe **X-Cache changes to hit**, confirming a **static directory cache rule based on `/resources`**.

## 4. Crafting the Exploit

1. In **Repeater**, go to the `/aaa/..%2fmy-account` request.
2. Construct an exploit by modifying the path:
   ```
   /resources/..%2fmy-account
   ```
   ![Screenshot from 2025-02-10 21-10-02](https://github.com/user-attachments/assets/4f2960bb-f3b3-495f-a1f7-69497e4ac245)

3. Send the request.
   - Observe that it receives a **200 response with your API key** and **X-Cache: miss**.
![Screenshot from 2025-02-10 21-10-10](https://github.com/user-attachments/assets/70de79d1-2631-4941-964d-799ab3a559f2)

4. Resend the request.
   - Observe **X-Cache updates to hit**.
5. In **Burp's browser**, click **Go to exploit server**.
6. In the **Body** section, craft an exploit to redirect the victim (`carlos`) to the **malicious cached URL**:
   ```html
   <script>document.location="https://YOUR-LAB-ID.web-security-academy.net/resources/..%2fmy-account?wcd"</script>
   ```
7. Click **Deliver exploit to victim**.
8. When the victim accesses the link, their **API key response is cached**.

## 5. Extracting the API Key and Submitting the Solution

1. Visit the **cached exploit URL**:
   ```
   https://YOUR-LAB-ID.web-security-academy.net/resources/..%2fmy-account?wcd
   ```
2. Observe that the response contains **carlos’ API key**.
3. Copy the API key.
4. Click **Submit solution** and paste **carlos' API key** to complete the challenge.
```
API Key is: dFIYEjZiLBBp6GQYmNkvpqwFcxbcPq3F
```
![Screenshot from 2025-02-10 21-09-58](https://github.com/user-attachments/assets/de2eae55-05a5-442b-aaa3-dd873b33594a)
