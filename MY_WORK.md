# 📝 MY_WORK: Student Information, Development Log, Reflection & Answers

> This is the **only file** your instructor reads to grade Parts 3 and 4 (documentation and video). Everything you write here must be **in your own words**.

---

## 🛑 STOP: Read This Before You Do Anything Else

> ### 1️⃣ Read the whole `README.md` first
> The `README.md` in this repository contains the full instructions: how this assignment continues Assignment 1, the Chapter 5 recap, class descriptions, feature specifications, question prompts and the video script. **If you skip it, you will lose marks.**
>
> ### 2️⃣ Understand the full code before answering any question
> Open `SchedulerSimulationSync.java` and read it **from top to bottom**. You must be able to explain what `SharedStats`, `ReadyQueue`, `CpuWorker`, `RaceWindow.pause()`, `ReentrantLock` and `Semaphore` do **before** you write a single answer in Parts B and C. **Run the original starter first** (it is intentionally broken) and **save its output**: you need it for Question 1 of Part C.
>
> ### 3️⃣ Use only the code of this repository
> Work **only** on `SchedulerSimulationSync.java` from this repository. Do **not** copy or paste code from your previous assignments.
>
> ### 4️⃣ Commit many times, not once
> A single commit, or all commits made in the last hour, costs you **-0.5 mark**. See the [Commit Rules](#-commit-rules-mandatory) below.

**How to use this file:**
1. Fill in your **Student Information** (below) right now.
2. Follow the steps in the **Work Roadmap** in order.
3. Update the **Development Log** *every time* you work on the assignment, not at the end.
4. Do not delete any section header. Replace the `[...]` placeholders with your own text.

---

## 👤 Student Information

> ⚠️ **WARNING:** Fill this in first. Your name and ID must match the student ID you set in `SchedulerSimulationSync.java` (line 304) and the one you say in your video.

| Field | Your Answer |
|-------|-------------|
| **Full Name** | Aya Ahmad Alali |
| **Student ID** | 445052784 |
| **University Email** | 445052784@std.psau.edu.sa |
| **GitHub Username** | Ayya-coder |
| **Repository Link** | https://github.com/Ayya-coder/OS-Assignment3-Aya-Alali |
 
---

## 🎥 Video Link

**Video Link**: [Paste your video link here]

> ⚠️ **WARNING:** The video must be **publicly accessible** ("Anyone with the link can view") on **Google Drive**, **YouTube (Unlisted or Public)** or any other cloud file-sharing system. A private, restricted or broken link counts as a **missing video (-1 mark)**.
>
> 💡 **TIP:** Open the link in a **private/incognito window** before you submit. If it asks you to log in or request access, it is not public.
>
> 📌 **NOTE:** The link goes in **this file only** (`MY_WORK.md`), **not** in `README.md`. Name your video file `StudentID_Assignment3_Demo.mp4`. It must last **2 to 3 minutes**.

---

## 🗺️ Work Roadmap (follow in this order)

| Step | What to do | Where | Marks |
|:----:|------------|-------|:-----:|
| 0 | Read `README.md`, read the full code, **run the original starter and save its output** (it fails on purpose) | Your IDE | – |
| 1 | Fork, rename, keep the repo **PUBLIC**, set your student ID (line 304), **commit** | GitHub + code | Part 1 (1) |
| 2 | Feature 1: Mutex lock on the ready queue, **commit** | Code | Part 2 (0.25) |
| 3 | Feature 2: Mutex lock(s) on the shared statistics, **commit** | Code | Part 2 (0.25) |
| 4 | Feature 3: Counting semaphore instead of busy waiting, **commit** | Code | Part 2 (0.5) |
| 5 | Development Log (5+ entries, different dates) | This file, Part A | Part 3 (0.5) |
| 6 | Reflection (4 questions) | This file, Part B | Part 3 (0.5) |
| 7 | Technical Answers (4 questions) | This file, Part C | Part 3 (0.5) |
| 8 | Record the video, upload it, paste the link above | Video + this file | Part 4 (1.5) |
| 9 | Final check, then submit the repo link on Blackboard | Blackboard | – |

> 💡 **TIP:** Tick each step off as you go. Do not leave the log, the reflection or the video for the last day.

---

## 🔁 Commit Rules (MANDATORY)

> ### ⚠️ MANY COMMITS ARE REQUIRED. A single bulk commit is penalized (-0.5 mark).

**Minimum: 4 meaningful commits. Aim for 7 or more.**

| # | Commit | Example message |
|:-:|--------|-----------------|
| 1 | Student ID set | `Set my student ID: 441234567` |
| 2 | Feature 1: Ready queue lock | `Feature 1: Protected the ready queue with a ReentrantLock` |
| 3 | Feature 2: Statistics locks | `Feature 2: Added locks to the shared statistics counters` |
| 4 | Feature 3: Semaphore | `Feature 3: Replaced busy waiting with a counting semaphore` |
| 5 | Development log entries | `Docs: Added development log entries 1-3` |
| 6 | Reflection and answers | `Docs: Completed reflection and technical answers` |
| 7 | Video link | `Docs: Added demo video link` |

**Rules:**
- ✅ **One commit per feature.** Do not put all three features in one commit.
- ✅ **Commit after each work session**, and after each part of this file.
- ✅ **Spread your commits over different dates.** Not all in one day.
- ❌ **Do not make all commits in the last hour** before the deadline (November 14, 2026).
- ❌ **No vague messages** like `done`, `update` or `final version`.

> 💡 **TIP:** Your commit history is checked and you **show it in your video** (at least 4 commits visible). Your development log dates should match your commit dates.
>
> 💡 **TIP:** **Use VS Code** (see *Recommended Development Environment* in `README.md` for the full setup). Sign in to GitHub in VS Code, then commit from the Source Control panel (Ctrl+Shift+G) → stage → write a message → Commit → Sync/Push. You can edit and commit this file the same way. **Pushing** matters: commits that are not pushed to GitHub are invisible to the instructor.

---

# Part A: Development Log (0.5 mark)

> ⚠️ **WARNING:** Minimum **5 entries**, spread over **different dates**. Five entries written on the same day, or written all at once at the end, will lose marks and look like a copy. Entry dates should be **between the start of the assignment and the deadline (November 14, 2026)**.
>
> 💡 **TIP:** Write an entry at the **end of each work session**, while you still remember what happened. It takes 5 minutes.
>
> 💡 **TIP:** Be specific. "Worked on the code" is a weak entry. "Added a `ReentrantLock queueLock` to `ReadyQueue` and wrapped the body of `poll()` in `lock()` / `try` / `finally { unlock() }`" is a strong one.
>
> 📌 **NOTE:** Each entry needs: date and time, what you did, details, challenges, solution, and time spent. Real challenges are fine (and expected). Do not invent fake ones.

## Example Entry (do not copy it, write your own)

### Entry 1 - [November 2, 2026, 4:30 PM]
**What I did**: Forked the repository, set my student ID, and ran the original starter

**Details**:
- Forked the starter repository and renamed it
- Changed student ID on line 304 to my actual ID (441234567)
- Ran the original program: 4 CPUs started, three of them picked `P1`, then `IllegalThreadStateException` and the watchdog stopped the program
- Saved the output in a text file for Question 1
- Committed and pushed: `Set my student ID: 441234567`

**Challenges**: I did not understand why the same process was picked by three CPUs

**Solution**: Read `ReadyQueue.poll()` again and found the `RaceWindow.pause()` between reading the head and removing it

**Time spent**: 45 minutes

---

## Your Development Log

### Entry 1 - [October 26, 2026, 8:00 PM]
**What I did**: Logged into GitHub, forked the project repository, and changed the assigned ID in the code.

**Details**: Successfully authenticated with GitHub and created a fork of the original repository. Cloned the forked repository locally, opened the project in Visual Studio, and updated the assigned student ID within the code files. Verified that the changes were saved correctly and prepared the repository for further development.

**Challenges**: Making sure the repository was connected correctly and the ID was changed in the correct file.

**Solution**: Checked the project instructions and verified the changes before saving and committing the code.

**Time spent**: 30 minutes.

---

### Entry 2 - [October 29, 2026, 10:30 PM]
**What I did**: Implemented the required concurrency feature and modified the synchronization mechanism using the One Lock Per Variable approach. Tested the program execution to ensure correct thread behavior and data consistency.

**Details**: Reviewed the existing code structure and identified shared variables accessed by multiple threads. Replaced the previous synchronization strategy with independent locks for each variable to reduce contention and improve parallel execution. Executed several test runs and monitored the output to verify that all updates were performed correctly.

**Challenges**: Initially, the program continued to show synchronization issues due to incorrect lock usage. It was also necessary to determine which variables should have separate locks and ensure that all critical sections were protected properly.

**Solution**: Assigned a dedicated lock to each shared variable and updated the critical sections accordingly. Re-tested the application multiple times and verified that the race conditions were eliminated and the outputs became consistent.

**Time spent**: 3.5 hours.

---

### Entry 3 - [October 30, 2026, 7:38 PM]
**What I did**: Completed the Reflection Part of the assignment by answering all reflection questions related to synchronization concepts, challenges faced during development, real-world applications, and lessons learned from the project.

**Details**: Reviewed the completed implementation and reflected on my experience working with synchronization, locks, race conditions, and semaphores. Wrote detailed answers based on the challenges encountered during the project and the solutions used to overcome them.

**Challenges**: It was difficult to summarize the technical concepts clearly while connecting them to my actual experience in the project. I also needed to explain my reasoning and learning process in my own words.

**Solution**: I revisited the project code, reviewed the synchronization mechanisms that were implemented, and reflected on the debugging and testing process. This helped me provide accurate and meaningful answers to all reflection questions.

**Time spent**: 1 hour.

---

### Entry 4 - [October 31, 2026, 5:44 PM]
**What I did**: Completed the synchronization and concurrency analysis questions for the CPU scheduling project.

**Details**: Tested the original starter code and compared the verification results with the corrected version. I identified lost updates caused by race conditions, documented the critical sections and lock granularity decisions, and explained how semaphores were used to synchronize access to the ready queue. I also analyzed the Producer-Consumer model and deadlock scenarios.

**Challenges**: Understanding how a lost update occurs between multiple CPUs and explaining the behavior of semaphores and deadlocks in a clear way. It was also challenging to determine the correct lock granularity and justify using separate locks for independent shared variables.

**Solution**: Used one lock per independent shared statistic and a separate lock for the ready queue to reduce contention. Replaced busy waiting with semaphores to eliminate unnecessary CPU usage and followed proper synchronization ordering to avoid deadlocks. Verified the final implementation by ensuring all expected and observed values matched.

**Time spent**: 2 hours.

---

### Entry 5 - [Date and Time]
**What I did**:

**Details**:

**Challenges**:

**Solution**:

**Time spent**:

---

### Entry 6 - [Optional - Date and Time]
**What I did**:

**Details**:

**Challenges**:

**Solution**:

**Time spent**:

---

## Development Log Summary

> 💡 **TIP:** Fill this in **last**, after all entries are written.

**Total time spent on assignment**: [X hours]

**Most challenging part**:

**Most interesting learning**:

**What I would do differently next time**:

---

# Part B: Reflection (0.5 mark)

> 🛑 **STOP:** Do **not** start this part until you have read the `README.md`, read the **entire** `SchedulerSimulationSync.java`, run it, and finished the three features.
>
> ⚠️ **WARNING:** Each answer must be **5 to 7 sentences**, in **your own words**. Copied or AI-generated answers without understanding get **0 marks for the whole assignment**. You may be asked to explain them in person.
>
> 💡 **TIP:** Mention concrete things you actually did: a method you wrote, an error you hit, a line of output you saw. Generic answers score low.
>
> 💡 **TIP:** Draft your answer in a few bullet points first, then turn them into sentences.

## Question 1: What did you learn about synchronization?

> 💡 **TIP:** Talk about race conditions (read-modify-write), critical sections, mutual exclusion with `ReentrantLock` and `try/finally`, the role of a counting semaphore, and what surprised you when the starter failed.

**Your Answer:** *(5-7 sentences)*

Through this assignment, I learned that synchronization is necessary when multiple threads access and modify the same shared data. Without proper synchronization, race conditions can occur because different threads may read and update a variable at the same time, leading to incorrect results. I also learned how critical sections should be protected using ReentrantLock to ensure mutual exclusion and prevent conflicts between threads. Using try/finally was important because it guarantees that a lock is released even if an exception occurs. In addition, I understood how a counting semaphore can limit the number of threads that are allowed to access a resource concurrently. What surprised me most was how the starter program produced inconsistent output due to thread interference, even though the code looked correct at first glance.

## Question 2: What was the most challenging part of this assignment?

> 💡 **TIP:** Pick **one** specific challenge (understanding why the starter failed, choosing the lock granularity, the initial value of the semaphore, the shutdown of the CPUs, Git, the video) and say *why* it was hard.

**Your Answer:** *(5-7 sentences)*

The most challenging part of this assignment was choosing the appropriate lock granularity. At first, I was unsure whether to use a single lock for the entire shared state or separate locks for each variable. After analyzing the code and discussing the independence of the shared variables, I decided to use the one-lock-per-variable approach. The difficult part was making sure that every critical section used the correct lock without introducing new synchronization problems. I tested the program several times and carefully checked the outputs to verify that race conditions were removed. This experience helped me understand how lock granularity affects both correctness and performance in concurrent programs.

## Question 3: How did you overcome the challenges you faced?

> 💡 **TIP:** Describe your method: reading the slides and the documentation, adding `System.out.println` to debug, running the program many times, re-reading the README, testing after each small change, asking for help.

**Your Answer:** *(5-7 sentences)*

I overcame the challenges by working on the project step by step instead of changing many things at once. I re-read the README several times to make sure I understood the requirements before modifying the code. When I was unsure about the synchronization design, I compared different approaches and analyzed whether the shared variables were independent or not. I also ran the program repeatedly after each small change and checked the output to see if the behavior was correct. Whenever I encountered a problem, I reviewed the code carefully and asked for help to better understand the issue. This process helped me identify mistakes early and gradually reach a stable and correct solution.

## Question 4: Where can synchronization concepts be applied in real-world applications?

> 💡 **TIP:** Use real applications you know (banking and money transfers, ticket or seat booking, a shared counter on a website, a print queue, a database) and connect each one to what you built here (shared data, critical section, lock or semaphore).

**Your Answer:** *(5-7 sentences)*

Synchronization concepts are widely used in real-world applications where multiple users or processes access shared resources at the same time. For example, in an online ticket booking system, synchronization prevents two users from reserving the same seat simultaneously. In banking applications, locks are used to protect account balances during deposits and withdrawals so that money is updated correctly. Databases also use synchronization techniques to maintain data consistency when many transactions occur at once. This assignment helped me understand how locks and critical sections protect shared data from race conditions. The use of semaphores can also be applied to limit access to a resource, similar to controlling how many users can use a service at the same time.

### Optional: What would you like to learn more about?

[Any topics related to synchronization or operating systems that you're curious about?]

I would like to learn more about deadlocks and how operating systems detect and prevent them. I am also interested in learning about advanced synchronization techniques that are used in large-scale applications. Understanding how modern operating systems manage threads and CPU scheduling would help me deepen my knowledge of concurrent programming

### Optional: How confident do you feel about synchronization concepts now?

[Beginner / Intermediate / Confident. What do you understand well? What needs more practice?]

I would describe my confidence level as intermediate. I now understand race conditions, critical sections, locks, and semaphores much better than before starting the assignment. I am comfortable applying basic synchronization techniques, but I still need more practice with choosing the best synchronization strategy and debugging complex multithreaded programs.

### Optional: Feedback on the assignment

[Any comments? Was it helpful? Too easy or hard? Suggestions?]

The assignment was challenging but very helpful for understanding synchronization concepts in a practical way. Implementing the features and testing the program helped me connect the theoretical concepts from class with real code. The project required careful thinking and debugging, which improved my problem-solving skills. Overall, I found the assignment valuable and educational.

---

# Part C: Technical Answers (0.5 mark)

> 🛑 **STOP:** You cannot answer these questions without understanding the code. Re-read `SchedulerSimulationSync.java` and **run it** first. Your answers must reference **your own code and your own output** (your student ID makes your output and your expected values unique).
>
> ⚠️ **WARNING:** Each answer must be **3 to 5 sentences**, with specific examples from your code or output. Use correct terms: race condition, critical section, mutual exclusion, mutex lock, semaphore, busy waiting, deadlock, producer, consumer.
>
> 💡 **TIP:** Keep the output of the **original starter** and the output of your **final version** in text files or screenshots, so you can copy real snippets for Question 1.

## Question 1: Race Condition in Your Own Output

**Question**: Run the **original starter** (before any fix) and paste the **verification block** of that run. Which value was wrong (expected vs observed)? Explain **step by step**, with two named threads/CPUs, how **one lost update** happens on one of the shared variables (read, `RaceWindow.pause()`, write). Why can the same bug happen on a real machine even without `RaceWindow`, and why is it hard to detect?

> ⚠️ **WARNING:** The snippet must come from **your own run** (with your student ID), not from a classmate or from this README.
>
> 💡 **TIP:** Use a small timeline: *CPU-1 reads `contextSwitchCount` = 12 → CPU-2 reads 12 → CPU-1 writes 13 → CPU-2 writes 13 → one increment is lost.* If your starter run crashed before printing numbers, run it several times (or paste the exception and the watchdog message and explain it).

**Your Answer:** *(3-5 sentences)*

The wrong value in my original run was the Context switches count, which was expected to be 30 but observed as 29. A lost update can occur when CPU-1 reads a shared variable value (for example 12) and is paused before writing the incremented value. While CPU-1 is paused, CPU-2 reads the same value 12 and writes 13. When CPU-1 resumes, it also writes 13, causing one increment to be lost because both CPUs updated the same old value. This bug can happen on a real machine even without RaceWindow because thread scheduling and context switches are unpredictable, and it is difficult to detect since the program may sometimes produce correct results and sometimes fail.

Verification block of the **original starter**:

<img width="671" height="215" alt="Output befor any Change" src="https://github.com/user-attachments/assets/62d18291-ff69-4a32-8c71-f24250aa9be9" />



Verification block of **my final version**:

<img width="599" height="150" alt="Output after Feature 3" src="https://github.com/user-attachments/assets/41add7bf-e8ea-443d-9e84-c01814996ed6" />



**Explanation of the timeline:**
[Explain the lost update step by step.]

CPU-1 reads contextSwitchCount = 28 but pauses before writing. CPU-2 then reads the same value (28), increments it, and writes 29. When CPU-1 resumes, it also increments its old copy and writes 29. One increment is therefore lost, so the final value becomes 29 instead of the expected 30. This race condition can happen on real machines because thread scheduling is unpredictable, making the bug difficult to reproduce consistently.

## Question 2: Critical Sections and Lock Granularity

**Question**: List the **critical sections** you protected (variables and methods) and the lock(s) you used. **Lock granularity:** did you use **one lock** for the three statistics or **one lock per variable**? Why, and what is the trade-off? Why is the **ready queue** protected by a **different** lock than the statistics? Explain how your solution provides **mutual exclusion**, and discuss **progress** and **bounded waiting** (a default `ReentrantLock` is not fair: what does that imply?).

> 💡 **TIP:** The three counters are independent: does updating `contextSwitchCount` need to block an update of `totalServiceTime`? Also think about what could go wrong (deadlock, or long waiting) if the queue lock and the statistics lock were the **same** lock and a thread held it while doing something slow.

**Your Answer:** *(3-5 sentences)*

I protected the shared statistics variables (contextSwitchCount, completedProcesses, and totalServiceTime) using separate locks, with one lock per variable. I also used a different lock for ready queue operations because queue access is independent from statistics updates. This lock granularity reduces contention since a thread updating one statistic does not block updates to other statistics. The separate queue lock guarantees mutual exclusion when adding or removing processes from the ready queue, while keeping the scheduler independent from statistics operations. Progress is preserved because threads eventually acquire the required lock, but bounded waiting is not strictly guaranteed since the default ReentrantLock is non-fair and may allow a thread to acquire the lock multiple times before others.

| Critical section (variable / method) | Lock used | Why it needs protection |
|---|---|---|
| [e.g., `ReadyQueue.poll()`] | [lock name] | [reason] |

| contextSwitchCount update | contextSwitchLock | Prevent lost updates when multiple CPUs increment the counter simultaneously |
| completedProcesses update | completedProcessesLock | Ensure completed process count remains accurate |
| totalServiceTime update | totalServiceTimeLock | Prevent concurrent writes from corrupting the accumulated time |
| ReadyQueue.offer() | readyQueueLock | Prevent concurrent modifications of the queue structure |
| ReadyQueue.poll() | readyQueueLock | Ensure only one thread removes a process at a time |

## Question 3: Busy Waiting vs Semaphore

**Question**: Describe the **busy waiting** of the starter and its cost (give the `Busy-waiting polls` number from your **starter** run). What does the **value of your semaphore** represent? What is its **initial value** and why? What is its value right after the `numProcesses` initial `add()` calls? Why does the last CPU call `release(totalCPUs)` at the end, and what would happen if it did not?

> 💡 **TIP:** Think about what a CPU does when the queue is empty in the starter, and what a CPU does in your final version (is it running, ready or blocked?). Also check what happens to a CPU blocked in `acquire()` when no process will ever be added again.

**Your Answer:** *(3-5 sentences)*

In the starter version, the CPUs used busy waiting by repeatedly checking the ready queue when it was empty, wasting CPU time and producing 1099 busy-waiting polls. In my final version, I replaced busy waiting with a semaphore so CPUs block in acquire() until a process becomes available. The semaphore value represents the number of processes available in the ready queue. Its initial value is 0 because the queue is initially empty, and after the numProcesses initial add() calls, its value becomes equal to the number of added processes. The last CPU calls release(totalCPUs) to wake up any CPUs blocked in acquire() after all work is finished; otherwise, some CPUs could remain blocked forever and the simulation would never terminate correctly.
Using a semaphore improves efficiency because blocked CPUs do not consume processor time while waiting, unlike busy waiting where CPUs continuously poll the queue even when no work is available.

## Question 4: Classical Problems and Deadlock

**Question**: The CPUs and the ready queue look like a classical synchronization problem of Chapter 5. **Name it**, and say who plays the **producer** and the **consumer** roles (careful: here the CPUs also re-enqueue processes). If the ready queue were **bounded** (capacity `K`), name the **three semaphores** you would need and their **initial values**. Describe a **deadlock scenario** that appears if a thread acquired the **mutex before** waiting on the semaphore (or if two locks were acquired in different orders), and say **which of the four necessary conditions** you would break to prevent it.

> 💡 **TIP:** Recall the bounded-buffer solution of the slides (`empty`, `full`, `mutex`) and the order of `wait()` calls in the producer and the consumer. For the deadlock, think about a producer that holds the mutex while waiting for a free slot that only a consumer (who needs the mutex) can create.

**Your Answer:** *(3-5 sentences)*

**Classical problem and roles**:
[Name the problem; who is the producer, who is the consumer?]

The problem is the Producer-Consumer (Bounded Buffer) problem. The ready queue acts as the shared buffer between producers and consumers. New process arrivals act as producers because they add processes to the ready queue, while CPUs act as consumers because they remove processes from the queue and execute them. CPUs can also temporarily play the producer role when they re-enqueue a process that has not finished its execution.

**Semaphores for a bounded queue (name and initial value)**:
[Name the three semaphores and their initial values.]

empty = K  (number of free slots in the queue)
full = 0   (number of occupied slots in the queue)
mutex = 1  (provides mutual exclusion when accessing the queue)

**Deadlock scenario and prevention**:
[Describe the scenario and the necessary condition you would break.]

A deadlock can occur if a producer acquires the mutex lock first and then waits on the empty semaphore when the queue is full. The producer holds the mutex while waiting, so a consumer cannot acquire the mutex to remove an item and free space in the queue. As a result, both threads wait forever. To prevent this, threads should wait on the semaphore before acquiring the mutex, following the standard producer-consumer order. This breaks the hold-and-wait condition, one of the four necessary conditions for deadlock.

## Summary

**Key concepts I understood through these questions:**
1.How race conditions occur when multiple threads access shared variables simultaneously, leading to lost updates and inconsistent results.
2.How locks and critical sections provide mutual exclusion, and how using one lock per independent variable can reduce contention and improve concurrency.
3.The difference between busy waiting and blocking synchronization using semaphores, as well as how the Producer-Consumer pattern and proper lock ordering help prevent deadlocks.

**Concepts I need to study more:**
1.Deadlock prevention strategies and the four necessary conditions for deadlock.
2.Fairness, bounded waiting, and advanced synchronization techniques using semaphores and locks.

---

# ✅ Final Checklist (complete before submitting)

> ⚠️ **WARNING:** Go through every line. Blackboard does not accept submissions after the deadline, which is **November 14, 2026**.

**Repository**
- [ ] Repository is **PUBLIC** (Settings → Danger Zone → Visibility)
- [ ] Repository is renamed to `OS-Assignment3-YourFirstName-YourLastName`
- [ ] GitHub account uses the university email (`@std.psau.edu.sa`)

**Code**
- [ ] Student ID is set in `SchedulerSimulationSync.java` (line 304)
- [ ] My code is based **only** on the starter of this repository (nothing copied from previous assignments)
- [ ] Code compiles and runs with no errors, and terminates by itself (no `⛔ WATCHDOG`)
- [ ] Feature 1 (ready queue lock), Feature 2 (statistics locks) and Feature 3 (semaphore, no busy waiting) all work
- [ ] **The four verification lines are ✔ in 5 consecutive runs**
- [ ] Every lock is released in a `finally` block
- [ ] `RaceWindow`, `Config`, the verification block and the watchdog are **unchanged**
- [ ] Each feature has clear comments

**Commits**
- [ ] **At least 4 meaningful commits, ideally 7 or more**
- [ ] **One commit per feature**
- [ ] Commits are spread over **different dates** (not all in the last hour)
- [ ] Everything is **pushed** to GitHub

**This file (`MY_WORK.md`)**
- [ ] Full name and student ID filled in at the top
- [ ] Development log has **5+ entries** on different dates
- [ ] Reflection: 4 questions, 5-7 sentences each
- [ ] Technical answers: 4 questions, 3-5 sentences each, with examples from **your** output (starter output **and** final output pasted in Question 1)
- [ ] No `[...]` placeholders left
- [ ] No section headers deleted

**Video**
- [ ] 2-3 minutes long, named `StudentID_Assignment3_Demo.mp4`
- [ ] Shows your name, ID, repository, 3 features (and your lock granularity choice), IDE execution with the verification block, one synchronization concept, and commit history
- [ ] Link is **public** (tested in an incognito window) and pasted in the **Video Link** section above

**Blackboard**
- [ ] Submit **only** the link to your public GitHub repository

> 🎯 **Good luck!** Start early, commit regularly, and make sure you can explain every line you submit.
