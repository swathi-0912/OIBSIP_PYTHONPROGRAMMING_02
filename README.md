# OIBSIP_PYTHONPROGRAMMING_02
BMI CALCULATOR

import tkinter as tk
from tkinter import messagebox
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

# CSV File to store user data
CSV_FILE = "bmi_records.csv"

def calculate_bmi(weight, height):
    return weight / (height ** 2)

def classify_bmi(bmi):
    if bmi < 18.5:
        return "Underweight"
    elif 18.5 <= bmi < 24.9:
        return "Normal weight"
    elif 25 <= bmi < 29.9:
        return "Overweight"
    else:
        return "Obese"

def save_data(name, weight, height, bmi, category):
    df = pd.DataFrame([{
        "Date": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "Name": name,
        "Weight (kg)": weight,
        "Height (m)": height,
        "BMI": round(bmi, 2),
        "Category": category
    }])
    
    try:
        df.to_csv(CSV_FILE, mode='a', header=not pd.io.common.file_exists(CSV_FILE), index=False)
    except Exception as e:
        messagebox.showerror("File Error", f"Could not save data: {e}")

def on_calculate():
    try:
        name = entry_name.get().strip()
        weight = float(entry_weight.get())
        height = float(entry_height.get())
        
        if not name:
            messagebox.showwarning("Input Error", "Please enter your name.")
            return

        if weight <= 0 or height <= 0:
            raise ValueError

        bmi = calculate_bmi(weight, height)
        category = classify_bmi(bmi)

        result_var.set(f"BMI: {bmi:.2f} ({category})")
        save_data(name, weight, height, bmi, category)

    except ValueError:
        messagebox.showerror("Input Error", "Please enter valid numbers for weight and height.")

def show_graph():
    try:
        df = pd.read_csv(CSV_FILE)
        df["Date"] = pd.to_datetime(df["Date"])
        df.sort_values("Date", inplace=True)

        plt.figure(figsize=(8, 4))
        for name in df["Name"].unique():
            user_df = df[df["Name"] == name]
            plt.plot(user_df["Date"], user_df["BMI"], marker='o', label=name)

        plt.xlabel("Date")
        plt.ylabel("BMI")
        plt.title("BMI Trend Over Time")
        plt.legend()
        plt.grid(True)
        plt.tight_layout()
        plt.show()
    except Exception as e:
        messagebox.showerror("Graph Error", f"Could not load or plot data: {e}")

# GUI Setup
root = tk.Tk()
root.title("Advanced BMI Calculator")
root.geometry("400x400")
root.config(bg="#f0f0f0")

tk.Label(root, text="Name").pack(pady=5)
entry_name = tk.Entry(root)
entry_name.pack()

tk.Label(root, text="Weight (kg)").pack(pady=5)
entry_weight = tk.Entry(root)
entry_weight.pack()

tk.Label(root, text="Height (m)").pack(pady=5)
entry_height = tk.Entry(root)
entry_height.pack()

tk.Button(root, text="Calculate BMI", command=on_calculate, bg="#4CAF50", fg="white").pack(pady=10)

result_var = tk.StringVar()
tk.Label(root, textvariable=result_var, font=("Arial", 12, "bold")).pack(pady=10)

tk.Button(root, text="Show BMI Graph", command=show_graph, bg="#2196F3", fg="white").pack(pady=5)

root.mainloop()
