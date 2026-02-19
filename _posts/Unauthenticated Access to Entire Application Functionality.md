---
layout: post
title: "Unauthenticated Access to Entire Application Functionality"
date: 2026-02-18
categories: vapt web security
---

## 1. Introduction
During a VAPT engagement, a critical access control weakness was identified in the web application’s backend APIs. Although the web interface enforces login redirection, the underlying APIs fail to properly validate authentication tokens or session states. This allows unauthenticated users to directly interact with protected endpoints, resulting in unauthorized access to sensitive application data and functionality.

---

## 2. Vulnerability Overview
The application fails to enforce authentication checks across a majority of its backend API endpoints and application functionality. While direct access to application pages through a browser redirects unauthenticated users to the login page, backend APIs can be accessed directly by sending crafted HTTP requests. The application processes these requests without validating authentication tokens or active session cookies, allowing unauthenticated users to interact with protected functionality. This indicates a fundamental access control flaw in the application’s authentication and authorization mechanism.

As a result of this issue, an unauthenticated attacker can access sensitive application data and functionality across the entire web application. Responses from vulnerable endpoints expose sensitive internal information, including user details, administrative accounts, application roles, and internal endpoint data. Since this behaviour is observed across most application endpoints, the vulnerability leads to complete loss of confidentiality and control over application resources.

---

## 3. Testing Scope & Methodology
- Assessment Type: Black-box
- Access Level: Unauthenticated
- Tools Used: Burp Suite
- Environment: Web application (sanitized)

---

## 4. Steps to Reproduce

1. Configure a browser to route traffic through an interception proxy (e.g., Burp Suite).

2. Navigate to the application URL:

   `https://website`

   Confirm that the application redirects unauthenticated users to the login page.

 

3. Without entering credentials, switch to **Burp Suite → Repeater**.

4. Create a new HTTP request in the Repeater tab.

5. Configure the request as example:

   - **Method:** POST  
   - **Endpoint:** `/getUsers`(put endpoints here)  
   - **Authentication:** None  
   - **Cookies:** Not included  
   - **Authorization Headers:** Not included  



6. Insert the following sanitized request structure:

POST /getUsers HTTP/1.1
Host: <redacted>
Auth-Token: false
Content-Type: application/json
Role: admin
Username: false

{"username":"false"}


7. Ensure that:
   - No session cookies are present  
   - No bearer tokens are included  
   - No authenticated headers are supplied  

8. Click **Send** in Repeater.

9. Observe the server response in the **Response** tab.

---

### Expected Result

The application should return:

- `HTTP/1.1 401 Unauthorized`  
  **or**  
- `HTTP/1.1 403 Forbidden`  

Because the request does not contain valid authentication credentials.

---

### Actual Result

The server responds with:

- `HTTP/1.1 200 OK`

The response contains sensitive user information, including:

- User email addresses  
- Phone numbers  
- Role assignments (admin/user)  
- Last login timestamps  



---

## Impact Analysis
This vulnerability allows attackers to:

1. Gain complete unauthenticated access to all backend APIs across the application.

2. Access and disclose sensitive information of both regular users and administrative accounts, including personally identifiable information and role-based details.

3. Enumerate application users, administrative accounts, and associated privilege levels without authorization.

4. Create, modify, or delete application data without any authentication or authorization checks.

5. Completely bypass all authentication and authorization mechanisms implemented in the application.

6. Cause a complete loss of confidentiality, integrity, and trust in the application and its data.

---

## Recommendations

1. Implement mandatory authentication checks on all backend API endpoints.

2. Enforce server-side validation of session tokens or authentication credentials for every request.

3. Remove reliance on client-controlled headers (such as role or authorization flags) for authorization decisions.

4. Apply a centralized authentication and authorization middleware across the entire application.

5. Ensure unauthenticated requests receive HTTP 401 (Unauthorized) responses.
