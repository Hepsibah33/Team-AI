# Team-AI


# Submitted by
B200444
B200508
B200427
B200803
B201302


# 🐾 HapiVet AI - Smart Appointment Scheduler & Reminder System

## 1. Problem Statement
Veterinary clinics often face challenges in efficiently managing pet appointments and sending timely reminders to pet owners. Manual tracking leads to missed appointments, poor communication, and delays in treatment.  

The **HapiVet AI Scheduler** is designed to automate the appointment management process. The system allows:

- Booking new appointments for pets with selected doctors.
- Viewing all scheduled appointments in an organized manner.
- Sending automated reminders to owners for upcoming appointments.
- Capturing owner responses to either **Accept** or **Reschedule** appointments.
- Handling emergency cases effectively.  

This tool improves clinic workflow, reduces missed appointments, and ensures timely veterinary care.

---

## 2. Tools/Technologies Used
- **Programming Language:** Python  
- **Framework:** Streamlit (for web interface and interactivity)  
- **Data Handling:** Pandas (for storing and managing appointment data)  
- **Date/Time Management:** datetime, timedelta  
- **Key Features Implemented:**
  - Add and view appointments
  - Automated reminder system for next-day appointments
  - Accept/Reschedule functionality for owners
  - Emergency case management

---

## 3. Code with Team Name
**Team Name:** Team-AI 

The main application is implemented in `hapivet_demo_web.py` using Streamlit. It provides a full-featured web interface for clinic staff to manage appointments and communicate with pet owners.  

### Code :

**Booking a New Appointment:**


import streamlit as st
import pandas as pd
from datetime import datetime, timedelta

st.set_page_config(page_title="HapiVet AI Scheduler", layout="wide")
st.title("🐾 HapiVet AI - Smart Appointment Scheduler & Reminder System")

# ----------------------------- #
# INITIAL SETUP
# ----------------------------- #
if "appointments" not in st.session_state:
    st.session_state.appointments = pd.DataFrame(columns=[
        "Pet Name", "Owner Name", "Doctor", "Appointment Date",
        "Time", "Report Date", "Status", "Reminder Sent"
    ])

# ----------------------------- #
# PAGE: ADD NEW APPOINTMENT
# ----------------------------- #
st.header("📅 Book a New Appointment")

pet_name = st.text_input("Pet Name:")
owner_name = st.text_input("Owner Name:")
doctor = st.selectbox("Select Doctor", ["Dr. Sharma", "Dr. Meena", "Dr. Raghav"])
appointment_date = st.date_input("Select Appointment Date:", min_value=datetime.now().date())
appointment_time = st.time_input("Select Appointment Time:")
report_date = st.date_input("Select Report Date:", value=datetime.now().date() + timedelta(days=1))
status = st.selectbox("Status", ["Scheduled", "Emergency"])

if st.button("➕ Add Appointment"):
    if pet_name and owner_name:
        new_appt = pd.DataFrame([{
            "Pet Name": pet_name,
            "Owner Name": owner_name,
            "Doctor": doctor,
            "Appointment Date": appointment_date,
            "Time": appointment_time.strftime("%H:%M"),
            "Report Date": report_date,
            "Status": status,
            "Reminder Sent": False
        }])
        st.session_state.appointments = pd.concat(
            [st.session_state.appointments, new_appt], ignore_index=True
        )
        st.success(f"✅ Appointment booked for {pet_name} with {doctor} on {appointment_date}.")
    else:
        st.warning("⚠️ Please fill all fields before adding.")

# ----------------------------- #
# PAGE: VIEW ALL APPOINTMENTS
# ----------------------------- #
st.header("📋 Appointment Details")

if not st.session_state.appointments.empty:
    st.dataframe(st.session_state.appointments)
else:
    st.info("No appointments yet.")

# ----------------------------- #
# PAGE: REMINDER SYSTEM
# ----------------------------- #
st.header("🔔 Automated Reminder System")

today = datetime.now().date()
tomorrow = today + timedelta(days=1)

df = st.session_state.appointments.copy()
df["Appointment Date"] = pd.to_datetime(df["Appointment Date"])

# Filter for tomorrow's appointments
upcoming = df[(df["Appointment Date"].dt.date == tomorrow) & (df["Reminder Sent"] == False)]

if not upcoming.empty:
    st.subheader("📨 Reminders for Tomorrow’s Appointments")

    for i, row in upcoming.iterrows():
        with st.expander(f"🐶 {row['Pet Name']} | Owner: {row['Owner Name']} | Doctor: {row['Doctor']} | {row['Appointment Date'].date()} {row['Time']}"):
            send_reminder = st.checkbox(f"Send Reminder to {row['Owner Name']}", key=f"reminder_{i}")

            if send_reminder and not st.session_state.appointments.loc[i, "Reminder Sent"]:
                # Mark reminder as sent
                st.session_state.appointments.loc[i, "Reminder Sent"] = True
                st.info(f"📤 Reminder sent to {row['Owner Name']} for {row['Pet Name']}!")

                st.markdown("### 📅 Appointment Response Options")
                action = st.radio(
                    f"Select an option for {row['Pet Name']}:",
                    ("Accept", "Reschedule"),
                    key=f"action_{i}"
                )

                if action == "Accept":
                    st.session_state.appointments.loc[i, "Status"] = "Confirmed"
                    st.success(f"✅ {row['Owner Name']} confirmed the appointment.")

                elif action == "Reschedule":
                    new_date = st.date_input(
                        f"Select new appointment date for {row['Pet Name']}",
                        value=row["Appointment Date"].date() + timedelta(days=1),
                        key=f"newdate_{i}"
                    )
                    if st.button(f"💾 Save New Date for {row['Pet Name']}", key=f"save_{i}"):
                        st.session_state.appointments.loc[i, "Appointment Date"] = pd.to_datetime(new_date)
                        st.session_state.appointments.loc[i, "Status"] = "Rescheduled"
                        st.success(f"✅ Appointment for {row['Pet Name']} rescheduled to {new_date}.")
else:
    st.success("✅ No pending reminders for tomorrow.")

# ----------------------------- #
# PAGE: EMERGENCY MANAGEMENT
# ----------------------------- #
st.header("🚨 Emergency Case Management")

emergency_cases = st.session_state.appointments[
    st.session_state.appointments["Status"] == "Emergency"
]
if not emergency_cases.empty:
    st.warning("⚠️ Active Emergency Cases:")
    st.dataframe(emergency_cases)
else:
    st.info("No active emergency cases.")

