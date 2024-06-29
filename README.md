# python-old-project
import tkinter as tk
from tkinter import simpledialog, messagebox, filedialog
import schedule
import time
import threading
from datetime import datetime
import json
import os

class TaskScheduler:
    def __init__(self, root):
        self.root = root
        self.root.title("Task Scheduler")

        self.tasks = []

        
        self.task_listbox = tk.Listbox(root, width=70)
        self.task_listbox.pack(padx=10, pady=10)

        
        self.add_button = tk.Button(root, text="Add Task", command=self.add_task, bg="red")
        self.add_button.pack(padx=10, pady=5, ipadx=10)

        self.edit_button = tk.Button(root, text="Edit Task", command=self.edit_task, bg="orange")
        self.edit_button.pack(padx=10, pady=5, ipadx=10)

        self.delete_button = tk.Button(root, text="Delete Task", command=self.delete_task, bg="lightcoral")
        self.delete_button.pack(padx=10, pady=5, ipadx=10)

        self.save_button = tk.Button(root, text="Save Tasks", command=self.save_tasks, bg="lightblue")
        self.save_button.pack(padx=10, pady=5, ipadx=10)

        self.load_button = tk.Button(root, text="Load Tasks", command=self.load_tasks, bg="lightgreen")
        self.load_button.pack(padx=10, pady=5, ipadx=10)

        self.exit_button = tk.Button(root, text="Exit", command=self.exit_app, bg="lightgray")
        self.exit_button.pack(padx=10, pady=5, ipadx=10)

        self.load_tasks_from_file()

        self.start_scheduler()

    def add_task(self):
        task_name = simpledialog.askstring("Add Task", "Enter task name:")
        task_time = simpledialog.askstring("Add Task", "Enter task time (HH:MM):")
        task_description = simpledialog.askstring("Add Task", "Enter task description:")
        task_priority = simpledialog.askstring("Add Task", "Enter task priority (1-5):")
        recurrence = simpledialog.askstring("Add Task", "Enter recurrence (daily, weekly, none):")

        if task_name and task_time:
            try:
                datetime.strptime(task_time, "%H:%M")  # Validate time format
                task_priority = int(task_priority)
                if task_priority < 1 or task_priority > 5:
                    raise ValueError
                self.tasks.append((task_name, task_time, task_description, task_priority, recurrence))
                self.sort_and_display_tasks()
            except ValueError:
                messagebox.showerror("Invalid Input", "Time format should be HH:MM and priority should be 1-5")
             
#edit task started
    def edit_task(self):
        selection = self.task_listbox.curselection()
        if selection:
            task = self.tasks[selection[0]]
            task_name = simpledialog.askstring("Edit Task", "Enter task name:", initialvalue=task[0])
            task_time = simpledialog.askstring("Edit Task", "Enter task time (HH:MM):", initialvalue=task[1])
            task_description = simpledialog.askstring("Edit Task", "Enter task description:", initialvalue=task[2])
            task_priority = simpledialog.askstring("Edit Task", "Enter task priority (1-5):", initialvalue=task[3])
            recurrence = simpledialog.askstring("Edit Task", "Enter recurrence (daily, weekly, none):", initialvalue=task[4])

            # Edit task in the list
            if task_name and task_time:
                try:
                    datetime.strptime(task_time, "%H:%M")  # Validate time format
                    task_priority = int(task_priority)
                    if task_priority < 1 or task_priority > 5:
                        raise ValueError
                    self.tasks[selection[0]] = (task_name, task_time, task_description, task_priority, recurrence)
                    self.sort_and_display_tasks()
                except ValueError:
                    messagebox.showerror("Invalid Input", "Time format should be HH:MM and priority should be 1-5")
#edit task code ended
    #delete task started
    def delete_task(self):
        selection = self.task_listbox.curselection()
        if selection:
            self.task_listbox.delete(selection)
            del self.tasks[selection[0]]
            self.sort_and_display_tasks()

    def exit_app(self):
        self.root.destroy()

    def sort_and_display_tasks(self):
        # Sort tasks by time and priority
        self.tasks.sort(key=lambda x: (x[1], x[3]))
        # Update the listbox
        self.task_listbox.delete(0, tk.END)
        for task_name, task_time, task_description, task_priority, recurrence in self.tasks:
            self.task_listbox.insert(tk.END, f"{task_name} - {task_time} - {task_description} - Priority: {task_priority} - Recurrence: {recurrence}")

    def start_scheduler(self):
        for task_name, task_time, task_description, task_priority, recurrence in self.tasks:
            if recurrence == "daily":
                schedule.every().day.at(task_time).do(self.execute_task, task_name, task_description, task_priority)
            elif recurrence == "weekly":
                schedule.every().week.at(task_time).do(self.execute_task, task_name, task_description, task_priority)
            else:
                schedule.every().day.at(task_time).do(self.execute_task, task_name, task_description, task_priority)

        # Start the scheduler in a new thread to avoid blocking the main thread
        scheduler_thread = threading.Thread(target=self.run_scheduler, daemon=True)
        scheduler_thread.start()

    def run_scheduler(self):
        while True:
            schedule.run_pending()
            time.sleep(1)

    def execute_task(self, task_name, task_description, task_priority):
        messagebox.showinfo("Task Execution", f"Task '{task_name}' executed at {time.strftime('%Y-%m-%d %H:%M:%S')}\nDescription: {task_description}\nPriority: {task_priority}")
        # deleter task ended
    #save task started

    def save_tasks(self):
        file_path = filedialog.asksaveasfilename(defaultextension=".json", filetypes=[("JSON files", "*.json"), ("All files", "*.*")])
        if file_path:
            with open(file_path, 'w') as file:
                json.dump(self.tasks, file)
            messagebox.showinfo("Save Tasks", "Tasks saved successfully!")
#save task ended 
    #load statted
    def load_tasks(self):
        file_path = filedialog.askopenfilename(defaultextension=".json", filetypes=[("JSON files", "*.json"), ("All files", "*.*")])
        if file_path:
            with open(file_path, 'r') as file:
                self.tasks = json.load(file)
            self.sort_and_display_tasks()
            messagebox.showinfo("Load Tasks", "Tasks loaded successfully!")

    def load_tasks_from_file(self):
        if os.path.exists("tasks.json"):
            with open("tasks.json", 'r') as file:
                self.tasks = json.load(file)
            self.sort_and_display_tasks()
#load ended
if __name__ == "__main__":
    root = tk.Tk()
    app = TaskScheduler(root)
    root.protocol("WM_DELETE_WINDOW", app.exit_app)
    root.mainloop()
