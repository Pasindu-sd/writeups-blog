
# #PortSwigger 


![[Pasted image 20260702225403.png]]


## Lab Description

The blog page for this lab contains a hidden blog post that has a secret password. To solve the lab, find the hidden blog post and enter the password.

---
---


### Step 1: Identify the Missing Blog Post

1. In Burp's browser, access the blog page
2. In Burp Proxy, examine the GraphQL response
3. Notice that blog posts have sequential IDs:
    - ID 1 --> Visible
    - ID 2 --> Visible
    - ID 3 --> **Missing!**
    - ID 4 --> Visible        

Blog post ID 3 is hidden.


---

### Step 2: Find the GraphQL Endpoint

1. In Burp Proxy, find the `POST /graphql/v1` request
2. Send it to Repeater

![[Pasted image 20260702232203.png]]


---

### Step 3: Run an Introspection Query

1. In Repeater, right-click in the Request panel
2. Select **GraphQL > Set introspection query**    
3. Send the request

**Introspection query:**
```
query {
  __schema {
    types {
      name
      fields {
        name
      }
    }
  }
}
```


![[Pasted image 20260702232654.png]]


---

### Step 4: Discover the `postPassword` Field

In the introspection response, look for the `BlogPost` type:
```
{
  "name": "BlogPost",
  "fields": [
    {"name": "id"},
    {"name": "title"},
    {"name": "content"},
    {"name": "postPassword"},
    ...
  ]
}
```

- The `postPassword` field exists on `BlogPost`.

![[Pasted image 20260702232813.png]]


---

### Step 5: Query the Hidden Post

1. In Repeater, switch to the **GraphQL tab**
2. In the **Variables** panel, set:
```
{
  "id": 3
}
```

3. In the **Query** panel, add the `postPassword` field:
```
query getBlogPost($id: Int!) {
  blogPost(id: $id) {
    id
    title
    content
    postPassword
  }
}
```

4. Send the request

---

### Step 6: Extract the Password

**Response:**
```
{
  "data": {
    "blogPost": {
      "id": 3,
      "title": "Secret Post",
      "content": "...",
      "postPassword": "the_secret_password"
    }
  }
}
```

![[Pasted image 20260702234731.png]]

Copy the password.

---

### Step 7: Submit the Password

1. Go back to the lab page
2. Click **Submit solution**
3. Paste the password
4. Click **Submit**


---

### Step 8: Lab Solved

![[Pasted image 20260702234832.png]]

---
---

