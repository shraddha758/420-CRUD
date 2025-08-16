# 420-CRUD
import tkinter as tk
from tkinter import messagebox
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["car_db"]
collection = db["cars"]

# GUI App
app = tk.Tk()
app.title("Car Management System")
app.geometry("500x500")


fields = ["Car ID", "Brand", "Model", "Year", "Price"]
entries = {}

for idx, field in enumerate(fields):
    label = tk.Label(app, text=field)
    label.grid(row=idx, column=0, padx=10, pady=5, sticky=tk.W)

    entry = tk.Entry(app, width=30)
    entry.grid(row=idx, column=1, padx=10, pady=5)
    entries[field] = entry


def clear_fields():
    for entry in entries.values():
        entry.delete(0, tk.END)

def create_car():
    data = {
        "car_id": entries["Car ID"].get(),
        "brand": entries["Brand"].get(),
        "model": entries["Model"].get(),
        "year": entries["Year"].get(),
        "price": entries["Price"].get(),
    }
    if not data["car_id"]:
        messagebox.showerror("Error", "Car ID is required.")
        return

    if collection.find_one({"car_id": data["car_id"]}):
        messagebox.showerror("Error", "Car with this ID already exists.")
        return

    collection.insert_one(data)
    messagebox.showinfo("Success", "Car added successfully.")
    clear_fields()

def read_car():
    car_id = entries["Car ID"].get()
    if not car_id:
        messagebox.showerror("Error", "Car ID is required to read.")
        return

    car = collection.find_one({"car_id": car_id})
    if car:
        entries["Brand"].delete(0, tk.END)
        entries["Brand"].insert(0, car.get("brand", ""))

        entries["Model"].delete(0, tk.END)
        entries["Model"].insert(0, car.get("model", ""))

        entries["Year"].delete(0, tk.END)
        entries["Year"].insert(0, car.get("year", ""))

        entries["Price"].delete(0, tk.END)
        entries["Price"].insert(0, car.get("price", ""))
    else:
        messagebox.showerror("Error", "Car not found.")

def update_car():
    car_id = entries["Car ID"].get()
    if not car_id:
        messagebox.showerror("Error", "Car ID is required to update.")
        return

    new_data = {
        "brand": entries["Brand"].get(),
        "model": entries["Model"].get(),
        "year": entries["Year"].get(),
        "price": entries["Price"].get(),
    }

    result = collection.update_one({"car_id": car_id}, {"$set": new_data})
    if result.matched_count:
        messagebox.showinfo("Success", "Car updated successfully.")
    else:
        messagebox.showerror("Error", "Car not found.")
    clear_fields()

def delete_car():
    car_id = entries["Car ID"].get()
    if not car_id:
        messagebox.showerror("Error", "Car ID is required to delete.")
        return

    result = collection.delete_one({"car_id": car_id})
    if result.deleted_count:
        messagebox.showinfo("Success", "Car deleted successfully.")
    else:
        messagebox.showerror("Error", "Car not found.")
    clear_fields()

# Buttons
button_frame = tk.Frame(app)
button_frame.grid(row=6, column=0, columnspan=2, pady=20)

tk.Button(button_frame, text="Create", width=10, command=create_car).grid(row=0, column=0, padx=5)
tk.Button(button_frame, text="Read", width=10, command=read_car).grid(row=0, column=1, padx=5)
tk.Button(button_frame, text="Update", width=10, command=update_car).grid(row=0, column=2, padx=5)
tk.Button(button_frame, text="Delete", width=10, command=delete_car).grid(row=0, column=3, padx=5)

# Run the GUI loop
app.mainloop()
