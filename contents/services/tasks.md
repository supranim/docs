---
title: "Task Scheduler"
description: "Schedule and manage background tasks in your Supranim application with the Task Scheduler service."
keywords: ["task scheduler", "background tasks", "cron jobs", "scheduled tasks"]
---

## About
The Task Scheduler service allows you to schedule and manage background tasks in your Supranim application. It can be used to run periodic tasks, for example, to send out email notifications, perform database cleanups, or execute any code at specified intervals.

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">
    You can use the Task Scheduler in your Supranim application or as a standalone Nimble package in any Nim project.
  </div>
</div>

## Features
- Schedule tasks to run at specific intervals (e.g., every hour, daily, weekly)
- Manage and monitor scheduled tasks through a simple API
- Based on Libevent for efficient event-driven task execution
- Support for both one-time and recurring tasks
- Threadpool-based execution for efficient task management

## Initialize Task Scheduler
To use the Task Scheduler service in your Supranim application, you need to initialize it in your main application file:

```nim
App.services do:
  # other services...
  tasks.init()
```

## Where to define tasks
In Supranim, tasks are stored in the `src/service/event/queue` directory, alongside the `listeners` directory where [event listeners](/services/events) are stored. 

All `.nim` files created in the `queue` directory are automatically discovered by the Task Scheduler service and can be used to define background tasks. We **recommended to organize your tasks separately** (one file per task or group related tasks) for better maintainability.


## Create a background task
Create a new task using the `newTask` macro, which takes a unique task name and a description as arguments. You can then specify the task's schedule and the code to execute when the task runs:
```nim
newTask fetchData, "Fetch data from an API every hour":
  interval: 1.hour
    # The task will run every hour
  repeat: true
    # This task will repeat every hour
  runner do:
    # Here you can write the code that you want to execute every hour
    echo "Fetching data from API..."
```

If you prefer, you can use the API provided by the Task Scheduler package to create tasks programmatically. This is useful when you want to create tasks dynamically based on user input or other conditions in your application. 

You can use `TasksInstance` singleton to access the Task Scheduler service:
```nim
TasksInstance.submitTask(
  Task(
    id: "task1",
    work: proc() =
      echo "Hello from task1"
      while true:
        sleep(1000) # Simulate long-running task
  )
)
```

## Standalone Usage
The Task Scheduler can also be used as a standalone Nimble package in any Nim project. Check the README of the [tasks package](https://github.com/supranim/tasks?tab=readme-ov-file#example-usage) for more details on how to use it outside of a Supranim application.

### API Reference
The API reference for this service: https://supranim.github.io/tasks
