# **What is Serialization?**

Serialization is the process of **converting complex data structures**, such as objects and their fields, **into a "flatter" format** that can be sent and received as a sequential stream of bytes.
In other words "Serialization is the process of converting an object (like a Python dictionary, a Java object, or a JSON structure) into a format that can be easily stored or transmitted (e.g., JSON, XML, or binary data)."

**Serializing data makes it much simpler to:**

 - Write complex data to inter-process memory, a file, or a database
 - Send complex data, for example, over a network, between different components of an application, or in an API call
 - Crucially, when serializing an object, its state is also persisted. In other words, the object's attributes are preserved, along with their assigned values.

**Deserialization** is the process of **restoring this byte stream** to a **fully functional replica** of the original object, in the exact state as when it was serialized. The website's logic can then interact with this deserialized object, just like it would with any other object.
In short it is the **reverse process** — taking that serialized data and converting it back into an object that the application can use.

## **EXAMPLE :**
```
import pickle  

# Serialization: Convert a Python object into bytes
data = {"username": "Lakshya", "role": "admin"}
serialized_data = pickle.dumps(data)  

# Deserialization: Convert bytes back into a Python object
deserialized_data = pickle.loads(serialized_data)  
print(deserialized_data)  # Output: {'username': 'Lakshya', 'role': 'admin'}

```
----
## **What is insecure deserialization?**
 - Insecure deserialization occurs when an application **blindly accepts** and **deserializes** untrusted data without **validation or sanitization**. If an attacker sends a malicious serialized object, they can execute arbitrary code, modify application logic, or escalate privileges.
 - It is even possible to replace a serialized object with an object of an entirely different class. Alarmingly, objects of any class that is available to the website will be deserialized and instantiated, regardless of which class was expected. For this reason, insecure deserialization is sometimes known as an **"object injection" vulnerability**.
 - An object of an unexpected class might cause an exception. By this time, however, the damage may already be done. Many deserialization-based attacks are completed before deserialization is finished. This means that the deserialization process itself can initiate an attack, even if the website's own functionality does not directly interact with the malicious object. For this reason, websites whose logic is based on strongly typed languages can also be vulnerable to these techniques.

----
----
### **How do insecure deserialization vulnerabilities arise?**

 - Insecure deserialization typically arises because there is a general lack of understanding of how dangerous deserializing user-controllable data can be. Ideally, user input should never be deserialized at all.

 - Insecure deserialization vulnerabilities arise because it is virtually impossible to implement validation or sanitization to account for every eventuality. These checks are also fundamentally flawed as they rely on checking the data after it has been deserialized, which in many cases will be too late to prevent the attack.

 - Vulnerabilities may also arise because deserialized objects are often assumed to be trustworthy. Especially when using languages with a binary serialization format, developers might think that users cannot read or manipulate the data effectively. However, while it may require more effort, it is just as possible for an attacker to exploit binary serialized objects as it is to exploit string-based formats.

 - Deserialization-based attacks are also made possible due to the number of dependencies that exist in modern websites. This creates a massive pool of classes and methods that is difficult to manage securely. As an attacker can create instances of any of these classes, it is hard to predict which methods can be invoked on the malicious data. 

 - In short, it can be argued that it is not possible to securely deserialize untrusted input.

----
----
### **What is the impact of insecure deserialization?**

 - The impact of insecure deserialization can be very severe because it provides an entry point to a massively increased attack surface. It allows an attacker to reuse existing application code in harmful ways, resulting in numerous other vulnerabilities, often remote code execution.

 - Even in cases where remote code execution is not possible, insecure deserialization can lead to :
  - Privilege escalation
  - Arbitrary file access
  - Denial-of-service attacks.
----
