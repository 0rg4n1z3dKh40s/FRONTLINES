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

## Global leaderboard

Uses the same Firebase project. Add this rule next to the `saves` one (scores are readable by anyone, writable only by the signed-in owner):

```
match /scores/{id} {
  allow read: if true;
  allow create, update: if request.auth != null
    && request.resource.data.uid == request.auth.uid
    && id == request.auth.uid + '_' + request.resource.data.run
    && request.resource.data.score is int
    && request.resource.data.score >= 0 && request.resource.data.score <= 100000
    && request.resource.data.name.size() <= 24;
}
```

Scores are worked out on the player's device, so this is a friendly board, not a cheat-proof one. Players must be signed in with Google to post; anyone can view.
