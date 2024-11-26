---
title: "My First App: Why Flutter and Firebase Make Great MVPs"
date: 2024-11-25 08:42:00 -500
categories: [Flutter, Firebase, Startups, MVPs]
tags: [for beginners, recommendations, efficiency]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/ClothingCAT/Swerv-app.png"
---

We're back with another Weaving Change/ Swerv post! I'm really excited to finally be writing these - this project was definitely pivotal in my career. I arrived with no experience and barely having a clue of how to code, and I left as a Software Engineer. As I mentioned in the [previous pos](https://www.cristinaponcela.com/posts/My-First-App-How-I-Built-a-ML-Model-with-No-Experience/), I spent 9 months working on this project. Out of these, the first 2.5 were spent on ClothingCAT (the ML model), then 2.5 were spent off since I was sitting my final university exams and then went on a sick trip around China and Japan (yey!), and the final 4 were spent as a Full-Stack Developer getting the application ready for the launch on November 16th.

This app was definitely a challenge, mainly because of how small the team was (I spent most of the time as the main developer, though I could always ask for support when needed), and because of how rich the range of functionalities we offer is. 

I developed a bit of everything - making accounts private, adding and accepting requests,  an encrypted end-to-end messaging system, the social aspects of the app (create post, like, comment, like comment, reply to comment, bookmark, etc), searching for users, updating your profile (avatar, bio, etc), built the wardrobe, added fetching and then caching for efficiency, and so many more. I will definitely write a lot more articles about these to showcase how exactly I did each of those, as each one is enough for a full post, but for now I'll focus on giving a good overview of what you can leverage as a startup to create an MVP.

When I joined the team, they had already started building the app in Flutter, which I had a bit of experience in thanks to a parallel project I was working on, and they used Firebase. So it was the time to get my hands dirty and master these technologies.

I have to admit I'm a big fan of Meta's philosophy, "move fast and break things". I have watched John Ousterhout's fantastic Google Talk ["A Philosophy of Software Design"](https://www.youtube.com/watch?v=bmSAYlu0NcY), and while I agree with absolutely everything, I also believe that, especially for startups, tactical tornadoes can have a lot of value. For those who don't want to watch the hour-long talk (which I do really recommend), this is a type of programmer that builds things fast, but perhaps not in the cleanest, most readable, most scalable way, resulting in a lot of investment being required down the line to fix the codebase. And while this is obviously annoying, it is sometimes the only way a startup can stay afloat with limited resources.

I speak from experience - while working for Weaving Change on Swerv, a project that is clearly becoming very successful and making big steps fast in the industry, I was working on a second project, a social media application. I worked on this project for 10 months (started it a month before joining Weaving Change), and I initially worked as a frontend developer. Around 4 months in, I had a viable MVP, but the company suddenly found itself without a backend developer, who left and whose code thus far was unusable for the app. After finding a replacement, the team decided to redesign the app (twice), and the new developer insisted on changing the entire structure of the codebase to use SOLID principles and wanted me to learn BloC and change all the state management. Don't get me wrong, I agree these practices are better, but the company was already struggling financially and this added 6 months of workload to a product we could have finished in the first 4 had they adopted a similar approach to Weaving Change and just used Firebase as an easy backend. Yes, this requires a lot of resources down the line to make a better, scalable backend, but that is exactly why you produce MVPs, get financial support from investors once you have a product to show, and then reinvest this to improve your product. It's a cycle, and watching a company with great potential waste the opportunity to make profit off of an imperfect MVP was very painful.

So if you're a coding beginner, or a startup looking to produce a working MVP fast to get your company off the ground, I would definitely recommend Flutter and Firebase. It requires little coding experience, and a lot less time to produce a working product, and so it is ideal for a low budget startup.

For example, Firebase is great because you don't need a backend developer to design a schema for your database. Just by adding a path, the corresponding data will be either added if the path already exists or create the path and add the data there if not. Of course, this can give many errors and headaches if you're not careful, especially as it is NoSQL, but deleting data is also very easy from the Firebase console's intuitive UI. And before launching, where you don't even have to migrate existing data upon changes, it makes development extremely fast and efficient.

This is how you would add something to Firebase storage:

```dart
// Reference to the destination path in the user's storage
final userAvatarRef = storage.ref('Users/$uid/Avatar/avatar.svg');

// Upload the SVG data to the user's avatar location
await userAvatarRef.putData(
  avatarBytes,
  SettableMetadata(contentType: 'image/svg+xml'),
);
```

And how you would retrieve it:

```dart
final Reference ref = FirebaseStorage.instance
          .ref()
          .child('AvatarTemplates/Avatar/AV01/M.svg');
      return await ref.getDownloadURL();
```

Also, Firebase offers 2 types of storage: Cloud Storage and Realtime Database. I would recommend you use Cloud Storage only for images or other files that are not just data, and Firestore for everything else (especially json-like data). This is because Firestore updates in real time and is both easier and faster to access.  

In terms of Flutter, its easy learning curve and intuitive syntax make it ideal to develop fast MVPs. Plus, my favourite feature in Flutter - the hot reload makes it super fast and dynamic to develop a product. I love you, iOS development, but I really miss this. Or for example, Flutter offers functions to rebuild the widget tree on command, to update the UI accordingly in an easy way. I realize other frameworks have similar features, like GRDB.swift and its DatabaseObserver @StateObject, but I did really enjoy the way this works in Flutter. 

Also, state management is pretty easy to integrate in Flutter. We used Riverpod for this project, and adding a notifier was as easy as:

```dart
final avatarProvider =
    StateNotifierProvider<AvatarModel, AvatarState>((ref) => AvatarModel());
```

Thus, you get rid of 3 of the hardest aspects of building an app: the database, the backend (no need for a server at all with Firebase), and the state management. Also! Firebase handles authentication for you, and you can also use it's Firebase Cloud Messaging (FCM) for notifications (which we definitely took advantage of too).This means you can essentially have a tech startup with an app as long as you know some frontend! Crazy world we live in.
