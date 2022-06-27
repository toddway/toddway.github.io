[Firebase](http://firebase.google.com) is a set of backend platform services (owned by Google and closely integrated with the Google Cloud Platform) for building web and mobile apps.  They have SDKs for Android, iOS, web, C++, Unity, Node.js, and Java.  Their generous free tier makes it easy to launch fully functional apps to a modest user base without cost.

Free and unlimited features include: 
Authentication, Analytics, App Indexing, Cloud Messaging, Crash Reporting, Dynamic Links, Notifications, Remote Config

Free features with usage limits: 
Realtime Database, Cloud Functions, Hosting, Storage, Test Lab

There is way too much to cover here.  If you want all the details, their [website docs](https://firebase.google.com/docs/database/) are some of the best I've encountered.  You can also demo many features right from the [web console](https://console.firebase.google.com/).  Here are a few of the highlights that I think can considerably reduce effort and improve quality for app development.

## Sign in/up simply
Firebase Authentication provides email, social, anonymous, and custom sign in methods out of the box.  Accounts can be managed and each method enabled or disabled from the Firebase web console.   Access tokens are based on the JWT feature of OpenID Connect which encrypts portable authorization data in each token.  This means multi-system architectures can share tokens and without the expense of server-to-server callbacks on client requests. 

## Realtime apps are no longer a luxury
The Realtime Database is a NoSQL cloud database with REST and SDK (websocket) support.  Data is synced across all clients in realtime, and remains available even when offline.   Network calls, cache updates, device resources, and intermittent connectivity are managed automatically by the SDK.  Clients simply listen for data changes and react with UI updates.

## Free backend - if needed
For the most part Firebase requires no server-side code, but if you want something to be handled in a trusted backend environment (e.g. push notification logic), there are two relatively simple and free-tier options: Cloud Functions and App Engine.  Cloud Functions is a hosted, private, and scalable Node.js environment where you can run JavaScript code and interact with Firebase.   App Engine is a hosted, private, and scalable environment that supports Java, Python, PHP, and Go.  If you're building an Android app, App Engine is a convenient option because your Java code and IDE tools can be shared between the two.

## Deep links that survive installs
A Dynamic Link is a deep link that can survive the optional app installation step (on Android and iOS) or fall back to a web link if the user is on a desktop machine.  Firebase will generate short links that contain all the details required.  This works great for letting users invite their friends to an app or tracking referral codes. 

## Final thoughts
Serverless isn't the right approach for every situation, but the potential for reduced effort, early feedback, and tighter operational management is compelling - especially in the early life stages of an app.  Firebase is one of the most complete Backend-as-a-Service platforms, but the field is still fairly new.  The competition will evolve and so will your app.  Design principles like [dependency inversion](https://en.wikipedia.org/wiki/Dependency_inversion_principle) can help minimize these future risks.

## Further Reading
- https://scotch.io/bar-talk/a-look-at-the-new-firebase-a-powerful-google-platform
- https://firebase.google.com/docs/
- https://martinfowler.com/articles/serverless.html
