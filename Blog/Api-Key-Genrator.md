
---

## 📘 API Key Management – Internal Design Document

---

### 🔐 Purpose

Possibly, we won't open up about the API key and building about the API's important aspect because we don't have a public API. Only the internal developers are allowed to call it. This makes it easier for them to use the API by just providing an API key for access.

The main advantage is that we don't need to go through a complex authentication flow (like JWT/CWT or system-to-system tokens). Instead, we can simply stick to a secure parameter approach.

This type of "API key as a shared secret" mechanism is also used by platforms like **Stripe** and **Trello**, so it is a proven and effective approach.

We’re using this not because we can’t do better, but because **this fits our use case well** — it is **organization-only**, not for individual user access.

---

### 📌 About API Exposure

We are not designing the full public API initially. As we develop the application further, we will expose the necessary APIs, and they should be **accessible only by the organization** that owns or manages them.

The API key will act as the access gateway.

---

### 🔑 Understanding the Two Types of API Keys

There are **two types of API key storage methods**:

#### 1. **Retrievable**

* The API key is stored in plaintext or reversible encryption.
* Can be viewed again from the system.
* Easier for developers, but less secure.

#### 2. **Irretrievable**

* The API key is shown only once at the time of creation.
* The actual key is **not stored**, only a **hash** of the key is stored.
* During validation, the incoming key is hashed and compared.
* Much more secure but has a downside: if lost, the key cannot be recovered.

We will go with the **Irretrievable model** — it is slightly more complex, but it's more secure and educates us on how modern systems like GitHub or Stripe handle secrets.

---

### ⚙️ Supporting Features and Considerations

#### 1. **Rolling Timeout (Planned Feature)**

* A timeout period can be set during which the organization can revoke and rotate their API keys.
* After revoking, the newly created key will take effect after a defined waiting period.

#### 2. **Checksum Validation**

* A checksum will be added to each API key and validated on each request.
* This ensures the key has not been tampered with or corrupted.

#### 3. **Minimal API Key Lifetime**

* API keys should have a short validity time by default.
* Expired keys should automatically be deactivated.

#### 4. **Metadata Support**

* Use snake\_case for all field names.
* Include `created_at`, `last_used_at`, `revoked_at`, and `status` fields.
* This helps organizations track key usage and perform revocations or rotations when necessary.

---

## 📌 Key Learnings / Design Notes

Here’s a breakdown of the important points captured from the design:

### ✅ Why API Keys?

* Internal APIs only (organization-level access).
* No need for complex token flows (JWT/OAuth).
* Lightweight authentication using shared secrets.

### 🔐 Irretrievable Key Storage

* Store only hashed values (like HMAC-SHA256 or bcrypt).
* No one, including the admin, can view the key after generation.
* Shows key once at creation — user must copy immediately.

### 🧠 API Key Format

* Proposed: `access_<prefix>_<hash(org_id)>_<random>`
* But better to avoid exposing org hashes in visible keys.
* Use a securely generated random token (UUIDv4 / base64).

### 📂 Storage Schema

```json
{
  "api_key_hash": "hashed_value",
  "org_id": "org-uuid",
  "created_at": "timestamp",
  "last_used_at": "timestamp",
  "revoked_at": "timestamp",
  "status": "active/revoked"
}
```

### 🔁 Rolling Timeout

* Allows delayed activation of new keys.
* Useful for smooth rotation.

### 🧪 Checksum Validation

* Add integrity checks to API keys.
* Reject tampered or malformed keys.

### 🚀 Future Improvements

* Redis caching for key validation to reduce DB load.
* Admin UI for listing all keys (with metadata, not value).
* Automatic key rotation support.
* Webhook notification on key usage or expiration.

---



Possibly, we won't open up about the API key
And building about the API important aspect because we don't have a public API and only the person
Developers to call it so it is easy for them to provide an API to give them an access to about we don't
The main advantage comes about as we don't have to think about like going about the complex authentication flow that we defined for the CWT as well as systems who or system so for that we are just simply
Stick to this parameter
Shift on the key is used by stripe as well as Treo TREWLO so that is a good approach active for it to do so so we are not as bad as but we should be using this as an important
I learn about like in my system, which I am creating. Now I need an API that expose my API that it should be handled by the organization only.
Preferably is about like it's only for the organization not for the individual us to handle about it like a particular individual so that's mean about it

Not designing about the API we are to is about as as we develop the application. We are gonna add things that we are going to develop.


 understanding about the types of APi keys there are two types of mainly
 1. Retrievable.
 2. Irretrievable.

So on the basis of our audience. What happens is about the in the  irr-> the  api key fades aways. as you genrate as what will happen 
is we will not stro the  whole api key . we will store about the hashes and verify with them  . that make about the things good\
but cons comes about here isthat user need to store at that instance . only .  we can  do  


1.Rolling time out thsi is th e time out given given to the organization of the uses about to when they rebook actually the API key
They have given time about the time they can after do that stuff of removing the about the starting when about the starting of the new API key will be working after revoking

2. The next comes about which validation checksum validation.
3. Then it comes to be it should be minimum. Let him see the time out of the API should be the key. It should be very less.
4. It comes about the snake case as well as you should also show the time in which the API created so help them to revoke the key according to that particular constraints.
