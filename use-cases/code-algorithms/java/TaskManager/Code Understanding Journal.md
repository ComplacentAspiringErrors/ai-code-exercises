## Exercise Part 1: Understanding Project Structure
i see that in the java project unders the source directory there are several subfolders containng java classes.

TaskManager.java --> Im not sure what it does, just that it contains code for storage and task status 
TaskManagerCli.java --> this is a class that handles the command line instructions for the class, what is valid and also help instructions to help the user

Model folder includes 3 classes
Task.java --> includes getters and setters for the task manager "tasks" 
TaskPriority --> code that allows the user to set the importance of a task 
TaskStatus --> Code that states the progress of the task 

TaskStorage.java --> reads a file or loads one, also a save method so i assume its read, edit and delete saved tasks on a json file

## Prompt 1: Understanding Project Structure and Technology Stack
Gradle is not a framework --> its a build tool, it compiles your code, runs your tests, and manages all the external libraries you need so you don't have to download JAR files manually. 

The tech stack is java 

src/main/java --> Contains the actual application source code
src/main/resources --> Non-code files like configuration, text files or database schemas.
src/test/java --> Contains the code that tests your main code.
gradle/ contains the "wrapper" files that ensure everyoine on the team uses the same version of Gradle

Entry points and key files 
Inside of src/main/java --> looking for a file that contains public static void main(String[] args). 
the main file is TaskManagerCli.java --> its the only class that contains the public static void main(String[] args)

the misconceptions i had was how each file worked, i didnt know the entry point file was the only file that contained a main method i always thought it was the class ending in App, 

the key components of TaskManager -->
Here are the four key components that make the whole thing work:

1. The Entry Point (CLI Layer)
File: TaskManagerCli.java

This is the Presentation Layer. It is the only part of the code that speaks to the human user. It uses the Apache Commons CLI library to turn your keyboard typing into something Java understands.

Role: Input validation and output formatting.

Analogy: The waiter at a restaurant who takes your order and brings you the plate.

2. The Coordinator (Application/Service Layer)
File: TaskManager.java

This is the Logic Layer. It doesn't care if you're using a terminal, a website, or a mobile app. It only cares about the rules of being a "Task Manager" (e.g., "you can't complete a task that doesn't exist").

Role: Orchestrating tasks—creating, filtering, and calculating statistics.

Analogy: The head chef who knows the recipes and tells the kitchen what to do.

3. The Blueprint (Model Layer)
Files: Task.java, TaskPriority.java, TaskStatus.java

These are Domain Objects. They define what a "Task" actually is—a title, a description, a priority, and a date.

Role: Holding the data in a structured way while the program is running.

Analogy: The actual ingredients and the final dish.

4. The Persistence (Storage Layer)
File: TaskStorage.java

This is the Data Access Layer. Since your computer clears its RAM when you turn it off, this component is responsible for writing your tasks to a permanent file (like tasks.json).

Role: Reading from and writing to the hard drive.

Analogy: The fridge or pantry where the food is kept so it doesn't spoil overnight.

## Exercise Part 2: Finding Feature Implementation
It should be inside TaskStorage class --> this class is responsible for saving tasks in a file, im not sure what the file type is.

the method void load, since its void it returns nothing so i assume we use it to open a file

key words i searched for: Loading file, file, anything ending with a .txt

CSV --> stands for Comma-Seoerated Values, its a simple text file where each line is a record (like a task) and each piece of data is seperated by a comma. Basicall a spreadsheet in a text file

my search was shallow --> Search for Data formats (CSV, JSON, String) and action verbs(Export, Write, Save, Pars)

Based on your file list, here is where the "Export to CSV" logic likely hides:

za.co.wethinkcode.taskmanager.util.TaskTextParser.java (High Probability): This is likely where the logic lives to turn a Task object into a line of text (the CSV format). "Parser" usually implies moving between text and objects.

za.co.wethinkcode.taskmanager.storage.TaskStorage.java: If the app saves the CSV to the hard drive, the TaskStorage will handle the actual file writing.

za.co.wethinkcode.taskmanager.app.TaskManager.java: This will have the method that says "Hey, get all tasks and send them to the CSV writer."

za.co.wethinkcode.taskmanager.cli.TaskManagerCli.java: This is where the user types export --format=csv. 

*** key words to search for *** 
"csv" (Case insensitive)

"append" or "FileWriter"

"String.join(\",\"" (This is how CSV lines are often built)

"export"

*** codebase implementation map *** 
CLI --> cli/TaskManagerCli.java --> Add a new case "export" to the switch statement and handle user arguments (like the filename).
Service -->  /app/TaskManager.java --> Create an exportTasksToCsv(String filePath) method that coordinates the data flow.
Utility --> /util/TaskTextParser.java --> Implement the logic to turn a single Task object into a CSV-formatted String.
Storage -- > storage/TaskStorage.java --> Add a method to actually write a list of Strings to a new file on the disk.

Implementation Plan (Step-by-Step)
Phase 1: The Converter (The "Internal" Logic)
Before you can save a file, you need to know what a single line looks like.

Open TaskTextParser.java.

Create a method: public String toCsvRow(Task task).

Inside, use String.join(",", ...) to combine the ID, Title, Priority, and Status into one line.

Phase 2: The Orchestrator
Open TaskManager.java.

Create a method that fetches all tasks from storage.getAllTasks().

Loop through those tasks and use your new TaskTextParser method to convert each one into a list of Strings.

Phase 3: The Writer (The "Plumbing")
In TaskStorage.java, create a method saveAsCsv(String path, List<String> lines).

Use Java's FileWriter or Files.write() to create the file and dump the data.

Phase 4: The Trigger (The "UI")
In TaskManagerCli.java, add the export command.

Connect it so that when a user types export my_tasks.csv, it calls the Manager, which calls the Parser, which tells the Storage to save.

## Exercise Part 3: Understanding Domain Model
Definition of domain relating to the exercise, Think of a Domain as the "Universe" or the "Subject Matter" that your software lives in. If you are building a banking app, the domain is Finance. If you are building a Task Manager, your domain is Productivity and Time Management.
A Domain Model is simply how we represent that real-world subject inside our code using objects and rules.


## Exercise Part 4: Reflection and Presentation
The application follows a modern Full-Stack JavaScript architecture, built for responsiveness and data integrity. It utilizes a Model-View-Controller (MVC) influenced structure to keep concerns separated:

Frontend: A React-based Single Page Application (SPA) that handles the UI and state management.

Backend: A Node.js and Express server that manages the RESTful API endpoints.

Database: A PostgreSQL relational database, ensuring that tasks and user data remain structured and persistent.

Communication: JSON is the primary data exchange format between the client and the server.

These prompts helped shift the mental model from "step-by-step execution" to "event-driven execution," which eventually made handling API calls second nature.

## Exercise: Algorithm Deconstruction Challenge
## Algorithm 1: Task Priority Sorting Algorithm

The algorithm is a scoring system, each task starts at 0 and earns or loses points based on 3 factors --> priority, due date and status.

*** How did Ai explanation change my understanding of the algorithm ***
priority --> The × 10 just gives room to add smaller bonuses on top without them "overpowering" priority.
due date --> depending on the due date depends on the urgency of the task or how close to the due date it is
status --> Tasks nearly done matter less.
Tag boost --> depending on a keyword linked to the task e.g "blocker", "critical" or "urgent" increases the priority.

*** What aspects were still difficult to understand after AI explanation? ***
long daysUntilDue = ChronoUnit.DAYS.between(LocalDateTime.now(), task.getDueDate()); --> this piece of code, ChronoUnit.DAYS.between(start, end) gives you a negative number when the end date is in the past.

*** How would you explain this algorithm to another junior developer? ***
Every task gets a score. You start with points based on how important it is, then you add bonus points if it's due soon or already late. Tasks that are done or nearly done lose points so there urgency decreases. Special tags like 'blocker' give a small boost. The task with the highest score is the most urgent."

*** How might you improve the algorithm based on your understanding? ***
Scores can go negative — a DONE + LOW priority task scores 10 - 50 = -40. That's fine functionally, but could cause bugs if anything downstream assumes scores are positive.

-----------------------------------------------------------------------------------------------------------

## Exercise: Code Documentation
*** Documentation for the text parser code (TaskTextParser.java) ***
*** Prompt 1: Comprehensive Function Documentation ***

TaskTextParser — Documentation
High-level overview
TaskTextParser converts a plain text string — the kind a user types in a chat box or CLI — into a fully populated Task object. It scans the string for special tokens (!priority, @tag, #date), strips them out of the title, and maps each one to a structured field. Everything left over becomes the task title.

Syntax reference
TokenFormatDescriptionTitleplain textAny text not starting with !, @, or # becomes the task title. Leftover whitespace is collapsed.Priority!1–!4 or !low / !medium / !high / !urgentSets the task priority. Case-insensitive. Defaults to MEDIUM if omitted.Tag@wordAdds a tag. Multiple allowed. Word characters only (\w+).Due date#today, #tomorrow, #monday…#sunday, #nextweek, #YYYY-MM-DDSets the due date. Only the first recognised date marker is used.

Method signature
javapublic static Task parseTaskFromText(String text)
Parameters
NameTypeDescriptiontextStringThe raw input string containing a task description with optional marker tokens. Must not be null; an empty string produces a Task with an empty title and default values.
Return value
TypeDescriptionTaskA new Task instance populated with the extracted title, priority, dueDate, and tags. The dueDate is null if no valid date marker was found. All other Task fields retain their constructor defaults.
Exceptions
TypeWhenNullPointerExceptionThrown if text is null. The method calls text.trim() immediately with no null guard.

Silently ignored: Unrecognised #date values that fail LocalDate.parse() — the parser moves on to the next date marker without throwing.


Parse pipeline — step by step

Initialise defaults — Title = full input. Priority defaults to MEDIUM. Due date = null. Tags = empty list.
Extract priority — Regex scans for !1–!4 or !word. First match wins. Token removed from title.
Extract tags — All @word tokens collected into a list. Scans the original text (not the cleaned title). All removed from title.
Extract dates — All #word tokens collected. First valid date parsed wins. Supports keywords + YYYY-MM-DD.
Clean title — Collapse multiple spaces. Trim leading/trailing whitespace. Assemble and return Task.


Key logic — annotated
Weekday calculation (getNextWeekday)
java// Modular arithmetic trick: finds days to the next target weekday
int daysToAdd = (targetDayOfWeek - currentDayOfWeek + 7) % 7;

// +7 prevents negative result when target < current (e.g. Mon=1 from Fri=5)
// %7 brings values >= 7 back into 0–6 range
// result 0 means today IS the target day → force to next week
if (daysToAdd == 0) daysToAdd = 7;
ScenarioCalculationResultToday is Friday (5), target is Monday (1)(1 − 5 + 7) % 7 = 33 days ahead ✓Today is Monday (1), target is Monday (1)(1 − 1 + 7) % 7 = 0 → forced to 7Next week ✓
Why tags scan the original text
java// Tags scan the ORIGINAL text, not the title being cleaned
Matcher tagMatcher = tagPattern.matcher(text);

// Dates also scan the original text
Matcher dateMatcher = datePattern.matcher(text);

// But priority scans `title` (which is already title = text.trim())
// Priority regex requires a leading SPACE → won't match at position 0
Matcher priorityMatcher = priorityPattern.matcher(title);
Tags are extracted from text (the original parameter) rather than from the title variable, which is being progressively cleaned. This ensures tokens are captured correctly regardless of strip order. The downside is the priority regex requires a leading \s, so !urgent at position 0 will never match.

Assumptions and edge cases
Null input crashes
text.trim() is called immediately. Passing null throws a NullPointerException with no guard.
First-date-wins, silently
Multiple # markers: only the first valid one is used. #monday #friday → Monday. Friday is discarded with no warning.
Priority at start of string
"!urgent Buy milk" — no leading space before !, so the regex won't match. Priority silently defaults to MEDIUM.
Day-of-week always means next occurrence
Writing #monday on a Monday gives you next Monday (+7 days), not today. This is intentional. Use #today for same-day.
Unrecognised dates silently skipped
#asap or #q4 fail the LocalDate.parse(), catch the exception quietly, and are ignored — no error is surfaced to the caller.
Multiple tags work correctly
@work @project @urgent are all captured. Tags are read from the original string even after title cleaning.

Example usage
Basic task with all markers
javaString input = "Finish report for client XYZ !urgent #friday @work @project";
Task task = TaskTextParser.parseTaskFromText(input);

task.getTitle();     // "Finish report for client XYZ"
task.getPriority();  // TaskPriority.URGENT
task.getDueDate();   // next Friday at 00:00:00
task.getTags();      // ["work", "project"]
Numeric priority + explicit date
javaString input = "Buy milk @shopping !2 #2025-12-31";
Task task = TaskTextParser.parseTaskFromText(input);

task.getTitle();     // "Buy milk"
task.getPriority();  // TaskPriority.MEDIUM
task.getDueDate();   // 2025-12-31T00:00:00
No markers — defaults applied
javaString input = "Call the plumber";
Task task = TaskTextParser.parseTaskFromText(input);

task.getTitle();     // "Call the plumber"
task.getPriority();  // TaskPriority.MEDIUM  (default)
task.getDueDate();   // null
task.getTags();      // []

Private helper
javaprivate static LocalDateTime getNextWeekday(LocalDateTime date, int targetDayOfWeek)
Computes the next occurrence of a given day of the week (1=Monday … 7=Sunday) from date. If date already falls on targetDayOfWeek, it advances by 7 days (i.e. next week, not today). Used internally by parseTaskFromText.

Suggested improvements
1. Add a null guard
A single check turns a cryptic NullPointerException into a clear contract violation:
javaif (text == null) throw new IllegalArgumentException("text must not be null");
2. Fix the leading-space priority bug
Change the pattern so a priority marker at position 0 is also matched:
java// Before
Pattern.compile("\\s!([1-4]|urgent|high|medium|low)\\b", Pattern.CASE_INSENSITIVE)

// After
Pattern.compile("(?:^|\\s)!([1-4]|urgent|high|medium|low)\\b", Pattern.CASE_INSENSITIVE)
3. Compile patterns once as static constants
Pattern.compile() is called on every method invocation. Move them to class-level fields:
javaprivate static final Pattern PRIORITY_PATTERN =
    Pattern.compile("(?:^|\\s)!([1-4]|urgent|high|medium|low)\\b", Pattern.CASE_INSENSITIVE);

private static final Pattern TAG_PATTERN =
    Pattern.compile("\\s@(\\w+)");

private static final Pattern DATE_PATTERN =
    Pattern.compile("\\s#(\\w+)");
4. Extract the day map as a static constant
Map.ofEntries(…) inside the date loop recreates the map on every call. Extract it:
javaprivate static final Map<String, Integer> DAY_MAP = Map.ofEntries(
    Map.entry("monday", DayOfWeek.MONDAY.getValue()),
    Map.entry("mon",    DayOfWeek.MONDAY.getValue()),
    // ... etc
);
5. Replace the priority switch with a Map lookup
javaprivate static final Map<String, TaskPriority> PRIORITY_MAP = Map.of(
    "1", TaskPriority.LOW,    "low",    TaskPriority.LOW,
    "2", TaskPriority.MEDIUM, "medium", TaskPriority.MEDIUM,
    "3", TaskPriority.HIGH,   "high",   TaskPriority.HIGH,
    "4", TaskPriority.URGENT, "urgent", TaskPriority.URGENT
);

// Usage
priority = PRIORITY_MAP.getOrDefault(priorityText, TaskPriority.MEDIUM);
6. Surface parse feedback to the caller
Right now unrecognised date markers and a priority-at-position-0 are silent failures. Consider a wrapper return type:
javapublic record ParseResult(Task task, List<String> warnings) {}
This lets the UI tell the user "I didn't understand #asap" rather than silently ignoring it.\

*** Prompt 2: Intent and Logic Explanation *** 

TaskPriorityManager — Documentation
High-level overview
TaskPriorityManager calculates a numeric urgency score for each task based on multiple weighted factors, then uses that score to sort and filter task lists. Think of it as a points table — each task starts at 0 and earns or loses points depending on its priority level, due date, status, tags, and recency. The task with the highest final score is treated as the most urgent.
FINAL SCORE = Priority Points + Due Date Points − Status Penalty + Tag Bonus + Recency Bonus

Methods
calculateTaskScore
javapublic static int calculateTaskScore(Task task)
Calculates a single integer score representing the urgency of a task.
Parameters
NameTypeDescriptiontaskTaskThe task to score. Must not be null. Assumes task.getPriority(), task.getStatus(), task.getTags(), and task.getUpdatedAt() are all non-null. task.getDueDate() may be null — the due date factor is skipped if so.
Return value
TypeDescriptionintA score representing urgency. Higher = more urgent. Can be negative (e.g. a DONE + LOW priority task scores −40).

sortTasksByImportance
javapublic static List<Task> sortTasksByImportance(List<Task> tasks)
Returns a new list of tasks sorted by score descending (highest urgency first). Does not mutate the input list.
Parameters
NameTypeDescriptiontasksList<Task>The tasks to sort. Must not be null. Can be empty.
Return value
TypeDescriptionList<Task>A new List<Task> ordered from highest to lowest score.

getTopPriorityTasks
javapublic static List<Task> getTopPriorityTasks(List<Task> tasks, int limit)
Returns the top N tasks by urgency score.
Parameters
NameTypeDescriptiontasksList<Task>The full task list. Must not be null.limitintMaximum number of tasks to return. If limit exceeds the list size, all tasks are returned.
Return value
TypeDescriptionList<Task>A new List<Task> of at most limit tasks, ordered highest score first.

Scoring breakdown
1. Priority (base score)
PriorityWeightScoreLOW1 × 1010MEDIUM2 × 1020HIGH3 × 1030URGENT4 × 1040
The × 10 multiplier creates headroom so smaller bonuses don't overpower the base priority level.
2. Due date bonus
ConditionPoints addedOverdue (past due date)+30Due today+20Due in 1–2 days+15Due within 7 days+10Due further away or no due date+0
3. Status penalty
StatusPoints removedDONE−50REVIEW−15All others−0
4. Tag boost
If any of the task's tags match "blocker", "critical", or "urgent" → +8
5. Recency boost
If the task was updated within the last 24 hours → +5

Concrete example
FactorTask ATask BPriorityHIGH → +30MEDIUM → +20Due dateNext week → +10Overdue → +30StatusIn progress → +0In progress → +0Tagsnone → +0"blocker" → +8RecencyNot recent → +0Not recent → +0Total4058
Task B wins despite Task A having a higher priority — because it is overdue and tagged as a blocker. This is an intentional design decision: urgency is contextual, not just a label.

Key logic — annotated
java// ChronoUnit.DAYS.between(start, end) returns a NEGATIVE number when end is in the past.
// So daysUntilDue < 0 means the task is overdue — the due date has already passed.
long daysUntilDue = ChronoUnit.DAYS.between(LocalDateTime.now(), task.getDueDate());
if (daysUntilDue < 0) {   // overdue
    score += 30;
} else if (daysUntilDue == 0) {  // due today
    score += 20;
}
java// .stream().anyMatch() checks if AT LEAST ONE tag is in the special list.
// It short-circuits — stops checking as soon as it finds the first match.
if (task.getTags().stream().anyMatch(tag ->
        List.of("blocker", "critical", "urgent").contains(tag))) {
    score += 8;
}
java// daysSinceUpdate uses the reverse order: between(updatedAt, now).
// A positive result means time has passed since the update.
// < 1 means updated less than 24 hours ago.
long daysSinceUpdate = ChronoUnit.DAYS.between(task.getUpdatedAt(), LocalDateTime.now());
if (daysSinceUpdate < 1) {
    score += 5;
}

Assumptions and edge cases
Scores can go negative
A DONE + LOW priority task scores 10 − 50 = −40. This is fine functionally but could cause bugs if any downstream code assumes scores are always positive.
calculateTaskScore is called once per sort comparison
sortTasksByImportance uses Comparator.comparing(TaskPriorityManager::calculateTaskScore). Java's sort calls the comparator multiple times — meaning calculateTaskScore is called more than once per task during sorting. For large lists this is inefficient.
Magic numbers throughout
All point values (30, 20, 15, 10, 8, 5, 50) are hardcoded. There is no way to tune scoring behaviour without editing the source.
Tag boost is weak relative to other factors
+8 for a "blocker" tag is less than the recency bonus and far less than the overdue bonus. A blocker that is due next month scores lower than a low-priority task due today.
No penalty for stale tasks
A task created weeks ago with no progress gets no urgency boost from age. Only recency (last updated) is considered, and only as a bonus — never as a negative signal.

Suggested improvements
1. Cache scores before sorting
calculateTaskScore is called multiple times per task during sort. Precompute once:
javapublic static List<Task> sortTasksByImportance(List<Task> tasks) {
    return tasks.stream()
        .map(t -> Map.entry(t, calculateTaskScore(t)))  // compute once
        .sorted(Map.Entry.<Task, Integer>comparingByValue().reversed())
        .map(Map.Entry::getKey)
        .collect(Collectors.toList());
}
2. Extract scoring constants
Replace magic numbers with named constants so the weights can be understood and adjusted:
javaprivate static final int OVERDUE_BONUS      = 30;
private static final int DUE_TODAY_BONUS    = 20;
private static final int DUE_SOON_BONUS     = 15;
private static final int DUE_THIS_WEEK_BONUS = 10;
private static final int BLOCKER_TAG_BONUS  = 8;
private static final int RECENT_UPDATE_BONUS = 5;
private static final int DONE_PENALTY       = 50;
private static final int REVIEW_PENALTY     = 15;
3. Guard against negative scores downstream
If anything consuming these scores assumes non-negative values, add a floor:
javareturn Math.max(0, score);
Or document clearly that negative scores are valid and expected.
4. Add task age as a creeping urgency factor
Tasks that have been sitting untouched for a long time arguably deserve a boost:
javalong daysSinceCreation = ChronoUnit.DAYS.between(task.getCreatedAt(), LocalDateTime.now());
if (daysSinceCreation > 14) score += 5;
if (daysSinceCreation > 30) score += 10;
5. Make scoring configurable
For teams with different workflows, consider accepting a ScoringConfig object rather than hardcoding all weights. This keeps the algorithm logic intact while allowing the values to be tuned without code changes.



### Combined version of both prompts

Task Manager — Developer Documentation

This document covers two utility classes that work together to handle task creation from free-form text and urgency-based prioritisation: TaskTextParser and TaskPriorityManager.


Table of contents

Overview — how the two classes connect
TaskTextParser

High-level intent
Syntax reference
Method signature
Parse pipeline
Key logic annotated
Assumptions and edge cases
Example usage
Suggested improvements


TaskPriorityManager

High-level intent
Methods
Scoring breakdown
Key logic annotated
Assumptions and edge cases
Suggested improvements




Overview — how the two classes connect <a name="overview"></a>
These two classes form a pipeline. TaskTextParser sits at the entry point — it takes raw user input and produces a structured Task. TaskPriorityManager sits downstream — it takes a list of Task objects and ranks them by urgency.
User types:  "Fix login bug !urgent #tomorrow @backend"
                          │
                          ▼
               TaskTextParser.parseTaskFromText()
                          │
                          ▼
              Task { title="Fix login bug",
                     priority=URGENT,
                     dueDate=tomorrow,
                     tags=["backend"] }
                          │
                          ▼
         TaskPriorityManager.calculateTaskScore()
                          │
                          ▼
                      score = 75
                    (ranks near top)
The fields that TaskTextParser sets — priority, dueDate, and tags — are exactly the fields that TaskPriorityManager scores. They are designed to be used together.

TaskTextParser <a name="tasktextparser"></a>
High-level intent <a name="tasktextparser-intent"></a>
TaskTextParser converts a plain text string into a fully populated Task object. It scans for special marker tokens (!priority, @tag, #date), strips them from the title, and maps each one to a structured field. Everything left over becomes the task title.

Syntax reference <a name="syntax-reference"></a>
TokenFormatDescriptionTitleplain textAny text not starting with !, @, or # becomes the task title. Leftover whitespace is collapsed.Priority!1–!4 or !low / !medium / !high / !urgentSets task priority. Case-insensitive. Defaults to MEDIUM if omitted.Tag@wordAdds a tag. Multiple allowed. Word characters only (\w+).Due date#today, #tomorrow, #monday…#sunday, #nextweek, #YYYY-MM-DDSets due date. Only the first recognised marker is used.

Method signature <a name="tasktextparser-method"></a>
javapublic static Task parseTaskFromText(String text)
Parameters
NameTypeDescriptiontextStringRaw input string with optional marker tokens. Must not be null. An empty string produces a Task with an empty title and all defaults.
Return value
TypeDescriptionTaskA new Task populated with extracted title, priority, dueDate, and tags. dueDate is null if no valid date marker was found. All other fields use constructor defaults.
Exceptions
TypeWhenNullPointerExceptionThrown if text is null. text.trim() is called immediately with no null guard.

Silently ignored: Unrecognised #date values that fail LocalDate.parse() — the parser moves on to the next date marker without throwing.


Parse pipeline <a name="parse-pipeline"></a>
StepWhat happens1. Initialise defaultsTitle = full input. Priority = MEDIUM. Due date = null. Tags = empty list.2. Extract priorityRegex scans for !1–!4 or !word. First match wins. Token removed from title.3. Extract tagsAll @word tokens collected into a list. Scans the original text. All removed from title.4. Extract datesAll #word tokens collected. First valid date wins. Supports keywords + YYYY-MM-DD.5. Clean titleCollapse multiple spaces. Trim. Assemble and return Task.

Key logic annotated <a name="tasktextparser-logic"></a>
Weekday calculation
java// Modular arithmetic: finds days until the next occurrence of a target weekday.
int daysToAdd = (targetDayOfWeek - currentDayOfWeek + 7) % 7;

// +7 prevents a negative result when the target day is earlier in the week than today.
// %7 brings any value >= 7 back into the 0–6 range.
// A result of 0 means today IS the target day → push to next week instead.
if (daysToAdd == 0) daysToAdd = 7;
ScenarioCalculationResultToday is Friday (5), target Monday (1)(1 − 5 + 7) % 7 = 3+3 days ✓Today is Monday (1), target Monday (1)(1 − 1 + 7) % 7 = 0 → forced to 7Next week ✓
Why tags scan the original text
java// Tags and dates scan `text` — the original, unmodified input parameter.
Matcher tagMatcher  = tagPattern.matcher(text);
Matcher dateMatcher = datePattern.matcher(text);

// Priority scans `title` (= text.trim()), which is progressively cleaned.
// IMPORTANT: the priority regex requires a leading \s — so a priority marker
// at position 0 of the string will NEVER match.
Matcher priorityMatcher = priorityPattern.matcher(title);

Assumptions and edge cases <a name="tasktextparser-edge-cases"></a>
TypeScenarioBehaviour💥 Crashtext is nullNullPointerException — no null guard exists⚠️ SilentPriority marker at start of string: "!urgent Buy milk"No leading space → regex won't match → defaults to MEDIUM⚠️ SilentMultiple date markers: "#monday #friday"First valid one wins (Monday). Friday silently discarded.⚠️ SilentUnrecognised date: "#asap" or "#q4"DateTimeParseException caught quietly. No error surfaced.ℹ️ Intentional#monday used on a MondaySets due date to next Monday (+7 days). Use #today for same-day.✅ WorksMultiple tags: @work @project @urgentAll captured correctly from the original text.

Example usage <a name="tasktextparser-examples"></a>
All markers
javaString input = "Finish report for client XYZ !urgent #friday @work @project";
Task task = TaskTextParser.parseTaskFromText(input);

task.getTitle();     // "Finish report for client XYZ"
task.getPriority();  // TaskPriority.URGENT
task.getDueDate();   // next Friday at 00:00:00
task.getTags();      // ["work", "project"]
Numeric priority + explicit date
javaString input = "Buy milk @shopping !2 #2025-12-31";
Task task = TaskTextParser.parseTaskFromText(input);

task.getTitle();     // "Buy milk"
task.getPriority();  // TaskPriority.MEDIUM
task.getDueDate();   // 2025-12-31T00:00:00
No markers — all defaults
javaString input = "Call the plumber";
Task task = TaskTextParser.parseTaskFromText(input);

task.getTitle();     // "Call the plumber"
task.getPriority();  // TaskPriority.MEDIUM  (default)
task.getDueDate();   // null
task.getTags();      // []

Suggested improvements <a name="tasktextparser-improvements"></a>
1. Add a null guard
javaif (text == null) throw new IllegalArgumentException("text must not be null");
2. Fix the leading-space priority bug
java// Before — won't match if ! is the first character
Pattern.compile("\\s!([1-4]|urgent|high|medium|low)\\b", Pattern.CASE_INSENSITIVE)

// After — matches at start of string OR after whitespace
Pattern.compile("(?:^|\\s)!([1-4]|urgent|high|medium|low)\\b", Pattern.CASE_INSENSITIVE)
3. Compile patterns once as static constants
java// Before — recompiled on every method call
Pattern priorityPattern = Pattern.compile("...");

// After — compiled once at class load
private static final Pattern PRIORITY_PATTERN = Pattern.compile("...");
private static final Pattern TAG_PATTERN      = Pattern.compile("\\s@(\\w+)");
private static final Pattern DATE_PATTERN     = Pattern.compile("\\s#(\\w+)");
4. Extract the day map as a static constant
java// Before — Map.ofEntries(...) called inside the loop on every parse
// After
private static final Map<String, Integer> DAY_MAP = Map.ofEntries(
    Map.entry("monday", DayOfWeek.MONDAY.getValue()),
    Map.entry("mon",    DayOfWeek.MONDAY.getValue()),
    // ... etc
);
5. Replace the priority switch with a Map lookup
javaprivate static final Map<String, TaskPriority> PRIORITY_MAP = Map.of(
    "1", TaskPriority.LOW,    "low",    TaskPriority.LOW,
    "2", TaskPriority.MEDIUM, "medium", TaskPriority.MEDIUM,
    "3", TaskPriority.HIGH,   "high",   TaskPriority.HIGH,
    "4", TaskPriority.URGENT, "urgent", TaskPriority.URGENT
);

// Usage — replaces the entire switch block
priority = PRIORITY_MAP.getOrDefault(priorityText, TaskPriority.MEDIUM);
6. Surface parse warnings to the caller
java// Consider a wrapper return type so the UI can tell the user what was ignored
public record ParseResult(Task task, List<String> warnings) {}

TaskPriorityManager <a name="taskprioritymanager"></a>
High-level intent <a name="taskprioritymanager-intent"></a>
TaskPriorityManager calculates a numeric urgency score for each task based on multiple weighted factors, then uses that score to sort and filter task lists. Each task starts at 0 and earns or loses points depending on its priority level, due date, status, tags, and recency.
FINAL SCORE = Priority Points + Due Date Bonus − Status Penalty + Tag Bonus + Recency Bonus

Methods <a name="taskprioritymanager-methods"></a>
calculateTaskScore
javapublic static int calculateTaskScore(Task task)
ParameterTypeDescriptiontaskTaskThe task to score. getPriority(), getStatus(), getTags(), and getUpdatedAt() must be non-null. getDueDate() may be null — the due date factor is skipped if so.
Return typeDescriptionintUrgency score. Higher = more urgent. Can be negative (e.g. DONE + LOW priority = −40).

sortTasksByImportance
javapublic static List<Task> sortTasksByImportance(List<Task> tasks)
Returns a new list sorted by score descending. Does not mutate the input list.

getTopPriorityTasks
javapublic static List<Task> getTopPriorityTasks(List<Task> tasks, int limit)
Returns the top N tasks by urgency score. If limit exceeds list size, all tasks are returned.

Scoring breakdown <a name="scoring-breakdown"></a>
1. Priority (base)
PriorityScoreLOW+10MEDIUM+20HIGH+30URGENT+40
The × 10 multiplier creates headroom so smaller bonuses don't overpower the base.
2. Due date bonus
ConditionPointsOverdue+30Due today+20Due in 1–2 days+15Due within 7 days+10Due further away / no due date+0
3. Status penalty
StatusPointsDONE−50REVIEW−15All others−0
4. Tag boost
If any tag matches "blocker", "critical", or "urgent" → +8
5. Recency boost
If updated within the last 24 hours → +5

Scoring in practice
FactorTask ATask BPriorityHIGH → +30MEDIUM → +20Due dateNext week → +10Overdue → +30StatusIn progress → +0In progress → +0Tagsnone → +0"blocker" → +8RecencyNot recent → +0Not recent → +0Total4058
Task B wins despite Task A having a higher priority — because it is overdue and tagged as a blocker. Urgency is contextual, not just a label.

Key logic annotated <a name="taskprioritymanager-logic"></a>
Due date: why negative means overdue
java// ChronoUnit.DAYS.between(start, end) returns NEGATIVE when end is in the past.
// So daysUntilDue < 0 means the due date has already passed.
long daysUntilDue = ChronoUnit.DAYS.between(LocalDateTime.now(), task.getDueDate());

if (daysUntilDue < 0) {        // overdue → highest bonus
    score += 30;
} else if (daysUntilDue == 0) { // due today
    score += 20;
}
Tag check: short-circuit stream
java// anyMatch() stops at the first matching tag — it does not scan the whole list.
// List.of() creates a small throwaway list on every call (see improvements).
if (task.getTags().stream().anyMatch(tag ->
        List.of("blocker", "critical", "urgent").contains(tag))) {
    score += 8;
}
Recency: between order is reversed
java// Note the argument order is flipped vs the due date check above.
// between(updatedAt, now) gives a POSITIVE number for time that has passed.
// < 1 means updated less than 24 hours ago.
long daysSinceUpdate = ChronoUnit.DAYS.between(task.getUpdatedAt(), LocalDateTime.now());
if (daysSinceUpdate < 1) {
    score += 5;
}

Assumptions and edge cases <a name="taskprioritymanager-edge-cases"></a>
TypeScenarioBehaviour⚠️ Possible bugDONE + LOW taskScore = −40. Valid logically, but may cause issues if downstream code assumes non-negative scores.⚠️ PerformancecalculateTaskScore called multiple times per task during sortJava's sort calls the comparator repeatedly — score is recomputed each time instead of cached.⚠️ DesignTag boost (+8) is weakA "blocker" tag adds less than the recency bonus and far less than the overdue bonus.ℹ️ IntentionalNo task age factorOnly recency (last updated) is considered, not how long a task has existed without progress.ℹ️ HardcodedAll point values are magic numbers30, 20, 15, 8, 5, 50 etc. cannot be tuned without editing source.

Suggested improvements <a name="taskprioritymanager-improvements"></a>
1. Cache scores before sorting
calculateTaskScore is called more than once per task by the sort comparator. Precompute to fix this:
javapublic static List<Task> sortTasksByImportance(List<Task> tasks) {
    return tasks.stream()
        .map(t -> Map.entry(t, calculateTaskScore(t)))  // score computed once per task
        .sorted(Map.Entry.<Task, Integer>comparingByValue().reversed())
        .map(Map.Entry::getKey)
        .collect(Collectors.toList());
}
2. Extract scoring constants
javaprivate static final int OVERDUE_BONUS        = 30;
private static final int DUE_TODAY_BONUS      = 20;
private static final int DUE_SOON_BONUS       = 15;
private static final int DUE_THIS_WEEK_BONUS  = 10;
private static final int BLOCKER_TAG_BONUS    = 8;
private static final int RECENT_UPDATE_BONUS  = 5;
private static final int DONE_PENALTY         = 50;
private static final int REVIEW_PENALTY       = 15;
3. Extract the special tags set as a constant
java// Before — new List created on every tag check
List.of("blocker", "critical", "urgent").contains(tag)

// After
private static final Set<String> BOOSTED_TAGS = Set.of("blocker", "critical", "urgent");

// Usage — Set.contains() is also O(1) vs List.contains() O(n)
if (task.getTags().stream().anyMatch(BOOSTED_TAGS::contains)) {
    score += BLOCKER_TAG_BONUS;
}
4. Guard against negative scores downstream
If any consumer assumes non-negative values, add a floor:
javareturn Math.max(0, score);
5. Add task age as a creeping urgency factor
javalong daysSinceCreation = ChronoUnit.DAYS.between(task.getCreatedAt(), LocalDateTime.now());
if (daysSinceCreation > 14) score += 5;
if (daysSinceCreation > 30) score += 10;
6. Make scoring configurable
For teams with different workflows, consider a ScoringConfig parameter rather than hardcoded weights. This keeps the algorithm intact while allowing values to be tuned per environment without code changes.

-----------------------------------------------------------------------------------------------------------

## Exercise: API Documentation 
*** enpoint API code ***
/**
 * Product API endpoints
 */
const productRouter = express.Router();

// Get all products with filtering and pagination
productRouter.get('/', async (req, res) => {
  try {
    const {
      category,
      minPrice,
      maxPrice,
      sort = 'createdAt',
      order = 'desc',
      page = 1,
      limit = 20,
      inStock
    } = req.query;

    // Build filter
    const filter = {};

    if (category) {
      filter.category = category;
    }

    if (minPrice !== undefined || maxPrice !== undefined) {
      filter.price = {};
      if (minPrice !== undefined) filter.price.$gte = parseFloat(minPrice);
      if (maxPrice !== undefined) filter.price.$lte = parseFloat(maxPrice);
    }

    if (inStock === 'true') {
      filter.stockQuantity = { $gt: 0 };
    }

    // Calculate pagination
    const skip = (parseInt(page) - 1) * parseInt(limit);

    // Determine sort order
    const sortOptions = {};
    sortOptions[sort] = order === 'asc' ? 1 : -1;

    // Execute query
    const products = await ProductModel.find(filter)
      .sort(sortOptions)
      .skip(skip)
      .limit(parseInt(limit));

    // Get total count for pagination
    const totalProducts = await ProductModel.countDocuments(filter);

    return res.status(200).json({
      products,
      pagination: {
        total: totalProducts,
        page: parseInt(page),
        limit: parseInt(limit),
        pages: Math.ceil(totalProducts / parseInt(limit))
      }
    });
  } catch (error) {
    console.error('Error fetching products:', error);
    return res.status(500).json({
      error: 'Server error',
      message: 'Failed to fetch products'
    });
  }
});

// Get product by ID
productRouter.get('/:productId', async (req, res) => {
  try {
    const { productId } = req.params;

    const product = await ProductModel.findById(productId);

    if (!product) {
      return res.status(404).json({
        error: 'Not found',
        message: 'Product not found'
      });
    }

    return res.status(200).json(product);
  } catch (error) {
    console.error('Error fetching product:', error);

    // Check if error is invalid ObjectId format
    if (error.name === 'CastError') {
      return res.status(400).json({
        error: 'Invalid ID',
        message: 'Invalid product ID format'
      });
    }

    return res.status(500).json({
      error: 'Server error',
      message: 'Failed to fetch product'
    });
  }
});

module.exports = productRouter;

*** Prompt 1: Endpoint Documentation Generation *** 
Product API — Endpoint Documentation

Base path: /products
All responses are JSON.


Table of contents

GET /products
GET /products/:productId
Authentication
Rate limiting and considerations
Error reference


GET /products <a name="get-products"></a>
Description
Returns a paginated list of products. Supports optional filtering by category, price range, and stock availability. Results can be sorted by any product field in ascending or descending order.

Query parameters
ParameterTypeRequiredDefaultDescriptioncategorystringNo—Filter by product category. Exact match. Example: "electronics"minPricenumberNo—Returns products with price ≥ this value.maxPricenumberNo—Returns products with price ≤ this value.inStockbooleanNo—Pass "true" to return only products where stockQuantity > 0.sortstringNo"createdAt"Field to sort results by. Any valid product field e.g. "price", "name".orderstringNo"desc"Sort direction. Accepts "asc" or "desc".pageintegerNo1Page number. Starts at 1.limitintegerNo20Number of results per page.

Response — 200 OK
json{
  "products": [
    {
      "_id": "64a7f3c2e1b2a3d4e5f67890",
      "name": "Wireless Headphones",
      "category": "electronics",
      "price": 79.99,
      "stockQuantity": 42,
      "createdAt": "2024-01-15T08:30:00.000Z"
    }
  ],
  "pagination": {
    "total": 84,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
}
FieldTypeDescriptionproductsarrayProducts matching the filter criteria.pagination.totalintegerTotal matching products across all pages.pagination.pageintegerCurrent page number.pagination.limitintegerResults per page.pagination.pagesintegerTotal number of pages (Math.ceil(total / limit)).

Example 1 — Filter by category, sorted by price
Request
GET /products?category=electronics&sort=price&order=asc&page=1&limit=10
Response — 200 OK
json{
  "products": [
    {
      "_id": "64a7f3c2e1b2a3d4e5f67891",
      "name": "USB-C Cable",
      "category": "electronics",
      "price": 12.99,
      "stockQuantity": 200,
      "createdAt": "2024-02-01T10:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 53,
    "page": 1,
    "limit": 10,
    "pages": 6
  }
}

Example 2 — Filter by price range, in stock only
Request
GET /products?minPrice=50&maxPrice=200&inStock=true
Response — 200 OK
json{
  "products": [
    {
      "_id": "64a7f3c2e1b2a3d4e5f67893",
      "name": "Bluetooth Speaker",
      "category": "electronics",
      "price": 149.99,
      "stockQuantity": 17,
      "createdAt": "2024-01-20T09:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 1,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}

Error responses
StatusErrorMessageCause500"Server error""Failed to fetch products"Unexpected database or server failure.

GET /products/:productId <a name="get-products-id"></a>
Description
Returns a single product by its unique MongoDB ObjectId. Returns 404 if no product exists with that ID and 400 if the ID format is invalid.

Path parameters
ParameterTypeRequiredDescriptionproductIdstringYesA valid MongoDB ObjectId (24-character hex string). Example: "64a7f3c2e1b2a3d4e5f67890"

Response — 200 OK
json{
  "_id": "64a7f3c2e1b2a3d4e5f67890",
  "name": "Wireless Headphones",
  "category": "electronics",
  "price": 79.99,
  "stockQuantity": 42,
  "createdAt": "2024-01-15T08:30:00.000Z",
  "updatedAt": "2024-03-10T11:45:00.000Z"
}

Example 1 — Valid product ID
Request
GET /products/64a7f3c2e1b2a3d4e5f67890
Response — 200 OK
json{
  "_id": "64a7f3c2e1b2a3d4e5f67890",
  "name": "Wireless Headphones",
  "category": "electronics",
  "price": 79.99,
  "stockQuantity": 42,
  "createdAt": "2024-01-15T08:30:00.000Z",
  "updatedAt": "2024-03-10T11:45:00.000Z"
}

Example 2 — Product not found
Request
GET /products/64a7f3c2e1b2a3d4e5f00000
Response — 404 Not Found
json{
  "error": "Not found",
  "message": "Product not found"
}

Example 3 — Invalid ID format
Request
GET /products/not-a-valid-id
Response — 400 Bad Request
json{
  "error": "Invalid ID",
  "message": "Invalid product ID format"
}

Error responses
StatusErrorMessageCause400"Invalid ID""Invalid product ID format"productId is not a valid MongoDB ObjectId.404"Not found""Product not found"No product exists with the given productId.500"Server error""Failed to fetch product"Unexpected database or server failure.

Authentication <a name="authentication"></a>
Neither endpoint currently implements authentication. Both routes are publicly accessible as written. Before deploying to production, consider adding JWT middleware or API key validation depending on your use case.

Rate limiting and special considerations <a name="rate-limiting"></a>
No rate limiting is applied to these endpoints. Three things in the current implementation are also worth addressing before going live:
sort is not validated — any string can be passed, including fields that don't exist on the document. MongoDB will silently return unexpected results. Whitelist the allowed sort fields.
limit has no maximum — a caller can pass ?limit=999999 and pull the entire collection in one request. Enforce a cap with Math.min(parseInt(limit), 100).
page and limit have no type validation — passing ?limit=abc results in NaN. Add input validation before using these values in the query.

Error reference <a name="error-reference"></a>
Status codeWhen it occurs200 OKRequest succeeded.400 Bad RequestInvalid productId format (GET /:productId only).404 Not FoundNo product found with the given productId.500 Internal Server ErrorUnhandled database or server exception.

*** Prompt 2: API Reference Conversion *** 

openapi: 3.0.3
info:
  title: Product API
  description: >
    RESTful API for managing and querying products. Supports filtering by
    category, price range, and stock availability, as well as sorting and
    pagination.
  version: 1.0.0

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: http://localhost:3000/v1
    description: Local development server

tags:
  - name: Products
    description: Endpoints for product retrieval and management

paths:
  /products:
    get:
      tags:
        - Products
      summary: List all products
      description: >
        Returns a paginated list of products. Supports filtering by category,
        price range, and stock status. Results can be sorted by any product
        field in ascending or descending order.
      operationId: getProducts
      parameters:
        - name: category
          in: query
          description: Filter products by category name.
          required: false
          schema:
            type: string
            example: electronics
        - name: minPrice
          in: query
          description: Filter products with price greater than or equal to this value.
          required: false
          schema:
            type: number
            format: float
            minimum: 0
            example: 10.00
        - name: maxPrice
          in: query
          description: Filter products with price less than or equal to this value.
          required: false
          schema:
            type: number
            format: float
            minimum: 0
            example: 500.00
        - name: inStock
          in: query
          description: When set to `true`, returns only products with stock quantity greater than 0.
          required: false
          schema:
            type: string
            enum: [true, false]
            example: true
        - name: sort
          in: query
          description: Field name to sort results by.
          required: false
          schema:
            type: string
            default: createdAt
            example: price
        - name: order
          in: query
          description: Sort direction.
          required: false
          schema:
            type: string
            enum: [asc, desc]
            default: desc
            example: asc
        - name: page
          in: query
          description: Page number for pagination (1-indexed).
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
            example: 1
        - name: limit
          in: query
          description: Number of products to return per page.
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
            example: 20
      responses:
        "200":
          description: A paginated list of products matching the given filters.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProductListResponse"
              example:
                products:
                  - id: "64b8f3a2c9e77b001f4e3d12"
                    name: "Wireless Headphones"
                    category: "electronics"
                    price: 79.99
                    stockQuantity: 150
                    createdAt: "2024-01-15T10:30:00Z"
                    updatedAt: "2024-03-01T08:00:00Z"
                  - id: "64b8f3a2c9e77b001f4e3d13"
                    name: "Mechanical Keyboard"
                    category: "electronics"
                    price: 129.99
                    stockQuantity: 42
                    createdAt: "2024-02-20T14:45:00Z"
                    updatedAt: "2024-03-05T11:30:00Z"
                pagination:
                  total: 84
                  page: 1
                  limit: 20
                  pages: 5
        "500":
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ServerError"
              example:
                error: "Server error"
                message: "Failed to fetch products"

  /products/{productId}:
    get:
      tags:
        - Products
      summary: Get a product by ID
      description: >
        Returns a single product identified by its unique MongoDB ObjectId.
        Returns 404 if no product exists with the given ID, or 400 if the
        ID format is invalid.
      operationId: getProductById
      parameters:
        - name: productId
          in: path
          description: The unique MongoDB ObjectId of the product.
          required: true
          schema:
            type: string
            pattern: "^[a-fA-F0-9]{24}$"
            example: "64b8f3a2c9e77b001f4e3d12"
      responses:
        "200":
          description: The product matching the given ID.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Product"
              example:
                id: "64b8f3a2c9e77b001f4e3d12"
                name: "Wireless Headphones"
                category: "electronics"
                price: 79.99
                stockQuantity: 150
                createdAt: "2024-01-15T10:30:00Z"
                updatedAt: "2024-03-01T08:00:00Z"
        "400":
          description: The provided product ID is not a valid MongoDB ObjectId.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/BadRequestError"
              example:
                error: "Invalid ID"
                message: "Invalid product ID format"
        "404":
          description: No product was found with the given ID.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/NotFoundError"
              example:
                error: "Not found"
                message: "Product not found"
        "500":
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ServerError"
              example:
                error: "Server error"
                message: "Failed to fetch product"

components:
  schemas:

    Product:
      type: object
      description: Represents a single product in the catalog.
      required:
        - id
        - name
        - category
        - price
        - stockQuantity
        - createdAt
        - updatedAt
      properties:
        id:
          type: string
          description: Unique MongoDB ObjectId for the product.
          pattern: "^[a-fA-F0-9]{24}$"
          example: "64b8f3a2c9e77b001f4e3d12"
        name:
          type: string
          description: Display name of the product.
          example: "Wireless Headphones"
        category:
          type: string
          description: Category the product belongs to.
          example: "electronics"
        price:
          type: number
          format: float
          description: Price of the product in the store's default currency.
          minimum: 0
          example: 79.99
        stockQuantity:
          type: integer
          description: Number of units currently in stock.
          minimum: 0
          example: 150
        createdAt:
          type: string
          format: date-time
          description: ISO 8601 timestamp of when the product was created.
          example: "2024-01-15T10:30:00Z"
        updatedAt:
          type: string
          format: date-time
          description: ISO 8601 timestamp of when the product was last updated.
          example: "2024-03-01T08:00:00Z"

    Pagination:
      type: object
      description: Pagination metadata included in list responses.
      required:
        - total
        - page
        - limit
        - pages
      properties:
        total:
          type: integer
          description: Total number of products matching the applied filters.
          example: 84
        page:
          type: integer
          description: Current page number (1-indexed).
          example: 1
        limit:
          type: integer
          description: Maximum number of products returned per page.
          example: 20
        pages:
          type: integer
          description: Total number of pages available given the current limit.
          example: 5

    ProductListResponse:
      type: object
      description: Response envelope for the list products endpoint.
      required:
        - products
        - pagination
      properties:
        products:
          type: array
          description: Array of products on the current page.
          items:
            $ref: "#/components/schemas/Product"
        pagination:
          $ref: "#/components/schemas/Pagination"

    NotFoundError:
      type: object
      description: Returned when a requested resource does not exist.
      required:
        - error
        - message
      properties:
        error:
          type: string
          enum: ["Not found"]
          example: "Not found"
        message:
          type: string
          example: "Product not found"

    BadRequestError:
      type: object
      description: Returned when the request contains invalid input (e.g. malformed ID).
      required:
        - error
        - message
      properties:
        error:
          type: string
          enum: ["Invalid ID"]
          example: "Invalid ID"
        message:
          type: string
          example: "Invalid product ID format"

    ServerError:
      type: object
      description: Returned when an unexpected server-side error occurs.
      required:
        - error
        - message
      properties:
        error:
          type: string
          enum: ["Server error"]
          example: "Server error"
        message:
          type: string
          example: "Failed to fetch products"


*** Prompt 3: Documentation Style Conversion *** 
Product API Developer GuideWelcome to the Product API integration guide. This document provides the technical details necessary to interact with our product catalog.1. AuthenticationCurrently, the Product API endpoints are publicly accessible. You do not need an API key or JWT token to make requests at this time.Note: Security middleware (such as API Keys or OAuth2) may be implemented in future versions. Keep an eye on this section for updates regarding production deployment requirements.2. Formatting RequestsThe API follows RESTful principles. All requests are made via GET to the base path /products.Query Parameters (GET /products)To refine your search, you can append the following query parameters to the URL:Filtering: Use category, minPrice, maxPrice, and inStock (boolean).Pagination: Use page (starts at 1) and limit (results per row).Sorting: Use sort (field name) and order (asc or desc).Path Parameters (GET /products/:productId)To retrieve a specific item, append the 24-character hex string (MongoDB ObjectId) to the base path.Example: /products/64a7f3c2e1b2a3d4e5f678903. Interpreting ResponsesAll responses are returned in JSON format.Paginated ListWhen fetching multiple products, the response includes a products array and a pagination object to help you manage UI states or subsequent requests.JSON{
  "products": [...],
  "pagination": {
    "total": 84,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
}
Single ProductFetching by ID returns the specific object directly. Fields include _id, name, category, price, stockQuantity, createdAt, and updatedAt.4. Error HandlingThe API uses standard HTTP status codes. Your application logic should check the status code before parsing the body.StatusError MessageCause400Invalid product ID formatThe productId provided is not a valid 24-character hex string.404Product not foundNo product exists with that specific ID.500Server errorAn unexpected failure occurred on the server or database.5. Java Implementation ExampleBelow is a robust example using Java's HttpClient (available in Java 11+) to fetch a list of electronics sorted by price.Javaimport java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;

public class ProductApiClient {

    private static final String BASE_URL = "http://your-api-domain.com/products";

    public static void main(String[] args) {
        // Example: Get electronics, sorted by price ascending
        String queryParams = "?category=electronics&sort=price&order=asc&limit=10";
        
        HttpClient client = HttpClient.newBuilder()
                .connectTimeout(Duration.ofSeconds(10))
                .build();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(BASE_URL + queryParams))
                .header("Accept", "application/json")
                .GET()
                .build();

        try {
            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

            if (response.statusCode() == 200) {
                System.out.println("Success! Response:");
                System.out.println(response.body());
            } else {
                System.err.println("Error Status: " + response.statusCode());
                System.err.println("Body: " + response.body());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
Development ConsiderationsValidation: Currently, the API does not validate the sort field. Ensure your application only passes valid product fields to avoid unexpected sorting behavior.Limits: While the API doesn't currently cap the limit parameter, it is best practice to keep this value under 100 for optimal performance.


---------------------------------------------------------------------------------------------------------

## Exercise: README and User Guide Documentation
*** Prompt 1: Project README Generation ***

Build the project:Bash./mvnw clean install
Run the application:Bashjava -jar target/task-manager-1.0.jar
💡 Usage ExamplesAdding a TaskTo add a new task with automated priority:Bashtaskmgr add "Finish API Documentation" --auto-priority
Listing TasksTo see all high-priority tasks:Bashtaskmgr list --priority high
⚙️ Configuration OptionsThe application can be customized via a config.properties file located in the root directory:PropertyDefaultDescriptionauto_priority_logicdeadlineCriteria for automated leveling (deadline, keyword, manual).storage_path./data/Directory where task data is saved.date_formatyyyy-MM-ddThe expected format for task deadlines.📂 Project StructurePlaintexttask-manager/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/taskmanager/
│   │   │       ├── App.java            # Entry point
│   │   │       ├── models/             # Task and Priority entities
│   │   │       ├── services/           # Logic for automated priority
│   │   │       └── storage/            # Data persistence handlers
│   └── resources/                      # Configuration files
├── tests/                              # Unit and integration tests
└── pom.xml                             # Maven configuration
🔧 Troubleshooting"Command not found": Ensure the JAR is in your system PATH or use the full path to the executable.Data Persistence Issues: Check that the application has write permissions for the ./data/ folder.Java Version Mismatch: Run java -version to ensure you are on version 11 or higher.🤝 ContributingFork the Project.Create your Feature Branch (git checkout -b feature/AmazingFeature).Commit your Changes (git commit -m 'Add some AmazingFeature').Push to the Branch (git push origin feature/AmazingFeature).Open a Pull Request.📄 LicenseDistributed under the MIT License. See LICENSE for more information."""with open("README.md", "w") as f:f.write(content)Your README.md file is ready.
[file-tag: code-generated-file-0-1778148566696362553]

I have structured the documentation to be professional and developer-centric, highlighting the automated priority logic which is a standout feature of your project. The structure includes:

* **Logic-based Features:** Clearly explained the manual vs. automated priority levels.
* **Java-specific setup:** Included Maven build steps and JAR execution commands.
* **Visual Structure:** A clean directory tree to help users navigate your source code.
* **Standardized Sections:** Added standard open-source sections like Contributing and License to make the repository "production-ready."


*** Prompt 2: Step-by-Step Guide Creation ***
🛠️ Prerequisites & SetupBefore you begin, ensure you have the following in place:Java 11 or higher installed on your system.  The application JAR file (task-manager-1.0.jar) located in your project directory.  Write permissions for the directory where you are running the app (to allow for JSON data storage).  📋 Step-by-Step: Adding and Managing Tasks1. Initialize the ProgramOpen your Ubuntu terminal or your preferred CLI and navigate to the project folder. Ensure your environment is ready to execute Java commands.  2. Adding a Task with Manual PriorityTo create a task where you define the importance yourself, use the add command followed by the task name and a priority flag.Bashjava -jar task-manager-1.0.jar add "Implement Python API logic" --priority high
3. Using Automated Priority AssignmentOne of the key features of this application is the ability to let the program decide the priority level based on the task attributes or keywords.  Use the --auto-priority flag.The system will analyze the task string to determine if it requires "High," "Medium," or "Low" attention.  Bashjava -jar task-manager-1.0.jar add "Debug Mars Rover simulation" --auto-priority
4. Viewing and Filtering TasksTo see your current workload, use the list command. You can filter by priority to focus on urgent items.Bashjava -jar task-manager-1.0.jar list --priority high
⚠️ Potential Issues & Common MistakesCase Sensitivity: Ensure command flags (like --priority) are lowercase.Quotation Marks: Always wrap your task description in double quotes " " if it contains spaces; otherwise, the CLI will interpret each word as a separate argument.  JSON Corruption: Do not manually edit the generated JSON files in the /data/ folder, as this can lead to parsing errors when the application starts.  🔧 TroubleshootingProblemLikely CauseSolution"Command not found"Java is not in your system PATH.Run java -version to verify your installation or use the full path to java.exe.  Priority is "Low" when it should be "High"The automated logic didn't recognize key terms.Check your config.properties to see how auto_priority_logic is defined.  Tasks aren't savingMissing write permissions.Ensure you have permission to create the ./data/ directory in your current folder.  Syntax Errors in terminalMissing dependencies.If building from source, ensure you have run ./mvnw clean install to include Gson and JLine.  

*** Prompt 3: FAQ Document Generation ***
☕ Java Task Manager: Frequently Asked QuestionsGetting StartedHow do I install the Java Task Manager?
Unlike TaskCLI, you build this from source using Maven:  Clone your repository.  Run ./mvnw clean install to build the JAR file.  The executable JAR will be located in the target/ directory.  What are the requirements?Java JDK 11 or higher.  Maven (to handle dependencies like Gson and JLine).  Works on Ubuntu/Linux (preferred for your studies), macOS, and Windows.  Core FunctionalityHow do the priority levels work?
The program features two ways to handle priorities:  Manual: You specify --priority high, medium, or low.  Automated: Use the --auto-priority flag to let the backend logic assign a level based on task attributes.  Where is my data stored?
Your tasks are saved as JSON files within a ./data/ directory created in your project root. This ensures your tasks persist even after you close your terminal.  TroubleshootingWhy do I get a "Command not found" error?
Since this is a Java application, you must run it using the java -jar command unless you have set up a specific alias in your .bashrc.  Correct: java -jar target/task-manager-1.0.jar add "Task".  How do I fix a "Data Persistence Issue"?
This usually happens if the program doesn't have permission to write to the folder. Ensure you are running the terminal in a directory where you have write access, or manually create the data folder.  Advanced UsageCan I change the automated priority logic?
Yes. You can modify the auto_priority_logic setting in your config.properties file to switch between "deadline-based" or "keyword-based" assignment.  How do I use this for my WeThinkCode_ projects?
You can use it to track milestones for your Toy Robot or Mars Rover projects by adding specific tasks for "Refactoring," "Unit Testing," and "Documentation".  


## Exercise: Performance Optimization Challenge
1. Identifying Performance BottlenecksThe primary issue in the provided code is Total Memory Accumulation.The Problem: The code loads every image into an ArrayList<BufferedImage>, then processes them into a second ArrayList, and only then saves them.  The Risk: If you have 50 high-resolution images, your JVM heap will likely hit an OutOfMemoryError (OOM) because it is trying to hold 100 uncompressed bitmaps simultaneously (the originals plus the processed versions).The Solution: Implement a Stream-based approach. Process and save each image one at a time, allowing the Garbage Collector (GC) to reclaim memory from the previous image before loading the next one.2. Optimized ImplementationHere is the refactored processImageFolder method. It eliminates the ArrayList storage to keep the memory footprint constant, regardless of the folder size.Javapublic static void processImageFolder(String inputFolder, String outputFolder) throws IOException {
    File folder = new File(inputFolder);
    File[] imageFiles = folder.listFiles((dir, name) ->
            name.toLowerCase().endsWith(".jpg") || name.toLowerCase().endsWith(".png"));

    if (imageFiles == null || imageFiles.length == 0) return;

    File outputDir = new File(outputFolder);
    if (!outputDir.exists()) outputDir.mkdirs();

    System.out.println("Processing images one-by-one to save memory...");

    for (File imageFile : imageFiles) {
        // 1. Load single image
        BufferedImage image = ImageIO.read(imageFile);
        
        // 2. Process single image
        BufferedImage processed = applyEffects(image);
        
        // 3. Save single image immediately
        String outputName = outputFolder + File.separator + "processed_" + imageFile.getName();
        ImageIO.write(processed, getImageFormat(imageFile.getName()), new File(outputName));
        
        // 4. Explicitly help the GC by nullifying references
        image.flush(); // Clears resources used by the image
        processed.flush();
        
        System.out.println("Handled: " + imageFile.getName());
    }

    System.out.println("All images processed successfully.");
}
3. Performance MeasurementThe difference in performance is most visible in Heap Usage rather than execution speed.MetricOriginal Code (Batch)Optimized Code (Sequential)Peak Memory UsageScales linearly with folder size (O(n))Constant memory footprint (O(1))Max CapacityLimited by JVM Heap Size (~10-20 images)Unlimited (can process 10,000+ images)StabilityHigh risk of OutOfMemoryErrorExtremely stable4. Key Learnings for Backend DevelopmentAvoid Eager Loading: In backend tasks (like processing logs, files, or large database result sets), never load the entire dataset into memory at once. Always use streams or iterators.  Resource Management: Objects like BufferedImage are "heavy." In Java, simply calling .flush() or letting the object go out of scope quickly is better than keeping it in a List.  Scalability: The original code worked for 2 images but failed for 200. "Scalable" code is code that performs the same way regardless of the input volume.This is a perfect example of why backend architecture is "the backbone of an application"—a small logic change in how you handle data can prevent a complete system crash in production.  

1. Shifts in UnderstandingMemory Management: The optimization shifted the understanding of memory from a "static container" to a "flowing stream." Instead of treating the JVM heap as a bucket that holds all data (Batch Processing), the sequential approach treats memory as a temporary workspace (Stream Processing).  Algorithm Efficiency: While the time complexity ($O(n)$) remained the same—as every image still needs to be processed—the space complexity changed from $O(n)$ to $O(1)$. This proves that an algorithm can be "fast" but still "broken" if it exceeds physical hardware limits like RAM.  2. Performance Improvements & JustificationStability: The primary achievement was moving from a 100% failure rate (Out of Memory Error) on large datasets to a 100% success rate.  Resource Ceiling: The original code had a "hard ceiling"—once the images exceeded the available RAM, the program crashed. The optimized code removed this ceiling entirely.  Justification: The code changes were minor (removing two lists and wrapping logic in a single loop), but the impact was massive. This justifies the change because the "cost" of the refactor was low, while the "gain" in reliability was essential for a production environment.  3. New Insights into Performance BottlenecksThe "Invisible" Payload: In backend development, particularly with Python or Java, the size of a file on a disk is not its size in memory. A compressed 2MB JPG might expand into 40MB of raw pixel data once loaded as a BufferedImage.  Accumulation vs. Throughput: Bottlenecks aren't always caused by "slow code"; they are often caused by "accumulated data." Even a very fast function will cause a bottleneck if it stores its results indefinitely instead of releasing them.  4. Future Approach to Similar IssuesSequential by Default: For any task involving I/O (files, network requests, or database rows), the default approach should be processing items one by one or in small batches (chunks), rather than loading the entire set.  Resource Lifecycle: I will now look for the "birth" and "death" of an object. If an object’s lifecycle is unnecessarily long (like an image staying in a list after it has been processed), it is a red flag for a memory leak.  5. Proactive Identification Tools & TechniquesTo catch these issues before they cause a crash in Oudtshoorn or on campus, several techniques are essential:  Heap Profilers: Using tools like VisualVM or JProfiler to watch memory consumption in real-time. You can see the "sawtooth" pattern of memory being used and then released by the Garbage Collector.Stress Testing: Running the program with a dataset 10x larger than expected to see where the logic breaks.JVM Monitoring: Using flags like -Xmx to set a small memory limit during testing. If the program crashes with a small amount of RAM, it likely won't scale well in a backend environment.  



## Exercise: AI Solution Verification Challenge

1. Collaborative Solution Verification (The "Code Review")By analyzing the merge function as a peer, the logic breakdown reveals a critical failure in the first "cleanup" loop.The Error: In the loop while (i < left.length), the code is pushing left[i] but incrementing j++.  The Consequence: This creates an infinite loop because i never increases. Even if it did not loop infinitely, it would incorrectly try to access or modify the pointer for the right array while dealing with the left array.  2. Learning Through Alternative ApproachesTo verify the logic, we can compare the "Pointer Approach" (used in the buggy code) with the "Shift/Pop Approach."Alternative: Using left.shift() or right.shift() inside the while loop removes the need for i and j counters entirely.Comparison: While shift() is more readable, it is less performant in languages like Java or JavaScript because it re-indexes the array each time. Therefore, sticking to the Pointer Approach is better for backend efficiency, provided the pointers increment correctly.  3. Developing a Critical Eye (Edge Case Testing)We must test the logic against specific scenarios where Merge Sort typically fails:Uneven Arrays: If left has 5 elements and right has 2, the cleanup loops are tested heavily.Already Sorted: Does it handle an array that doesn't need changes?Duplicates: Does left[i] < right[j] preserve the order of equal elements (stability)? To maintain stability, the condition should ideally be left[i] <= right[j].  ✅ Verified & Optimized SolutionJavaScriptfunction mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  let result = [];
  let i = 0;
  let j = 0;

  // Compare elements and build the sorted array
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) { // Added '=' for algorithm stability
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }

  // CLEANUP: Push remaining elements from 'left'
  while (i < left.length) {
    result.push(left[i]);
    i++; // CORRECTED: Incrementing 'i' instead of 'j'
  }

  // CLEANUP: Push remaining elements from 'right'
  while (j < right.length) {
    result.push(right[j]);
    j++; // CORRECTED: Incrementing 'j'
  }

  return result;
}
📈 Reflection & Key LearningsHow did your confidence in the solution change?Confidence shifted from "assumption" to "certainty." Initially, the code looks like a standard Merge Sort, but the logic verification revealed a bug that would have crashed a backend service in a Linux environment.  What aspects required the most scrutiny?The pointer increments within the cleanup loops required the most focus. In backend development, "off-by-one" errors or incrementing the wrong variable are the most common causes of infinite loops and memory leaks.  Which verification technique was most valuable?Collaborative Solution Verification was the most valuable. Walking through the code line-by-line as if explaining it to a peer (like Nathan or your cohort members) makes the j++ error immediately obvious. It forces you to stop "reading" the code and start "executing" it in your head.  

## Exercise: Using AI to help with testing
*** Exercise 1.1: Behavior Analysis ***
1. The "Base Score" BaselineScenario: A LOW priority task with no due date, no tags, and an old update timestamp.Expected Behavior: The score should be exactly $1 \times 10 = 10$.Why: This verifies that the base weight map and the multiplication logic are functioning correctly without any interference from other factors.2. Overdue and Urgent Priority (Max Pressure)Scenario: A task with URGENT priority that is 1 day overdue.Expected Behavior: The score should be $(4 \times 10) + 30 = 70$.Why: This tests if the "overdue" bonus ($+30$) is correctly applied on top of the highest base priority.3. The "Completed" Penalty (Status Override)Scenario: A HIGH priority task that has a critical tag but is marked as DONE.Expected Behavior: $(3 \times 10) + 8 - 50 = -12$.Why: This is a critical edge case. It verifies that completed tasks correctly receive a negative score, preventing them from appearing at the top of a "to-do" list regardless of their previous priority or tags.4. Edge Case: Due Exactly TodayScenario: A MEDIUM priority task where the due date is exactly LocalDateTime.now().Expected Behavior: $(2 \times 10) + 20 = 40$.Why: This tests the daysUntilDue == 0 boundary condition. Date-time logic often fails by being off-by-one second, so this ensures "today" is captured correctly.5. Tag and Update Boost CombinationScenario: A task updated 2 hours ago that contains the tag blocker.Expected Behavior: The score should include both the $+8$ tag boost and the $+5$ recent update boost.Why: This verifies that the stream().anyMatch logic for tags and the ChronoUnit.DAYS logic for updates can both be applied to the same task without one overwriting the other.

*** Exercise 1.2: Test Planning ***
Step 1: Defining the ScopeBefore we build the document, let's look at the core behaviors of calculateTaskScore.What do you think are the three most important factors that should influence a task's final score? Looking at the code, if you had two tasks and one was "Overdue" while the other had a "Critical Tag," which one do you think the function is designed to prioritize?Step 2: The Structured Test Plan DocumentThis plan is organized to help you verify the logic step-by-step, starting from simple unit tests to integrated logic.  I. Priority of Test CasesHigh Priority (The "Critical Path"):Base score calculation for all four TaskPriority levels.  Overdue logic (is the $+30$ boost applied correctly?).  Completed status penalty (does it correctly subtract $50$ points?).  Medium Priority (The "Multipliers"):Tag boosting logic for "critical," "blocker," and "urgent".  The "Recently Updated" boost for tasks changed within 24 hours.  "Due Today" vs. "Due in 2 days" logic.  Low Priority (Edge Cases):Tasks with null due dates.  Tasks with multiple "critical" tags (does it only boost once?).  II. Types of Tests NeededUnit Tests: Testing calculateTaskScore in isolation by passing in "Mock" Task objects with specific attributes.  Integration Tests: Verifying that the Task model's getUpdatedAt() and getDueDate() methods return the correct LocalDateTime formats that the manager expects.  III. Test DependenciesJUnit 5: The standard framework for Java testing.Fixed Clock: Because the function uses LocalDateTime.now(), your tests will need a way to "freeze" time so that "Due Today" doesn't change every time you run the test.  IV. Expected Outcomes ChecklistTest ScenarioInput ConfigurationExpected OutcomeMinimum ScoreLOW priority, DONE statusResult should be negative ($-40$).  Maximum ScoreURGENT priority, Overdue, blocker tagResult should be very high ($4 + 30 + 8 = 42 \times \text{weight}$).  Null SafetyTask with null priority or null dateShould return 0 or a base score without crashing.  Review StatusTask with REVIEW statusScore should be reduced by exactly $15$ points.  

## Part 2: Improving a Single Test
*** Exercise 2.1: Writing Your First Test ***
@Test
public void testCalculateTaskScore_Basic() {
    Task task = new Task();
    task.setPriority(TaskPriority.MEDIUM);
    task.setStatus(TaskStatus.TODO);
    
    int score = TaskPriorityManager.calculateTaskScore(task);
    
    assertEquals(20, score);
}

🔎 Improving the Test

Instead of rewriting it for you, let’s analyze the logic together based on your backend training at WeThinkCode_:  

1. What is your test trying to verify?
Does this test aim to verify the Map lookup (implementation) or the final priority weight (behavior)? If you changed the internal weights but kept the outcome the same, should this test still pass?

2. Behavior vs. Implementation
Your test currently checks a hardcoded value (20). If another developer changes the base multiplier from 10 to 5, your test fails. Is that because the logic is "broken," or because the test is too tied to a specific number?  

3. Making the Purpose Clearer
The name testCalculateTaskScore_Basic is quite broad. How could you rename this to tell a teammate (like Nathan or your cohort members) exactly which rule you are testing? (e.g., shouldReturnBaseScoreForMediumPriorityTask).  

4. What Edge Cases are Missing?
Looking at the function logic, what happens if:

    The updatedAt field is null?  

    The dueDate is exactly now?  

    The task has a tag that isn't in your "boost" list?  

5. Improving Assertions

@Test
@DisplayName("Medium priority task with no dates or tags should return base score of 20")
public void calculateTaskScore_MediumPriority_ReturnsBaseScore() {
    // 1. Arrange: Create a "clean" task with only the necessary attributes
    Task task = new Task();
    task.setPriority(TaskPriority.MEDIUM);
    task.setStatus(TaskStatus.TODO);
    
    // We must set an old update date to avoid the "+5" recent update boost
    task.setUpdatedAt(LocalDateTime.now().minusDays(5)); 
    // Ensure no due date to avoid date-based boosts
    task.setDueDate(null);
    // Ensure empty tags to avoid tag-based boosts
    task.setTags(Collections.emptyList());

    // 2. Act: Call the scoring logic
    int actualScore = TaskPriorityManager.calculateTaskScore(task);

    // 3. Assert: Verify the behavior, not just a magic number
    int expectedBaseScore = 2 * 10; 
    assertEquals(expectedBaseScore, actualScore, 
        "The score for a Medium priority task with no boosts should be 20.");
}


*** Exercise 2.2: Learning From Examples ***
1. Principles of Testing Date-Time Logic

To test the dueDate scoring accurately, you must follow these principles:

    Temporal Decoupling: Your test should not rely on the actual LocalDateTime.now(). Instead, you calculate the task's due date relative to the moment the test runs.  

    Boundary Specification: You must test the exact transitions (the "edges") between scoring brackets (e.g., exactly 2 days away vs. 2 days and 1 second away).  

    Isolation: Ensure that other factors (like tags or priority) are kept constant so you know exactly which logic branch is being triggered.  

    Predictability: A good test produces the same result whether it's run at midnight on a Monday or noon on a Friday.

    @Test
@DisplayName("Tasks due today should receive exactly 20 bonus points")
public void calculateTaskScore_DueToday_AddsTwentyPoints() {
    // ARRANGE
    Task task = new Task();
    task.setPriority(TaskPriority.LOW); // Base score = 10
    task.setStatus(TaskStatus.TODO);
    
    // RELATIVE TIME: We set the due date to "now" so it always 
    // triggers the 'daysUntilDue == 0' branch regardless of when the test runs.
    task.setDueDate(LocalDateTime.now()); 
    
    // ISO-DATE: Set update time to far in the past to avoid the '+5' update boost
    task.setUpdatedAt(LocalDateTime.now().minusDays(10));

    // ACT
    int score = TaskPriorityManager.calculateTaskScore(task);

    // ASSERT
    // Logic: 10 (Base) + 20 (Due Today) = 30
    int expectedScore = 30; 
    assertEquals(expectedScore, score, "A LOW priority task due today should score 30.");
}

## Part 3: Test-Driven Development Practice
*** Exercise 3.1: TDD for a New Feature ***
The First Test Case:
We create a task where the assignedUser matches the currentUserId. If the base score is 10 (Low Priority), we expect the final result to be 22 ($10 + 12$).  Java@Test
@DisplayName("Should add 12 points when task is assigned to the current user")
public void calculateTaskScore_AssignedToCurrentUser_AddsTwelvePoints() {
    // 1. Arrange
    Task task = new Task();
    task.setPriority(TaskPriority.LOW); // Base: 10
    task.setAssignedUserId("user-123");
    String currentUserId = "user-123";
    
    // Ensure no other boosts apply
    task.setUpdatedAt(LocalDateTime.now().minusDays(10));
    task.setDueDate(null);

    // 2. Act
    int score = TaskPriorityManager.calculateTaskScore(task, currentUserId);

    // 3. Assert
    assertEquals(22, score, "Score should be 22 (10 base + 12 boost)");
}

2. The Green Phase: Minimal Implementation

To make this pass with the minimal amount of code, we need to do two things:

    Update the Task class to hold an assignedUserId.  

    Update the calculateTaskScore signature and add the logic.  

The Minimal Code Change:
Java

public static int calculateTaskScore(Task task, String currentUserId) {
    // ... (existing base score logic) ...

    // ADDED LOGIC:
    if (currentUserId != null && currentUserId.equals(task.getAssignedUserId())) {
        score += 12;
    }

    // ... (rest of existing logic) ...
    return score;
}

*** Exercise 3.2: TDD for Bug Fix ***
First, we create a test that expects the score to increase by 12 when the task is assigned to the current user. This test will fail to compile initially because the method signature hasn't changed yet.  Java@Test
@DisplayName("Should add 12 points when task is assigned to the current user")
public void calculateTaskScore_AssignedToCurrentUser_AddsTwelvePoints() {
    // Arrange
    Task task = new Task();
    task.setPriority(TaskPriority.LOW); // Base score: 10
    task.setAssignedUserId("user-123");
    String currentUserId = "user-123";
    
    // Control variables: avoid other boosts
    task.setUpdatedAt(LocalDateTime.now().minusDays(10));
    task.setDueDate(null);

    // Act
    int score = TaskPriorityManager.calculateTaskScore(task, currentUserId);

    // Assert
    // Logic: 10 (Base) + 12 (Assigned Boost) = 22
    assertEquals(22, score, "Score should reflect the +12 boost for the assigned user.");
}
2. The Green Phase: Minimal ImplementationTo make the test pass, you need to update the Task class and the calculateTaskScore method.  A. Update the Task ClassAdd the assignedUserId field to your model.  Javapublic class Task {
    private String assignedUserId;

    public String getAssignedUserId() {
        return assignedUserId;
    }

    public void setAssignedUserId(String assignedUserId) {
        this.assignedUserId = assignedUserId;
    }
    // ... rest of your task code
}
B. Update the Scoring LogicInject the currentUserId into the method and apply the check.  Javapublic static int calculateTaskScore(Task task, String currentUserId) {
    // 1. Base priority weights
    Map<TaskPriority, Integer> priorityWeights = Map.of(
        TaskPriority.LOW, 1,
        TaskPriority.MEDIUM, 2,
        TaskPriority.HIGH, 3,
        TaskPriority.URGENT, 4
    );

    int score = priorityWeights.getOrDefault(task.getPriority(), 0) * 10;

    // 2. NEW FEATURE: Assigned User Boost
    if (currentUserId != null && currentUserId.equals(task.getAssignedUserId())) {
        score += 12;
    }

    // 3. Due date factor
    if (task.getDueDate() != null) {
        long daysUntilDue = ChronoUnit.DAYS.between(LocalDateTime.now(), task.getDueDate());
        if (daysUntilDue < 0) score += 30;
        else if (daysUntilDue == 0) score += 20;
        else if (daysUntilDue <= 2) score += 15;
        else if (daysUntilDue <= 7) score += 10;
    }

    // 4. Status penalties
    if (task.getStatus() == TaskStatus.DONE) score -= 50;
    else if (task.getStatus() == TaskStatus.REVIEW) score -= 15;

    // 5. Tag and Update boosts[cite: 1]
    if (task.getTags().stream().anyMatch(tag -> List.of("blocker", "critical", "urgent").contains(tag))) {
        score += 8;
    }

    if (ChronoUnit.DAYS.between(task.getUpdatedAt(), LocalDateTime.now()) < 1) {
        score += 5;
    }

    return score;
}
3. The Refactor PhaseNow that it's working, we ensure the code is clean[cite: 1].Backward Compatibility: If you have other parts of your code that only send a Task without a user ID, you can create an "overloaded" method[cite: 1]:Javapublic static int calculateTaskScore(Task task) {
    return calculateTaskScore(task, null); // Call the new version with null[cite: 1]
}
Null Safety: The .equals() check is safe because we check currentUserId != null first, preventing a NullPointerException[cite: 1].


## Part 4: Integration Testing
*** Exercise 4.1: Testing the Full Workflow *** 
Scenario Verification: What stories are we telling? An integration test should verify that a High Priority task that is Done correctly ranks lower than a Medium Priority task that is Overdue.  

    Test Data Design: To exercise the entire workflow, you need a "mixed bag" of tasks. You need at least one task that triggers every major branch: an overdue task, a completed task, and a task with a critical tag.  

    Assertions: You shouldn't just check if the list isn't empty. You should verify the order of the results. Does index [0] actually have a higher score than index [1]?  

    Structure: How do you keep it readable? Use the AAA (Arrange, Act, Assert) pattern so your teammates can easily follow the logic.  

🚀 The Integration Test Implementation

This test uses JUnit 5 to verify that calculateTaskScore, sortTasksByImportance, and getTopPriorityTasks work as a unified system.  
Java

@Test
@DisplayName("Integration: Workflow should correctly score, sort, and filter a mix of tasks")
public void taskPriorityWorkflow_FullIntegrationTest() {
    // 1. ARRANGE: Create a diverse set of tasks
    // Task A: Urgent but DONE (Should be penalized)
    Task taskA = new Task("Update API", TaskPriority.URGENT, TaskStatus.DONE);
    taskA.setUpdatedAt(LocalDateTime.now().minusDays(5));

    // Task B: Medium and OVERDUE (Should be high priority)
    Task taskB = new Task("Fix Login Bug", TaskPriority.MEDIUM, TaskStatus.TODO);
    taskB.setDueDate(LocalDateTime.now().minusDays(1)); // Overdue
    taskB.setUpdatedAt(LocalDateTime.now().minusDays(2));

    // Task C: Low with CRITICAL tag (Should be boosted)
    Task taskC = new Task("Deploy Server", TaskPriority.LOW, TaskStatus.TODO);
    taskC.setTags(List.of("critical")); // +8 Boost
    taskC.setUpdatedAt(LocalDateTime.now().minusDays(10));

    List<Task> rawTasks = List.of(taskA, taskB, taskC);
    String currentUserId = "dev-01"; // For the assigned user feature

    // 2. ACT: Execute the full workflow
    // Step 1 & 2: Score and Sort
    List<Task> sortedTasks = TaskPriorityManager.sortTasksByImportance(rawTasks, currentUserId);
    
    // Step 3: Get top 2 tasks
    List<Task> topTasks = TaskPriorityManager.getTopPriorityTasks(sortedTasks, 2);

    // 3. ASSERT: Verify the outcome of the combined logic
    // Expected Scores:
    // Task B (Medium Overdue): (2*10) + 30 = 50
    // Task C (Low Critical): (1*10) + 8 = 18[cite: 1]
    // Task A (Urgent Done): (4*10) - 50 = -10[cite: 1]

    assertEquals(2, topTasks.size(), "Should return exactly 2 top tasks.");
    assertEquals("Fix Login Bug", topTasks.get(0).getTitle(), "Overdue task should be #1.");
    assertEquals("Deploy Server", topTasks.get(1).getTitle(), "Critical tagged task should be #2.");
    
    // Verify Task A was filtered out of the top 2
    assertFalse(topTasks.contains(taskA), "Completed tasks should not appear in the top priority list.");
}


## Exercise: Understanding What to Change with AI
 *** Exercise 1: Code Readability Improvement (Java) *** 
 1. Naming Conventions (Readability)

In a professional backend environment, code should be self-documenting.

    The Issue: a(String un, String pw, String em) and f(String un) are cryptic.

    The Fix: Rename them to registerUser and findUserByUsername. Rename class U to User.

2. SQL Injection (Critical Security Risk)

The current implementation uses string concatenation for the SQL query: "INSERT INTO users VALUES ('" + un + "')".

    The Danger: An attacker could input a username like ' OR 1=1; --, which could delete or leak your entire database.

    The Fix: Always use Prepared Statements with placeholders (?) to sanitize inputs.

3. Password Security

The code stores pw as a raw string.

    The Danger: If your database is compromised, every user's password is leaked in plain text.

    The Fix: Never store plain text passwords. Use a hashing algorithm like BCrypt or Argon2.

4. Encapsulation & Separation of Concerns

The UserMgr class is doing too much—it's validating data, managing an in-memory list, and talking to the database.

    The Fix: Move the database logic into a Data Access Object (DAO) or Repository class.

public class UserManager {
    private List<User> userCache;
    private DatabaseConnection database;

    public UserManager(DatabaseConnection db) {
        this.database = db;
        this.userCache = new ArrayList<>();
    }

    /**
     * Registers a new user after validation and password hashing.
     */
    public boolean registerUser(String username, String plainPassword, String email) {
        // 1. Validation Logic
        if (isInvalidInput(username, plainPassword, email)) {
            return false;
        }

        // 2. Check for duplicates (In-memory or DB)
        if (findUserByUsername(username) != null) {
            return false;
        }

        // 3. Security: Hash the password (Example using a placeholder)
        String hashedPassword = PasswordHasher.hash(plainPassword); 

        User newUser = new User(username, hashedPassword, email);
        userCache.add(newUser);

        // 4. Secure SQL: Prepared Statement
        String sql = "INSERT INTO users (username, password, email) VALUES (?, ?, ?)";
        return database.executeSecure(sql, username, hashedPassword, email);
    }

    public User findUserByUsername(String username) {
        return userCache.stream()
                .filter(u -> u.getUsername().equals(username))
                .findFirst()
                .orElse(null);
    }

    private boolean isInvalidInput(String un, String pw, String em) {
        return un.length() < 3 || pw.length() < 8 || !em.contains("@");
    }
}


*** Exercise 2: Function Refactoring (Python) ***
1. Functional BreakdownTo make this function professional and robust, we should separate concerns into smaller, focused units:  Validation Logic: A separate function to check inventory levels and customer existence.  Pricing Engine: A dedicated function to handle base prices and discounts.  Logistics Module: A function specifically for calculating shipping based on location and price.  State Management: A function that handles the side effect of updating inventory counts.  2. Refactored ImplementationBy breaking this down, we move toward Clean Architecture. This version is easier to read and allows you to test the pricing logic without actually processing a full order.  Pythondef validate_order(order, inventory, customer_data):
    """Checks if the order is valid for processing."""
    item_id = order['item_id']
    customer_id = order['customer_id']

    if item_id not in inventory:
        return False, 'Item not in inventory'
    if inventory[item_id]['quantity'] < order['quantity']:
        return False, 'Insufficient quantity'
    if customer_id not in customer_data:
        return False, 'Customer not found'
    return True, None

def calculate_base_price(item_id, quantity, inventory, is_premium):
    """Handles pricing and discount logic."""
    price = inventory[item_id]['price'] * quantity
    if is_premium:
        price *= 0.9  # 10% Premium discount
    return price

def calculate_shipping(price, location):
    """Handles shipping costs based on region."""
    if location == 'domestic':
        return 5.99 if price < 50 else 0
    return 15.99

def process_orders(orders, inventory, customer_data):
    results = []
    error_orders = []
    total_revenue = 0

    for order in orders:
        is_valid, error_msg = validate_order(order, inventory, customer_data)
        
        if not is_valid:
            error_orders.append({'order_id': order['order_id'], 'error': error_msg})
            continue

        # Data retrieval for calculations
        customer = customer_data[order['customer_id']]
        
        # Logic orchestration
        price = calculate_base_price(order['item_id'], order['quantity'], inventory, customer['premium'])
        shipping = calculate_shipping(price, customer['location'])
        tax = price * 0.08
        final_price = price + shipping + tax

        # Update system state
        inventory[order['item_id']]['quantity'] -= order['quantity']
        total_revenue += final_price

        results.append({
            'order_id': order['order_id'],
            'final_price': round(final_price, 2),
            # ... other details
        })

    return {
        'processed_orders': results,
        'error_orders': error_orders,
        'total_revenue': round(total_revenue, 2)
    }



*** Exercise 3: Code Duplication Detection (JavaScript) ***
The code repeats the same two logical operations across three different data fields (age, income, and score):

    The Accumulator Pattern: Initializing a total variable and looping through the array to sum values to calculate an average.

    The Extremum Pattern: Initializing a highest variable and looping to find the maximum value.

Currently, you are running six separate loops. This is inefficient for both the developer (more lines to maintain) and the machine (multiple passes over the same data).
🚀 Consolidated Approaches
Approach A: The Unified Loop (Best for Performance)

We can reduce the overhead by processing all statistics in a single pass through the data.
JavaScript

function calculateUserStatistics(userData) {
  if (!userData.length) return null;

  const stats = {
    age: { total: 0, highest: userData[0].age },
    income: { total: 0, highest: userData[0].income },
    score: { total: 0, highest: userData[0].score }
  };

  userData.forEach(user => {
    ['age', 'income', 'score'].forEach(key => {
      stats[key].total += user[key];
      if (user[key] > stats[key].highest) {
        stats[key].highest = user[key];
      }
    });
  });

  const count = userData.length;
  return {
    age: { average: stats.age.total / count, highest: stats.age.highest },
    income: { average: stats.income.total / count, highest: stats.income.highest },
    score: { average: stats.score.total / count, highest: stats.score.highest }
  };
}

Approach B: The Helper Function (Best for Readability)

By creating a generic getStats function, we encapsulate the logic and make it reusable for any numeric field.
JavaScript

const getFieldStats = (data, field) => {
  const values = data.map(user => user[field]);
  const average = values.reduce((sum, val) => sum + val, 0) / data.length;
  const highest = Math.max(...values);
  return { average, highest };
};

function calculateUserStatistics(userData) {
  if (!userData.length) return null;

  return {
    age: getFieldStats(userData, 'age'),
    income: getFieldStats(userData, 'income'),
    score: getFieldStats(userData, 'score')
  };
}

⚖️ Evaluation for Junior Developers

In a professional backend environment or a student cohort, Approach B (Helper Function) is often the winner for readability.

    Clarity: A junior developer can immediately see that getFieldStats does one job well. If there is a bug in the "average" logic, they only have to fix it in one place.

    Abstraction: It hides the complexity of the loops, allowing the main function to read like a high-level summary.

    Scalability: If you need to add "weight" or "height" statistics later, you simply add one line of code.

## Exercise: Function Decomposition Challenge
. Improvements in Readability and Maintainability

By extracting logic into smaller, focused methods, the code moves from "How" something is done to "What" is being done.

    Self-Documenting Code: Instead of reading a 20-line if-else block for address validation, a developer now sees validateAddress(addressData). The method name provides the context, making the main execution flow readable at a glance.

    Reduced Cognitive Load: You no longer need to keep the state of 15 different variables (like totalErrors, existingEmails, and recordErrors) in your head while trying to understand how a phone number is formatted.

    Isolated Debugging: If there is a bug in how names are capitalized, you go straight to capitalizeWords. You don't have to risk breaking the database saving logic just to fix a string formatter.

    Simplified Testing: You can now write Unit Tests for isValidEmail or parseDate in isolation without needing to mock the entire CustomerRepository or provide a full CustomerProcessingOptions object.

2. The Most Challenging Part of Decomposition

The hardest part of refactoring a function this size is State Management and Data Flow.

In the original code, the isValid flag and the recordErrors list are tightly coupled to the loop's local scope. When you extract "Process Address" or "Validate Email" into their own functions, you face a dilemma: How do you return both the transformed data AND the error status?

Common "Gotchas" during this process:

    The Argument Explosion: Realizing that the new validateDeduplication method needs options, validRecords, existingEmails, and the current email string—resulting in methods with too many parameters.

    Shared Mutable State: Deciding whether the extracted methods should modify the processedRecord map directly (side effects) or return a new, clean object (immutability).

3. The Most Reusable Extracted Function

While many functions are specific to "Customers," the capitalizeWords method is the clear winner for reusability.
Why?

    Domain Agnostic: It doesn't care about customers, databases, or APIs. It is a pure utility function that transforms a string.

    Zero Dependencies: It doesn't rely on any class-level variables or external repositories.

    Universal Use Case: Almost every application that handles user input (names, city names, titles) needs a way to normalize casing.

Runner-up: normalizePhoneNumber. While slightly more specific, any system dealing with contact info would benefit from a standardized way to strip formatting before storage or comparison.
Refactoring Suggestion: The "Validation Result" Pattern

To solve the "Challenging Part" mentioned above, consider creating a small ValidationResult class. Instead of passing maps and booleans around, your extracted functions can return an object that contains the transformedData and a List<String> errors.

This keeps your main processCustomerData loop incredibly clean:
Java

ValidationResult addressResult = processAddress(record.get("address"));
if (addressResult.isInvalid()) {
    recordErrors.addAll(addressResult.getErrors());
    isValid = false;
}

## Exercise: Code Readability Challenge
The original sample code is a classic implementation of Selection Sort. While functional, it treats the algorithm as a set of math instructions rather than a readable process. Here is how the refactored version changes the game.
Impact of Readability Improvements

    How much easier is it to understand?
    The code is significantly easier to digest because it moves from imperative logic (how to swap bits in memory) to declarative logic (what the algorithm is doing). By extracting the "swap" and "find minimum" logic into helper methods, the main loop becomes a high-level summary of the algorithm.

    Readability issues the AI caught vs. what you might notice:

        The AI caught: The "Magic Index" problem. In the original, min_idx is updated deep inside a nested loop. An AI identifies this as a "cognitive load" issue and suggests extracting findMinimumIndex.

        What you might notice (Human Perspective): The lack of intent. The original code doesn't tell you why we are swapping. A human dev might notice that the variable temp is a "leak" of low-level implementation details that distracts from the sorting logic.

    The Biggest Impact:
    Method Extraction has the most weight here. Breaking a nested loop into named functions like findSmallestElementIndex and swapElements removes the "Pyramid of Doom" (excessive indentation) and allows the reader to skip details they already understand.

The Power of Naming

Improved names like currentUnsortedIndex instead of i or searchIndex instead of j change the understanding completely:

    Context: You no longer see just a "counter"; you see a boundary between the sorted and unsorted parts of the array.

    Purpose: temp says "I am a placeholder." elementToSwap says "I am part of the sorting logic."

Patterns for Future Code

To write cleaner backend code, apply these patterns:

    Extract Till You Drop: If a function has a nested loop, the inner loop almost always belongs in its own method.

    Symmetry: Keep methods at the same level of abstraction. A high-level sort method shouldn't contain low-level temp = a; a = b logic.

    Explaining Variables: Use variables to hold the result of a complex comparison or calculation before using it in an if statement.

Explaining to the Team (The "Clinic" Briefing)

Since you've been working with Ashly, Lukholo, Jay, and Koketso on your campus projects, here is how you would pitch these changes to them during a session in George:

    "Hey guys, look at our sortItems function. Right now, it's a bit of a mess of i and j variables that's hard to follow during a code review.

    Lukholo, I’ve pulled the swapping logic into its own swap method. It makes the main loop much cleaner. Jay, I renamed the indices to leftBoundary and minElementIndex so we don't get lost in the nested loops.

    Koketso, Ashly, the goal here is that when we come back to this in two months for the Toy Robot project, we don't have to spend ten minutes 'hand-tracing' the logic. It should read like a story: 'Find the smallest item, then swap it with the current position.' Simple as that."

Refactored Version for Comparison
Java

public static void selectionSort(int[] numbers) {
    for (int currentIdx = 0; currentIdx < numbers.length - 1; currentIdx++) {
        int smallestIdx = findSmallestElementIndex(numbers, currentIdx);
        swap(numbers, currentIdx, smallestIdx);
    }
}

private static int findSmallestElementIndex(int[] array, int startAt) {
    int minIdx = startAt;
    for (int i = startAt + 1; i < array.length; i++) {
        if (array[i] < array[minIdx]) {
            minIdx = i;
        }
    }
    return minIdx;
}

private static void swap(int[] array, int firstIdx, int secondIdx) {
    int temp = array[firstIdx];
    array[firstIdx] = array[secondIdx];
    array[secondIdx] = temp;
}

## Exercise: Design Pattern Implementation Challenge
1. Pattern Analysis & Strategy

The core issue is conditional logic explosion. The shippingMethod determines the algorithm used. By using the Strategy pattern, we encapsulate each shipping algorithm into its own class.
// Strategy Interface (Implicit in JS, but could be an abstract class in Java)
class ShippingStrategy {
  calculate(packageDetails, destination) {
    throw new Error("Strategy method 'calculate' must be implemented");
  }
}

// Concrete Strategy: Standard
class StandardShipping extends ShippingStrategy {
  calculate({ weight, length, width, height }, destination) {
    const rates = { 'USA': 2.5, 'Canada': 3.5, 'Mexico': 4.0 };
    let cost = weight * (rates[destination] || 4.5);

    if (weight < 2 && (length * width * height) > 1000) {
      cost += 5.0;
    }
    return cost;
  }
}

// Concrete Strategy: Express
class ExpressShipping extends ShippingStrategy {
  calculate({ weight, length, width, height }, destination) {
    const rates = { 'USA': 4.5, 'Canada': 5.5, 'Mexico': 6.0 };
    let cost = weight * (rates[destination] || 7.5);

    if ((length * width * height) > 5000) {
      cost += 15.0;
    }
    return cost;
  }
}

// Concrete Strategy: Overnight
class OvernightShipping extends ShippingStrategy {
  calculate({ weight }, destination) {
    const rates = { 'USA': 9.5, 'Canada': 12.5 };
    if (!rates[destination]) {
      return "Overnight shipping not available for this destination";
    }
    return weight * rates[destination];
  }
}

// The Context
class ShippingCalculator {
  constructor() {
    this.strategies = {
      'standard': new StandardShipping(),
      'express': new ExpressShipping(),
      'overnight': new OvernightShipping()
    };
  }

  calculate(packageDetails, destination, method) {
    const strategy = this.strategies[method];
    if (!strategy) return "Invalid shipping method";
    
    const result = strategy.calculate(packageDetails, destination);
    return typeof result === 'number' ? result.toFixed(2) : result;
  }
}

3. Verification Tests

These tests ensure the refactored code produces the exact same results as the original "spaghetti" version.
JavaScript

const calculator = new ShippingCalculator();
const testPkg = { weight: 5, length: 10, width: 10, height: 10 };

function runTests() {
  const results = [
    { desc: "Standard USA", expected: "12.50", actual: calculator.calculate(testPkg, 'USA', 'standard') },
    { desc: "Express Canada", expected: "27.50", actual: calculator.calculate(testPkg, 'Canada', 'express') },
    { desc: "Overnight Mexico (Invalid)", expected: "Overnight shipping not available for this destination", 
      actual: calculator.calculate(testPkg, 'Mexico', 'overnight') },
    { desc: "Standard Dim Surcharge", expected: "7.50", 
      actual: calculator.calculate({weight: 1, length: 20, width: 10, height: 10}, 'USA', 'standard') }
  ];

  results.forEach(t => {
    console.log(`[${t.actual === t.expected ? 'PASS' : 'FAIL'}] ${t.desc}: Expected ${t.expected}, got ${t.actual}`);
  });
}

runTests();

4. Benefits Gained

    Scalability: If you want to add "Drone Delivery," you simply create a DroneShipping class. You don't touch the ShippingCalculator or the other shipping classes.

    Testability: You can now write unit tests for OvernightShipping specifically without worrying about the complexity of Standard rules.

    Reduced Complexity: The calculate method in the Context class went from ~50 lines of nested if/else to a 3-line lookup.

    Clean Architecture: This mimics the "Back-end Developer" mindset you're training for—moving logic into specific services or handlers rather than letting it bloat a single controller.
    Context: The ShippingCalculator.

    Strategy Interface: A common interface (or abstract class) that defines a calculate(package, destination) method.

    Concrete Strategies: StandardShipping, ExpressShipping, and OvernightShipping.


## Exercise: Applying AI to deepen programming language understanding
public static void sortItems(int[] array) {
    int n = array.length;
    for (int i = 0; i < n - 1; i++) {
        int min_idx = i;
        for (int j = i + 1; j < n; j++) {
            if (array[j] < array[min_idx]) {
                min_idx = j;
            }
        }
        int temp = array[min_idx];
        array[min_idx] = array[i];
        array[i] = temp;
    }
}

1. Suggestions for Idiomatic Java

    Use camelCase for variables: In Java, min_idx is considered "un-idiomatic." Use minIndex instead.

    Leverage the final keyword: If a variable like n (length) isn't changing, mark it final to signal intent.

    In-place naming: Instead of n, use array.length directly or a descriptive name like unsortedBoundary.

    Standard Library Awareness: While this is a sorting exercise, idiomatic Java often favors Arrays.sort() for production code.

2. Why these changes follow Best Practices

    Readability (The "Clean Code" Rule): Java is verbose by design. Using minIndex over min_idx aligns with the standard library's naming conventions, making your code "look" like official Java.

    Separation of Concerns: Idiomatic Java often extracts the "swap" logic into a helper method. This follows the Single Responsibility Principle—a method should do one thing.

3. Language Features to Advantage

    Enhanced For-Loops (where applicable): While not ideal for selection sort (since we need indices), knowing when to use for (int num : array) is a core Java skill.

    Methods as Building Blocks: Java is heavily Object-Oriented. Even in static utility methods, extracting logic into private static helpers is the professional standard.


## Exercise: Understanding FastAPI Code Patterns

Part 1: Analyzing the Design Patterns
The Repository Pattern & Generic[T]

The Repository Pattern acts as a mediator between your business logic and the database.

    Why use it? It keeps your database queries in one place. If you ever switch from SQLAlchemy to another library, you only change the Repository, not your entire service layer.

    Purpose of Generic[T]: The T is a placeholder. Instead of writing a "list" or "get" function for every single table (Users, Products, Orders), you write it once in the Repository class.

    Maintainability: It enforces consistency. Every model in your system will automatically have list() and get_by_id() functionality without you writing redundant code.

Dependency Injection (DI)

FastAPI uses Depends() to manage DI. It works in layers:

    Level 1 (Infrastructure): get_db handles the database connection lifecycle.

    Level 2 (Security): get_current_user depends on get_db and the oauth2_scheme.

    Level 3 (Authorization): requires_role depends on get_current_user.

    Benefit: You don't "hardcode" a database connection inside your functions. This makes testing easy because you can "inject" a fake database during tests.

Part 2: Tracing Execution Flow (/admin/users/)

When a request hits the admin endpoint, it follows this sequence:

    Middleware: TimingMiddleware starts a timer.

    Authentication: get_current_user runs. It decodes the JWT. If the token is fake or expired, it throws a 401 Unauthorized.

    Database Lookup: get_current_user calls UserRepository to find the user in the DB.

    Authorization: The @requires_role("admin") decorator checks if is_superuser is true. If not, it throws a 403 Forbidden.

    Route Logic: The list_users function finally executes.

    Data Fetching: The UserRepository queries the database for the list of users.

    Response: The data is converted into UserSchema (Pydantic), the timer in the middleware stops, and the result is sent to the client.

Part 3: Simplifying Advanced Concepts

    lifespan: Think of this as the "Power On" and "Power Off" switch for your app. You use it to connect to the database when the server starts and disconnect when it stops.

    TimingMiddleware: It’s a "wrapper" around your whole app. It stands at the door, notes the time someone enters (request), and notes the time they leave (response) to calculate how fast the server is.

    JWT Flow:

        User gives username/password.

        Server gives back a "Digital ID Card" (The JWT).

        User shows this card for every future request.

        The server just checks the "stamp" (signature) on the card to see if it's real, rather than checking the password every single time.

Part 4: Implementation Challenge (Logging System)

To implement a logging system using these patterns, you would follow these three steps:
1. Create the Model and Repository

First, define what an "Audit Log" looks like and create a generic repository for it.
Python

class AuditLog(Base):
    __tablename__ = "audit_logs"
    id: int
    user_id: int
    action: str  # e.g., "login", "view_admin_users"
    timestamp: datetime

class AuditRepository(Repository[AuditLog]):
    # Inherits list() and get_by_id() from Repository
    async def create_log(self, db: AsyncSession, user_id: int, action: str):
        log = AuditLog(user_id=user_id, action=action, timestamp=datetime.utcnow())
        db.add(log)
        await db.commit()

2. Extend the Service Layer

Add the logging logic to the UserService.
Python

async def authenticate_user(self, db: AsyncSession, username: str, password: str) -> Optional[User]:
    user = await self.repository.get_by_username(db, username)
    if user and self.verify_password(password, user.hashed_password):
        # Add a log entry here
        audit_repo = AuditRepository(AuditLog)
        await audit_repo.create_log(db, user.id, "successful_login")
        return user
    return None

