# CS3701 Operating Systems - Assignment 3: Synchronization
## Making the Multi-CPU Scheduler Simulation Thread-Safe (Mutex Locks and Semaphores)

### 📋 Assignment Overview

This assignment evaluates your ability to apply the **process synchronization** tools of Chapter 5 (**mutex locks** and **semaphores**) in Java, while continuing the professional practices introduced in Assignment 1 (version control with GitHub, documentation, video presentation). It is the **continuation of Assignment 1**: you work with the **same Round-Robin CPU scheduler simulation** (same `Process` class, same ready queue, same colored output, same random values generated from your student ID), but the machine now has **several CPUs that run at the same time** and share the ready queue and the statistics. Shared data + concurrent threads = **race conditions**, and your job is to find them and fix them.

**What You Will Do:**
- Fork a starter repository from GitHub and personalize it with your information
- Run the starter code, observe the failures caused by **race conditions**, and understand why they happen
- Modify the Java code to add three synchronization features (a mutex lock on the ready queue, mutex locks on the shared statistics, and a counting semaphore that replaces busy waiting)
- Verify your solution with the **built-in verification block** (the expected values are computed from your own student ID)
- Document your development process and reflect on what you learned
- Create a video demonstration explaining your work and understanding
- Submit your work through your public GitHub repository

**Important**: This assignment is worth **5%** of your final grade. **Deadline: November 14, 2026.**

> ### 🛠️ Recommended: use Visual Studio Code (VS Code) for this assignment
> Work on **everything in VS Code**: edit `SchedulerSimulationSync.java` and `MY_WORK.md`, run the program (needed for your video), and **link VS Code directly to your GitHub account** so you can **commit and push with a few clicks, no command line needed**.
>
> 👉 **Step-by-step setup, GitHub sign-in, and commit/push guide: see [Recommended Development Environment: VS Code + GitHub](#️-recommended-development-environment-vs-code--github) below.** (It is the same setup as Assignment 1.)
>
> ⚠️ Commits are graded: a single bulk commit costs **-0.5 mark**. VS Code makes it easy to commit after every feature and every work session.
>
> 🛑 **Before anything else:** read this whole README, then read and run the full `SchedulerSimulationSync.java` **before** answering any question.
>
> 📌 **Use ONLY the code of this repository** (`SchedulerSimulationSync.java`). Do **not** copy or paste any code from your previous assignments: this starter already contains everything it needs, and the verification block only works with this code.

---

## 🎯 Learning Objectives

By completing this assignment, you will:
- **Recognize Race Conditions:** Identify shared data and the read-modify-write sequences that lose updates when several threads run concurrently
- **Solve the Critical-Section Problem:** Protect critical sections with a **mutex lock** (`ReentrantLock`) that guarantees mutual exclusion, and release it safely with `try/finally`
- **Apply Semaphores:** Use a **counting semaphore** (`Semaphore`) to count available resources and make threads **block instead of busy-wait**
- **Choose Lock Granularity:** Decide between one lock for everything and one lock per independent shared variable, and justify the choice
- **Avoid Deadlock and Starvation Pitfalls:** Understand why locks must always be released and why shutdown of blocked threads must be handled
- **Verify Concurrent Code:** Test a concurrent program with deterministic checks instead of "it seems to work"
- **Practice Professional Habits:** Continue using Git/GitHub with meaningful commits, document your work, and explain it in a video

---

## 📚 Background: Chapter 5 in Five Minutes

> 📖 Course slides: **Chapter 5 - Synchronization**. Textbook (*Operating System Concepts*, 10th edition): **Chapter 6 (Synchronization Tools)** and **Chapter 7 (Synchronization Examples)**.

### Race condition
A **race condition** occurs when several threads access and modify the same shared data and the result depends on the order in which the accesses happen. The classic example is `count++`, which is **three** steps (read, add, write), not one:

```java
int tmp = counter;     // 1) read
tmp = tmp + 1;         // 2) modify
counter = tmp;         // 3) write
```
If two threads interleave their steps (both read `5`, then both write `6`), **one increment is lost**: the counter ends at `6` instead of `7`.

### Critical-section problem
The code that accesses shared data is a **critical section**. A correct solution must provide:
1. **Mutual exclusion**: only one thread at a time inside the critical section
2. **Progress**: if no thread is inside, a thread that wants to enter must be able to decide who enters, without waiting forever
3. **Bounded waiting**: a thread cannot be forced to wait indefinitely while others keep entering

### Mutex lock → `ReentrantLock`
```java
private final ReentrantLock lock = new ReentrantLock();

lock.lock();            // acquire(): blocks if another thread owns the lock
try {
    // critical section
} finally {
    lock.unlock();      // release(): ALWAYS in finally, even if an exception is thrown
}
```

### Semaphore → `java.util.concurrent.Semaphore`
A semaphore is an integer `S` accessed only through two atomic operations. The slides' names map to Java like this:

| Course slides | Java | Meaning |
|---|---|---|
| `wait(S)` / `P(S)` | `semaphore.acquire()` | decrement `S`; **block** the thread if `S` would become negative |
| `signal(S)` / `V(S)` | `semaphore.release()` | increment `S`; wake up one blocked thread (if any) |
| `S` initial value | `new Semaphore(initialValue)` | `1` = binary (mutex-like), `N` = counting (N identical resources), `0` = "nothing available yet" |

A semaphore initialized to **0** that counts *items available* is the standard way to make consumer threads **sleep until a producer signals** (instead of busy waiting).

### Busy waiting
A thread that repeatedly tests a condition in a loop (`while (!condition) { ... }`) wastes CPU time: this is **busy waiting** (a *spinlock* is the same idea applied to a lock). A semaphore avoids it because the waiting thread is **blocked** (it does not consume the CPU) until another thread calls `release()`.

---

## 🔍 Background: Understanding the Starter Code

The program simulates a machine with **several CPUs** (2 to 4, depending on your student ID). All CPUs take processes from **one shared FIFO ready queue**, run **one time quantum**, and put the process back in the queue if it is not finished. The file `SchedulerSimulationSync.java` contains these classes:

| Class | Role | Status |
|---|---|---|
| `Colors` | ANSI colors for the console | Same as Assignment 1 |
| `RaceWindow` | `RaceWindow.pause()` sleeps 1 ms **between a read and the following write** to simulate the OS preempting a thread at the worst moment, so that races appear **every time** | ⛔ **Do not modify or remove** |
| `Config` | `TIME_SCALE` (simulation speed) | Do not change |
| `SharedStats` | Three **shared variables** updated by all the process threads: `contextSwitchCount`, `completedProcessCount`, `totalServiceTime` (+ `busyWaitPolls`, a measurement counter) | **You modify** (Feature 2) |
| `ReadyQueue` | The shared FIFO queue (`LinkedList<Thread>` + `Map<Thread, Process>`): `add()`, `poll()`, `processOf()`, `snapshot()` | **You modify** (Feature 1 and Feature 3) |
| `Process` | Same as Assignment 1: runs **one quantum** when its thread is started (calls `SharedStats` at the beginning and at the end of the quantum) | Do not change |
| `CpuWorker` | **One CPU = one thread.** Its loop is the scheduler loop of Assignment 1: take a process, `start()` its thread, `join()`, re-enqueue it if not finished | **You modify** (Feature 3) |
| `SchedulerSimulationSync` | `main`: reads your student ID, generates the processes, starts the CPUs, runs a **watchdog**, then prints the **verification block** and the **summary table** | Only change `studentID` |

**Flow of the program:**
```
main ──► creates processes ──► ReadyQueue (shared)
   │                                  ▲        │ poll()
   │ starts                    add()  │        ▼
   └──► CPU-1 ... CPU-N  ──►  take a process → Process thread: start() → one quantum → join()
                                         │            │ updates SharedStats (shared!)
                                         └────────────┘ not finished? add() it back
```

**The verification block** (printed at the end, **expected values computed from your student ID**):
```
═══ VERIFICATION (expected values are computed from YOUR seed) ═══
  Context switches (quanta executed)     expected 31       observed 31       ✔
  Completed processes                    expected 17       observed 17       ✔
  Total service time (ms)                expected 46593    observed 46593    ✔
  Busy-waiting polls (must be 0)         expected 0        observed 0        ✔
  ✔ All shared values are consistent.
```
- *Context switches* must equal the **total number of quanta** = the sum over all processes of `ceil(burstTime / timeQuantum)`.
- *Completed processes* must equal the number of processes.
- *Total service time* must equal the **sum of all burst times**.
- *Busy-waiting polls* counts how many times a CPU found the queue empty and polled it again: it must be **0** once Feature 3 is done.

**The watchdog:** if the CPUs do not terminate within a reasonable time, `main` prints `⛔ WATCHDOG`, shows the verification block, and stops the program. It means a process was lost or a CPU is blocked forever.

**Note:** the complete code is in the starter repository you will fork from GitHub. Lines containing `TODO Feature 1/2/3` show **exactly where** you have to work.

---

## 🚀 Getting Started

### Step 1: Fork This Repository

1. **Use your GitHub account** created with your **university email** (@std.psau.edu.sa) in Assignment 1 (the same account, never a new one).

2. **Fork this repository**:
   - Go to the starter repository: `https://github.com/makopt/OS-Assignment3-Starter-481`
   - ⚠️ **Fork ONLY this starter repository, never another student's repository** (zero marks for both students + referral to the College Student Issues Committee)
   - Make sure you're logged in to GitHub with your university email account
   - Click the **"Fork"** button at the top right of the page
   - Your repository will be at: `https://github.com/YOUR_USERNAME/OS-Assignment3-Starter-481`

3. **Rename your forked repository**:
   - Go to your forked repository **Settings**
   - Change the repository name to: `OS-Assignment3-YourFirstName-YourLastName`
   - Example: `OS-Assignment3-Mohammed-Ahmed`

4. **Make sure your repository is PUBLIC**:
   - Go to Settings → scroll down to "Danger Zone"
   - Verify visibility is set to "Public"
   - If private, click "Change visibility" → "Make public"

### Step 2: Clone Your Repository to Your Computer

**Option A: VS Code (recommended, no command line)**
1. Press **Ctrl+Shift+P** (Cmd+Shift+P on Mac) → **Git: Clone** → **Clone from GitHub**.
2. Select **your own renamed repository** (`OS-Assignment3-YourFirstName-YourLastName`), choose a folder, then click **Open**.
3. (If VS Code is not linked to your GitHub account yet: see *Recommended Development Environment* below.)

**Option B: Command line**
```bash
git clone https://github.com/YOUR_USERNAME/OS-Assignment3-YourFirstName-YourLastName.git
cd OS-Assignment3-YourFirstName-YourLastName
```

### Step 3: Set Up Your Student ID

**⚠️ CRITICAL FIRST STEP ⚠️**

Open `SchedulerSimulationSync.java` and **immediately** change line 304:

```java
// BEFORE (DO NOT SUBMIT THIS):
int studentID = 123456789;

// AFTER (Use YOUR real ID):
int studentID = 441234567;  // Replace with YOUR actual student ID
```

**Save this change and commit it right away** (use **one** of the two options):

**Option A: VS Code (recommended, no command line)**
1. Save the file (Ctrl+S) and open the **Source Control** panel (Ctrl+Shift+G).
2. Click **+** next to `SchedulerSimulationSync.java` to stage it.
3. Type the message `Set my student ID: 441234567` (with your own ID) in the box and click **✓ Commit**.
4. Click **Sync Changes** (or **Push**) to send it to GitHub.

**Option B: Command line**
```bash
git add SchedulerSimulationSync.java
git commit -m "Set my student ID: 441234567"
git push origin main
```

✅ This personalizes your assignment: the quantum, the number of processes and CPUs, and the expected values of the verification block all depend on your ID.

---

## 📂 Repository Structure

Your repository should have these files:

```
OS-Assignment3-YourName/
├── README.md                       # This file - project overview
├── SchedulerSimulationSync.java    # Main code file (YOU WILL MODIFY THIS)
├── MY_WORK.md                      # Student info + dev log + reflection + answers + video link (FILL THIS)
└── .gitignore                      # Git ignore file (already configured)
```

---

## ✅ Your Tasks (4 Parts - 5 Marks Total)

### **Part 1: GitHub Account & Repository Setup (1 mark)**

Same procedure as Assignment 1:

1. **Use your GitHub account** created with your university email (`[yourid]@std.psau.edu.sa`).
   - **Note:** Using a non-university email results in a -0.25 marks penalty
2. **Fork the starter repository** `https://github.com/makopt/OS-Assignment3-Starter-481` (never another student's repository).
3. **Rename your repository** to `OS-Assignment3-YourFirstName-YourLastName`.
4. **Keep the repository PUBLIC** (Settings → Danger Zone → Visibility).
5. **Set your student ID** in `SchedulerSimulationSync.java` (line 304) and **commit it** with the message `Set my student ID: [YOUR-ID]`. This is mandatory: your output must be unique!

**Important:** Read this `README.md` and `MY_WORK.md` in the repository: they contain detailed instructions for every step!

---

### **Part 2: Code Implementation (1 mark)**

> 🛑 **Step 0 (before writing any code): run the starter and observe the failure.** Set your student ID, run `SchedulerSimulationSync`, and **keep a copy of the output** (you need it for Question 1 of `MY_WORK.md` and for your video). The starter is **intentionally broken**. You will typically see:

| What you see in the console | What it means |
|---|---|
| Several CPUs `picks P1` at the same time, then `IllegalThreadStateException` | **Feature 1 missing**: two CPUs read the same head of the ready queue (`poll()` is not atomic) |
| A process disappears (never finishes), then `⛔ WATCHDOG` after some seconds | **Feature 1 missing**: `poll()` removed a *different* process than the one it read, so one process was lost |
| `Context switches ... expected 31 observed 28 ✘ MISMATCH` (no exception) | **Feature 2 missing**: lost updates on the shared counters |
| `Completed processes ... observed 16` and `⛔ WATCHDOG` | **Feature 2 missing**: a lost update on `completedProcessCount` makes the CPUs wait for a count that never reaches the total |
| `Busy-waiting polls (must be 0) ... observed 7 ✘` (everything else ✔) | **Feature 3 missing**: the CPUs poll the empty queue in a loop |

**Your Task:** Modify `SchedulerSimulationSync.java` to add three synchronization features. **Do the features in this order**: the failures of Feature 1 hide those of Feature 2 until the queue is fixed.

**Important Guidelines:**
- Make **one commit per feature** - do NOT commit everything at once!
- Test each feature (run the program **at least 5 times**) before committing
- Add clear comments explaining your additions
- Keep the existing functionality working
- ⛔ **Never modify or remove `RaceWindow`, `Config`, or the verification block** to make the problem "disappear": this is considered cheating (-1 mark)

**Commit Examples:**
- ✅ `"Feature 1: Protected the ready queue with a ReentrantLock"`
- ✅ `"Feature 2: Added locks to the shared statistics counters"`
- ✅ `"Feature 3: Replaced busy waiting with a counting semaphore"`
- ❌ `"done"` or `"update"` or `"final version"` (too vague!)

#### 2.1: Feature 1 - Protect the Shared Ready Queue with a Mutex Lock (0.25 mark)
- Add `import java.util.concurrent.locks.ReentrantLock;`
- In `ReadyQueue`, declare a `ReentrantLock` (see `TODO Feature 1`) protecting `list` and `map`
- Protect **every** access to `list` and `map`: `add()`, `poll()`, `processOf()` and `snapshot()` (the `snapshot()` iteration is also a critical section: a `ConcurrentModificationException` can occur otherwise)
- Always use `lock()` + `try { ... } finally { unlock(); }`
- **Expected result:** no more exceptions, and no duplicated or lost process. The counters of the verification block may still be wrong, and the watchdog may still fire occasionally because of a lost update on `completedProcessCount`: that is Feature 2
- Add clear comments explaining your additions

> 💡 **TIP:** In `poll()` the `return` statements are inside the `try` block: the `finally` block still releases the lock. Do **not** call `unlock()` before the `return`.

#### 2.2: Feature 2 - Protect the Shared Statistics with Mutex Lock(s) (0.25 mark)
- In `SharedStats`, declare the lock(s) (see `TODO Feature 2`) and protect the **three update methods** (`incrementContextSwitch()`, `incrementCompletedProcess()`, `addServiceTime()`) **and the three getters**
- **Design decision (lock granularity):** will you use **one lock** for the three variables, or **one lock per variable**? The three variables are independent. **You must justify your choice** in `MY_WORK.md` (Question 2) and in your video
- Always use `lock()` + `try { ... } finally { unlock(); }`
- **Expected result:** the first three lines of the verification block are ✔ **in every run**
- Add clear comments explaining your additions

#### 2.3: Feature 3 - Replace Busy Waiting with a Counting Semaphore (0.5 mark)
In the starter, a CPU that finds the queue empty sleeps 20 ms and tries again (**busy waiting**), and it stops only when `completedProcessCount == totalProcesses`. You must replace this polling by a **blocking wait** on a **counting semaphore** that counts the processes in the ready queue:
- Add `import java.util.concurrent.Semaphore;` and declare in `ReadyQueue` a `Semaphore` whose value is **the number of processes currently waiting in the ready queue** (think carefully about its **initial value**)
- In `add()`: signal that one more process is available (`release()`)
- In `CpuWorker.run()`: **before** taking a process, wait until one is available (`acquire()`), and **delete the polling code** (`Thread.sleep(20)`, the `if (thread == null)` branch)
- **Clean shutdown:** when the **last process has completed**, the CPUs blocked on `acquire()` would sleep forever. The CPU that observes that all the processes are completed must wake up the others (`release(SharedStats.totalCPUs)`), and a CPU that wakes up because of this signal must stop instead of polling the queue. `SharedStats.totalCPUs` is provided for you
- **Expected result:** all four lines of the verification block are ✔, **`Busy-waiting polls = 0`**, and the program terminates by itself (no watchdog)
- Add clear comments explaining your additions

> ⚠️ **WARNING:** If you forget the shutdown signal, the program will **hang** at the end and the watchdog will stop it: this is a deadlock-like situation (threads waiting forever).

**Commit after EACH feature** (use **one** of the two options):

**Option A: VS Code (recommended, no command line)**
1. Save the file (Ctrl+S) and open the **Source Control** panel (Ctrl+Shift+G).
2. Click **+** next to `SchedulerSimulationSync.java` to stage it.
3. Type a message such as `Feature 1: Protected the ready queue with a ReentrantLock` and click **✓ Commit**.
4. Click **Sync Changes** (or **Push**) to send it to GitHub.

**Option B: Command line**
```bash
git add SchedulerSimulationSync.java
git commit -m "Feature N: [brief description]"
git push origin main
```

> 💡 **TIP:** Full details, including the Command Palette method, are in *Recommended Development Environment* below.

#### 🧪 How to test your solution
1. Run the program **at least 5 times** (concurrency bugs are not always visible in one run).
2. In **every** run: no exception, no `⛔ WATCHDOG`, and **all four lines ✔** with the message `✔ All shared values are consistent.`
3. The program must **terminate by itself**.
4. In `MY_WORK.md` (Part C, Question 1) you will compare the **starter's** output with **your final** output.

#### ⚠️ Common Mistakes
1. **Unlock outside `finally`:**
   ```java
   // WRONG - if an exception occurs, the lock is never released (all the other threads block forever)
   lock.lock();
   doWork();
   lock.unlock();

   // CORRECT
   lock.lock();
   try {
       doWork();
   } finally {
       lock.unlock();
   }
   ```
2. **Protecting only the writers:** a getter that reads a variable while another thread is updating it must also be protected.
3. **Holding a lock while waiting for a semaphore (or the reverse):** never call `readyCount.acquire()` while you hold `queueLock` (the thread would block *with* the lock, and no one could ever call `add()`: **deadlock**).
4. **Initial value of the semaphore:** a wrong initial value (for example `1`) lets a CPU take a process from an empty queue.
5. **Forgetting the shutdown signal** (Feature 3): the CPUs stay blocked in `acquire()` forever.
6. **Using `synchronized` or `AtomicInteger` instead of the requested tools:** the assignment asks for `ReentrantLock` and `Semaphore` (Chapter 5 tools); other solutions get **no marks** for the feature.

---

### **Part 3: Documentation (1.5 marks)**

> 🛑 **Before you start:** read this whole `README.md` and the full `SchedulerSimulationSync.java` (run it too) **before** answering any question. Then open `MY_WORK.md`: it contains a **work roadmap**, **commit rules**, and ⚠️ warnings / 💡 tips at every step to guide you.
>
> ⚠️ **Commit often:** a single commit, or all commits in the last hour, costs **-0.5 mark**. Make at least 4 meaningful commits (setup + one per feature), spread over different dates, and push them.

Complete the single `MY_WORK.md` file in your repository to document your work and demonstrate your understanding. **First fill in your full name and student ID** in its *Student Information* table. It then contains three sections:

#### 3.1: Development Log section (0.5 mark)
Track your development process with **minimum 5 entries**:
- Date and time of work session
- What you worked on (specific features or tasks)
- Challenges encountered during this session
- Solutions you implemented
- Time spent (approximate)
- **Template with example format is provided in the file!**
- Entries should be spread across different dates (not all at once)

#### 3.2: Reflection section (0.5 marks)
Answer the **4 reflection questions** provided in the file:
1. What did you learn about synchronization from this assignment?
2. What was the most challenging part of this assignment and why?
3. How did you overcome the challenges you faced?
4. Where can the synchronization concepts be applied in real-world applications?
- Each answer: **minimum 5-7 sentences**
- Write in your own words, showing genuine reflection
- Connect concepts to your actual experience with the code

#### 3.3: Answers section (0.5 marks)
Answer the **4 technical questions** in the provided template (see the section *Questions to Answer* below):
- Each answer: **minimum 3-5 sentences**
- Include specific examples from **your** code and **your** output
- Use proper technical terminology (race condition, critical section, mutual exclusion, mutex lock, semaphore, busy waiting, deadlock, etc.)
- Reference relevant parts of your code where appropriate

### **Part 4: Video Demonstration (1.5 marks)**

Create a **2-3 minute video** (not shorter, not longer) showing:

1. **Introduction (20 seconds)**:
   - State your name and student ID
   - Show your GitHub repository homepage
   - Verify repository is public and uses university email

2. **Code Walkthrough (1 minute)**:
   - Navigate through your repository files
   - Show your three modifications: the **lock on the ready queue**, the **locks on the statistics** (say whether you used one lock or one lock per variable, **and why**), and the **semaphore** (initial value, `release()` in `add()`, `acquire()` in `CpuWorker`, shutdown)
   - Briefly explain each feature you added

3. **Execution Demo (40 seconds)**:
   - Open your IDE (VS Code, IntelliJ IDEA, Eclipse, or similar)
   - Show the `SchedulerSimulationSync.java` file open in the IDE
   - Compile and run using your IDE's run button/feature
   - Show the console output with your student ID, time quantum, number of processes and **number of CPUs**, and the **verification block with all ✔** (including `Busy-waiting polls = 0`)

4. **Concept Explanation (30 seconds)**:
   - Pick ONE synchronization concept you learned
   - Options: how a **race condition** loses an update (use `contextSwitchCount` and `RaceWindow.pause()` as the example), why **`finally`** is required when releasing a lock, what the **value of the semaphore** means, or why **busy waiting** is wasteful
   - Explain it in your own words using your code as example

5. **Closing (10 seconds)**:
   - Show your commit history (at least 4 commits)
   - Thank you message

**Video Requirements:**
- Upload video to **Google Drive**, **YouTube** (unlisted or public) or any other cloud sharing system
- Set sharing permissions so it is publicly accessible ("Anyone with the link can view")
- Name your video file: `StudentID_Assignment3_Demo.mp4`
- Add the video link in your `MY_WORK.md` file (**not** in README.md)
- Show your face OR use clear voice narration
- Screen recording showing your code and execution
- Good audio quality (test your microphone first!)

**Recording Tools:**
- **Mac:** QuickTime Player (File → New Screen Recording)
- **Windows:** Xbox Game Bar (Win + G) or OBS Studio
- **Linux:** SimpleScreenRecorder or OBS Studio

**Important Notes:**
- Use your IDE (VS Code, IntelliJ IDEA, Eclipse, etc.) to compile and run the code - NOT the terminal
- Make sure the IDE console output is clearly visible in your video (pause on the verification block)
- Show your unique simulation parameters: student ID, time quantum, number of processes and number of CPUs
- Explain concepts in your own words to demonstrate genuine understanding

**Upload your video and add the shareable link to your `MY_WORK.md` file (not README.md). Make sure the link is accessible to anyone with the link; a private, restricted or broken link counts as a missing video (-1 mark).**

---

## 🛠️ Recommended Development Environment: VS Code + GitHub

> 💡 **We strongly recommend Visual Studio Code (VS Code)** for this assignment. You can edit `SchedulerSimulationSync.java` **and** `MY_WORK.md`, run the program, and **commit + push to GitHub, all in one window**, without typing Git commands. This makes it easy to commit often (which is required).

### Why VS Code?
- **Free** and works on Windows, macOS and Linux: [download VS Code](https://code.visualstudio.com/download)
- **Built-in Git and GitHub support**: commit and push with a few clicks
- **Runs Java in the IDE** (needed for the video): the ▶ Run button shows the output in the IDE console
- **Previews Markdown**: edit `MY_WORK.md` and see it rendered side by side

### Step 1: Install the tools
1. Install a **JDK** (version 17 or later) and **Git** ([git-scm.com](https://git-scm.com/downloads)). VS Code needs Git installed to commit.
2. Install VS Code, then install these **extensions** (Extensions icon in the left bar, or Ctrl+Shift+X):
   - **Extension Pack for Java** (Microsoft): run and debug Java
   - **GitHub Pull Requests** (GitHub): sign-in and GitHub integration
   - **GitLens**: shows the history of each line and your commit history
   - **Markdown All in One**: helps edit `MY_WORK.md`

> ⚠️ **WARNING:** If VS Code says *"Git not found"*, install Git and **restart VS Code**.

### Step 2: Link VS Code to your GitHub account
1. Click the **account icon** (bottom-left corner) → **Sign in with GitHub**.
2. Your browser opens: log in with **the GitHub account that uses your university email** and click **Authorize**.
3. Back in VS Code, your GitHub username now appears under the account icon.
4. Tell Git who you are (**one time only**). Open the terminal in VS Code (Ctrl+`) and run:
   ```bash
   git config --global user.name "Your Full Name"
   git config --global user.email "yourid@std.psau.edu.sa"
   ```

> ⚠️ **WARNING:** Use your **university email** here. Commits made with another email can count as a non-university email (**-0.25 mark**).

### Step 3: Clone **your** fork into VS Code
1. Press **Ctrl+Shift+P** (Cmd+Shift+P on Mac) → type **Git: Clone** → press Enter.
2. Choose **Clone from GitHub**, then select **your own renamed repository** (`OS-Assignment3-YourFirstName-YourLastName`), **not** the starter repository and **never another student's repository**.
3. Pick a folder on your computer, then click **Open** when VS Code asks.

> 💡 **TIP:** Save your work in a normal folder (e.g., Documents), **not** inside a synced or shared folder that other people can access.

### Step 4: Edit, then commit and push (repeat after every change!)
First **edit** `SchedulerSimulationSync.java` or `MY_WORK.md` and **save** (Ctrl+S). Then use **either** method below. Both work entirely with mouse and menus, **no Git command line needed**.

#### Method A: Source Control panel (recommended)
1. Open the **Source Control** panel (Ctrl+Shift+G, the branch icon in the left bar). Your changed files are listed under *Changes*.
2. Click a file to see **exactly what you changed** (green = added, red = removed). Check it before committing.
3. **Stage** the files: click the **+** next to each file (or next to *Changes* for all files).
4. Type a **clear commit message** in the box, e.g. `Feature 1: Protected the ready queue with a ReentrantLock`.
5. Click **✓ Commit**.
6. Click **Sync Changes** (or **Push**) to send the commit to GitHub.

#### Method B: Command Palette + status bar (no Source Control panel)
1. Press **Ctrl+Shift+P** (Cmd+Shift+P on Mac) and run **Git: Stage All Changes** (or **Git: Stage** for one file).
2. Press **Ctrl+Shift+P** again, run **Git: Commit**, type your commit message in the box that appears and press **Enter**.
3. Press **Ctrl+Shift+P** once more and run **Git: Push**. Alternatively, click the **🔄 sync icon** in the blue **status bar** at the bottom-left (next to the branch name `main`) to sync.

#### Verify (both methods)
- Open your repository page on **github.com** and check that your new commit appears in the commit list.

> 🛑 **STOP:** A commit that is **not pushed** stays on your computer only. The instructor **cannot see it**. Always click **Sync Changes / Push** after committing.
>
> ⚠️ **WARNING (commits are graded):** Make **one commit per feature**, and commit **after each work session** and after each part of `MY_WORK.md`. Do **not** commit everything at once at the end (**-0.5 mark**).
>
> 💡 **TIP:** If VS Code asks *"Would you like Code to periodically run git fetch?"* or asks you to *Sync*, answer **Yes**. If it asks *"There are no staged changes, commit all?"*, **Yes** is fine as long as you checked the changes in step 3.

### Step 5: Run the program in VS Code (for your video)
1. Open `SchedulerSimulationSync.java`.
2. Click the **▶ Run** button at the top-right of the editor (or **Run | Debug** above `main`).
3. The output appears in the **Terminal / Debug Console** panel at the bottom of VS Code.

> 💡 **TIP:** You must run the program **from the IDE** in your video (not from an external terminal), so practice this now.

### Common problems
| Problem | Fix |
|---------|-----|
| *"Git not found"* | Install Git, restart VS Code |
| Push asks for a password | Use **Sign in with GitHub** (Step 2); passwords no longer work for GitHub |
| Push is rejected / *"behind remote"* | Click **Sync Changes** (pull, then push). This happens if you edited a file on github.com |
| Cloned the wrong repository | Delete the folder and clone **your own** fork again (Step 3) |
| Commit shows another name/email | Re-run the two `git config` commands in Step 2 |
| Colors look strange in the console (`←[36m`) | The output uses ANSI colors. Use the VS Code Terminal or Debug Console; it is only cosmetic |

### Useful tutorials
- [VS Code GitHub Integration Tutorial](https://www.youtube.com/watch?v=i_23KUAEtUM)
- [Java Development in VS Code Tutorial](https://www.youtube.com/watch?v=BB0gZFpukJU)
- Official guide: search for *"VS Code Source Control"* in the VS Code documentation

**Note:** You may use another IDE (IntelliJ IDEA, Eclipse, etc.), but the instructor's guidance and troubleshooting are based on VS Code, which has the smoothest GitHub integration.

---

## 📝 Questions to Answer (in MY_WORK.md)

These questions help you demonstrate technical understanding. Each answer should be thoughtful and specific, and must refer to **your own code and your own output** (your student ID makes your output unique).

### Question 1: Race Condition in Your Own Output
- **Run the original starter** (before any fix) and keep its output. **Paste the verification block** of that run in `MY_WORK.md`
- **Which value was wrong** (expected vs observed) and by how much?
- **Explain step by step**, with two named threads/CPUs, how **one lost update** happens (read, `RaceWindow.pause()`, write) using one of the shared variables
- **Why can the same bug occur on a real machine** even without `RaceWindow`, and why is it hard to detect?
- Minimum: 3-5 sentences

### Question 2: Critical Sections and Lock Granularity
- **List the critical sections** you protected (variables and methods) and the lock(s) you used
- **Lock granularity:** did you use one lock for the three statistics or one lock per variable? **Why?** What is the trade-off?
- **Why is the ready queue protected by a different lock** than the statistics?
- Explain how your solution provides **mutual exclusion**, and discuss **progress** and **bounded waiting** (a default `ReentrantLock` is *not fair*: what does that imply?)
- Minimum: 3-5 sentences

### Question 3: Busy Waiting vs Semaphore
- **Describe the busy waiting** of the starter and its cost (give the `Busy-waiting polls` number of the starter run)
- **What does the value of your semaphore represent?** What is its **initial value** and why? What is its value right after the `numProcesses` initial `add()` calls?
- **Why does the last CPU call `release(totalCPUs)`** at the end, and what would happen if it did not?
- Minimum: 3-5 sentences

### Question 4: Classical Problems and Deadlock
- The CPUs and the ready queue look like a classical synchronization problem of Chapter 5. **Name it** and say who plays the **producer** and the **consumer** roles (careful: here the CPUs also re-enqueue processes)
- If the ready queue were **bounded** (capacity `K`), name the **three semaphores** you would need and their **initial values**
- Describe a **deadlock scenario** that appears if a thread acquired the **mutex before** waiting on the semaphore in that bounded version (or if two locks were acquired in different orders) and say **which of the four necessary conditions** you would break to prevent it
- Minimum: 3-5 sentences

---

## ⚠️ Academic Integrity

### Critical Warnings:

1. **AI-Generated Code**: 
   - Pure AI-generated submissions without understanding will receive **zero marks**
   - You may be asked to explain your code in person (for example: *"why is this `return` inside the `try` block?"*, *"what happens if the initial value of your semaphore is 1?"*)
   - You must be able to answer questions about your implementation

2. **Plagiarism**: 
   - Copying from classmates results in **zero marks for all parties**
   - Each student has unique output (and unique expected values) based on their student ID

   - **Forking another student's repository** (instead of the starter repository) is plagiarism: it results in **zero marks for BOTH students** and the case is **referred to the College Student Issues Committee**

3. **Commit History**: 
   - Single bulk commits show poor practice and will be penalized
   - Commits must be spread over different dates/times

4. **Video Authenticity**: 
   - You must be able to answer questions about your work
   - Video should show genuine understanding, not memorized explanations

5. **Tampering with the verification**: 
   - Modifying `RaceWindow`, `Config`, the verification block, or the watchdog so that the problems "disappear" is **cheating** (-1 mark and possible academic misconduct report)

---

## 📅 Submission Instructions

### What to Submit on Blackboard:

Submit **only the link to your public GitHub repository** (no other files or information):

```
https://github.com/[your-username]/OS-Assignment3-[YourFirstName]-[YourLastName]
```

Everything else (name, student ID, development log, reflection, answers, and the video link) is read from your repository, in `MY_WORK.md`. Make sure the repository is public and complete before the deadline.

### Before You Submit - Final Checklist:

- [ ] Repository is **PUBLIC** (check Settings → Visibility)
- [ ] Student ID is set correctly in `SchedulerSimulationSync.java` (line 304)
- [ ] My code is based **only** on the starter of this repository (nothing copied from previous assignments)
- [ ] Code compiles without errors: `javac SchedulerSimulationSync.java`
- [ ] Code runs successfully: `java SchedulerSimulationSync` (terminates by itself, no `⛔ WATCHDOG`)
- [ ] **All four verification lines are ✔ in 5 consecutive runs**
- [ ] Every lock is released in a `finally` block
- [ ] No busy waiting left (`Busy-waiting polls = 0`) and `RaceWindow` untouched
- [ ] At least 4 meaningful commits with good messages (setup + one per feature)
- [ ] Commits spread over different dates (not all at once!)
- [ ] `MY_WORK.md` completed - Development Log (minimum 5 entries), Reflection (4 questions), and Answers (4 questions)
- [ ] Video is 2-3 minutes long
- [ ] Video uploaded (Google Drive, YouTube or similar) with a public link added to MY_WORK.md
- [ ] GitHub account uses university email (@std.psau.edu.sa)

---

## 🏆 Grading Summary (Total: 5 marks)

Your work will be evaluated based on the following criteria:

- **Part 1 - GitHub Setup & Personalization (1 mark):** 
  - GitHub account created with university email (@std.psau.edu.sa) (0.25 marks)
  - Repository properly forked and renamed (0.25 mark)
  - Student ID correctly set in code (0.25 mark)
  - Repository is public and accessible (0.25 marks)

- **Part 2 - Code Implementation (1 mark):** 
  - Feature 1: Ready queue protected with a `ReentrantLock` in all four methods, `try/finally` (0.25 mark)
  - Feature 2: Statistics protected (3 updates + 3 reads), `try/finally`, granularity choice justified (0.25 mark)
  - Feature 3: Counting semaphore (correct initial value, `release()` in `add()`, `acquire()` in the CPU loop), busy waiting removed, clean shutdown (0.5 mark)
  - Code compiles without errors, terminates, and the **four verification lines are ✔** in every run
  - Clear comments added to explain modifications
  - Separate commits for each feature

- **Part 3 - Documentation (1.5 marks):** 
  - `MY_WORK.md` Development Log complete with 5+ meaningful entries (0.5 mark)
  - `MY_WORK.md` Reflection with 4 questions answered thoroughly (0.5 marks)
  - `MY_WORK.md` Answers with 4 technical questions answered correctly (0.5 marks)
  - All answers demonstrate understanding and use proper terminology

- **Part 4 - Video Demonstration (1.5 marks):** 
  - Video is 2-3 minutes long (0.25 marks)
  - Shows repository, code walkthrough, and execution with the verification block (0.5 marks)
  - Explains one synchronization concept clearly (0.5 marks)
  - Good audio/visual quality and accessible link (0.25 marks)

- **Professional Practices:**
  - Minimum 4 meaningful commits with descriptive messages
  - Commits spread over multiple sessions (not all at once)
  - Proper file organization and naming conventions

### Penalties:
- Missing video: -1 mark
- Single commit or all commits in the last hour: -0.5 mark
- Private repository: -0.5 mark
- Not using university email for GitHub: -0.25 marks
- Lock or semaphore not released in a `finally` block: -0.25 mark each (maximum -0.5)
- A feature implemented with `synchronized`/`AtomicInteger` instead of `ReentrantLock`/`Semaphore`: no marks for that feature
- Modifying `RaceWindow`, `Config`, the verification or the watchdog: -1 mark
- AI-generated without understanding: 0 marks (entire assignment)
- Plagiarism: 0 marks + academic misconduct report
- Forking another student's repository: 0 marks for both students + referral to the College Student Issues Committee

---

## 📚 Additional Resources

### Java Synchronization Basics:
- Oracle Java Tutorials: [Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
- Java API: [`ReentrantLock`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html) and [`Semaphore`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/Semaphore.html)
- Course slides: **Chapter 5 - Synchronization** (race condition timing analysis, critical-section problem, mutex locks, semaphores, producer-consumer)
- Your textbook: Chapter 6 (Synchronization Tools) and Chapter 7 (Synchronization Examples)

### Git & GitHub:
- [GitHub Guides](https://guides.github.com/)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)

### Round-Robin Scheduling (reminder from Assignment 1):
- Your textbook: CPU Scheduling chapter
- Wikipedia: [Round-robin scheduling](https://en.wikipedia.org/wiki/Round-robin_scheduling)

---

## 🎓 Important Information

**Deadline:** November 14, 2026

**Late Policy:** Blackboard does not accept submissions after the deadline

Good luck! Start early, commit regularly, and demonstrate your understanding of synchronization through this assignment.

