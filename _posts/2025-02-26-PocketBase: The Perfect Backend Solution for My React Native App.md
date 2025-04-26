---
title: "PocketBase: The Perfect Backend Solution for My React Native App"
date: 2025-02-26 08:42:00 -500
categories: [ PocketBase, TypeScript, ReactNative]
tags: [production, backend, database, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/pocketbase-banner.png"
---

I recently started a new personal project, and since I plan it to be quite small scale at least for now, I wanted to find a cheap, easy to use, scalable backend solution. After some exploring, I found PocketBase, and it has greatly exceeded my expectations so far.

Usually, I would say I’m starting a new project because I’m bored and have too much free time. But that is no longer the case - as a full time software engineer, I can assure you the workload is nothing to mess with. Instead, I felt the urge to start a new project because I am a digital nomad, and I find myself frequently being annoyed at how hard it is to find a good working spot. Italians kick me out from their cafes, Indian places rarely have reliable Wi-Fi, and Tunisian spots always have a few smoking enthusiasts. I wanted to build a solution for this, and I thought I might as well do it in React Native - I have experience in React and TypeScript, and have dabbled using Metro Bundler before. I thought “why not? A free extra language for my repertoire”. And we all know recruiters truly love listing those 15 frameworks that you’ve probably never heard of and never will. 

So, the question now was: how can I streamline the setup and deployment of the backend as much as possible? Welcome to this beautiful solution I had never heard about.


## What is PocketBase?

PocketBase is an ✨open-source✨ backend solution built with Go that provides a SQLite database, authentication system, admin dashboard, and a realtime API. It's lightweight, portable, and perfect for smaller projects that might scale later.

I already had a blast using SQLite with HabitTracker, and I didn’t think using relational databases could become any easier! As we will see, PocketBase allows you to create these from the dashboard, defining the schema and everything. Cool indeed!


## Getting Started: Ridiculously Simple Setup
One of the most impressive aspects of PocketBase is how quickly you can get it up and running. The entire initialization process can be completed in just a few minutes:

1. Download the PocketBase executable for your platform
2. Run it with a simple command, should be something like:

```bash
cristinaponcela@Cristinas-MacBook-Air NomadSpot % ./pocketbase serve
2025/04/05 12:31:50 Server started at http://127.0.0.1:8090
├─ REST API:  http://127.0.0.1:8090/api/
└─ Dashboard: http://127.0.0.1:8090/_/
```

Spoiler, it does serve.

3\. Access the admin UI at localhost:8090

That's it! No complex configuration files, no dependency hell, just a single executable ready to serve your app. And a cute UI at the port!

![Desktop View](/assets/img/pocketbase-dashboard.png){: .normal}


## Authentication Made Easy

I don’t think I know a single developer that builds auth from scratch for your usual app. Fear not! PocketBase provides a complete authentication system out of the box (pretty bare minimum if we are going to pick it as a solution, but still, it is so easy to set up and works like a charm).

It supports:
- Email/password authentication
- OAuth providers
- Custom authentication flows
- Password reset functionality
- Email verification

Hours of senseless development saved, happy developer.


## User-Generated Content: Handling Reviews with Ease

My app’s main data is user reviews, and PocketBase makes handling this super straightforward. I created a "spots" collection with just a few clicks in the admin dashboard, defining the schema visually without writing a single line of SQL.

PocketBase's API automatically provides CRUD operations for all collections, with proper validation and error handling. When users submit reviews through my React Native app, I will simply make API calls to PocketBase using their JavaScript SDK:

```typescript
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

// Create a new review
async function submitReview(reviewData) {
  try {
    const record = await pb.collection('spots').create(reviewData);
    console.log(record);
  } catch (error) {
    console.error(error);
  }
}
```
Easyyyyyyyy.

And another super important point, something that SQLite won my heart over - just like @StateObject, PocketBase’s SDK also provides realtime subscriptions, allowing me to update the UI instantly when new reviews are added. This dynamic, responsive experience for users is a necessity in any startup app.


## Development Environment: Two Commands and You're Running

I confess, I have been spoiled by Bending Spoons - at StreamYard, we have the best dev env I’ve ever used. 1 command and we’re running! I didn’t want my project to be any different, though I do have 2:

1. Start PocketBase: ./pocketbase serve
2. Start React Native: npx expo start

This minimalist approach keeps my development process clean and focused. No need to manage multiple terminal windows running different services or remember complex command combinations (or rather copy/paste them from the README lol).


## Scalability: Starting Small, Ready to Grow

While my app is currently small-scale, I always build with growth in mind. PocketBase offers several options for scaling:
• Self-hosting on VPS providers like DigitalOcean or AWS
• Using Docker for containerized deployment
• Backing up and migrating data as needs evolve

The SQLite database is surprisingly capable, easily handling thousands of records with excellent performance. And even if the time comes and I eventually outgrow it, migrating to a larger solution won't be overly complex as long as you build on solid foundations.


## Cost-Effective for Early-Stage Projects

As an indie developer, keeping costs low during development is crucial. PocketBase's self-hosted nature means I'm not locked into monthly subscription fees that eat into my budget before the app generates revenue.

The only cost I incur is the hosting itself, which can be as low as $5/month on basic VPS providers. This affordability gives me the freedom to develop at my own pace without financial pressure. And trust me, I need this leisurely pace when juggling several individual projects, travelling and a job hehe.


## How PocketBase Compares to Alternatives

Before settling on PocketBase, I of course researched a couple popular alternatives. Here's how they stack up:

### Supabase

Supabase is another excellent backend-as-a-service option that many of my colleagues religiously use for client projects. Really, they never use anything else. 

It offers:
- PostgreSQL database (more powerful than PocketBase's SQLite)
- Real-time subscriptions
- Row-level security
- Storage solutions
- Excellent dashboard UI

While Supabase is undoubtedly powerful, I found PocketBase's simplicity more appealing for my specific use case. Supabase requires more configuration and has a steeper learning curve. The PostgreSQL foundation makes it more suitable for complex data relationships and larger scale applications, but this power comes with added complexity.

My colleagues who use Supabase for client work love it, but I suspect PocketBase might actually be better for many of their use cases - especially when rapid development of smaller apps is the priority. I may or may not have decided to go with PocketBase just in case I was right and it’s better, purely so I can troll them. But that was before I actually used it, and was very positively surprised. I'm looking forward to having some friendly debates about this once I've completed my app!


### Firebase

Firebase is the veteran in this space, and I have recommended it in the past too. I still think it is amazing for small-scale development especially, but scalability and costs are its biggest weaknesses when compared to PocketBase. As a newly full-fledged adult, I have to think about these too :( womp womp.

It offers:
- NoSQL database (and now also a SQL option)
- Authentication
- Cloud functions
- Analytics
- Crash reporting
- Push notifications

Its pricing model is… very not great when you scale. While the free tier is generous, costs can escalate quickly once you exceed those limits. Many developers have war stories about unexpected Firebase bills when their apps gained traction.

The vendor lock-in is also significant - once you're deep into the Firebase ecosystem, migrating away becomes increasingly difficult. Trust me, nothing is more stress-inducing than having to migrate your user’s data, especially when the task is made harder by a Google owned database (GCP PTSD has entered the chat).


## Conclusion

Adopting PocketBase for my React Native project is a decision I’m very happy with so far. It strikes the perfect balance between simplicity and capability, letting me focus on building features rather than wrestling with infrastructure. We will see how deployment goes, but I suspect it should be easy enough.

If you're building a mobile app and want a backend solution that won't slow you down, do look into it. I had never heard of it before, but as usual open-source solutions tend to make me a happy kiddo. It has also been an ideal companion for React Native development.

For small to medium projects, it might just be the perfect backend - powerful enough to support your needs, but simple enough to never give you headaches. While alternatives like Supabase and Firebase are also definitely worth considering, PocketBase's simplicity, cost-effectiveness, and scalability make it a pretty good choice.


