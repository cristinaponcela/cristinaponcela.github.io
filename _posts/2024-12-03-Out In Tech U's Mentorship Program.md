---
title: "Out In Tech U's Mentorship Program"
date: 2024-12-03 08:42:00 -500
categories: [OiT, SwiftUI, Blog, MVPs]
tags: [for beginners, recommendations, efficiency, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/out-in-tech.jpg"
---

[Out in Tech U’s Mentorship Program](https://outintech.com/mentorship-program/) is an 8-week program designed to help LGBTQ+ youth (17-24 year olds) from all backgrounds start a career in tech. Each participant is matched with an industry professional with many years of experience. Together, you build a project to showcase at the final "graduation", which is extremely helpful for those looking to find a job in the tech industry. As you will quickly learn, interviewers and hirers appreciate a good GitHub prtfolio much more than a resume. Besides, you get to do invaluable networking, and all projects are put in an online book that is then made accessible to a bunch of companies that partner with OiT, which may become promising job prospects.

The program is entirely free, fully virtual, and open to residents across the UK and USA. Typically work with your mentor for 2 hours per week and then are left to build your project independently, though you can always ask for help as everyone is super friendly. My kind of deal. 

The team is great and they have poured so much effort and time into making this as valuable as possible for us. Thanks to them! It’s more than mentorship; it’s a launchpad for the next generation of queer tech talent. I cannot recommend it enough. 


## My Experience

Over the past eight weeks, I had the incredibleluck of being paired with [Peter Steiberger](https://x.com/steipete). He has truly gone above and beyond, not just to help me learn but to boost my career and provide invaluable advice.


## Building my Online Persona

I love coding and am a bit of a workaholic, so the actual process of learning a new language and building a personal project is not something I struggle too much with. But what I hadn't realized, and is the kind of advice that only a tenured professional can give you, is that I was sorely lacking an online persona. I have done cool things and have experience in some sought-after programming languages, but I wasn't selling myself properly. I didn't have a personal website, any open source contributions or a [Twitter account](https://x.com/cristinaponcela), which I now consider the LinkedIn of Software Engineers.

The first step was getting these set up - the very blog you're reading is a part of my personal website (duh), and I've tried my best to add regular updates on whatever I'm working on. Counting this one, I've posted 10 articles in these 8 weeks, and uploaded 3 [gists](https://gist.github.com/cristinaponcela). 

![Desktop View](/assets/img/my-blog.png){: .normal}


![Desktop View](/assets/img/gist.png){: .normal}

One of my blog posts, the one about [Xcode Completion vs. GitHub Copilot](https://www.cristinaponcela.com/posts/Xcode-Completion-vs.-GitHub-Copilot-Which-Is-Better-for-Improving-Your-Coding-Efficiency/), was referenced in an [issue of iOS Dev Weekly](https://iosdevweekly.com/issues/686#start)! This is a great way to get discovered by tech professionals, and I am super grateful for [Dave Verwer](https://x.com/daveverwer) for considering my work. 

![Desktop View](/assets/img/iosdevweekly.png){: .normal}


## Starting a New Job with Advice

For the first half of the internship, I was working for 2 startups in the UK, and then I got a job at Bending Spoons as a Software Engineer, which I have recently started. Going through this transition with the support of a mentor was invaluaable - Peter kept giving me advice on how to improve my work, what I should ask for, contract guidance, and so on. 

It made the entire process way more manageable and less scary. Not to mention that, thanks to his tip on getting started with MacOS and iOS development, I was way more prepared when I started my new job.


## Building HabiTracker, my First App in Swift

Peter has a background in iOS development, and as such he recommended I used the project to learn Swift and get acquainted with all things Apple. As I trust him with my life, I invested in a MacBook Air and got to work.

Before joining the program, I had never used SwiftUI or [GRDB](https://github.com/groue/GRDB.swift) — a super useful Swift database library. However, with Peter's guidance, I have built an app called HabitTracker, which helps users manage their daily habits in an efficient and balanced way.

> You can find the full source code in my GitHub repo [HabitTracker](https://github.com/cristinaponcela/HabitTracker)!
{: .prompt-tip }

This idea came about because I work remotely and travel a lot, so keeping up with friends has been a challenge. I ended up creating a llist of friends that I wanted to call regularly - since I had around 30 people on this list, this made for a perfect monthly schedule to have a monthly call with each friend. The more I explained this to my friends, the more they kept saying it was a great idea and they had started doing the same. And as any geeky developer does, I decided to automate it.

But since this felt a bit bare for an app, I decided to kill two birds with one stone and solve my second biggest problem due to this lifestlye - keeping my work/life balance in check. 

HabitTracker allows you to add events and friends to a calendar, with a UI very similar to that of the Apple Calendar but way more intuitive and easy to use. For example, you can type event names directly into the Calendar View, as each hour is a text field. You can also easily change to Weekly and Monthly Views by using two fingers to zoom out. Frequent events and calls with your friends then get automatically added, and you get reeminded so you're held accountable for maintaining your habits. In all three of the Daily, Weekly and Monthly Views, you can intuitively see how your work/life balance is doing by looking at the charts, separated by the categories you create.

![Desktop View](/assets/img/HabitTracker/DailyView.png){: width="140vw" height="210vw" }{: .left}

![Desktop View](/assets/img/HabitTracker/WeeklyView.png){: width="140vw" height="210vw" }{: .normal}

![Desktop View](/assets/img/HabitTracker/Monthly.png){: width="140vw" height="210vw" }{: .right}

![Desktop View](/assets/img/HabitTracker/AddFriend.png){: width="140vw" height="210vw" }{: .left}

![Desktop View](/assets/img/HabitTracker/ScheduledCall.png){: width="140vw" height="210vw" }{: .right}



## GRDB's Thread Safety

As all fun projects are, this was incredibly annoying at times. I had many issues when dealing with the database, and I quickly met my nemeses: "Fatal error: Database methods are not reentrant" and "Fatal error: Database was not used on the correct thread.". 

A lot of big words incoming, but I swear it's easier than it sounds:

Imagine you have a function that tries to update a shared resource (like a variable, file, or database). If the function is called again before the first call finishes, it might lead to unexpected behavior (e.g., the resource getting overwritten or corrupted). This is called *reentrancy* — when the function enters itself again before it finishes executing. 

SwiftUI avoids this by structuring code with *single transaction models*, ensuring that operations like database writes happen *sequentially*. By passing the database reference (db) between parent and child functions, SwiftUI prevents reentrancy by ensuring that tasks don’t overlap. 

This synchronization ensures only one task modifies shared resources at a time, preserving data integrity and preventing race conditions. SwiftUI's approach keeps state updates ordered, making the app predictable and stable.

I discovered this the hard way. Thinking I was about 2 days away from release, I added the call scheduling function within the main fetching and displaying function, as the last step for events and calls to be automatically added and displayed. Everything broke.

It turns out, you can't add nested `db.write` or `db.read` read operations withought initializing a `dbQueue` variable, and then passing the `db: Database` variable to all child functions to ensure you are performing operations on the correct (in this case, the only) thread. This is because SQLite (which I used with GRDB) is not thread-safe by default for read/write operations. The structure should be as follows:

```swift
private func parentFunction() {
    do {
        // Perform the write block
        try DatabaseManager.shared.dbQueue.write { db in
            // Clear the events data before fetching new ones
            eventsByDate.removeAll()

            // Call child functions with the db instance
            try childFunction1(db: db)
            try childFunction2(db: db)
            try childFunction3(db: db)
        }
    } catch {
        print("Error performing database operations: \(error)")
    }
}
```

```swift
private func childFunctionN(db: Database) throws {
    for date in daysInMonth {
        do {
            let events = try fetchEvents(for: date, db: db)
            eventsByDate[date] = events
        } catch {
            print("Error fetching events for date \(date): \(error)")
            throw error // Propagate the error back to the parent function
        }
    }
}
```

## GRDB's Database Observers

And my personal favorite feature of GRDB: you can declare a `@StateObject` variable to be a `DatabaseObserver`, so that the UI automatically updates and displays changes as they occur in the database.

For example, we would start by defining the model of the type of data we want to observe, which should correspond to the database table we will observe:

```swift
import GRDB

struct Task: Identifiable, Codable {
    var id: Int64
    var name: String
    var isComplete: Bool
}

extension Task {
    static let databaseTableName = "tasks"
}
```

Then, we create the `DatabaseObserver` class:

```swift
import SwiftUI
import GRDB

class DatabaseObserver: ObservableObject {
    @Published var tasks: [Task] = []
    
    private var dbQueue: DatabaseQueue
    
    init(dbQueue: DatabaseQueue) {
        self.dbQueue = dbQueue
        fetchTasks()
    }
    
    func fetchTasks() {
        do {
            try dbQueue.read { db in
                // Fetch tasks from the database
                self.tasks = try Task.fetchAll(db)
            }
        } catch {
            print("Error fetching tasks: \(error)")
        }
    }
    
    // Add more methods here for inserting, updating, or deleting tasks
    func addTask(name: String) {
        do {
            try dbQueue.write { db in
                let task = Task(id: 0, name: name, isComplete: false)
                try task.insert(db)
            }
            fetchTasks() // Reload tasks after adding one
        } catch {
            print("Error adding task: \(error)")
        }
    }
}
```

And finally, we call this in the View or wherever in the UI we want changes to automatically display:

```swift
import SwiftUI

struct TaskListView: View {
    @StateObject private var databaseObserver = DatabaseObserver(dbQueue: DatabaseManager.shared.dbQueue)
    
    var body: some View {
        VStack {
            List(databaseObserver.tasks) { task in
                HStack {
                    Text(task.name)
                    Spacer()
                    if task.isComplete {
                        Image(systemName: "checkmark")
                    }
                }
            }
            
            Button(action: {
                // Add a new task when button is clicked
                databaseObserver.addTask(name: "New Task")
            }) {
                Text("Add Task")
            }
            .padding()
        }
        .onAppear {
            // Fetch tasks when the view appears
            databaseObserver.fetchTasks()
        }
    }
}
```

And that's a wrap! No need to set up any more interactions with the database!


## Bonus

I have been trying out different IDEs throughout the program, changing environment every 2 weeks or so. I used VSC, Xcode with Xcode Completion, Xcode with GitHub Copilot, and now I'm really enjoying Windsurf and it's in-built AI Cascade. I find it to be my favorite inline AI suggester yet! I will write more on this when I get around to tryig Cursor too, which is next on the list B) 

This mentorship was an excellent exercise in learning, debugging, using docu and StackOverflow, and essentially doing what a developer does every day. It was an extremely entertaining and challenging project, and I feel like I have learned in 8 weeks what could take months or years thanks to Peter's expert guidance. I'm looking forward to working with Peter again in the future to continue learning and growing as a developer.