# LandTech 2026 · P2 — Deed & Ownership Verification

**Team:** Draft Zero (ID: LA-134) · **Track:** Digital Land Records, Verification & Digitization
**Institution:** Daffodil International University · **Team:** Tausif Hasan (Leader), Umme Salman Afrin

> Hackathon prototype. All data is synthetic. This is not an official government system.

## 1. Project Overview (P2)

**Problem.** In Bangladesh, forged deeds and conflicting Khatians (CS, SA, RS, BS) are a leading source of land disputes. Sub-Registry offices and AC Land offices keep records that are not instantly cross-checkable, so a citizen or bank often cannot tell in seconds whether a paper deed is genuine.

**Solution.** A zero-backend web app with two portals:

| Portal | Purpose |
|---|---|
| **Sub-Registry Portal (Issue & Stamp)** | Officer enters deed particulars, uploads the deed file, and the app computes its real SHA-256 fingerprint and generates a QR stamp. |
| **Public Citizen Portal (Verify Deed)** | Citizen scans the QR, sees the registered owner and Khatian, then uploads the deed file. Any change to the file, even 1 byte, produces a different hash and a red forgery alert. |

**How it works**

1. The file is read with `FileReader` and hashed with `window.crypto.subtle.digest('SHA-256', …)` in the browser.
2. The record is saved to `localStorage`, and a QR is generated for `origin + pathname + ?verify=1&hash=…&khatian=…&owner=…`.
3. Opening that link auto-selects the Verify tab and shows the **AUTHENTICATED OFFICIAL DEED** card from the URL parameters. No shared storage is needed, so an iPhone can scan an iPad screen.
4. Uploading a file in the Verify tab recomputes its hash and compares it with the QR record (if the page was opened from a QR) or with the device ledger / current session.
5. **Test Tampered File** flips one byte, re-hashes, and shows the mismatched hashes with differing characters highlighted.

**Tech stack:** HTML5, Tailwind CSS (CDN), qrcodejs, Web Crypto API. No backend, no build step, no external data calls.

## 2. Setup Instructions

**Requirements:** a modern browser. Web Crypto requires a secure context (**HTTPS or `localhost`**).

1. Save the code as `index.html` in an empty folder.
2. **Local test:** run `python3 -m http.server 8080` in that folder and open `http://localhost:8080`. (A QR from `localhost` will not open on another device.)
3. **Deploy to Vercel:** push the folder to GitHub, then in Vercel choose *Add New → Project*, import the repo, set Framework Preset to **Other**, leave the build command empty, and deploy. Alternatively run `npx vercel --prod` from the folder.
4. Open the deployed `https://…vercel.app` URL on both devices.

**Two-device demo script**

1. iPad → *Sub-Registry* tab → **Fill Sample Synthetic Data** → **Use built-in synthetic deed** → **Issue & Stamp Deed**.
2. iPhone camera → scan the QR → the green **AUTHENTICATED OFFICIAL DEED** card appears.
3. iPhone → **Use built-in sample deed** → green "FILE MATCHES OFFICIAL RECORD".
4. iPhone → **Test Tampered File** → red **WARNING: FORGED OR ALTERED DEED DETECTED** with both hashes.

## 3. Synthetic Data Compliance

- Every record is **100% synthetic**. Sample values (Mouza "Dhanmondi", Khatian "RS-4092", Dag "1104", owner "Md. Rahim Uddin - Synthetic Record") are fictional.
- The built-in sample deed is a generated text file containing only those synthetic values.
- No real deed, NID number, name, or government record was used or collected.
- Everything runs client-side. Uploaded files are hashed in memory and never transmitted. The only persisted data is the synthetic ledger in the user's own `localStorage`, which can be cleared in the app.
- URL parameters are rendered with `textContent` (never `innerHTML`) to prevent script injection through crafted links.

## 4. Security Model & Known Limitations

- **Integrity, not issuer authenticity.** SHA-256 proves a file is byte-identical to the one that was stamped. In this prototype the QR link itself is not signed, so anyone could craft a link with arbitrary values. The green card shows what the link claims.
- **Production path:** the Sub-Registry signs `{hash, khatian, owner, issuedAt}` with an ECDSA private key held in an HSM or server. The QR carries the signature, and the verifier checks it with the published public key. Records are anchored to the e-Mutation / AC Land systems.
- **Scan variations:** a rescanned or re-photographed legacy deed will hash differently from the original file. A countrywide rollout needs digital-native PDFs issued at registration, or perceptual matching for legacy scans.
- `localStorage` is per-browser and per-device, which is why the QR payload carries the verification data.

## 5. AI Tool Disclosure

- **Claude (Anthropic)** was used to generate the initial `index.html` code and this README.
- **Gemini (Google)** was used to draft the Concept Note.
- No AI tool was given real personal or land-record data. The team is responsible for reviewing and testing all AI-generated output before submission.
