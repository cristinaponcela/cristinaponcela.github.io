---
title: How To Create a Dynamic Search Bar in Flutter
date: 2024-10-29 14:59:00 -500
categories: [Flutter, gist, Efficient Functions]
tags: [flutter, for beginners, recommendations, efficiency]
image: "https://raw.githubusercontent.com/cristina22999/cristina22999.github.io/refs/heads/main/assets/img/search.jpeg"
---

As you may know, Flutter is known for its intuitiveness and ease of use. One of my favourite features is the `onChanged` functionality, which lets you create dynamic and efficient user interfaces with minimal code. Particularly, I tend to use it to create search bars that update results dynamically as you type.


## Why `onChanged` is Cool

1. **Real-Time Updates**: `onChanged` automatically triggers the given function every time the text in `TextField` changes.

2. **Minimal Code Overhead**:In many programming languages, setting up a dynamic search bar often requires multiple steps, like adding listeners, boilerplate code, managing debouncing and handling input. Flutter’s `onChanged` functionality handles this with just one line of code; it is an accessible and elegant solution.

3. **Efficient Resource Use**: Because `onChanged` works with Flutter’s state management, it allows us to handle search queries and UI updates with minimal resource consumption. This reduces unnecessary processing, as `onChanged` ensures the state only updates when a user changes the text input, keeping the app’s memory and processing needs low. No need to make a billion HTTP requests!

4. **Better User Experience**: results being updated in real-time makes your app feel responsive and interactive.


Here is how I implement it:

> Add `onChanged: _filterUsernames,` in the `TextField`, then define the function as below.
{: .prompt-tip }

```dart
// Function to update search results by filtering input
void _filterUsernames(String query) {
    setState(() {
      filteredUsers = dummyUsers
          .where((user) =>
              user['username']!.toLowerCase().startsWith(query.toLowerCase()))
          .toList();
    });
  }
```

Check out my full solution in my [gist](https://gist.github.com/cristinaponcela/3259485155a7e4aa783551caf352b3c6).