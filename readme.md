celery command: python -m celery -A django_celery worker -l info








Why use celery?
Third-Party API Calls: Offload long-running API calls to background tasks, ensuring the main application remains responsive.

High CPU-Intensive Tasks: Manage computationally heavy tasks asynchronously to free up resources for handling user requests.

Periodic/Scheduled Tasks: Use Celery Beat to schedule tasks at specific intervals for routine maintenance, data cleanup, and more.

Improving User Experience: Handle long-running tasks in the background to keep your application fast and responsive.









Q: Tell me the difference between ClockedSchedule tasks vs SolicitudeSchedule tasks vs Intervalschedule task vs crontabschedule task.

A: The answer is …

IntervalSchedule: This type of schedule is used to execute a task at a fixed interval of time. It requires a every parameter, which defines the time interval in seconds, minutes or hours at which the task should be executed. For example, every=10 will execute the task every 10 seconds.

CrontabSchedule: This type of schedule is used to execute a task based on a cron-like schedule. A cron schedule is defined using a string that specifies the exact time and date when the task should be executed. For example, This will execute the task every day at midnight.

ClockedSchedule : This type of schedule is used to execute a task at a specific time of day. It requires a clocked_time parameter, which defines the exact time of day when the task should be executed. For example, clocked_time=datetime.time(10, 0) will execute the task at 10:00am every day. Found in from celery.schedules import ClockedSchedule

SolicitudeSchedule : This type of schedule is used to execute a task based on a specific condition. It requires a condition parameter, which defines the condition when the task should be executed. For example, condition=lambda: datetime.now().hour == 10 will execute the task only if the current hour is 10.





Q: How to limit the number of tasks executed concurrently ?
A: per task level:

from celery import Celery

app = Celery('my_app')
app.conf.update(
    worker_concurrency=3,
)



# Retry
1. Exponential Backoff
Exponential backoff is a strategy for retrying failed operations (e.g., reconnecting to a server, sending a request). Instead of retrying at constant intervals, the time between retries increases exponentially.


2. Random Jitter
Random jitter refers to adding a small, random variation to the backoff delay. For example:

If the calculated delay is 4 seconds, jitter might adjust it to a random value between 3 and 5 seconds.


```python
from celery import Celery, Task

app = Celery('my_app', broker='redis://localhost:6379/0')

@app.task(
    autoretry_for=(Exception,),  # Retry on any exception
    retry_kwargs={'max_retries': 5, 'countdown': 10},  # Retry up to 5 times, 10s apart
    retry_backoff=True,  # Use exponential backoff
    retry_jitter=True    # Add random jitter to delay
)
def my_task(x, y):
    result = x / y
    return result
```


Manual Retry in Tasks
For more control, you can manually retry tasks using the retry method:

```python
@app.task(bind=True)
def my_manual_task(self, x, y):
    try:
        result = x / y
    except ZeroDivisionError as e:
        raise self.retry(exc=e, countdown=5, max_retries=3)
    return result

```

# Dynamic Pool Control
Dynamic pool control allows you to adjust the number of worker processes dynamically without restarting Celery. This is particularly useful for handling fluctuating workloads efficiently.

How It Works
Celery workers use pools to manage worker processes.
You can resize the pool at runtime using Celery's remote control commands.
This is typically achieved with the celery control command or via Flower.

# Increase the Pool Size:
```bash
celery -A my_app control pool_grow 2

# Decrease the Pool Size:


celery -A my_app control pool_shrink 1
```
# Automatic Scaling
To automate pool control, you can integrate custom logic using Celery's autoscaler:

The autoscaler adjusts the pool size based on the task queue's length and system load.
Example: Enabling Autoscaler
```bash
celery -A my_app worker --autoscale=10,3
# 10: Maximum number of worker processes.
# 3: Minimum number of worker processes.
```




When you start a celery worker, you specify the pool, concurrency, autoscale etc. in the command.

Pool: Determines the type of execution (thread, child process, worker itself etc.).
Concurrency: Decides the size of the pool.
Autoscale: Dynamically adjusts the pool size based on the load.

```Bash
celery -A <project>.celery worker --pool=prefork --concurrency=5 --autoscale=10,3 -l info
```



```python

```