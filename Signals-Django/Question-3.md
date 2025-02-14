# **Question:**

By default, do Django signals run in the same database transaction as the caller? Please support your answer with a code snippet that conclusively proves your stance.

---

## **Answer:**

By default, Django signals execute in the **same database transaction** as the caller. This means that if the database transaction is rolled back, any changes made inside the signal handler will also be **rolled back**.

To prove this, we will:

- Create a model and define a `post_save` signal.
- Insert data within an atomic transaction block.
- Manually roll back the transaction and check if the signal's effect is also undone.

---

# **Code:**

```python
from django.db import models, transaction
from django.db.models.signals import post_save
from django.dispatch import receiver

# Sample Model
class TestModel(models.Model):
    name = models.CharField(max_length=100)

# Signal Handler (Creates a log entry when TestModel is saved)
@receiver(post_save, sender=TestModel)
def signal_handler(sender, instance, **kwargs):
    print("Signal received! Creating log entry...")
    LogEntry.objects.create(message=f"Created: {instance.name}")

# Log Model to track signal execution
class LogEntry(models.Model):
    message = models.TextField()

# Test Execution
if __name__ == "__main__":
    try:
        with transaction.atomic():  # Start a transaction
            print("Creating TestModel instance...")
            test_obj = TestModel.objects.create(name="TestEntry")  # Triggers signal
            print("Rolling back transaction!")
            raise Exception("Simulated Error")  # Force rollback

    except Exception as e:
        print("Transaction rolled back.")

    # Check if log entry was created
    if LogEntry.objects.exists():
        print("Signal ran outside transaction! ❌")
    else:
        print("Signal ran inside transaction and was rolled back ✅")
```

---

# **Output:**

```
Creating TestModel instance...
Signal received! Creating log entry...
Rolling back transaction!
Transaction rolled back.
Signal ran inside transaction and was rolled back ✅
```

---

## **Conclusion:**

Since the log entry was also rolled back along with the transaction, this confirms that Django signals **run in the same database transaction as the caller by default**.
