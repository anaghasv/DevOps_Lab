# Jenkins Hello World Job

## Objective
Create a Jenkins Freestyle Job that fetches a shell script from GitHub, executes it, and displays the output in the Jenkins Console Output.

## Tools Used
- **Jenkins** – Automates and executes the build job.
- **Git and GitHub** – Stores and manages the source code.
- **Docker** – Runs Jenkins in a container.
- **Shell Script** – Prints the Hello World message.

## Project Structure

```text
8-Jenkins-Hello-World-Job/
├── README.md
└── hello-world.sh
```

## Shell Script

The `hello-world.sh` file contains:

```bash
#!/bin/bash
echo "Hello, Jenkins!"
```

## Steps to Execute

### 1. Push the Code to GitHub

Add and push the exercise files to the GitHub repository.

```bash
git add 8-Jenkins-Hello-World-Job/
git commit -m "Add Jenkins Hello World job exercise"
git push origin main
```

### 2. Create a Jenkins Freestyle Job

1. Open Jenkins at `http://localhost:8080`.
2. Click **New Item**.
3. Enter the job name `HelloWorld`.
4. Select **Freestyle project** and click **OK**.

### 3. Configure Source Code Management

1. Under **Source Code Management**, select **Git**.
2. Enter the repository URL:

   `https://github.com/anaghasv/DevOps_Lab`

3. Set the branch specifier to `*/main`.
4. Save the configuration.

### 4. Configure the Build Step

1. Open the job configuration.
2. Under **Build Steps**, select **Execute shell**.
3. Enter the following commands:

```bash
cd 8-Jenkins-Hello-World-Job
sh hello-world.sh
```

4. Click **Save**.

### 5. Run the Job

1. Open the `HelloWorld` job.
2. Click **Build Now**.
3. Open the latest build from **Build History**.
4. Click **Console Output** to view the result.

## Expected Output

```text
Hello, Jenkins!
Finished: SUCCESS
```

## Result

Successfully configured and executed a Jenkins Freestyle Job that retrieves a shell script from GitHub and runs it, displaying the output in the Jenkins Console Output.

## Key Learnings

- Creating and configuring a Jenkins Freestyle Job.
- Integrating Jenkins with a GitHub repository.
- Fetching source code from GitHub.
- Executing shell scripts through Jenkins.
- Verifying build execution through Console Output.