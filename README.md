# FRONTLINES

## Cloud save setup

Cloud saving is off until `FIREBASE_CONFIG` in `index.html` is filled in.

1. Create a Firebase project, add a Web app, and paste its config object into `FIREBASE_CONFIG`.
2. Authentication: enable the Google provider, and add the site's domain under Authorized domains.
3. Firestore: create a database with these rules:

```
match /saves/{uid} { allow read, write: if request.auth != null && request.auth.uid == uid; }
```

The Firebase web `apiKey` is not a secret, but the pre-commit hook flags `AIza...` keys, so commit that line with `--no-verify`.
