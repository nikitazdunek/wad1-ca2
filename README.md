# Console Archive

A web app for cataloguing gaming consoles by manufacturer. Users sign up, log in and keep their own collection, with manufacturer logos uploaded to Cloudinary.

Built for Web App Development 1 at SETU Waterford (Assignment 2, May 2026). The first version is in [wad1-ca1](https://github.com/nikitazdunek/wad1-ca1).

## Features

* Sign up, log in and log out
* A dashboard showing only your own manufacturers, with search and A to Z sorting
* Add and delete manufacturers, with an optional logo upload
* Add, edit and delete consoles for each manufacturer
* A stats page with totals and the average number of consoles per manufacturer

## Built with

Node.js, Express 5, Handlebars, LowDB (JSON file storage), Cloudinary and Fomantic UI

## Run it locally

```bash
git clone https://github.com/nikitazdunek/wad1-ca2.git
cd wad1-ca2
npm install
cp .env.example .env    # then add your Cloudinary details
npm start
```

Open http://localhost:3000 and sign up for an account.

## Security review

After submitting the assignment I went back over the app looking for security problems, and tested each one against a local copy. It was built to the course brief rather than for real use, and the code is left as submitted, so the problems are written up here rather than fixed.

| Problem | What an attacker can do | Fix | OWASP 2025 |
|---|---|---|---|
| The login cookie is just the user's email, unsigned (`controllers/accounts.js`) | Set the cookie to someone else's email in the browser and be logged in as them, no password needed | Server side sessions with a random ID (`express-session`), with the cookie set `httpOnly` and `sameSite` | A07 Authentication Failures |
| Passwords stored and compared in plain text (`models/user-store.json`) | Anyone who gets hold of the JSON file has every password | Hash with bcrypt or argon2 | A07, A04 Cryptographic Failures |
| Routes that delete, add or edit data never check the login (`dashboard.deleteCollection`, `controllers/console-details.js`) | Delete or change data without logging in, just by visiting a URL | Check the session on every route that changes data | A01 Broken Access Control |
| No ownership check on collections (`consoleDetails.createView`) | Open another user's collection by changing the ID in the URL (IDOR) | Compare the collection's `userid` with the logged in user | A01 Broken Access Control |
| Deletes run over GET (`routes.js`) | A link or image on another site can trigger a delete (CSRF) | Use POST with a CSRF token | A01 Broken Access Control |
| Weak passwords allowed (`accounts.register`) | Sign up with an empty password or an email already in use; the seeded demo account is admin / admin | Validate input, set a minimum password length, reject emails already registered | A07 Authentication Failures |

What it does get right: Handlebars escapes output by default, which protects the views against stored XSS, and the Cloudinary keys live in `.env`, which is kept out of the repo.
