# Railway-platform-display-board-
import tkinter as tk
import threading
from gtts import gTTS
import pygame
import os
import time
from datetime import datetime, timedelta

# ===== INITIALIZATION =====
pygame.mixer.init()
root = tk.Tk()
root.title("Railway Info System")
root.geometry("600x600")
root.configure(bg="black")  # Pure black background

# ===== TRAIN DATA =====
trains = [
    {"number": "12345", "name": "Rajdhani Express", "delayed": False,
     "scheduled_time": "12:30", "actual_time": "11:30", "platform": 2},
    {"number": "67890", "name": "Shatabdi Express", "delayed": False,
     "scheduled_time": "12:45", "actual_time": "11:45", "platform": 1},
    {"number": "11111", "name": "Tejas Express", "delayed": False,
     "scheduled_time": "12:50", "actual_time": "11:50", "platform": 3},
    {"number": "22222", "name": "Garib Rath", "delayed": True,
     "scheduled_time": "12:15", "actual_time": "11:25", "platform": 3},
    {"number": "33333", "name": "Duronto Express", "delayed": True,
     "scheduled_time": "12:10", "actual_time": "11:18", "platform": 4}
]

# ===== UTILITY FUNCTIONS =====
def speak(text):
    """ Speak text using gTTS and pygame """
    filename = f"temp_{int(time.time()*1000)}.mp3"
    tts = gTTS(text=text, lang='en')
    tts.save(filename)
    pygame.mixer.music.load(filename)
    pygame.mixer.music.play()
    while pygame.mixer.music.get_busy():
        time.sleep(0.1)
    os.remove(filename)

def play_announcements(trains_to_announce):
    """ Play announcements for both ON TIME and DELAYED """
    for train in trains_to_announce:
        if train["delayed"]:
            # Calculate delay in minutes
            sch = datetime.strptime(train["scheduled_time"], "%H:%M")
            actual = datetime.strptime(train["actual_time"], "%H:%M")
            delay_minutes = int((actual - sch).total_seconds() / 60)
            speak(f"{train['name']} is delayed by {delay_minutes} minutes.")
        else:
            speak(f"{train['name']} is on time.")

def is_train_in_next_15_minutes(train_time):
    """ Check if train arrives within next 15 minutes """
    now = datetime.now()
    train_dt = datetime.strptime(train_time, "%H:%M").replace(year=now.year, month=now.month, day=now.day)
    diff = (train_dt - now).total_seconds()
    return 0 <= diff <= 15 * 60

def auto_announcement():
    """ Check every 60 seconds for upcoming trains and announce """
    while True:
        for train in trains:
            if is_train_in_next_15_minutes(train["scheduled_time"]):
                speak(f"{train['name']} will arrive shortly at platform {train['platform']}.")
        time.sleep(60)

# ===== DISPLAY FUNCTION =====
def show_all_trains():
    """ Shows ON TIME and DELAYED trains when button clicked """
    for widget in display_frame.winfo_children():
        widget.destroy()

    on_time_trains = [t for t in trains if not t["delayed"]][:3]
    delayed_trains = [t for t in trains if t["delayed"]][:2]

    # ON TIME
    on_time_frame = tk.LabelFrame(display_frame, text="On Time Trains",
                                   font=("Helvetica", 12, "bold"),
                                   fg="green", bg="black", bd=2, relief=tk.RIDGE)
    on_time_frame.pack(pady=10, fill="x", padx=10)

    for train in on_time_trains:
        label = tk.Label(on_time_frame,
                         text=f"{train['name']} ({train['number']}) - {train['scheduled_time']} - Platform {train['platform']}",
                         font=("Courier", 10), bg="green", fg="white")
        label.pack(pady=2, padx=5, fill="x")

    # DELAYED
    delayed_frame = tk.LabelFrame(display_frame, text="Delayed Trains",
                                   font=("Helvetica", 12, "bold"),
                                   fg="red", bg="black", bd=2, relief=tk.RIDGE)
    delayed_frame.pack(pady=10, fill="x", padx=10)

    for train in delayed_trains:
        sch = datetime.strptime(train["scheduled_time"], "%H:%M")
        actual = datetime.strptime(train["actual_time"], "%H:%M")
        delay_minutes = int((actual - sch).total_seconds() / 60)

        label = tk.Label(delayed_frame,
                         text=f"{train['name']} ({train['number']}) - {train['scheduled_time']} ➔ {train['actual_time']} - Platform {train['platform']} (Delayed by {delay_minutes} mins)",
                         font=("Courier", 10), bg="red", fg="white")
        label.pack(pady=2, padx=5, fill="x")

    # SOUND ANNOUNCEMENT
    all_trains_for_announcement = on_time_trains + delayed_trains
    threading.Thread(target=play_announcements, args=(all_trains_for_announcement,), daemon=True).start()


# ===== TIME LOOP FUNCTION =====
def update_time():
    """ Update the clock every second """
    now = datetime.now()
    clock_label.config(text=now.strftime("%H:%M:%S"))
    root.after(1000, update_time)

# ===== UI ELEMENTS =====
station_label = tk.Label(root, text="New Delhi Station",
                         font=("Helvetica", 16, "bold"), bg="black", fg="yellow")
station_label.pack(pady=5)

clock_label = tk.Label(root, font=("Helvetica", 12), bg="black", fg="white")
clock_label.pack(pady=5)

show_trains_btn = tk.Button(root, text="Show Trains",
                            command=show_all_trains,
                            font=("Arial", 11), bg="blue", fg="white")
show_trains_btn.pack(pady=10)

# MAIN DISPLAY AREA
display_frame = tk.Frame(root, bg="black")
display_frame.pack(pady=10, fill="both", expand=True)

# START CLOCK
update_time()

# START AUTO ANNOUNCEMENT THREAD
threading.Thread(target=auto_announcement, daemon=True).start()

# MAIN LOOP
root.mainloop()
