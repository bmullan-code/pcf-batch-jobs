# pcf-batch-jobs
Tutorial on PCF Scheduled Batch Jobs


1) Create a project directory
```
mkdir app
cd app
```
2) Create a task script
task.sh
```
#!/bin/bash
echo "running in cf instance id $CF_INSTANCE_GUID on host $CF_INSTANCE_INTERNAL_IP"
```

```
chmod +x task.sh
```

3) Create a manifest.yml file to push to PCF

```
applications:
- name: task-app
  buildpacks:
  - binary_buildpack
  command: "./task.sh"
  disk_quota: 256M
  instances: 0
  memory: 256M
  health-check-type: none
  no-route: true
  ```
Push it to PCF with 
```
cf push
```

4) Run script as a task

```
cf run-task task-app "./task.sh" --name my-task
```

You can check the status of the run with 
```
cf tasks task-app

Getting tasks for app task-app in org CVSWksp / space BarryM as bmullan@pivotal.io...
OK

id   name      state       start time                      command
1    my-task   SUCCEEDED   Fri, 20 Dec 2019 17:55:00 UTC   ./task.sh
```
You can check the logs with 
```
cf logs task-app --recent
```

5) Scheduling Tasks [more info](https://docs.pivotal.io/scheduler/1-2/using-jobs.html)

You must first bind the app to an instance of pcf scheduler. You can find the name of the scheduler with 
```
cf services | grep scheduler
```
If there is no instance of pcf scheduler you can create one with 
```
cf cs scheduler-for-pcf standard my-scheduler
```

Bind the app to the scheduler
```
cf bind-service task-app my-scheduler
```

Create a job (similar to creating a task)
```
cf create-job task-app my-job "./task.sh"
```

Schedule the job (uses [cron syntax](https://docs.pivotal.io/scheduler/1-2/syntax.html))
```
cf schedule-job my-job "0 * * * ?"
```

You can check status and history with these commands
```
cf job-schedules
cf jobs
cf job-history my-job
```








  
