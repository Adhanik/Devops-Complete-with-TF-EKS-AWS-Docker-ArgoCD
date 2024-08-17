

# Workflow

Whenever an event is triggered, for eg, a pull request is made to your repo (which defines a event), a workflow is triggered, which in turns consists of several jobs, and each of these jobs gets triggered inside its own virtual machine runner, or inside a container. This jobs consist of one or more steps that either run a script that you define or run an action, which is a reusable extension that can simplify your workflow.

So Workflow is an automated process that runs one or more jobs. They are defined in a YAML file, and run when triggered by an event in our repository. They can also be triggered manually, or at a defined schedule.

Workflows are defined in the .github/workflows directory in a repository.


# Events

An event is a activity that triggers a workflow run. For eg, when someone creates a PR, opens a issue, or pushes a commit to a repo.


# JOBS

1. A job is a set of steps in workflow, which are executed once the event happens. The steps in job are either a shell script, or a action that will be run. All job steps are executed in order, and are dependent on each other.

2. All steps are executed on same runner, so can have a step that builds your application followed by a step that tests the application that was built.

3. By default, jobs have no dependencies and run in parallel. When a job takes a dependency on another job, it waits for the dependent job to complete before running.

# Actions - for frequently repeated task

1. Use an action to help reduce the amount of repetitive code that you write in your workflow files. An action can pull your Git repository from GitHub, set up the correct toolchain for your build environment, or set up the authentication to your cloud provider.

2. You can write your own actions, or you can find actions to use in your workflows in the GitHub Marketplace.

eg actions/checkout@v4   --> helps u checkout a specific repo

# Runners

1. A runner is a server that runs your workflows when they're triggered. Each runner can run a single job at a time. GitHub provides Ubuntu Linux, Microsoft Windows, and macOS runners to run your workflows. Each workflow run executes in a fresh, newly-provisioned virtual machine.

2. If you need a different operating system or require a specific hardware configuration, you can host your own runners.