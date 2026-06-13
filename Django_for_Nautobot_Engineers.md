# Django for Network Engineers — Nautobot-Ready in 9 Weeks

> **Perspective throughout this guide:** Everything is taught from the angle of *why Nautobot works the way it does*. Every Django concept maps directly to something you interact with in Nautobot Jobs, the Nautobot REST API, or Nautobot's web UI.

---

## How to Read This Guide

Each week follows this structure:

- **What you will learn** — the Django concept
- **Why it matters in Nautobot** — direct mapping to what Nautobot uses it for
- **Core theory** — explained in plain language first
- **Code** — all examples use network/Nautobot-relevant data models
- **AUDIT** — architecture breakdown
- **BUG** — common mistakes and how to avoid them
- **Practice exercise** — build something that directly mirrors Nautobot internals

**Prerequisites:** Python basics (functions, classes, loops, dicts, try/except). Nothing else.

---

---

# WEEK 1 — Python + HTTP Foundations

## What you will learn
How the web works at the HTTP level — requests, responses, status codes, headers, JSON. Django is an HTTP framework; without understanding HTTP, Django feels like magic for the wrong reasons.

## Why it matters in Nautobot
Every time your iAutomate playbook calls `ansible.builtin.uri` to hit the Nautobot REST API, it is making an HTTP request. When you write a Nautobot Job that calls `requests.get(...)` to query vManage, that is HTTP. The entire Nautobot API surface (GET /api/dcim/devices/, POST /api/ipam/ip-addresses/) is HTTP under the hood.

---

## 1.1 How HTTP Works

```
Client (Browser / Ansible / Python script)
        │
        │  HTTP Request
        │  ─────────────────────────────────
        │  GET /api/dcim/devices/?tenant=deapega HTTP/1.1
        │  Host: nautobot.example.com
        │  Authorization: Token abc123xyz
        │  Accept: application/json
        │
        ▼
Server (Django / Nautobot)
        │
        │  HTTP Response
        │  ─────────────────────────────────
        │  HTTP/1.1 200 OK
        │  Content-Type: application/json
        │
        │  {"count": 4, "results": [...]}
        │
        ▼
Client receives response
```

### Request components

| Part | Example | What it is |
|---|---|---|
| Method | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` | What operation you want |
| URL | `/api/dcim/devices/` | Which resource |
| Headers | `Authorization: Token abc123` | Metadata about the request |
| Body | `{"hostname": "vedge-01"}` | Data payload (POST/PATCH only) |
| Query params | `?tenant=deapega&is_active=true` | Filters on GET requests |

### HTTP status codes — the ones Nautobot uses

| Code | Meaning | When Nautobot returns it |
|---|---|---|
| `200 OK` | Success | GET returned data |
| `201 Created` | Resource created | POST succeeded |
| `204 No Content` | Success, no body | DELETE succeeded |
| `400 Bad Request` | You sent bad data | Invalid field in POST body |
| `401 Unauthorized` | No/invalid token | Missing Authorization header |
| `403 Forbidden` | Valid token, no permission | Token user lacks object permission |
| `404 Not Found` | Resource doesn't exist | Device ID doesn't exist |
| `500 Internal Server Error` | Server crashed | Bug in Nautobot/your Job |

---

## 1.2 Making HTTP Requests in Python

```python
import requests

NAUTOBOT_URL = "https://nautobot.example.com"
TOKEN        = "abc123xyz"

HEADERS = {
    "Authorization": f"Token {TOKEN}",
    "Content-Type":  "application/json",
    "Accept":        "application/json",
}

# GET — query devices for a customer
resp = requests.get(
    f"{NAUTOBOT_URL}/api/dcim/devices/",
    headers=HEADERS,
    params={"tenant": "deapega", "platform": "Viptela vEdge"},
)
resp.raise_for_status()   # raises HTTPError for 4xx/5xx

data    = resp.json()
devices = data["results"]

for device in devices:
    print(device["name"], device["primary_ip"]["address"])

# POST — create a new IP address
payload = {
    "address":  "10.100.1.1/30",
    "status":   {"name": "Active"},
    "tenant":   {"name": "deapega"},
}
resp = requests.post(
    f"{NAUTOBOT_URL}/api/ipam/ip-addresses/",
    headers=HEADERS,
    json=payload,
)
resp.raise_for_status()
print("Created:", resp.json()["id"])
```

---

## 1.3 JSON — The Language Nautobot Speaks

```python
import json

# Python dict → JSON string (for sending in POST body)
device = {"hostname": "vedge-01", "ip": "10.0.0.1", "active": True}
json_string = json.dumps(device)
# '{"hostname": "vedge-01", "ip": "10.0.0.1", "active": true}'

# JSON string → Python dict (for reading API response)
raw = '{"count": 2, "results": [{"name": "r1"}, {"name": "r2"}]}'
parsed = json.loads(raw)
print(parsed["count"])          # 2
print(parsed["results"][0])     # {"name": "r1"}

# requests does this automatically:
resp = requests.get(url, headers=HEADERS)
data = resp.json()   # equivalent to json.loads(resp.text)
```

---

## BUG — Week 1

**Bug: not calling raise_for_status() — silently swallowing errors**

```python
# BAD — a 401 or 500 response looks like success
resp = requests.get(url, headers=HEADERS)
data = resp.json()   # might be {"detail": "Authentication required"}

# GOOD — fails loudly on 4xx/5xx
resp = requests.get(url, headers=HEADERS)
resp.raise_for_status()   # raises requests.HTTPError if not 2xx
data = resp.json()
```

**Bug: forgetting that query params are case-sensitive in Nautobot**

```python
# WRONG — Nautobot platform names are case-sensitive
params = {"platform": "viptela vedge"}     # returns 0 results

# CORRECT — match the exact catalog value
params = {"platform": "Viptela vEdge"}     # returns correct results
```

---

## Practice — Week 1

Write a Python script (no Django yet) that:
1. Queries `GET /api/dcim/devices/` filtered by `tenant=deapega`
2. Prints each device's name and primary IP
3. Handles `404` and `401` explicitly with different error messages
4. Saves the result list to a `devices.json` file

---

---

# WEEK 2 — Django Project Setup + Settings

## What you will learn
How a Django project is structured, what every file does, and how Django's settings system controls the entire application's behaviour.

## Why it matters in Nautobot
Nautobot is a Django project. When you look at Nautobot's source code, you see `nautobot/settings.py`, `nautobot/urls.py`, `nautobot/apps/` — the exact same structure you will create this week. Understanding this means you can read Nautobot's source confidently.

---

## 2.1 Installing Django and Creating a Project

```bash
# Create a virtual environment (always — never install globally)
python3 -m venv venv
source venv/bin/activate       # Linux/Mac
# venv\Scripts\activate        # Windows

pip install django

# Create the project
django-admin startproject netinventory .
# The trailing dot puts files in current dir instead of a subdirectory

# Create your first app (Django calls feature modules "apps")
python manage.py startapp devices

# Run the development server
python manage.py runserver     # http://127.0.0.1:8000
```

---

## 2.2 Project Structure — Every File Explained

```
netinventory/                  ← Your Django project root
├── manage.py                  ← CLI entry point (runserver, migrate, etc.)
├── netinventory/              ← Project config package
│   ├── __init__.py
│   ├── settings.py            ← All configuration lives here
│   ├── urls.py                ← Root URL router (delegates to app URLconfs)
│   ├── wsgi.py                ← WSGI server entry point (production)
│   └── asgi.py                ← ASGI server entry point (async)
│
└── devices/                   ← Your "devices" feature app
    ├── __init__.py
    ├── models.py              ← DB schema (the data models)
    ├── views.py               ← Business logic
    ├── urls.py                ← URL patterns for this app
    ├── admin.py               ← Admin UI registration
    ├── forms.py               ← Form definitions
    ├── tests.py               ← Test cases
    └── migrations/            ← Auto-generated DB schema changes
        └── __init__.py
```

**Nautobot parallel:**

```
nautobot/
├── manage.py
├── core/settings.py           ← Nautobot's settings
├── core/urls.py               ← Root URL router
├── dcim/models.py             ← Device, Interface, Cable, Site models
├── ipam/models.py             ← IPAddress, Prefix, VLAN models
├── circuits/models.py         ← Circuit, Provider models
└── extras/models.py           ← Job, CustomField, Tag models
```

Everything is the same structure. Nautobot is just a Django project with network-specific apps.

---

## 2.3 settings.py — The Control Centre

```python
# netinventory/settings.py

import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# ── Security ──────────────────────────────────────────────────────────────
SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY", "dev-only-insecure-key")
DEBUG      = os.environ.get("DJANGO_DEBUG", "True") == "True"
ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS", "127.0.0.1").split(",")

# ── Installed Apps ──────────────────────────────────────────────────────────
# Django only knows about your app if it's listed here
INSTALLED_APPS = [
    "django.contrib.admin",        # Admin UI
    "django.contrib.auth",         # Authentication system
    "django.contrib.contenttypes", # Generic relations (Nautobot uses heavily)
    "django.contrib.sessions",     # Session management
    "django.contrib.messages",     # Flash messages
    "django.contrib.staticfiles",  # Static file serving

    # Third-party
    "rest_framework",              # Django REST Framework (Nautobot's API layer)

    # Your apps
    "devices",                     # The app you created
]

# ── Database ────────────────────────────────────────────────────────────────
# Nautobot requires PostgreSQL. For learning, SQLite is fine.
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}

# For PostgreSQL (what Nautobot uses in production):
# DATABASES = {
#     "default": {
#         "ENGINE": "django.db.backends.postgresql",
#         "NAME": os.environ["DB_NAME"],
#         "USER": os.environ["DB_USER"],
#         "PASSWORD": os.environ["DB_PASSWORD"],
#         "HOST": os.environ.get("DB_HOST", "localhost"),
#         "PORT": os.environ.get("DB_PORT", "5432"),
#     }
# }

# ── URLs ────────────────────────────────────────────────────────────────────
ROOT_URLCONF = "netinventory.urls"

# ── Static Files ─────────────────────────────────────────────────────────────
STATIC_URL = "/static/"

# ── Timezone ─────────────────────────────────────────────────────────────────
TIME_ZONE = "UTC"
USE_TZ    = True

# ── Default Primary Key ───────────────────────────────────────────────────────
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"
```

---

## 2.4 manage.py — The Swiss Army Knife

```bash
# Most important commands

python manage.py runserver          # Start dev server
python manage.py shell              # Django-aware Python REPL (very useful)
python manage.py makemigrations     # Generate migration files after model changes
python manage.py migrate            # Apply migrations to DB
python manage.py createsuperuser    # Create admin login
python manage.py test               # Run tests
python manage.py collectstatic      # Gather static files for production

# Nautobot uses the same manage.py interface:
nautobot-server runserver
nautobot-server migrate
nautobot-server createsuperuser
```

---

## BUG — Week 2

**Bug: app not in INSTALLED_APPS — models are invisible**

```python
# If "devices" is not in INSTALLED_APPS:
# - makemigrations finds no models
# - the admin can't register anything from it
# - the ORM can't query it

# Always add your app after creating it:
INSTALLED_APPS = [
    ...
    "devices",    # Required for Django to discover your models
]
```

**Bug: DEBUG=True in production**

```python
# DEBUG=True in production exposes:
# - Full stack traces with local variables to anyone who hits a 500 error
# - All your settings values
# - Internal path information

# Always:
DEBUG         = False              # In production
ALLOWED_HOSTS = ["yourdomain.com"] # Django refuses to serve without this in prod
```

---

## Practice — Week 2

1. Create the `netinventory` project and `devices` app
2. Configure PostgreSQL in settings.py using environment variables
3. Run `python manage.py migrate` (applies Django's built-in migrations)
4. Run `python manage.py createsuperuser`
5. Visit `http://127.0.0.1:8000/admin/` — you should see Django's admin UI

---

---

# WEEK 3 — Models: Your Database Schema in Python

## What you will learn
Django models are Python classes that define your database schema. Django translates them to SQL automatically. The ORM (Object-Relational Mapper) lets you query the database using Python — no raw SQL needed.

## Why it matters in Nautobot
`Device`, `Interface`, `IPAddress`, `Prefix`, `Circuit`, `VLAN` — everything in Nautobot is a Django model. When your Nautobot Job writes `Device.objects.filter(...)`, it is using the ORM on top of a model defined in `nautobot/dcim/models.py`. This week you build that intuition from scratch.

---

## 3.1 Defining Models

```python
# devices/models.py

from django.db import models


class Site(models.Model):
    """Network site / data centre location."""

    name = models.CharField(max_length=100, unique=True)
    city = models.CharField(max_length=100)
    country = models.CharField(max_length=2)   # ISO country code

    def __str__(self):
        return self.name

    class Meta:
        ordering = ["name"]


class Platform(models.Model):
    """Device operating system / platform (IOS-XE, Viptela vEdge, etc.)."""

    name = models.CharField(max_length=100, unique=True)
    napalm_driver = models.CharField(max_length=50, blank=True)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ["name"]


class Device(models.Model):
    """Network device — mirrors Nautobot's dcim.Device model."""

    class StatusChoices(models.TextChoices):
        ACTIVE  = "active",  "Active"
        PLANNED = "planned", "Planned"
        FAILED  = "failed",  "Failed"
        OFFLINE = "offline", "Offline"

    # Basic fields
    name       = models.CharField(max_length=100, unique=True)
    status     = models.CharField(max_length=20, choices=StatusChoices.choices,
                                  default=StatusChoices.ACTIVE)
    make       = models.CharField(max_length=50)
    model      = models.CharField(max_length=50)

    # Relationships (ForeignKey = many-to-one)
    site       = models.ForeignKey(Site, on_delete=models.PROTECT,
                                   related_name="devices")
    platform   = models.ForeignKey(Platform, on_delete=models.SET_NULL,
                                   null=True, blank=True, related_name="devices")

    # Network fields
    management_ip = models.GenericIPAddressField(null=True, blank=True)

    # Metadata
    customer_number = models.CharField(max_length=20, blank=True)
    created_at      = models.DateTimeField(auto_now_add=True)
    updated_at      = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ["name"]


class Interface(models.Model):
    """Network interface on a device."""

    class TypeChoices(models.TextChoices):
        PHYSICAL = "physical", "Physical"
        LOOPBACK = "loopback", "Loopback"
        VIRTUAL  = "virtual",  "Virtual"
        TUNNEL   = "tunnel",   "Tunnel"

    device   = models.ForeignKey(Device, on_delete=models.CASCADE,
                                 related_name="interfaces")
    name     = models.CharField(max_length=64)   # e.g. "GigabitEthernet0/0/0"
    type     = models.CharField(max_length=20, choices=TypeChoices.choices,
                                default=TypeChoices.PHYSICAL)
    ip       = models.GenericIPAddressField(null=True, blank=True)
    prefix_length = models.PositiveSmallIntegerField(null=True, blank=True)
    enabled  = models.BooleanField(default=True)
    description = models.CharField(max_length=200, blank=True)

    def __str__(self):
        return f"{self.device.name} — {self.name}"

    class Meta:
        unique_together = [("device", "name")]
        ordering = ["device__name", "name"]
```

---

## 3.2 Field Types — The Most Important Ones

```python
# String fields
models.CharField(max_length=100)              # Short strings — always set max_length
models.TextField()                             # Long text — no max_length needed
models.SlugField(max_length=50)               # URL-safe strings (kebab-case)

# Numeric fields
models.IntegerField()
models.PositiveIntegerField()
models.PositiveSmallIntegerField()             # 0–32767 — good for port numbers
models.DecimalField(max_digits=10, decimal_places=2)

# Boolean
models.BooleanField(default=True)

# Network-specific
models.GenericIPAddressField()                 # Stores IPv4 and IPv6
models.GenericIPAddressField(protocol="IPv4")  # IPv4 only

# Date/time
models.DateTimeField(auto_now_add=True)        # Set once on creation — never changes
models.DateTimeField(auto_now=True)            # Updated on every save()
models.DateField()

# Relationships
models.ForeignKey(OtherModel, on_delete=models.CASCADE)    # Many-to-one
models.ManyToManyField(OtherModel)                         # Many-to-many
models.OneToOneField(OtherModel, on_delete=models.CASCADE) # One-to-one

# on_delete options:
# CASCADE  — delete this row when the related row is deleted (Interface → Device)
# PROTECT  — prevent deletion of related row if this row exists (Device → Site)
# SET_NULL — set FK to null when related row is deleted (Device → Platform)

# Optional fields — both required together
models.CharField(max_length=50, blank=True)   # Allow empty string in forms
models.IntegerField(null=True, blank=True)     # Allow NULL in DB + empty in forms
```

---

## 3.3 Applying Models to the Database (Migrations)

```bash
# After defining or changing any model:

# Step 1 — Django detects changes and generates a migration file
python manage.py makemigrations devices
# Creates: devices/migrations/0001_initial.py

# Step 2 — Apply the migration to the database
python manage.py migrate
# Runs SQL: CREATE TABLE devices_device (...), CREATE TABLE devices_interface (...)

# View the SQL Django will run (without actually running it)
python manage.py sqlmigrate devices 0001
```

**What the migration file looks like (auto-generated — never edit manually):**

```python
# devices/migrations/0001_initial.py  (auto-generated)

from django.db import migrations, models
import django.db.models.deletion

class Migration(migrations.Migration):

    dependencies = []

    operations = [
        migrations.CreateModel(
            name="Site",
            fields=[
                ("id", models.BigAutoField(primary_key=True)),
                ("name", models.CharField(max_length=100, unique=True)),
                ("city", models.CharField(max_length=100)),
                ("country", models.CharField(max_length=2)),
            ],
        ),
        migrations.CreateModel(
            name="Device",
            fields=[
                ("id", models.BigAutoField(primary_key=True)),
                ("name", models.CharField(max_length=100, unique=True)),
                ("status", models.CharField(max_length=20)),
                ("make", models.CharField(max_length=50)),
                ("model", models.CharField(max_length=50)),
                ("management_ip", models.GenericIPAddressField(null=True)),
                ("site", models.ForeignKey(to="devices.Site",
                                           on_delete=django.db.models.deletion.PROTECT)),
            ],
        ),
        # ... more tables
    ]
```

---

## 3.4 The Django Shell — Your Testing Ground

```bash
python manage.py shell
```

```python
# In the shell — create and query objects

from devices.models import Site, Platform, Device, Interface

# ── Create records ──────────────────────────────────────────────────────────
site = Site.objects.create(name="IAD-DC1", city="Ashburn", country="US")
platform = Platform.objects.create(name="Viptela vEdge", napalm_driver="")

vedge1 = Device.objects.create(
    name="deapega-vedge-cpe2",
    status="active",
    make="Cisco",
    model="vEdge 100",
    site=site,
    platform=platform,
    management_ip="192.168.31.21",
    customer_number="304999",
)

# ── Query records ────────────────────────────────────────────────────────────
# Get all devices
Device.objects.all()

# Filter — returns QuerySet (list of matching objects)
Device.objects.filter(status="active")
Device.objects.filter(platform__name="Viptela vEdge")    # double-underscore = JOIN
Device.objects.filter(customer_number="304999", status="active")  # AND

# Get a single object
device = Device.objects.get(name="deapega-vedge-cpe2")
print(device.management_ip)   # "192.168.31.21"
print(device.site.name)       # "IAD-DC1"   (follows ForeignKey)
print(device.platform.name)   # "Viptela vEdge"

# ── Update ───────────────────────────────────────────────────────────────────
device.status = "offline"
device.save()

# Or bulk update (single SQL UPDATE, no Python loop needed)
Device.objects.filter(customer_number="304999").update(status="active")

# ── Delete ───────────────────────────────────────────────────────────────────
Device.objects.filter(status="failed").delete()
```

---

## 3.5 The double-underscore (__)  — Django's JOIN Syntax

This is the most important ORM pattern to understand. The `__` lets you traverse ForeignKey relationships and apply lookups in a single filter expression.

```python
# ForeignKey traversal — equivalent to SQL JOIN
Device.objects.filter(site__city="Ashburn")
Device.objects.filter(platform__name="Viptela vEdge")
Device.objects.filter(site__country="US", status="active")

# You can go multiple levels deep
Device.objects.filter(interfaces__type="loopback")   # devices that have a loopback

# Lookup types (after __ on a field)
Device.objects.filter(name__startswith="deapega")    # LIKE 'deapega%'
Device.objects.filter(name__icontains="vedge")        # case-insensitive LIKE
Device.objects.filter(management_ip__isnull=False)    # WHERE ip IS NOT NULL
Device.objects.filter(customer_number__in=["304999", "305000"])  # IN (...)
Device.objects.exclude(status="failed")               # NOT WHERE status = 'failed'

# Range / comparison
Device.objects.filter(created_at__gte="2025-01-01")  # >=
Device.objects.filter(created_at__lt="2026-01-01")   # <

# Exact match (default — these are equivalent)
Device.objects.filter(status="active")
Device.objects.filter(status__exact="active")
```

**Nautobot equivalent — these are real queries from Nautobot Jobs:**

```python
# In a Nautobot Job:
from nautobot.dcim.models import Device

# Exact same syntax — it's all Django ORM
vedges = Device.objects.filter(
    platform__name="Viptela vEdge",
    tenant__name="deapega",
    status__name="Active",
)
```

---

## BUG — Week 3

**Bug: missing both null=True and blank=True for optional fields**

```python
# WRONG — blank=True alone: form accepts empty, but DB rejects NULL → crash
management_ip = models.GenericIPAddressField(blank=True)

# WRONG — null=True alone: DB accepts NULL, but form won't submit empty
management_ip = models.GenericIPAddressField(null=True)

# CORRECT — both required for a truly optional field
management_ip = models.GenericIPAddressField(null=True, blank=True)
# Exception: CharField — use blank=True only (store "" not NULL for strings)
napalm_driver = models.CharField(max_length=50, blank=True)  # Correct
```

**Bug: using get() without try/except**

```python
# BAD — raises Device.DoesNotExist if not found — unhandled exception
device = Device.objects.get(name="router1")

# GOOD — handle the exception
try:
    device = Device.objects.get(name="router1")
except Device.DoesNotExist:
    device = None

# ALSO GOOD — get_or_create
device, created = Device.objects.get_or_create(
    name="router1",
    defaults={"make": "Cisco", "site": site},
)
```

**Bug: modifying a field but forgetting to call save()**

```python
device = Device.objects.get(name="router1")
device.status = "offline"
# Nothing happened to the DB yet!

device.save()   # Required — updates the DB row
```

---

## Practice — Week 3

1. Define the `Site`, `Platform`, `Device`, `Interface` models above
2. Run `makemigrations` and `migrate`
3. In the Django shell:
   - Create 2 sites, 2 platforms, 5 devices, 3 interfaces each
   - Filter devices by platform using the `__` traversal
   - Update all devices for one site to status="offline"
   - Delete all devices that are "failed"

---

---

# WEEK 4 — ORM Advanced: Querysets, Optimization, Q Objects

## What you will learn
Advanced ORM: chaining querysets, avoiding the N+1 query problem, Q objects for OR conditions, aggregations, and custom managers. This is where ORM performance is made or broken.

## Why it matters in Nautobot
Nautobot Jobs can process hundreds or thousands of devices. Without `select_related` and `prefetch_related`, a Job querying 500 devices could generate 500+ DB queries and time out. This week is about writing efficient Nautobot Jobs.

---

## 4.1 QuerySets are Lazy

```python
# QuerySets do NOT hit the database until evaluated
qs = Device.objects.filter(status="active")          # No DB hit
qs = qs.filter(platform__name="Viptela vEdge")        # No DB hit
qs = qs.exclude(customer_number="")                   # No DB hit

# DB hit happens on first evaluation:
for device in qs:            # DB hit — iteration
    print(device.name)

list(qs)                     # DB hit — forced list conversion
count = qs.count()           # DB hit — COUNT(*) query
first = qs.first()           # DB hit — LIMIT 1 query
exists = qs.exists()         # DB hit — most efficient existence check

# QuerySets are re-usable but NOT cached by default
for d in qs: print(d.name)  # DB hit #1
for d in qs: print(d.make)  # DB hit #2 — same query re-runs

# Cache by converting to list first
devices = list(qs)           # DB hit once
for d in devices: print(d.name)  # No DB hit
for d in devices: print(d.make)  # No DB hit
```

---

## 4.2 The N+1 Problem — The Most Critical Performance Issue

```python
# SCENARIO: Print each device's site name
devices = Device.objects.all()   # Query 1: SELECT * FROM devices

for device in devices:
    print(device.site.name)      # Query 2, 3, 4, 5... per device
                                  # 500 devices = 501 total queries!

# This is the N+1 problem:
# 1 query to get devices + N queries to get related data = N+1 queries

# ── FIX: select_related — for ForeignKey and OneToOne ──────────────────────
# Performs a SQL JOIN — fetches everything in ONE query
devices = Device.objects.select_related("site", "platform").all()

for device in devices:
    print(device.site.name)      # No extra query — already joined
    print(device.platform.name)  # No extra query — already joined

# ── FIX: prefetch_related — for reverse FK and ManyToMany ──────────────────
# Performs a second query and caches results in Python
devices = Device.objects.prefetch_related("interfaces").all()

for device in devices:
    for iface in device.interfaces.all():   # No extra query per device
        print(iface.name, iface.ip)

# ── COMBINE BOTH ────────────────────────────────────────────────────────────
devices = (
    Device.objects
    .select_related("site", "platform")      # JOIN for FK fields
    .prefetch_related("interfaces")           # Prefetch for reverse FK
    .filter(status="active")
)
# Total: 2 queries regardless of how many devices exist
```

---

## 4.3 Q Objects — OR Conditions and Complex Filters

```python
from django.db.models import Q

# Simple AND (default — same as passing multiple kwargs)
Device.objects.filter(status="active", make="Cisco")

# OR — must use Q objects
Device.objects.filter(Q(status="active") | Q(status="planned"))

# NOT — use ~Q
Device.objects.filter(~Q(status="failed"))

# Complex combination
Device.objects.filter(
    Q(platform__name="Viptela vEdge") | Q(platform__name="Viptela cEdge"),
    ~Q(status="offline"),
    customer_number="304999",    # AND with kwargs still works
)

# Nautobot Job real example — query BFD/TLOC-affected devices
from nautobot.dcim.models import Device

affected = Device.objects.filter(
    Q(platform__name="Viptela vEdge") | Q(platform__name="Viptela cEdge"),
    tenant__name="deapega",
    status__name="Active",
).select_related("primary_ip4", "tenant", "platform")
```

---

## 4.4 values() and values_list() — When You Don't Need Full Objects

```python
# Full objects — fetches ALL columns
devices = Device.objects.filter(status="active")
for d in devices:
    print(d.name)    # Works, but fetched make, model, created_at etc. too

# values() — returns dicts, fetches only specified columns
devices = Device.objects.filter(status="active").values("name", "management_ip")
# [{"name": "vedge-01", "management_ip": "192.168.31.21"}, ...]

# values_list() — returns tuples
Device.objects.values_list("name", flat=True)
# ["vedge-01", "vedge-02", "vedge-03"]   — flat list of names

# Use case — build an inventory dict for Ansible
inventory = {
    d["name"]: d["management_ip"]
    for d in Device.objects.filter(customer_number="304999")
                            .values("name", "management_ip")
}
# {"deapega-vedge-cpe2": "192.168.31.21", ...}
```

---

## 4.5 Aggregations and Annotations

```python
from django.db.models import Count, Max, Min, Avg, Sum

# Count total devices per platform
Device.objects.values("platform__name").annotate(count=Count("id"))
# [{"platform__name": "Viptela vEdge", "count": 12}, ...]

# Count interfaces per device — annotate adds computed column
devices = Device.objects.annotate(interface_count=Count("interfaces"))
for d in devices:
    print(d.name, d.interface_count)

# Aggregate across all rows
from django.db.models import Count
total = Device.objects.filter(status="active").aggregate(total=Count("id"))
print(total["total"])   # 47

# Last update time per site
Device.objects.values("site__name").annotate(last_updated=Max("updated_at"))
```

---

## 4.6 Custom Managers — Reusable Query Logic

```python
# devices/models.py

class DeviceQuerySet(models.QuerySet):
    """Reusable queryset methods for Device model."""

    def active(self):
        return self.filter(status="active")

    def for_customer(self, customer_number):
        return self.filter(customer_number=customer_number)

    def vedge(self):
        return self.filter(platform__name="Viptela vEdge")

    def cedge(self):
        return self.filter(platform__name="Viptela cEdge")

    def with_relations(self):
        """Always use this when traversing site/platform/interfaces."""
        return self.select_related("site", "platform").prefetch_related("interfaces")


class DeviceManager(models.Manager):
    def get_queryset(self):
        return DeviceQuerySet(self.model, using=self._db)

    # Proxy all custom methods to the queryset
    def active(self):         return self.get_queryset().active()
    def for_customer(self, n): return self.get_queryset().for_customer(n)
    def vedge(self):          return self.get_queryset().vedge()
    def cedge(self):          return self.get_queryset().cedge()
    def with_relations(self): return self.get_queryset().with_relations()


class Device(models.Model):
    ...
    objects = DeviceManager()   # Replace the default manager


# Usage — clean, readable, reusable
vedges = Device.objects.active().for_customer("304999").vedge().with_relations()
for d in vedges:
    print(d.name, d.management_ip, d.site.name)
```

---

## BUG — Week 4

**Bug: using len() on a queryset instead of .count()**

```python
# BAD — loads all rows into memory just to count
count = len(Device.objects.filter(status="active"))

# GOOD — single COUNT(*) SQL query
count = Device.objects.filter(status="active").count()
```

**Bug: calling .all() in a loop on a related manager**

```python
# BAD — N extra queries (N = number of devices)
devices = Device.objects.all()
for d in devices:
    for iface in d.interfaces.all():   # New query per device!
        print(iface)

# GOOD — prefetch resolves all interfaces in 2 total queries
devices = Device.objects.prefetch_related("interfaces").all()
for d in devices:
    for iface in d.interfaces.all():   # No extra query — uses prefetch cache
        print(iface)
```

---

## Practice — Week 4

1. Reproduce the N+1 problem: query 20 devices and print site names WITHOUT select_related. Run `python manage.py shell` and watch the queries using `django.db.connection.queries`
2. Fix it with `select_related`
3. Write a custom manager with methods: `active()`, `for_customer(n)`, `for_platform(name)`, `with_full_relations()`
4. Write a query using Q objects that returns all devices that are either "active" OR "planned" but NOT on platform "Viptela vEdge"

---

---

# WEEK 5 — Views, URLs, and Templates

## What you will learn
Django views are Python functions (or classes) that receive HTTP requests and return HTTP responses. URLs map request paths to views. Templates render HTML from Python data.

## Why it matters in Nautobot
Nautobot's UI pages are Django views. The `/dcim/devices/` list page, the device detail page, the Job run page — all are views registered in `nautobot/dcim/urls.py` and `nautobot/extras/urls.py`. Understanding views explains exactly how clicking a link in Nautobot's UI triggers data queries and renders a page.

---

## 5.1 Function-Based Views (FBV)

```python
# devices/views.py

import logging
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.http import JsonResponse
from .models import Device, Site

logger = logging.getLogger(__name__)


@login_required
def device_list(request):
    """List all devices with optional filtering."""
    status   = request.GET.get("status", "")      # ?status=active
    platform = request.GET.get("platform", "")    # ?platform=vedge

    qs = Device.objects.select_related("site", "platform").all()
    if status:
        qs = qs.filter(status=status)
    if platform:
        qs = qs.filter(platform__name__icontains=platform)

    context = {
        "devices": qs,
        "total": qs.count(),
        "filter_status": status,
    }
    return render(request, "devices/device_list.html", context)


@login_required
def device_detail(request, pk):
    """Single device detail page."""
    device     = get_object_or_404(Device, pk=pk)
    interfaces = device.interfaces.all().order_by("name")

    context = {
        "device":     device,
        "interfaces": interfaces,
    }
    return render(request, "devices/device_detail.html", context)


@login_required
def device_create(request):
    """Create a new device."""
    from .forms import DeviceForm

    if request.method == "POST":
        form = DeviceForm(request.POST)
        if form.is_valid():
            device = form.save()
            logger.info("Created device: %s", device.name)
            return redirect("device-detail", pk=device.pk)
    else:
        form = DeviceForm()

    return render(request, "devices/device_form.html", {"form": form})


def device_api(request, pk):
    """Quick JSON endpoint — returns device data as JSON."""
    device = get_object_or_404(Device, pk=pk)
    data = {
        "id":            device.pk,
        "name":          device.name,
        "status":        device.status,
        "management_ip": device.management_ip,
        "site":          device.site.name,
        "platform":      device.platform.name if device.platform else None,
    }
    return JsonResponse(data)
```

---

## 5.2 URL Configuration

```python
# devices/urls.py

from django.urls import path
from . import views

app_name = "devices"   # Namespace — prevents name collisions across apps

urlpatterns = [
    path("",                  views.device_list,   name="device-list"),
    path("<int:pk>/",         views.device_detail, name="device-detail"),
    path("add/",              views.device_create, name="device-create"),
    path("<int:pk>/api/",     views.device_api,    name="device-api"),
]
```

```python
# netinventory/urls.py — root URL config

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/",   admin.site.urls),
    path("devices/", include("devices.urls", namespace="devices")),
    # Nautobot does the same:
    # path("dcim/",    include("nautobot.dcim.urls")),
    # path("ipam/",    include("nautobot.ipam.urls")),
    # path("api/",     include("nautobot.api.urls")),
]
```

**URL pattern reference:**

```python
path("devices/",            view, name="device-list")      # /devices/
path("devices/<int:pk>/",   view, name="device-detail")    # /devices/42/
path("devices/<slug:slug>/", view, name="device-slug")     # /devices/vedge-01/
path("devices/<str:name>/", view, name="device-name")      # /devices/anything/
```

---

## 5.3 Templates — Django Template Language (DTL)

```html
<!-- devices/templates/devices/device_list.html -->

<!DOCTYPE html>
<html>
<head><title>Devices ({{ total }})</title></head>
<body>

<!-- Variables: {{ variable }} -->
<h1>Devices — {{ total }} total</h1>

<!-- Tags: {% tag %} -->
<!-- if / elif / else -->
{% if filter_status %}
  <p>Filtered by status: <strong>{{ filter_status }}</strong></p>
{% endif %}

<!-- for loop -->
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Status</th>
      <th>Platform</th>
      <th>Site</th>
      <th>Management IP</th>
    </tr>
  </thead>
  <tbody>
    {% for device in devices %}
    <tr>
      <td>
        <!-- url tag builds the URL by name -->
        <a href="{% url 'devices:device-detail' pk=device.pk %}">
          {{ device.name }}
        </a>
      </td>
      <td>{{ device.get_status_display }}</td>   <!-- human-readable choice -->
      <td>{{ device.platform.name|default:"—" }}</td>   <!-- filter: default if None -->
      <td>{{ device.site.name }}</td>
      <td>{{ device.management_ip|default:"—" }}</td>
    </tr>
    {% empty %}
    <!-- Shown when queryset is empty -->
    <tr><td colspan="5">No devices found.</td></tr>
    {% endfor %}
  </tbody>
</table>

</body>
</html>
```

```html
<!-- devices/templates/devices/base.html — template inheritance -->

<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}Net Inventory{% endblock %}</title>
</head>
<body>
  <nav>
    <a href="{% url 'devices:device-list' %}">Devices</a>
  </nav>

  <main>
    {% block content %}{% endblock %}   <!-- child templates fill this -->
  </main>
</body>
</html>
```

```html
<!-- devices/templates/devices/device_detail.html — extends base -->

{% extends "devices/base.html" %}

{% block title %}{{ device.name }}{% endblock %}

{% block content %}
<h1>{{ device.name }}</h1>
<dl>
  <dt>Status</dt>         <dd>{{ device.get_status_display }}</dd>
  <dt>Management IP</dt>  <dd>{{ device.management_ip|default:"Not set" }}</dd>
  <dt>Site</dt>           <dd>{{ device.site.name }}</dd>
  <dt>Platform</dt>       <dd>{{ device.platform.name|default:"Unknown" }}</dd>
  <dt>Customer</dt>       <dd>{{ device.customer_number }}</dd>
</dl>

<h2>Interfaces</h2>
<table>
  {% for iface in interfaces %}
  <tr>
    <td>{{ iface.name }}</td>
    <td>{{ iface.get_type_display }}</td>
    <td>{{ iface.ip|default:"—" }}/{{ iface.prefix_length|default:"" }}</td>
    <td>{% if iface.enabled %}Up{% else %}Down{% endif %}</td>
  </tr>
  {% endfor %}
</table>
{% endblock %}
```

---

## 5.4 Class-Based Views (CBV) — Nautobot's Preferred Style

```python
# devices/views.py — CBV equivalents

from django.views.generic import ListView, DetailView, CreateView, UpdateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from .models import Device
from .forms import DeviceForm


class DeviceListView(LoginRequiredMixin, ListView):
    """List all active devices. Equivalent to the FBV device_list above."""
    model               = Device
    template_name       = "devices/device_list.html"
    context_object_name = "devices"
    paginate_by         = 50   # auto-pagination — 50 rows per page

    def get_queryset(self):
        qs = Device.objects.select_related("site", "platform").filter(status="active")
        status = self.request.GET.get("status")
        if status:
            qs = qs.filter(status=status)
        return qs

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx["total"] = self.get_queryset().count()
        return ctx


class DeviceDetailView(LoginRequiredMixin, DetailView):
    model         = Device
    template_name = "devices/device_detail.html"

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx["interfaces"] = self.object.interfaces.all()
        return ctx


class DeviceCreateView(LoginRequiredMixin, CreateView):
    model         = Device
    form_class    = DeviceForm
    template_name = "devices/device_form.html"
    success_url   = reverse_lazy("devices:device-list")


class DeviceUpdateView(LoginRequiredMixin, UpdateView):
    model         = Device
    form_class    = DeviceForm
    template_name = "devices/device_form.html"
    success_url   = reverse_lazy("devices:device-list")
```

```python
# devices/urls.py — CBV registration (same URL patterns)
from .views import DeviceListView, DeviceDetailView, DeviceCreateView

urlpatterns = [
    path("",          DeviceListView.as_view(),   name="device-list"),
    path("<int:pk>/", DeviceDetailView.as_view(), name="device-detail"),
    path("add/",      DeviceCreateView.as_view(), name="device-create"),
]
```

---

## BUG — Week 5

**Bug: not using get_object_or_404 in views — raw get() causes 500 error**

```python
# BAD — DoesNotExist exception → 500 Internal Server Error
def device_detail(request, pk):
    device = Device.objects.get(pk=pk)   # crashes if pk doesn't exist

# GOOD — DoesNotExist → clean 404 Not Found response
from django.shortcuts import get_object_or_404
def device_detail(request, pk):
    device = get_object_or_404(Device, pk=pk)
```

**Bug: redirect loop when forgetting to check request.method in create views**

```python
# BAD — always shows empty form even after valid POST
def device_create(request):
    form = DeviceForm(request.POST)   # request.POST is empty on GET
    ...

# GOOD — differentiate GET (show form) from POST (process form)
def device_create(request):
    if request.method == "POST":
        form = DeviceForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("devices:device-list")
    else:
        form = DeviceForm()
    return render(request, "devices/device_form.html", {"form": form})
```

---

## Practice — Week 5

1. Build a device list view with filtering by `?status=` query parameter
2. Build a device detail view showing all interfaces
3. Add template inheritance — all pages extend a `base.html`
4. Redirect `/` to the device list view
5. Add pagination to the list view using Django's `Paginator`

---

---

# WEEK 6 — Django Admin: Nautobot's UI Engine

## What you will learn
Django Admin is the built-in, auto-generated CRUD interface. By registering your models, you get a full management UI instantly — no frontend code needed.

## Why it matters in Nautobot
This is not an optional Django feature. Nautobot's ENTIRE web UI — the device list page, the IP address management page, the Job run interface — is built on top of Django Admin, extended with custom views and templates. Learning Admin is learning how Nautobot's UI works at its core.

---

## 6.1 Basic Admin Registration

```python
# devices/admin.py

from django.contrib import admin
from .models import Site, Platform, Device, Interface


@admin.register(Site)
class SiteAdmin(admin.ModelAdmin):
    list_display  = ["name", "city", "country"]
    search_fields = ["name", "city"]
    ordering      = ["name"]


@admin.register(Platform)
class PlatformAdmin(admin.ModelAdmin):
    list_display  = ["name", "napalm_driver"]
    search_fields = ["name"]


@admin.register(Device)
class DeviceAdmin(admin.ModelAdmin):
    # Columns shown in the list view
    list_display = ["name", "status", "make", "model", "site", "platform",
                    "management_ip", "customer_number"]

    # Dropdowns in the right sidebar to filter by these fields
    list_filter  = ["status", "make", "site", "platform"]

    # Fields that power the search box
    search_fields = ["name", "management_ip", "customer_number"]

    # Clickable column (clicking the name opens the detail view)
    list_display_links = ["name"]

    # Default ordering
    ordering = ["name"]

    # Readonly fields — shown in detail view but not editable
    readonly_fields = ["created_at", "updated_at"]

    # Group fields in the detail form
    fieldsets = [
        ("Identity", {"fields": ["name", "status", "make", "model"]}),
        ("Network",  {"fields": ["management_ip", "platform"]}),
        ("Location", {"fields": ["site", "customer_number"]}),
        ("Metadata", {"fields": ["created_at", "updated_at"],
                      "classes": ["collapse"]}),   # Collapsible section
    ]

    # Inline editing of related objects
    inlines = []   # See below for InlineAdmin


@admin.register(Interface)
class InterfaceAdmin(admin.ModelAdmin):
    list_display  = ["device", "name", "type", "ip", "prefix_length", "enabled"]
    list_filter   = ["type", "enabled"]
    search_fields = ["device__name", "name", "ip"]
    raw_id_fields = ["device"]   # Use ID input for FK instead of dropdown (faster)
```

---

## 6.2 Inline Admin — Edit Related Objects on the Parent Page

```python
# devices/admin.py

class InterfaceInline(admin.TabularInline):
    """Shown inside the Device detail page — edit interfaces directly."""
    model      = Interface
    extra      = 1             # Number of empty rows to show
    fields     = ["name", "type", "ip", "prefix_length", "enabled"]
    show_change_link = True    # Link to full Interface edit page


@admin.register(Device)
class DeviceAdmin(admin.ModelAdmin):
    ...
    inlines = [InterfaceInline]   # Now interfaces appear inside the device form
```

---

## 6.3 Custom Admin Actions

```python
# devices/admin.py

@admin.register(Device)
class DeviceAdmin(admin.ModelAdmin):
    ...
    actions = ["mark_offline", "mark_active"]

    @admin.action(description="Mark selected devices as offline")
    def mark_offline(self, request, queryset):
        updated = queryset.update(status="offline")
        self.message_user(request, f"{updated} device(s) marked offline.")

    @admin.action(description="Mark selected devices as active")
    def mark_active(self, request, queryset):
        updated = queryset.update(status="active")
        self.message_user(request, f"{updated} device(s) marked active.")
```

---

## 6.4 Custom Admin Columns with Logic

```python
from django.utils.html import format_html

@admin.register(Device)
class DeviceAdmin(admin.ModelAdmin):
    list_display = ["name", "status_badge", "management_ip", "site"]

    def status_badge(self, obj):
        """Render status as a coloured badge."""
        colours = {
            "active":  "green",
            "planned": "blue",
            "failed":  "red",
            "offline": "grey",
        }
        colour = colours.get(obj.status, "black")
        return format_html(
            '<span style="color:{}; font-weight:bold;">{}</span>',
            colour,
            obj.get_status_display(),
        )
    status_badge.short_description = "Status"
    status_badge.admin_order_field = "status"   # Allow column sorting
```

---

## 6.5 Nautobot Admin Pattern — How Nautobot Extends It

```python
# This is (simplified) how Nautobot extends Django Admin for Device:

# nautobot/dcim/admin.py  (simplified view of Nautobot's actual code)
from django.contrib import admin
from nautobot.extras.admin import NautobotModelAdmin   # Nautobot's base class
from nautobot.dcim.models import Device

@admin.register(Device)
class DeviceAdmin(NautobotModelAdmin):     # Extends admin.ModelAdmin
    list_display = [
        "name", "tenant", "device_type", "platform",
        "primary_ip4", "primary_ip6", "status",
    ]
    list_filter  = ["status", "site", "rack", "tenant", "platform"]
    search_fields = ["name", "serial", "asset_tag"]
    fieldsets    = [...]

# NautobotModelAdmin adds:
# - Custom change history tracking
# - Soft-delete support
# - Custom permission checks
# - Job integration buttons
# All are Django Admin extensions — not a separate UI framework
```

---

## BUG — Week 6

**Bug: admin page shows all objects in a dropdown, freezing on large datasets**

```python
# BAD — on a table with 100,000 records, the FK dropdown loads all of them
class DeviceAdmin(admin.ModelAdmin):
    # No raw_id_fields — platform dropdown loads all Platform objects
    pass

# GOOD — use raw_id_fields for high-cardinality FKs
class DeviceAdmin(admin.ModelAdmin):
    raw_id_fields = ["site"]    # Shows a text field with a popup search
    # Or use autocomplete_fields (requires search_fields on the related admin)
    autocomplete_fields = ["platform"]
```

**Bug: forgetting @admin.register — model invisible in admin**

```python
# BOTH of these work — pick one style

# Style 1: decorator (preferred)
@admin.register(Device)
class DeviceAdmin(admin.ModelAdmin): ...

# Style 2: manual register (old style)
class DeviceAdmin(admin.ModelAdmin): ...
admin.site.register(Device, DeviceAdmin)

# NOT registering = the model doesn't appear in /admin/ at all
```

---

## Practice — Week 6

1. Register all four models in admin with appropriate `list_display`, `list_filter`, and `search_fields`
2. Add `InterfaceInline` to `DeviceAdmin`
3. Add a custom action to bulk-update customer number on selected devices
4. Add a `status_badge` computed column with coloured HTML output
5. Visit `/admin/` and test search, filter, bulk action, and inline editing

---

---

# WEEK 7 — Django REST Framework (DRF): Nautobot's API Layer

## What you will learn
Django REST Framework (DRF) turns your Django models into a fully-featured REST API with serialization, validation, authentication, and browsable documentation. It is the library behind every endpoint in the Nautobot API.

## Why it matters in Nautobot
Every API call your Ansible playbooks make to Nautobot — `GET /api/dcim/devices/`, `POST /api/ipam/ip-addresses/`, `PATCH /api/dcim/interfaces/{id}/` — is handled by a DRF ViewSet. Understanding DRF means you understand why Nautobot's API behaves the way it does, what the request/response structure means, and how to build custom API endpoints in a Nautobot plugin.

---

## 7.1 Installation and Setup

```bash
pip install djangorestframework
```

```python
# settings.py
INSTALLED_APPS = [
    ...
    "rest_framework",
]

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.TokenAuthentication",  # Same as Nautobot
        "rest_framework.authentication.SessionAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.LimitOffsetPagination",
    "PAGE_SIZE": 50,
}
```

---

## 7.2 Serializers — Translating Models to/from JSON

```python
# devices/api/serializers.py

from rest_framework import serializers
from devices.models import Site, Platform, Device, Interface


class SiteSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Site
        fields = ["id", "name", "city", "country"]


class PlatformSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Platform
        fields = ["id", "name", "napalm_driver"]


class InterfaceSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Interface
        fields = ["id", "name", "type", "ip", "prefix_length", "enabled", "description"]


class DeviceSerializer(serializers.ModelSerializer):
    """Nested serializer — mirrors Nautobot's device serializer structure."""

    # Nested objects (read only — shows full object in GET response)
    site     = SiteSerializer(read_only=True)
    platform = PlatformSerializer(read_only=True)
    interfaces = InterfaceSerializer(many=True, read_only=True)

    # Writable FK fields — accept ID for POST/PATCH
    site_id     = serializers.PrimaryKeyRelatedField(
        queryset=Site.objects.all(), source="site", write_only=True
    )
    platform_id = serializers.PrimaryKeyRelatedField(
        queryset=Platform.objects.all(), source="platform",
        write_only=True, required=False, allow_null=True
    )

    class Meta:
        model  = Device
        fields = [
            "id", "name", "status", "make", "model",
            "management_ip", "customer_number",
            "site", "site_id",
            "platform", "platform_id",
            "interfaces",
            "created_at", "updated_at",
        ]
        read_only_fields = ["created_at", "updated_at"]

    def validate_management_ip(self, value):
        """Custom field validation."""
        if value and Device.objects.filter(management_ip=value).exclude(
            pk=self.instance.pk if self.instance else None
        ).exists():
            raise serializers.ValidationError("This IP is already assigned to another device.")
        return value
```

---

## 7.3 ViewSets — The API Handler

```python
# devices/api/views.py

from rest_framework import viewsets, filters, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend   # pip install django-filter
from devices.models import Device, Interface
from .serializers import DeviceSerializer, InterfaceSerializer


class DeviceViewSet(viewsets.ModelViewSet):
    """
    Full CRUD API for devices.
    ModelViewSet provides: list, retrieve, create, update, partial_update, destroy
    """
    queryset         = Device.objects.select_related("site", "platform").prefetch_related("interfaces")
    serializer_class = DeviceSerializer

    # Filtering
    filter_backends  = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ["status", "make", "customer_number"]  # ?status=active
    search_fields    = ["name", "management_ip"]              # ?search=vedge
    ordering_fields  = ["name", "created_at"]                 # ?ordering=-created_at

    def get_queryset(self):
        """Apply customer_number filter from query param."""
        qs = super().get_queryset()
        customer = self.request.query_params.get("customer_number")
        if customer:
            qs = qs.filter(customer_number=customer)
        return qs

    @action(detail=True, methods=["post"], url_path="mark-offline")
    def mark_offline(self, request, pk=None):
        """Custom action: POST /api/devices/{pk}/mark-offline/"""
        device = self.get_object()
        device.status = "offline"
        device.save(update_fields=["status", "updated_at"])
        return Response({"status": "marked offline"}, status=status.HTTP_200_OK)

    @action(detail=False, methods=["get"], url_path="active-vedges")
    def active_vedges(self, request):
        """Custom action: GET /api/devices/active-vedges/"""
        qs = self.get_queryset().filter(
            status="active",
            platform__name="Viptela vEdge",
        )
        serializer = self.get_serializer(qs, many=True)
        return Response({"count": qs.count(), "results": serializer.data})
```

---

## 7.4 URL Registration with Router

```python
# devices/api/urls.py

from rest_framework.routers import DefaultRouter
from .views import DeviceViewSet

router = DefaultRouter()
router.register("devices", DeviceViewSet, basename="device")

urlpatterns = router.urls

# This auto-generates:
# GET    /api/devices/               → list
# POST   /api/devices/               → create
# GET    /api/devices/{pk}/          → retrieve
# PUT    /api/devices/{pk}/          → update
# PATCH  /api/devices/{pk}/          → partial_update
# DELETE /api/devices/{pk}/          → destroy
# POST   /api/devices/{pk}/mark-offline/  → custom action
# GET    /api/devices/active-vedges/      → custom action
```

```python
# netinventory/urls.py
urlpatterns = [
    path("admin/",       admin.site.urls),
    path("devices/",     include("devices.urls")),
    path("api/",         include("devices.api.urls")),   # REST API
    path("api-auth/",    include("rest_framework.urls")), # Login for browsable API
]
```

---

## 7.5 Token Authentication — Same as Nautobot

```python
# settings.py — add to INSTALLED_APPS
"rest_framework.authtoken",

# After migrate, create token for a user:
from rest_framework.authtoken.models import Token
from django.contrib.auth.models import User

user  = User.objects.get(username="admin")
token = Token.objects.create(user=user)
print(token.key)   # Use this in Authorization: Token <key>
```

```bash
# Test your API
curl -H "Authorization: Token abc123xyz" \
     http://127.0.0.1:8000/api/devices/?status=active

# Same pattern as Nautobot
curl -H "Authorization: Token abc123xyz" \
     https://nautobot.example.com/api/dcim/devices/?tenant=deapega
```

---

## 7.6 Nautobot API Parallel

```python
# Nautobot's ViewSet (simplified) — identical pattern to what you built
# nautobot/dcim/api/views.py

class DeviceViewSet(NautobotModelViewSet):          # extends DRF ModelViewSet
    queryset         = Device.objects.all()
    serializer_class = DeviceSerializer
    filterset_class  = DeviceFilterSet              # django-filter FilterSet

# NautobotModelViewSet adds:
# - Nautobot's permission model
# - Custom bulk operations
# - Changelog recording
# All built on top of standard DRF

# Nautobot's response structure:
# {"count": 4, "next": null, "previous": null, "results": [...]}
# This is LimitOffsetPagination from DRF — the same class you configured above
```

---

## BUG — Week 7

**Bug: serializer returns nested object but CREATE requires ID — confusing for callers**

```python
# Problem: GET returns full nested site object
# {"id": 1, "name": "r1", "site": {"id": 5, "name": "IAD-DC1"}}

# But POST requires site_id:
# {"name": "r2", "site_id": 5}  ← different field name!

# Solution: use write_only site_id + read_only site (as shown in 7.2)
# Or use SlugRelatedField for human-readable references:
site = serializers.SlugRelatedField(
    queryset=Site.objects.all(),
    slug_field="name"    # POST with {"site": "IAD-DC1"} — name instead of ID
)
```

**Bug: not using update_fields on save() — updates ALL columns unnecessarily**

```python
# BAD — rewrites every column on every save
device.status = "offline"
device.save()   # SQL: UPDATE devices SET name=..., make=..., model=..., status=..., ...

# GOOD — only update what changed
device.status = "offline"
device.save(update_fields=["status", "updated_at"])   # SQL: UPDATE devices SET status=..., updated_at=...
```

---

## Practice — Week 7

1. Build the full DRF API for Device (list, retrieve, create, update, delete)
2. Add a custom action `POST /api/devices/{pk}/bounce/` that sets status=offline then status=active and returns both timestamps
3. Add token authentication and test all endpoints with curl
4. Verify the response structure matches Nautobot's format: `{"count": N, "results": [...]}`

---

---

# WEEK 8 — Nautobot Jobs: Django in Production

## What you will learn
Nautobot Jobs are Python scripts that run inside the Nautobot platform — with a Django ORM session, logging, async execution, and an auto-generated UI form. This week you write production-grade Jobs with everything you have learned.

## Why it matters in Nautobot
This is the direct deliverable. Everything in Weeks 1–7 feeds into writing Jobs that are efficient, correct, safe, and debuggable in production.

---

## 8.1 Job Architecture

```
Nautobot Web UI (Job Run button)
         │  POST /extras/jobs/{job}/run/
         ▼
Django View (extras.views.JobRunView)
         │  Validates input form
         ▼
Celery Worker (async task queue)
         │  Runs Job.run() in background
         ▼
Your Job.run() method
         │  Django ORM queries
         │  Netmiko / REST API calls
         │  self.logger output
         ▼
JobResult object saved to DB
         │
         ▼
Nautobot UI shows live logs + final status
```

---

## 8.2 Basic Nautobot Job

```python
# jobs/vedge_triage.py

import logging
from nautobot.apps.jobs import Job, ObjectVar, StringVar, MultiObjectVar
from nautobot.dcim.models import Device

logger = logging.getLogger(__name__)


class VEdgeTriage(Job):
    """
    Basic triage job for Viptela vEdge devices.
    Demonstrates all core Job patterns.
    """

    # ── Input Variables (auto-rendered as form fields in the UI) ─────────────
    customer_number = StringVar(
        description="Customer number (e.g. 304999)",
        required=True,
    )
    device = ObjectVar(
        model=Device,
        description="Specific device to triage (optional)",
        required=False,
    )

    class Meta:
        name        = "vEdge Basic Triage"
        description = "Run basic health checks on vEdge devices for a customer"
        commit_default = False   # Require explicit commit checkbox in UI

    def run(self, customer_number, device=None):
        """Entry point — called by Nautobot when the Job runs."""
        self.logger.info("Starting vEdge triage for customer %s", customer_number)

        # ── Query using Django ORM ───────────────────────────────────────────
        qs = (
            Device.objects
            .filter(
                platform__name="Viptela vEdge",
                tenant__name=customer_number,
                status__name="Active",
            )
            .select_related("primary_ip4", "platform", "site", "tenant")
        )

        if device:
            qs = qs.filter(pk=device.pk)

        if not qs.exists():
            self.logger.warning("No active vEdge devices found for customer %s", customer_number)
            return

        self.logger.info("Found %d device(s) to triage", qs.count())

        results = []
        for dev in qs:
            result = self._triage_device(dev)
            results.append(result)

        # Log summary
        healthy = sum(1 for r in results if r["status"] == "healthy")
        self.logger.info(
            "Triage complete — %d/%d devices healthy", healthy, len(results)
        )

    def _triage_device(self, device):
        """Run triage checks on a single device."""
        self.logger.info("Triaging device: %s (%s)", device.name, device.primary_ip4)

        if not device.primary_ip4:
            self.logger.error("Device %s has no primary IP — cannot connect", device.name)
            return {"device": device.name, "status": "error", "reason": "no_primary_ip"}

        try:
            result = self._check_device_reachability(device)
            self.logger.info("Device %s is reachable", device.name)
            return {"device": device.name, "status": "healthy"}

        except Exception as exc:
            self.logger.error("Device %s triage failed: %s", device.name, exc)
            return {"device": device.name, "status": "error", "reason": str(exc)}

    def _check_device_reachability(self, device):
        """Placeholder — replace with real Netmiko/NAPALM logic."""
        ip = str(device.primary_ip4.address).split("/")[0]
        # In real code: connect with Netmiko, run 'show system status', parse output
        return {"reachable": True, "ip": ip}
```

---

## 8.3 Production Job Patterns

```python
# jobs/interface_triage.py

from nautobot.apps.jobs import Job, ObjectVar
from nautobot.dcim.models import Device, Interface
from nautobot.ipam.models import IPAddress
from django.db import transaction
import netmiko
import re


class InterfaceTriage(Job):
    """
    Production-grade Job — demonstrates all patterns used in iAutomate packages.
    """

    device = ObjectVar(model=Device, description="Device to triage")

    class Meta:
        name           = "Interface Triage"
        description    = "Detect flapping interfaces and classify LAN/WAN/Cellular"
        commit_default = False

    def run(self, device):
        self.logger.info("Starting interface triage for %s", device.name)

        # 1. Validate device has IP
        if not device.primary_ip4:
            raise ValueError(f"Device {device.name} has no primary IP — cannot proceed")

        mgmt_ip = str(device.primary_ip4.address).split("/")[0]

        # 2. Connect and run commands
        output = self._run_commands(mgmt_ip, device)

        # 3. Parse and classify interfaces
        interfaces = self._parse_interfaces(output)
        flapping   = self._detect_flapping(interfaces)

        # 4. Update Nautobot DB (only if commit=True)
        if self.commit:
            self._update_nautobot(device, interfaces)

        # 5. Log results
        if flapping:
            self.logger.warning("Flapping interfaces detected: %s",
                                ", ".join(i["name"] for i in flapping))
        else:
            self.logger.info("No flapping detected — all interfaces stable")

    def _run_commands(self, ip, device):
        """Connect via Netmiko and collect show output."""
        device_type = "cisco_ios" if "cEdge" in device.platform.name else "cisco_viptela"

        connection_params = {
            "device_type": device_type,
            "host":        ip,
            "username":    "admin",   # In production: from Nautobot secrets
            "password":    "secret",  # In production: from Nautobot secrets
        }

        try:
            with netmiko.ConnectHandler(**connection_params) as conn:
                return {
                    "interfaces": conn.send_command("show interfaces"),
                    "log":        conn.send_command("show log last 100"),
                }
        except netmiko.NetmikoTimeoutException as exc:
            self.logger.error("Connection timeout to %s: %s", ip, exc)
            raise
        except netmiko.NetmikoAuthenticationException as exc:
            self.logger.error("Auth failed for %s: %s", ip, exc)
            raise

    def _parse_interfaces(self, output):
        """Parse interface status from CLI output."""
        # Simplified — real parsing uses TextFSM or regex
        interfaces = []
        for line in output["interfaces"].splitlines():
            match = re.match(r"^(\S+) is (\w+), line protocol is (\w+)", line)
            if match:
                interfaces.append({
                    "name":          match.group(1),
                    "admin_status":  match.group(2),
                    "line_protocol": match.group(3),
                })
        return interfaces

    def _detect_flapping(self, interfaces):
        """Identify interfaces with protocol down."""
        return [i for i in interfaces if i["line_protocol"] == "down"
                and i["admin_status"] == "up"]

    @transaction.atomic
    def _update_nautobot(self, device, interfaces):
        """Update interface states in Nautobot — wrapped in a transaction."""
        for iface_data in interfaces:
            try:
                iface = Interface.objects.get(device=device, name=iface_data["name"])
                enabled = iface_data["admin_status"] == "up"
                if iface.enabled != enabled:
                    iface.enabled = enabled
                    iface.save(update_fields=["enabled"])
                    self.logger.info("Updated %s.%s → enabled=%s",
                                     device.name, iface_data["name"], enabled)
            except Interface.DoesNotExist:
                self.logger.debug("Interface %s not in Nautobot — skipping",
                                  iface_data["name"])
```

---

## 8.4 Reading from Nautobot Secrets

```python
# Production Jobs should NEVER hardcode credentials

from nautobot.extras.models import SecretsGroup

def _get_credentials(self, device):
    """Retrieve device credentials from Nautobot SecretsGroup."""
    try:
        secrets_group = device.secrets_group   # FK to SecretsGroup on Device
        username = secrets_group.get_secret_value(
            access_type="Generic",
            secret_type="Username",
        )
        password = secrets_group.get_secret_value(
            access_type="Generic",
            secret_type="Password",
        )
        return username, password
    except Exception as exc:
        self.logger.error("Failed to retrieve credentials for %s: %s", device.name, exc)
        raise
```

---

## BUG — Week 8

**Bug: running heavy Jobs synchronously — browser request times out**

```python
# BAD — synchronous Job blocks the web worker
# If it takes > 30s, the request times out and the Job keeps running orphaned

# GOOD — Nautobot Jobs always run via Celery (async)
# Never call Job().run() directly in a view
# Always use: Job.enqueue(request, data={...})
# Or use the UI / API trigger — Nautobot handles async automatically
```

**Bug: DB writes in a Job without transaction.atomic — partial writes on failure**

```python
# BAD — if the loop fails halfway, some devices are updated and some aren't
def run(self, ...):
    for device in devices:
        device.status = "active"
        device.save()   # Committed immediately — no rollback on later failure

# GOOD — wrap all writes in a transaction
from django.db import transaction

def run(self, ...):
    with transaction.atomic():
        for device in devices:
            device.status = "active"
            device.save()
    # If any save() fails, ALL are rolled back — consistent state
```

---

## Practice — Week 8

1. Write a Job that accepts `customer_number` and queries all active vEdge devices
2. For each device, log: name, primary IP, platform, site
3. Add a `dry_run` boolean variable — when True, only log; when False, update `updated_at`
4. Wrap all DB writes in `transaction.atomic()`
5. Handle `Device.DoesNotExist` and connection errors with proper `self.logger.error()`

---

---

# WEEK 9 — Advanced: Signals, Celery, Testing, and Plugin Architecture

## What you will learn
Django Signals (event hooks on model save/delete), Celery (async task queue — powers Nautobot Jobs), pytest-django (test your models, views, and Jobs), and how Nautobot's plugin system works.

## Why it matters in Nautobot
Signals are how Nautobot records change history (every time a Device is saved, a Signal fires and logs the diff to ChangeLog). Celery is how Nautobot runs Jobs asynchronously. Plugins are how you extend Nautobot with your own models, views, and Jobs — the endpoint of this learning path.

---

## 9.1 Django Signals — Event Hooks

```python
# devices/signals.py

import logging
from django.db.models.signals import post_save, pre_delete, post_delete
from django.dispatch import receiver
from .models import Device

logger = logging.getLogger(__name__)


@receiver(post_save, sender=Device)
def device_saved(sender, instance, created, **kwargs):
    """
    Fires every time a Device is saved.
    Nautobot uses this pattern to record ChangeLog entries.
    """
    action = "created" if created else "updated"
    logger.info("Device %s: %s (status=%s)", instance.name, action, instance.status)

    if not created and instance.status == "failed":
        # Example: trigger an alert when a device moves to 'failed'
        notify_noc(instance)


@receiver(pre_delete, sender=Device)
def device_pre_delete(sender, instance, **kwargs):
    """Fires before a Device is deleted — last chance to archive data."""
    logger.warning("Device %s is about to be deleted", instance.name)
    archive_device_data(instance)


def notify_noc(device):
    """Send an alert to NOC — placeholder."""
    logger.info("NOC alert: device %s is FAILED", device.name)


def archive_device_data(device):
    """Archive before delete — placeholder."""
    logger.info("Archiving data for device %s", device.name)
```

```python
# devices/apps.py — connect signals on app startup

from django.apps import AppConfig

class DevicesConfig(AppConfig):
    name = "devices"

    def ready(self):
        import devices.signals   # Importing connects all @receiver decorators
```

```python
# devices/__init__.py — tell Django to use the AppConfig
default_app_config = "devices.apps.DevicesConfig"
```

**How Nautobot uses Signals:**

```python
# Nautobot's ChangeLogging middleware hooks into post_save signals
# on every Nautobot model to record: who changed what, when, and what the diff was.
# This is why every object in Nautobot has a "Change Log" tab.
# It's all Django Signals under the hood.
```

---

## 9.2 Celery — Async Task Queue (Powers Nautobot Jobs)

```bash
pip install celery redis
```

```python
# netinventory/celery.py

import os
from celery import Celery

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "netinventory.settings")

app = Celery("netinventory")
app.config_from_object("django.conf:settings", namespace="CELERY")
app.autodiscover_tasks()
```

```python
# settings.py
CELERY_BROKER_URL  = "redis://localhost:6379/0"
CELERY_RESULT_BACKEND = "redis://localhost:6379/0"
```

```python
# devices/tasks.py

from celery import shared_task
import logging

logger = logging.getLogger(__name__)


@shared_task(bind=True, max_retries=3)
def triage_device_async(self, device_pk):
    """
    Async task — runs in Celery worker, not in the web request.
    This is exactly how Nautobot Jobs work internally.
    """
    from .models import Device

    try:
        device = Device.objects.select_related("site", "platform").get(pk=device_pk)
        logger.info("Triaging device: %s", device.name)
        # ... do work
        return {"status": "complete", "device": device.name}

    except Device.DoesNotExist:
        logger.error("Device %s not found", device_pk)
        return {"status": "error", "reason": "device_not_found"}

    except Exception as exc:
        logger.error("Task failed for device %s: %s", device_pk, exc)
        raise self.retry(exc=exc, countdown=60)   # Retry in 60s


# Enqueue from a view
def device_triage_view(request, pk):
    triage_device_async.delay(pk)   # Fire and forget — returns immediately
    return redirect("devices:device-list")
```

```bash
# Start Celery worker in a separate terminal
celery -A netinventory worker --loglevel=info
```

---

## 9.3 Testing with pytest-django

```bash
pip install pytest pytest-django factory-boy
```

```ini
# pytest.ini
[pytest]
DJANGO_SETTINGS_MODULE = netinventory.settings
```

```python
# devices/tests/test_models.py

import pytest
from devices.models import Site, Platform, Device, Interface


@pytest.fixture
def site(db):
    return Site.objects.create(name="IAD-DC1", city="Ashburn", country="US")


@pytest.fixture
def platform(db):
    return Platform.objects.create(name="Viptela vEdge")


@pytest.fixture
def device(db, site, platform):
    return Device.objects.create(
        name="deapega-vedge-cpe2",
        status="active",
        make="Cisco",
        model="vEdge 100",
        site=site,
        platform=platform,
        management_ip="192.168.31.21",
        customer_number="304999",
    )


class TestDeviceModel:

    def test_str_returns_name(self, device):
        assert str(device) == "deapega-vedge-cpe2"

    def test_default_status_is_active(self, site, platform, db):
        d = Device.objects.create(
            name="test-device", make="Cisco", model="vEdge 100",
            site=site, platform=platform,
        )
        assert d.status == "active"

    def test_filter_by_platform(self, device, site, db):
        Platform.objects.create(name="IOS-XE")
        Device.objects.create(
            name="ios-router", make="Cisco", model="ISR 4431",
            site=site, platform=Platform.objects.get(name="IOS-XE"),
        )
        vedges = Device.objects.filter(platform__name="Viptela vEdge")
        assert vedges.count() == 1
        assert vedges.first().name == "deapega-vedge-cpe2"

    def test_management_ip_uniqueness(self, device, site, platform, db):
        """Two devices cannot share a management IP — enforce at model level."""
        from django.db import IntegrityError
        # Depends on your model having unique=True on management_ip
        # If not, write the validation in a clean() method instead


class TestDeviceAPI:
    """Test the REST API endpoints."""

    def test_list_devices_requires_auth(self, client):
        resp = client.get("/api/devices/")
        assert resp.status_code == 401

    def test_list_devices_authenticated(self, client, device, db):
        from django.contrib.auth.models import User
        from rest_framework.authtoken.models import Token

        user  = User.objects.create_user("testuser", password="pass")
        token = Token.objects.create(user=user)

        resp = client.get(
            "/api/devices/",
            HTTP_AUTHORIZATION=f"Token {token.key}",
        )
        assert resp.status_code == 200
        data = resp.json()
        assert data["count"] == 1
        assert data["results"][0]["name"] == "deapega-vedge-cpe2"

    def test_create_device(self, client, site, platform, db):
        from django.contrib.auth.models import User
        from rest_framework.authtoken.models import Token

        user  = User.objects.create_superuser("admin", password="pass")
        token = Token.objects.create(user=user)

        payload = {
            "name":          "new-vedge-01",
            "status":        "active",
            "make":          "Cisco",
            "model":         "vEdge 100",
            "site_id":       site.pk,
            "platform_id":   platform.pk,
            "management_ip": "10.100.1.1",
        }
        resp = client.post(
            "/api/devices/",
            data=payload,
            content_type="application/json",
            HTTP_AUTHORIZATION=f"Token {token.key}",
        )
        assert resp.status_code == 201
        assert Device.objects.filter(name="new-vedge-01").exists()
```

---

## 9.4 Nautobot Plugin Architecture

```python
# A Nautobot plugin IS a Django app registered inside Nautobot
# Structure mirrors everything you built in Weeks 2–8

my_nautobot_plugin/
├── __init__.py          # Plugin metadata
├── models.py            # Custom network models (extends Nautobot base models)
├── views.py             # Custom UI views
├── urls.py              # URL patterns
├── admin.py             # Admin registration
├── api/
│   ├── serializers.py   # DRF serializers
│   ├── views.py         # DRF ViewSets
│   └── urls.py          # DRF routers
└── jobs.py              # Nautobot Jobs
```

```python
# my_nautobot_plugin/__init__.py

from nautobot.apps import NautobotAppConfig

class MyPluginConfig(NautobotAppConfig):
    name         = "my_nautobot_plugin"
    verbose_name = "My Network Automation Plugin"
    description  = "Custom jobs and models for our SD-WAN environment"
    version      = "1.0.0"
    author       = "Aadarsh"
    base_url     = "my-plugin"   # URLs mounted at /plugins/my-plugin/

config = MyPluginConfig
```

```python
# my_nautobot_plugin/models.py — custom model extending Nautobot

from django.db import models
from nautobot.dcim.models import Device
from nautobot.core.models import PrimaryModel   # Nautobot's base model


class SDWANOverlay(PrimaryModel):
    """
    Custom model tracking SD-WAN overlay relationships.
    PrimaryModel gives you: UUID pk, created, last_updated, tags, custom_fields.
    """

    vmanage_ip  = models.GenericIPAddressField()
    customer_id = models.CharField(max_length=20)
    devices     = models.ManyToManyField(Device, related_name="sdwan_overlays",
                                          blank=True)

    class Meta:
        verbose_name = "SD-WAN Overlay"

    def __str__(self):
        return f"Overlay {self.customer_id} ({self.vmanage_ip})"
```

```python
# nautobot_config.py — register your plugin with Nautobot
PLUGINS = ["my_nautobot_plugin"]

PLUGINS_CONFIG = {
    "my_nautobot_plugin": {
        "vmanage_default_port": 8443,
    }
}
```

---

## 9.5 Full Stack Summary — Django to Nautobot Mapping

```
What you built (Weeks 1–9)      →    Nautobot equivalent
─────────────────────────────────────────────────────────────────────────────
Device model (models.py)         →    nautobot.dcim.models.Device
Site, Platform models            →    nautobot.dcim.models.Site, Platform
Interface model                  →    nautobot.dcim.models.Interface
DeviceAdmin (admin.py)           →    nautobot.dcim.admin.DeviceAdmin
Device list/detail views         →    nautobot.dcim.views.DeviceListView/Detail
DeviceSerializer (DRF)           →    nautobot.dcim.api.serializers.DeviceSerializer
DeviceViewSet (DRF)              →    nautobot.dcim.api.views.DeviceViewSet
Router → /api/devices/           →    /api/dcim/devices/
VEdgeTriage(Job)                 →    Any Nautobot Job in extras/jobs.py
@receiver(post_save, Device)     →    Nautobot's ChangeLog signal hooks
Celery task + Redis broker       →    Nautobot's Job execution engine
pytest-django tests              →    nautobot-server test
my_nautobot_plugin (AppConfig)   →    Any installed Nautobot plugin
─────────────────────────────────────────────────────────────────────────────
```

---

## BUG — Week 9

**Bug: Signal fires in tests — causes unexpected DB writes or external calls**

```python
# Signals fire during tests by default — can cause:
# - ChangeLog writes cluttering test DB
# - External HTTP calls in signal handlers

# Solution: disconnect during tests
from django.test.utils import override_settings

@pytest.mark.django_db
def test_device_save_without_signal(settings):
    from django.db.models.signals import post_save
    from devices.signals import device_saved
    from devices.models import Device

    post_save.disconnect(device_saved, sender=Device)
    # ... run your test
    post_save.connect(device_saved, sender=Device)  # Reconnect after
```

**Bug: Celery tasks accessing stale Django ORM objects — race condition**

```python
# BAD — pass full object to Celery (can't be serialized correctly, and stale by run time)
triage_device_async.delay(device)   # Will fail — objects can't be serialized to JSON

# GOOD — always pass the primary key, re-fetch inside the task
triage_device_async.delay(device.pk)
# Inside task: device = Device.objects.get(pk=device_pk)  ← fresh from DB
```

---

## Practice — Week 9

1. Add a `post_save` signal on Device that logs a warning whenever status changes to "failed"
2. Write a Celery task that accepts a list of device PKs and runs triage on each
3. Write a `pytest-django` test suite covering:
   - Device model creation and `__str__`
   - ORM filter by platform
   - API list endpoint (auth required)
   - API create endpoint (validates management_ip uniqueness)
4. Structure your code as a Nautobot plugin with `NautobotAppConfig`

---

---

# Appendix A — Quick Reference: Django ORM Cheatsheet

```python
from devices.models import Device

# ── Create ───────────────────────────────────────────────────────────────────
Device.objects.create(name="r1", status="active", ...)
d = Device(name="r1"); d.save()
obj, created = Device.objects.get_or_create(name="r1", defaults={...})

# ── Read ─────────────────────────────────────────────────────────────────────
Device.objects.all()
Device.objects.filter(status="active")
Device.objects.filter(platform__name="Viptela vEdge")   # JOIN
Device.objects.exclude(status="failed")
Device.objects.get(pk=1)                                # Single — raises if missing
Device.objects.get_or_create(name="r1", defaults={...})
Device.objects.first()
Device.objects.last()
Device.objects.count()
Device.objects.exists()
Device.objects.none()                                   # Empty queryset

# ── Filter lookups ────────────────────────────────────────────────────────────
filter(name__exact="r1")        # exact match (default)
filter(name__iexact="R1")       # case-insensitive exact
filter(name__contains="edge")   # LIKE '%edge%'
filter(name__icontains="EDGE")  # case-insensitive LIKE
filter(name__startswith="vedge")
filter(name__in=["r1", "r2"])   # IN (...)
filter(pk__in=[1, 2, 3])
filter(created_at__gte="2025-01-01")  # >=
filter(created_at__lt="2026-01-01")   # <
filter(management_ip__isnull=True)    # IS NULL
filter(management_ip__isnull=False)   # IS NOT NULL

# ── Q objects ─────────────────────────────────────────────────────────────────
from django.db.models import Q
filter(Q(status="active") | Q(status="planned"))     # OR
filter(~Q(status="failed"))                          # NOT

# ── Update ───────────────────────────────────────────────────────────────────
Device.objects.filter(status="planned").update(status="active")  # Bulk SQL UPDATE
d.status = "active"; d.save()                        # Single object
d.save(update_fields=["status", "updated_at"])        # Only update these columns

# ── Delete ───────────────────────────────────────────────────────────────────
Device.objects.filter(status="failed").delete()      # Bulk delete
d.delete()                                            # Single object

# ── Optimization ─────────────────────────────────────────────────────────────
select_related("site", "platform")          # JOIN for FK — single query
prefetch_related("interfaces")             # Separate query, cached — for reverse FK
values("name", "management_ip")            # Return dicts instead of objects
values_list("name", flat=True)             # Return flat list of values
only("name", "status")                     # Fetch only these columns
defer("description")                       # Fetch all EXCEPT these columns

# ── Aggregation ───────────────────────────────────────────────────────────────
from django.db.models import Count, Max, Min, Avg
Device.objects.count()
Device.objects.aggregate(total=Count("id"))
Device.objects.values("platform__name").annotate(count=Count("id"))
Device.objects.annotate(iface_count=Count("interfaces"))

# ── Ordering ─────────────────────────────────────────────────────────────────
Device.objects.order_by("name")
Device.objects.order_by("-created_at")   # Descending
Device.objects.order_by("site__name", "name")
```

---

# Appendix B — Quick Reference: DRF Cheatsheet

```python
# Serializer field types
serializers.CharField(max_length=100)
serializers.IntegerField()
serializers.BooleanField()
serializers.IPAddressField()
serializers.DateTimeField(read_only=True)
serializers.SerializerMethodField()          # Computed field — add get_<name>(self, obj) method
serializers.PrimaryKeyRelatedField(queryset=Site.objects.all())
serializers.SlugRelatedField(slug_field="name", queryset=Site.objects.all())

# ViewSet method mapping
list()           → GET    /resource/
create()         → POST   /resource/
retrieve()       → GET    /resource/{pk}/
update()         → PUT    /resource/{pk}/
partial_update() → PATCH  /resource/{pk}/
destroy()        → DELETE /resource/{pk}/

# Custom actions
@action(detail=True,  methods=["post"])   # POST /resource/{pk}/action/
@action(detail=False, methods=["get"])    # GET  /resource/action/
```

---

# Appendix C — Environment Setup Checklist

```bash
# 1. Virtual environment
python3 -m venv venv && source venv/bin/activate

# 2. Install packages
pip install django djangorestframework django-filter \
            celery redis netmiko requests \
            pytest pytest-django factory-boy

# 3. Create project
django-admin startproject netinventory .
python manage.py startapp devices

# 4. Initial setup
python manage.py migrate
python manage.py createsuperuser

# 5. Run development server
python manage.py runserver

# 6. Run Celery worker (separate terminal)
celery -A netinventory worker --loglevel=info

# 7. Run tests
pytest

# 8. Django shell (your best debugging tool)
python manage.py shell
```

---

*End of Guide — Django for Network Engineers: Nautobot-Ready in 9 Weeks*
