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