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