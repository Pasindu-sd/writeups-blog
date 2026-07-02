
# #PortSwigger 


![[Pasted image 20260703004241.png]]

## Lab Description

The user login mechanism for this lab is powered by a GraphQL API. The API endpoint has a rate limiter that returns an error if it receives too many requests from the same origin in a short space of time.

**Objective:** Brute force the login mechanism to sign in as carlos. Use the list of authentication lab passwords as your password source.

**Tip:** Use **GraphQL aliases** to send multiple login attempts in a single request.

---
---


### Step 1: Capture the Login Mutation

1. In Burp's browser, access the lab
2. Attempt to log in with any credentials
3. In Burp Proxy, find the GraphQL login request, `POST /graphql/v1 HTTP/2`

**Example request:**
```
mutation {
  login(input: {username: "carlos", password: "test"}) {
    token
    success
  }
}
```

![[Pasted image 20260703004541.png]]


---

### Step 2: Generate the Aliased Query

**Use the provided JavaScript script to generate aliases:**
1. Open the lab in Burp's browser
2. Right-click --> **Inspect**
3. Go to the **Console** tab
4. Paste the script and press **Enter**

![[Pasted image 20260703005255.png]]

![[Pasted image 20260703005339.png]]

**The script generates a query like:**
```
mutation {
  bruteforce0: login(input: {password: "123456", username: "carlos"}) {
    token
    success
  }
  bruteforce1: login(input: {password: "password", username: "carlos"}) {
    token
    success
  }
  bruteforce2: login(input: {password: "12345678", username: "carlos"}) {
    token
    success
  }
  ...
}
```


---

### Step 3: Send the Request

1. Copy the generated aliased query
2. In Burp Repeater, replace the body with the aliased query
3. Send the request

GraphQL:
![[Pasted image 20260703005756.png]]

Response:
![[Pasted image 20260703005647.png]]


---

### Step 4: Find the Correct Password

**Response:**
![[Pasted image 20260703005844.png]]


The alias with `success: true` reveals the correct password.

![[Pasted image 20260703010043.png]]


---

### Step 5: Log In as Carlos

1. Use the discovered password to log in
2. Access Carlos's account   
	1. Username: `carlos`
	2. Password: `charlie`

![[Pasted image 20260703010219.png]]


---

### Step 6: Lab Solved

![[Pasted image 20260703010239.png]]

---
---

