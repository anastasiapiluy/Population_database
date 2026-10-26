# Population_database

## Пароль администратора: 123

### Программа
```python
from tkinter import *
import json
import hashlib
import secrets
from tkinter import messagebox

access = "user"

def give_rights():
    moder_panel.grid_forget()
    admin_panel.grid_forget()

    if access == "moderator":
        moder_panel.grid(row=4, column=0,columnspan=5, pady=10)
        admin_panel.grid(row=5, column=0, columnspan=5, pady=10)
        delete_btn.grid_forget()
        cp_btn.grid_forget()
        reset_btn.grid_forget()
        statistics()
    elif access == "admin":
        moder_panel.grid(row=4, column=0, columnspan=5, pady=10)
        admin_panel.grid(row=5, column=0, columnspan=5, pady=10)

        delete_btn.grid(row=2,column=4,padx=10,pady=10)
        reset_btn.grid(row=4, column=1,columnspan=2, padx=10, pady=10)
        cp_btn.grid(row=4, column=3, padx=10, pady=10)
        statistics()
    else:
        moder_panel.grid_forget()
        admin_panel.grid_forget()

def account():
    global access
    login = Toplevel()
    login.title("Authorization")
    login.geometry("200x200")
    login.grab_set()

    Label(login, text = "Choose an account:").grid(row=0, column=1, pady=10)

    def login_admin():
        user_btn.grid_forget()
        moder_btn.grid_forget()
        admin_btn.grid_forget()
        Label(login, text="Password:").grid(row=1, column=0, pady=10)
        entry = Entry(login, show="*")
        entry.grid(row=1, column=1, pady=10)

        def check():
            global access
            try:
                if check_password(entry.get(),"admin"):
                    access = "admin"
                    login.destroy()
                    give_rights()
                else:
                    messagebox.showerror("Error", "Invalid password")
            except FileNotFoundError:
                messagebox.showerror("Error", "File is not found")

        Button(login, text="Login", command=check).grid(row=2, column=1, pady=10)

    def login_moder():
        global access
        user_btn.grid_forget()
        moder_btn.grid_forget()
        admin_btn.grid_forget()
        Label(login, text="Password:").grid(row=1, column=0, pady=10)
        entry = Entry(login, show="*")
        entry.grid(row=1, column=1, pady=10)

        def check():
            global access
            try:
                if check_password(entry.get(),"moderator"):
                    access = "moderator"
                    login.destroy()
                    give_rights()
                else:
                    messagebox.showerror("Error", "Invalid password")
            except FileNotFoundError:
                messagebox.showerror("Error", "File is not found")

        Button(login, text="Login", command=check).grid(row=2, column=1, pady=10)

    def login_user():
        global access
        access = "user"
        login.destroy()
        give_rights()

    user_btn = Button(login, text="User", command=login_user)
    user_btn.grid(row=1, column=1, pady=10)
    moder_btn = Button(login, text="Moderator", command=login_moder)
    moder_btn.grid(row=2, column=1, pady=10)
    admin_btn = Button(login, text="Admin", command=login_admin)
    admin_btn.grid(row=3, column=1, pady=10)

def hash_password(password, status):
    salt = secrets.token_hex(16)
    pass_hash = hashlib.sha256((password + salt).encode("utf-8")).hexdigest()
    try:
        with open(f"{status}_password.txt", "w") as f:
            f.write(pass_hash + "\n")
            f.write(salt)
    except Exception as e:
        raise e

def check_password(password, access):
    with open(f"{access}_password.txt", "r") as f:
        old_hash = f.readline().strip()
        salt = f.readline().strip()
        new_hash = hashlib.sha256((password + salt).encode("utf-8")).hexdigest()
        return new_hash == old_hash

def password_interface():
    if access=="admin":
        password_frame = Toplevel(root)
        password_frame.title("Moderator password Manager")
        password_frame.geometry("500x200")
        password_frame.resizable(width=False, height=False)
        password_frame.grab_set()

        Label(password_frame, text="New password:").grid(row=0, column=0, padx=10, pady=10)
        new_pass_entry = Entry(password_frame, show="*")
        new_pass_entry.grid(row=0, column=1, padx=10, pady=10)
        Label(password_frame, text="Confirm new password:").grid(row=1, column=0, padx=10, pady=10)
        confirm_entry = Entry(password_frame, show="*")
        confirm_entry.grid(row=1, column=1, padx=10, pady=10)

        def change_password():
            new_password = new_pass_entry.get()
            new_password2 = confirm_entry.get()

            if not new_password or not new_password2:
                messagebox.showerror("Error", "New password is required")
                return
            if new_password == new_password2:
                hash_password(new_password, "moderator")
                messagebox.showinfo("Success", "Password has been successfully changed")
                password_frame.destroy()
            else:
                messagebox.showerror("Error", "Passwords do not match")

        change_btn = Button(password_frame, text="Confirm", command=lambda: change_password())
        change_btn.grid(row=5,column=2,padx=10,pady=10)

    else:
        messagebox.showerror("Error", "Admin access required")

def reset_password():
    if access != "admin":
        messagebox.showerror("Error", "Admin access required")
        return
    hash_password("123", "moderator")
    messagebox.showinfo("Password reset", "Password has been successfully reset")

def statistics():
    insert_res(total_entry,str(len(cities)))
    mil = 0
    small=0
    others=0
    for city,data in cities.items():
        if data["population"] > 1000000:
            mil+=1
        elif data["population"] < 20000:
            small+=1
        else:
            others+=1

    insert_res(mil_entry, str(mil))
    insert_res(small_entry, str(small))
    insert_res(other_entry, str(others))

def update_file(new_file):
    with open('population_size.json',"w", encoding="utf-8") as f:
        json.dump(new_file, f, ensure_ascii=False, indent=4)

def create_list():
    cities = {}
    try:
        with open('population_size.json', encoding="utf-8") as f:
            cities = json.load(f)
            for city, info in cities.items():
                if isinstance(info,int) and not isinstance(info,bool):
                    cities[city]={"population":info,"status":False}
        return cities
    except Exception as e:
        print(f"Error: {e}")
        return {}

def parse(city):
    city_low = city.strip().lower()
    for new_city in cities.keys():
        if new_city.lower() == city_low:
            return new_city
    return None

def add_city(name, population, stat):
    global cities

    if not isinstance(name,str) or not name.strip():
        raise ValueError("Name field cannot be empty")
    name = name.strip()
    if parse(name) is not None:
        raise ValueError("Name is already taken")
    try:
        population = int(population)
    except (TypeError, ValueError):
        raise ValueError("Population must be an integer")
    if population < 0:
        raise ValueError("Population cannot be less than 0")
    cities[name] = {"population":int(population),"status":stat}
    update_file(cities)
    statistics()

def change_population(city,new_population,stat):
    global cities
    if not isinstance(city,str) or not city.strip():
        raise ValueError (f"Name field cannot be empty")
    city = parse(city)
    if city is None:
        raise ValueError("Name is not found")
    try:
        new_population = int(new_population)
    except (TypeError, ValueError):
        raise ValueError("Population must be an integer")
    if new_population < 0:
        raise ValueError("Population cannot be less than 0")
    elif access=="moderator":
        if cities[city]["population"] != 0:
            difference = abs(int(new_population) - cities[city]["population"])/cities[city]["population"]*100
            if difference > 5:
                raise ValueError("You don't have enough rights to do this")
        else:
            raise ValueError("You don't have enough rights to do this")

    cities[city] = {"population":int(new_population),"status":stat}
    update_file(cities)
    statistics()

def delete_city(name):
    if access != "admin":
        raise PermissionError("Error", "Admin access required")
    global cities
    if not isinstance(name, str) or not name.strip():
        raise ValueError(f"Name field cannot be empty")
    name = parse(name)
    if name is None:
        raise ValueError("Name is not found")
    del cities[name]
    update_file(cities)
    statistics()

def pattern_check(pattern):
    pattern = pattern.lower()
    if len(pattern) < 3:
        return []
    cits = [city for city in cities if pattern.lower() in city.lower()]
    return sorted(cits)

def typing(entry=None):
    text = searching_field.get()
    listbox.delete(0, "end")
    if len(text) < 3:
        return
    for city in pattern_check(text):
        listbox.insert("end", city)

def add_in_entry(entry=None):
    if not listbox.curselection():
        return
    index = listbox.curselection()[0]
    city = listbox.get(index)
    searching_field.delete(0, "end")
    searching_field.insert(0, city)

def search():
    city = searching_field.get().strip()
    if not city:
        messagebox.showinfo("Error", "City field cannot be empty")
        insert_res(res,"")
        insert_res(status_res,"")
        return
    city = parse(city)
    if city:
        insert_res(res,str(cities[city]["population"]))
        insert_res(status_res, "Yes" if cities[city]["status"] else "No")
    else:
        insert_res(res,"Unknown city")
        insert_res(status_res,"")

def insert_res(res,text):
    res.config(state="normal")
    res.delete(0,"end")
    res.insert(0,text)
    res.config(state="readonly")

def admin(action):
    city = adm_entry.get().strip()
    population = adm_entry2.get().strip()
    stat = status.get()
    try:
        if action == "add":
            add_city(city, population,stat)
            messagebox.showinfo("Success", "City Added")
        elif action == "remove":
            delete_city(city)
            messagebox.showinfo("Success", "City Removed")
        elif action == "update":
            change_population(city,population,stat)
            messagebox.showinfo("Success", "City Updated")
        typing()
    except Exception as e:
        messagebox.showerror("Error", str(e))

root = Tk()
root.title("Find out the population size")
root.minsize(900,600)
root.maxsize(900,600)

cities = create_list()
Label(root, text = "City Name").grid(row=0,column=0,padx=10,pady=10)
searching_field = Entry(root)
searching_field.grid(row=0,column=1,padx=10,pady=10)

Label(root, text = "Population").grid(row=2,column=0,padx=10,pady=10)
res = Entry(root, state="readonly", justify="center")
res.grid(row=2,column=1,padx=10,pady=10)
Label(root, text = "Special status").grid(row=2,column=2,padx=10,pady=10)
status_res = Entry(root, state="readonly", justify="center")
status_res.grid(row=2,column=3,padx=10,pady=10)

listbox = Listbox(root)
listbox.grid(row=1,column=1,padx=10,pady=3)

searching_field.bind("<KeyRelease>", typing)
listbox.bind("<<ListboxSelect>>", add_in_entry)

btn = Button(root, text="Search", command=search, width=20, height=2)
btn.grid(row=0,column=2,columnspan=2,padx=20,pady=5)

mode_btn = Button(root, text="Mode", command=lambda:account())
mode_btn.grid(row=0,column=4,padx=200,pady=10,sticky="w")

# Moder_panel

moder_panel = Frame(root)

Label(moder_panel, text="Data amount: ").grid(row=0,column=0,padx=10,pady=10)
total_entry = Entry(moder_panel, state="readonly")
total_entry.grid(row=1,column=0,padx=10,pady=10)

Label(moder_panel, text="Million-plus cities: ").grid(row=0,column=1,padx=10,pady=10)
mil_entry = Entry(moder_panel, state="readonly")
mil_entry.grid(row=1,column=1,padx=10,pady=3)

Label(moder_panel, text="Cities with population < 20.000: ").grid(row=0,column=2,padx=10,pady=10)
small_entry = Entry(moder_panel, state="readonly")
small_entry.grid(row=1,column=2,padx=10,pady=10)

Label(moder_panel, text="Others: ").grid(row=0,column=3,padx=10,pady=3)
other_entry = Entry(moder_panel, state="readonly")
other_entry.grid(row=1,column=3,padx=10,pady=10)

Label(moder_panel, text = "Name of the city:").grid(row=2,column=0,padx=10,pady=10)
adm_entry = Entry(moder_panel)
adm_entry.grid(row=2,column=1,padx=10,pady=10)

Label(moder_panel, text = "Population of the city:").grid(row=3,column=0,padx=10,pady=10)
adm_entry2 = Entry(moder_panel)
adm_entry2.grid(row=3,column=1,padx=10,pady=10)

add_btn = Button(moder_panel, text="Add", command=lambda:admin("add"))
add_btn.grid(row=2,column=2,padx=10,pady=10)
update_btn = Button(moder_panel, text="Update", command=lambda:admin("update"))
update_btn.grid(row=2,column=3,padx=10,pady=10)
delete_btn = Button(moder_panel, text="Delete", command=lambda:admin("remove"))
delete_btn.grid(row=2,column=4,padx=10,pady=10)

status = BooleanVar()
Checkbutton(moder_panel, text="Special", variable=status).grid(row=5,column=0,padx=10,pady=10)

upd_stat_btn = Button(moder_panel, text="Update statistics",command=lambda:statistics())
upd_stat_btn.grid(row=1,column=4,padx=10,pady=10)

# Admin_panel

admin_panel = Frame(root)
reset_btn = Button(admin_panel, text="Reset moderator password", command=lambda:reset_password())
cp_btn = Button(admin_panel, text="Change moderator password", command=lambda:password_interface())

root.mainloop()

```
