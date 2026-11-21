<div align="center">

# 🏥 Smart Healthcare System

**A role-based hospital management app built for a 2nd-year CSE DBMS course, with Streamlit and MySQL.**



</div>

---

## 📌 Overview

The Smart Healthcare System replaces paper hospital records with a normalized MySQL database and a web interface. Patients, doctors and hospital staff each register and log in, and see a dashboard built for their role. Every action in the app (booking, prescribing, assigning a room, restocking medicine) is backed by SQL queries with foreign keys, joins and transactions.

## ✨ Features

### 👤 Patient
- Register and log in
- Book an appointment with any available doctor
- View medical records and diagnoses, filtered by date range
- View prescriptions together with their medicines
- View lab test results

### 🩺 Doctor
- Set availability (Available / Not Available)
- View appointments by date range and status (Completed / Not Completed / Both), with an optional count
- Mark appointments as completed
- Write prescriptions: instructions, diagnosis and medicines picked from inventory. This also creates the medical record.

### 🧑‍💼 Staff
- View their shift schedule (shifts are assigned automatically at registration: Morning 09:00–17:00, Evening 17:00–01:00)
- View medicine inventory and filter by expiry date (before / on / after a date)
- Add new medicines and update stock quantities
- Assign patients to available hospital rooms and free rooms again

## 🛠️ Tech Stack

| Layer | Technology |
|:------|:-----------|
| Frontend | Streamlit |
| Backend | Python, `mysql-connector-python` |
| Database | MySQL 8 |
| Data handling | Pandas |

## 🗂️ Database Design

The database is called `healthcaresystem` and has **17 tables**.

| Group | Tables |
|:------|:-------|
| Accounts | `users` (Patient / Doctor / Staff / Nurse / Others) |
| People | `patients`, `doctors`, `staff`, `emergencycontacts`, `insurancedetails` |
| Care | `appointment`, `prescriptions`, `prescriptionmedicines`, `medicalrecords`, `labtests` |
| Pharmacy | `medicineinventory`, `suppliers` |
| Facilities | `hospitalrooms`, `roomassignments`, `shiftschedule` |
| Finance | `billing` |

### Key relationships
- Every `patient`, `doctor` and `staff` row links to a `users` account.
- `appointment` joins a `patient` and a `doctor`.
- A `prescription` belongs to a patient and a doctor. It has many medicines through `prescriptionmedicines` (a many-to-many link to `medicineinventory`) and produces a `medicalrecord`.
- `medicineinventory` rows reference a `supplier`.
- `roomassignments` link `hospitalrooms` to `patients`.
- Deleting a patient or doctor cascades to their appointments, lab tests, bills and similar records.

> `billing`, `insurancedetails` and `emergencycontacts` are in the schema with sample data, but the app doesn't use them yet.

## 📁 Project Structure

```
DBMS-Smart-Healthcare-System/
├── src/
│   ├── app.py               # Streamlit entry point (login / dashboard routing)
│   ├── ui_elements.py       # Login, registration and role dashboards
│   └── backend_queries.py   # All MySQL queries
├── Schema/
│   └── SHCS - Schema Github Export.zip   # One .sql file per table (structure + sample data)
├── LICENSE
└── README.md
```

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- MySQL Server 8.0+

### 1. Clone the repository

```bash
git clone https://github.com/madddx/DBMS-Smart-Healthcare-System.git
cd DBMS-Smart-Healthcare-System
```

### 2. Install dependencies

```bash
pip install streamlit mysql-connector-python pandas
```

### 3. Set up the database

Unzip the schema export, then create the database and import the tables in dependency order:

```bash
unzip "Schema/SHCS - Schema Github Export.zip" -d schema
mysql -u root -p -e "CREATE DATABASE healthcaresystem;"

cd "schema/SHCS - Schema Github Export"
for t in users patients doctors staff suppliers hospitalrooms medicineinventory \
         prescriptions prescriptionmedicines medicalrecords appointment labtests \
         roomassignments shiftschedule billing insurancedetails emergencycontacts; do
  mysql -u root -p healthcaresystem < healthcaresystem_$t.sql
done
cd ../..
```

### 4. Configure the connection

Open `src/backend_queries.py` and set your own MySQL credentials in `init_connection()`:

```python
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="<your-mysql-password>",
    database="healthcaresystem"
)
```

### 5. Run the app

```bash
streamlit run src/app.py
```

Open the URL Streamlit prints (usually `http://localhost:8501`). Register as a Patient, Doctor or Staff member, or log in with one of the sample accounts in the `users` table.

## ⚠️ Known Limitations

This is a course project, so a few things are not production-ready:
- Passwords are stored and compared in plain text. Hash them (for example with `bcrypt`) before any real use.
- Database credentials are hard-coded in `backend_queries.py`. Move them to environment variables or `st.secrets`.
- The Staff dashboard's **Free Room** button calls `backend_queries.free_room()`, which isn't defined yet, so that action will fail until it's added.
- `write_prescription` commits after each insert instead of using a single transaction.

## 🔮 Future Improvements

- Billing and insurance screens (the tables already exist)
- Lab test entry for doctors
- Password hashing and secrets management
- Admin and Nurse dashboards
- Appointment date/time selection and reminders

## 📄 License

Released under the [MIT License](LICENSE).
