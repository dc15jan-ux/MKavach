# Hacking OWASP Juice Shop: Forged Review (Broken Access Control)

**Author:** Team-09 
**Category:** A01:2021 — Broken Access Control  
**Difficulty:** ⭐⭐ Medium  
**Target:** OWASP Juice Shop (localhost:3000)

---

## Introduction

This lab — **Forged Review** — challenges us to *"Post a product review as another user or edit any user's existing review."*

This is a classic example of one of the most critical web application vulnerabilities:  
**A01:2021 — Broken Access Control**, which allows attackers to perform actions they shouldn't be authorized to do.

---

## Step 1: Understanding the Target Feature

First, we need to understand how the review system works. As a logged-in user, we can navigate to any product and leave a review.

**We start by logging into our own account and finding a product, like the "Carrot Juice".**

![Step 1 - Navigate to Carrot Juice product page and login](./Evidences/1.JPG)

---

**Next, we write and submit a simple review.**

![Step 2 - Write and submit a review on the product](./Evidences/2.JPG)

---

This helps us understand the normal process. After submitting, our review appears alongside others — this is the **expected behavior**.

![Step 3 - Review appears on the product page](./Evidences/3.JPG)

---

## Step 2: Intercepting the Request with Burp Suite

To find a flaw, we need to look at what's happening behind the scenes. We'll use **Burp Suite** to intercept the request sent when we submit a review.

![Step 4 - Burp Suite proxy setup and interception](./Evidences/4.JPG)

---

**With Burp Suite's proxy enabled, we submit another review.**

By examining the HTTP history in Burp, we find the `PUT` request sent to the `/rest/products/{id}/reviews` endpoint.

![Step 5 - HTTP PUT request captured in Burp Suite HTTP History](./Evidences/5.JPG)

> This is the request that creates the review.

---

## Step 3: Manipulating the Request to Impersonate Another User

Here's where the vulnerability lies. The body of the `PUT` request contains the review message and an **"author"** field.

> **The application trusts the user-provided author email instead of verifying it against the user's session.**

We can exploit this by changing the `author` field to another user's email. We find another user's email from the existing reviews — for example: `uvogin@juice-sh.op`

**We modify the body of the request in Burp Repeater, changing the "author" to our target's email.**

![Step 6 - Burp Repeater with modified author field](./Evidences/6.JPG)

---

### The Modified JSON Body Looks Like This:

```json
{
  "message": "This is a forged review!",
  "author": "uvogin@juice-sh.op"
}
```

![Step 7 - Modified JSON body in Burp Repeater](./Evidences/7.JPG)

---

## Step 4: Verifying the Exploit

Now let's check the product page to see if our attack worked.

**Refreshing the "Carrot Juice" page, we can see our forged review — but now it appears to have been posted by the user we impersonated.**

![Step 8 - Forged review appearing under victim's name on product page](./Evidences/8.JPG)

---

**Success! The application notifies us that we've solved the "Forged Review" challenge.**

![Step 9 - Juice Shop challenge solved notification](./Evidences/9.JPG)

---

## Conclusion

This lab demonstrates a classic case of **Broken Access Control**.

| What Went Wrong | What Should Have Happened |
|-----------------|--------------------------|
| App trusted user-supplied `author` field | Server should ignore client-supplied author |
| No session verification on review author | Author should be pulled from session token |
| Any email could be forged in the request | Validate author against authenticated user |

### Prevention

- **Always perform server-side authorization checks** based on the user's session
- **Never trust user-controllable data** for identity or ownership fields
- **Enforce that the author of a review = the currently logged-in user**, not a value passed in the request body
- Use **JWT/session tokens** to determine the author server-side

---

## Tools Used

| Tool | Purpose |
|------|---------|
| OWASP Juice Shop | Target vulnerable web application |
| Burp Suite Community | HTTP interception and request manipulation |
| Firefox | Browser for navigating the app |

---

*This writeup is for educational purposes only. Practice only on intentionally vulnerable applications like OWASP Juice Shop.*
