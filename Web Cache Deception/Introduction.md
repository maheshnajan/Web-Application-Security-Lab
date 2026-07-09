# Web Cache Deception Vulnerability

### **Web Cache Deception Vulnerability: A Detailed Explanation**  

#### **What is Web Cache Deception?**  
**Web Cache Deception (WCD)** is a vulnerability that occurs when a web server **incorrectly caches sensitive content** (such as user-specific pages) and serves it to other users. It allows an attacker to trick a caching proxy or CDN into storing and serving private information publicly.  

**Root Cause**: Misconfigured caching mechanisms combined with improper URL handling.  
**Impact**: Unauthorized access to sensitive data such as **user profiles, private messages, session details, or invoices**.  

---

### **How Does Web Cache Deception Work?**  

1. **Targeting a Non-Cached Page:**  
   - Many websites do **not cache** dynamic pages, such as `/profile` or `/dashboard`, because they contain private user data.  

2. **Appending a Fake Extension:**  
   - If a web application does **not properly validate URL extensions**, an attacker can append a **static-looking** file extension like `.css`, `.jpg`, or `.js` to trick caching mechanisms.  
   - Example:
     - **Original URL (non-cached)**: `https://example.com/profile`
     - **Modified URL (cached)**: `https://example.com/profile.css`

3. **Forcing the Cache to Store Private Content:**  
   - If the caching proxy or Content Delivery Network (CDN) sees `profile.css` as a **static asset**, it might store and serve the **private content of `/profile`** to **anyone** requesting `profile.css`.  

4. **Attacker Accesses Cached Private Data:**  
   - The attacker visits `https://example.com/profile.css` later and sees **cached content from another user’s session**.  

---

### **Example Scenario: Web Cache Deception Attack**  

#### **Step 1: Identifying a Non-Cached Page**  
A website has a **dashboard page** for logged-in users:  
`https://example.com/dashboard` (Contains personal details, not cached)  

#### **Step 2: Appending a Fake Static File Extension**  
An attacker tricks the cache by adding `.css`:  
`https://example.com/dashboard.css`  

#### **Step 3: The Web Cache Stores the Sensitive Page**  
If the server does **not validate extensions properly**, the CDN **mistakenly** caches the full **HTML of the dashboard page**.  

#### **Step 4: Attacker Retrieves Cached Data**  
Now, **anyone** (even without authentication) can visit:  
`https://example.com/dashboard.css`  
And see cached **sensitive data** from another user’s session!  

![image](https://github.com/user-attachments/assets/c185cb04-8b67-49b2-8c34-e539f29c1aa3)

### **Breaking Down: Constructing a Web Cache Deception Attack**  

A **Web Cache Deception attack** exploits **misconfigurations in caching mechanisms** to make a web cache store **private, dynamic user data** as if it were a **public static file**. This allows an attacker to later retrieve another user's sensitive information.

---

## **Step-by-Step Breakdown of the Attack Process**  

### **1️.Identify a Target Endpoint that Returns a Dynamic Response Containing Sensitive Information**  

**Meaning:** The attacker first finds a **web page that contains private user data** but is **not meant to be cached**.  
- Example:  
  - `/profile` (Shows user account details)  
  - `/dashboard` (Contains billing info, messages)  
  - `/inbox` (Displays private messages)  

**Why this is important?**  
- The attacker needs **a dynamic page** that should be **user-specific** and **not publicly available**.  

**How to Find These Pages?**  
- Use **Burp Suite** to analyze HTTP responses for **sensitive information** like:  
  - **Session tokens** (`Set-Cookie`)  
  - **Usernames, email addresses**  
  - **Financial data**  
  - **Personal messages**  

**Security Tip:** Some sensitive info **does not appear on the rendered page** but can be seen **in raw HTTP responses** in Burp Suite.

---

### **2️.Identify a Discrepancy in How the Cache and Origin Server Parse the URL Path**  

**Meaning:** The attacker looks for **differences in how URLs are processed** by the **cache server** vs. the **origin server**.  

**Common Discrepancies:**  
1. **Mapping URLs to Resources**  
   - Example:  
     - `https://example.com/profile` → Displays **user data (not cached)**  
     - `https://example.com/profile.css` → **Cache incorrectly stores** the user data as a CSS file.  

2. **Processing Delimiter Characters** (`?`, `#`, `;`, `/`)  
   - Some caching mechanisms **ignore certain characters** while origin servers **interpret them** differently.  
   - Example:  
     - `https://example.com/profile;css` → Cache sees it as a static file but still loads sensitive data.  

3. **Path Normalization Issues**  
   - Example:  
     - `https://example.com/dashboard/..;/dashboard.css`  
     - Cache **normalizes** this to `/dashboard.css`, but the origin **still returns private data**.  

**Security Tip:** These discrepancies allow attackers to **trick the cache into storing private responses as public files**.

---

### **3️.Craft a Malicious URL that Exploits the Discrepancy**  

**Meaning:** The attacker **modifies the URL** in a way that tricks the cache into **storing user-specific responses** as public files.  

**Example:**  
1️. The attacker **appends a fake static file extension** to a private page:  
   - **Original (not cached):**  
     ```
     https://example.com/profile
     ```
   - **Tricked URL (cached incorrectly):**  
     ```
     https://example.com/profile.css
     ```
   - The cache **mistakenly stores the private profile page** as if it were a static CSS file.  

2️. The attacker **shares the malicious URL with a victim** (via phishing, hidden scripts, etc.).  

3. When the victim **visits the malicious URL (`profile.css`)**, their private profile page **gets stored in the cache**.  

4. The attacker **later requests `profile.css`** and retrieves the cached response **containing the victim's data**.  

**Security Tip:** The attacker must ensure that:  
- The request is processed by the cache before the **cache expires**.  
- The victim **accesses the URL** so their **private data is stored** in the cache.

---

### **4️. Retrieve the Cached Response Using Burp Suite**  

**Meaning:** The attacker **requests the same URL later** (without authentication) and sees the cached victim’s data.  

**Steps to Exploit:**  
1️. **Victim accesses the modified URL:**  
   - `https://example.com/dashboard.css`  
2️. **Cache stores private data** incorrectly.  
3️. **Attacker later requests the same URL** via Burp Suite:  
   ```http
   GET /dashboard.css HTTP/1.1
   Host: example.com
   ```
4️. **Cached private data is returned.**  

**Security Tip:**  
- **Do NOT test this in a browser** because:  
  - Some web apps **redirect non-authenticated users** (causing the attack to fail).  
  - Some apps **invalidate session data**, hiding vulnerabilities.  
- Instead, **use Burp Suite** to send raw requests and analyze responses.

---

## Web cache deception attacks exploit how cache rules are applied, so it's important to know about some different types of rules, particularly those based on defined strings in the URL path of the request. For example:
 - **Static file extension rules** - These rules match the file extension of the requested resource, for example .css for stylesheets or .js for JavaScript files.
 - **Static directory rules** - These rules match all URL paths that start with a specific prefix. These are often used to target specific directories that contain only static resources, for example /static or /assets.
 - **File name rules** - These rules match specific file names to target files that are universally required for web operations and change rarely, such as robots.txt and favicon.ico.
