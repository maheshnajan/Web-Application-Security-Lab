# Write-up: Exploiting an API endpoint using documentation

In this lab, we explore an API vulnerability that allows unauthorized access to sensitive information and user deletion. The objective is to analyze the API structure, retrieve documentation, and exploit it to delete the user *Carlos*.

#### **Step 1: Logging in and Modifying User Information**
1. Open Burp’s browser and log in to the application using the credentials `wiener:peter`.
2. Navigate to the account settings and attempt to update the email address.
3. In Burp Suite, go to **Proxy > HTTP history** and locate the `PATCH /api/user/wiener` request.
4. Right-click the request and select **Send to Repeater** for further testing.

#### **Step 2: Manipulating API Endpoints**
5. Go to the **Repeater** tab and send the `PATCH /api/user/wiener` request. The response includes user credentials for *wiener*.
6. Modify the request by removing `/wiener` from the endpoint, changing it to `/api/user`, then send the request again. This results in an error due to the missing user identifier.
7. Further modify the request by removing `/user`, making the endpoint `/api`, then send it. This exposes API documentation.

#### **Step 3: Accessing and Using API Documentation**
8. Right-click the response in **Repeater** and select **Show response in browser**.
9. Copy the provided URL and open it in Burp’s browser.
10. The interactive API documentation is displayed, providing access to various API functionalities.

#### **Step 4: Exploiting the API to Delete a User**
11. In the API documentation interface, locate the **DELETE** method.
12. Enter `carlos` as the target user and submit the request.
13. The user *Carlos* is deleted, successfully solving the lab.

This vulnerability highlights the risks of improper API endpoint exposure, which can lead to unauthorized data access and administrative actions. Proper authentication and endpoint validation are essential to mitigate such risks.

### **Write-up: Finding and exploiting an unused API endpoint**  

This lab demonstrates how an attacker can manipulate API endpoints to modify product prices due to improper authorization checks and misconfigured request validation.  

---

### **Step 1: Identifying the API Endpoint**
1. Open **Burp’s browser** and navigate to the lab.  
2. Click on a product to view its details.  
3. In **Burp Suite**, go to **Proxy > HTTP history** and locate the API request fetching product information, such as:  
   ```
   GET /api/products/1/price
   ```
4. Right-click the request and select **Send to Repeater** for further testing.  

---

### **Step 2: Identifying Supported HTTP Methods**
5. In the **Repeater** tab, change the request **method** from `GET` to `OPTIONS` and send the request.  
6. The response reveals that the **GET** and **PATCH** methods are allowed, meaning price modifications may be possible.  

---

### **Step 3: Checking for Authorization Requirements**
7. Change the request method from `GET` to `PATCH` and send the request.  
8. The response returns an **Unauthorized** message, indicating authentication is required to modify prices.  

---

### **Step 4: Logging in and Reattempting the Request**
9. In **Burp’s browser**, log in using the provided credentials:  
   ```
   Username: wiener  
   Password: peter  
   ```
10. Click on the **Lightweight "l33t" Leather Jacket** product.  
11. In **Proxy > HTTP history**, locate the `GET /api/products/1/price` request for the jacket and send it to **Repeater**.  
12. Change the request method from `GET` to `PATCH` and send the request.  
13. The response returns an error due to a missing `Content-Type` header.  

---

### **Step 5: Bypassing Input Validation**
14. Add the following **Content-Type** header:  
   ```
   Content-Type: application/json
   ```
15. Add an **empty JSON body** `{}` and send the request.  
16. The response returns an error indicating that a required parameter (`price`) is missing.  
17. Modify the JSON body to include the price parameter:  
   ```json
   {"price": 0}
   ```
18. Send the request. If successful, the product price is now **$0.00**.  

---

### **Step 6: Exploiting the Price Manipulation**
19. In **Burp’s browser**, reload the **Leather Jacket** product page and confirm that the price has changed to **$0.00**.  
20. Add the leather jacket to your basket.  
21. Proceed to checkout and click **Place order** to solve the lab.  

---

### **Write-up: Exploiting a mass assignment vulnerability**  

This lab demonstrates how an attacker can manipulate API requests to apply unauthorized discounts by exploiting a **mass assignment vulnerability**. The vulnerability arises when an API automatically processes parameters that were not explicitly included in the original request.  

---

### **Step 1: Identifying the Checkout Process**
1. Open **Burp’s browser** and log in with the provided credentials:  
   ```
   Username: wiener  
   Password: peter  
   ```
2. Navigate to the **Lightweight "l33t" Leather Jacket** product page and add it to your basket.  
3. Proceed to the **checkout page** and attempt to place the order.  
4. Notice that the transaction fails due to **insufficient credit**.  

---

### **Step 2: Analyzing API Requests**
5. In **Burp Suite**, go to **Proxy > HTTP history** and locate both:  
   - **GET /api/checkout** (retrieving the checkout details).  
   - **POST /api/checkout** (submitting the purchase request).  
6. Observe that the **GET response** includes a `chosen_discount` parameter:  
   ```json
   {
       "chosen_discount": {
           "percentage": 0
       },
       "chosen_products": [
           {
               "product_id": "1",
               "quantity": 1
           }
       ]
   }
   ```
7. The `POST` request, however, does not include the `chosen_discount` parameter.  

---

### **Step 3: Injecting the Discount Parameter**
8. Right-click the **POST /api/checkout** request and select **Send to Repeater**.  
9. Modify the JSON body to **include the `chosen_discount` field**:  
   ```json
   {
       "chosen_discount": {
           "percentage": 0
       },
       "chosen_products": [
           {
               "product_id": "1",
               "quantity": 1
           }
       ]
   }
   ```
10. Send the request. Notice that the **request does not return an error**, confirming that the server accepts this parameter.  

---

### **Step 4: Testing Input Validation**
11. Modify the **chosen_discount** value to a non-numeric string (`"x"`) and send the request:  
   ```json
   {
       "chosen_discount": {
           "percentage": "x"
       },
       "chosen_products": [
           {
               "product_id": "1",
               "quantity": 1
           }
       ]
   }
   ```
12. Observe that the server returns an **error**, indicating that the input is being processed and validated.  

---

### **Step 5: Exploiting the Vulnerability**
13. Change the `chosen_discount` value to **100%** to get the product for free:  
   ```json
   {
       "chosen_discount": {
           "percentage": 100
       },
       "chosen_products": [
           {
               "product_id": "1",
               "quantity": 1
           }
       ]
   }
   ```
14. Send the modified request and confirm that the order is successfully placed.  
15. Refresh the **checkout page** in **Burp’s browser** to verify that the **leather jacket was purchased at $0.00**, solving the lab.  

---

# **Write-Up: Exploiting server-side parameter pollution in a query string**  

## **Overview**  
In this lab, we exploited **server-side parameter pollution (SSPP)** to extract the **reset token** for the administrator user. By manipulating URL-encoded parameters, we were able to access the **password reset endpoint** and take over the administrator account.  

---

## **Step 1: Triggering a Password Reset**  
1. Open **Burp’s browser** and initiate a **password reset** for the administrator account.  
2. In **Burp Suite**, navigate to **Proxy > HTTP history** and locate the `POST /forgot-password` request.  
3. Identify the associated JavaScript file **`/static/js/forgotPassword.js`**, which may reveal how the reset function operates.  

---

## **Step 2: Sending the Request to Repeater**  
1. **Right-click** on the `POST /forgot-password` request and select **Send to Repeater**.  
2. In **Repeater**, resend the request to confirm the response remains consistent.  

---

## **Step 3: Testing for Invalid Usernames**  
1. Modify the **username** parameter from `administrator` to `administratorx`.  
2. Send the request and observe the **Invalid username** error message.  
   - This confirms that invalid usernames are properly rejected.  

---

## **Step 4: Injecting Additional Parameters**  
1. Attempt to inject a second parameter using a URL-encoded `&`:  
   ```
   username=administrator%26x=y
   ```
2. Send the request. Observe that the response returns **Parameter is not supported**.  
   - This suggests that the internal API interprets `&x=y` as a **separate parameter**.  

---

## **Step 5: Truncating Query String with `#`**  
1. Modify the **username** parameter using a URL-encoded `#` to truncate backend queries:  
   ```
   username=administrator%23
   ```
2. Send the request. Observe the **Field not specified** error message.  
   - This suggests that a required parameter (possibly `field`) was removed by the `#` character.  

---

## **Step 6: Injecting a Field Parameter**  
1. Modify the **username** parameter to add a new field:  
   ```
   username=administrator%26field=x%23
   ```
2. Send the request. Observe the **Invalid field** error message.  
   - This suggests that the **`field` parameter is recognized** by the backend.  

---

## **Step 7: Brute-Forcing the Field Parameter**  
1. **Right-click** the `POST /forgot-password` request and select **Send to Intruder**.  
2. In **Intruder**, set the **payload position** inside the `field` parameter:  
   ```
   username=administrator%26field=§x§%23
   ```
3. Use the **built-in Server-side variable names** payload list and start the attack.  
4. Review the results and observe that `email` and `username` return **200 OK responses**.  

---

## **Step 8: Extracting the Reset Token**  
1. Modify the **field parameter** to use `email`:  
   ```
   username=administrator%26field=email%23
   ```
2. Send the request and verify that the response structure remains unchanged.  
3. Review the **`/static/js/forgotPassword.js`** file in **Proxy > HTTP history**.  
   - The script contains a reference to:  
     ```
     /forgot-password?reset_token=${resetToken}
     ```
   - This confirms that **reset_token** is a valid field type.  

4. Modify the request to replace `email` with `reset_token`:  
   ```
   username=administrator%26field=reset_token%23
   ```
5. Send the request. **Observe the response containing the password reset token**.  

---

## **Step 9: Resetting the Administrator Password**  
1. Copy the extracted **reset token**.  
2. In **Burp’s browser**, navigate to the **password reset endpoint**:  
   ```
   /forgot-password?reset_token=123456789
   ```
3. Set a **new password** for the administrator account.  
4. Log in as `administrator` using the newly set password.  

---

## **Step 10: Deleting Carlos (Lab Completion)**  
1. Navigate to the **Admin Panel**.  
2. Delete the user **Carlos** to complete the lab.  

---
