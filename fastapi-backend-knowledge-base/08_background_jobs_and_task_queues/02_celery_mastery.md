# 02. Celery Mastery: Distributed Task Queue

## 1. Introduction

**Celery** is the industry standard for Python background tasks. It's powerful, mature, and integrates perfectly with FastAPI.

It requires two things:
1.  **A Broker**: To send messages (Redis/RabbitMQ).
2.  **A Backend**: To store results (Redis/Database).

---

## 2. Setting up with FastAPI

Project Structure:
```
project/
  main.py
  worker.py  <-- Celery App definition
  tasks.py   <-- Actual task logic
```

### `worker.py`
```python
from celery import Celery

# Configure Celery to use Redis as Broker and Backend
celery_app = Celery(
    "worker",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/0"
)

# Optional: Configure default task settings
celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
)
```

### `tasks.py`
```python
import time
from .worker import celery_app

@celery_app.task(name="send_email_task")
def send_email_task(email: str, subject: str):
    # Simulate long running process
    time.sleep(5)
    return f"Email sent to {email} with subject {subject}"
```

### `main.py` (FastAPI)
```python
from fastapi import FastAPI
from .tasks import send_email_task

app = FastAPI()

@app.post("/send-email")
def send_email(email: str):
    # .delay() is the magic method to push to queue
    task = send_email_task.delay(email, "Welcome!")
    return {"task_id": task.id, "status": "Processing"}
```

---

## 3. Advanced Patterns

### Retries (Resilience)
Network glitches happen. Celery can auto-retry failed tasks.

```python
@celery_app.task(bind=True, max_retries=3, default_retry_delay=10)
def unstable_task(self, data):
    try:
        # Do something risky
        process_payment(data)
    except ConnectionError as exc:
        # Retry in 10s, 20s, 40s (Exponential Backoff)
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)
```

### Chains (Workflows)
Execute tasks in a specific order.
`Task A -> Task B -> Task C`

```python
from celery import chain
from .tasks import fetch_data, process_data, save_data

# The output of fetch_data is passed as input to process_data, etc.
workflow = chain(fetch_data.s(), process_data.s(), save_data.s())
workflow.apply_async()
```

### Periodic Tasks (Celery Beat)
Run a task every Monday at 8 AM.

```python
from celery.schedules import crontab

celery_app.conf.beat_schedule = {
    "daily-report": {
        "task": "generate_report_task",
        "schedule": crontab(hour=8, minute=0), # Every day at 8am
    },
}
```

---

## 4. Running It

You need to run the FastAPI server AND the Celery worker.

**Terminal 1 (API):**
```bash
uvicorn main:app --reload
```

**Terminal 2 (Worker):**
```bash
# -A points to the file where celery_app is defined
# --loglevel=info to see what's happening
celery -A project.worker worker --loglevel=info
```

**Terminal 3 (Beat - Optional for periodic tasks):**
```bash
celery -A project.worker beat --loglevel=info
```

### Monitoring
Use **Flower**, a web UI for Celery.
```bash
pip install flower
celery -A project.worker flower
```
Visit `localhost:5555` to see active tasks, success rates, and worker health.
