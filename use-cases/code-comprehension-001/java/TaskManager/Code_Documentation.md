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
