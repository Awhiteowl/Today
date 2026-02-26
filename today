import tkinter as tk
from tkinter import ttk, colorchooser, font
import jdatetime
import json
import os
import sys
import pystray
from PIL import Image, ImageDraw
import threading
import ctypes

# ================= پیدا کردن مسیر اجرای برنامه (برای نسخه EXE) =================
if getattr(sys, 'frozen', False):
    BASE_PATH = os.path.dirname(sys.executable)
else:
    BASE_PATH = os.path.dirname(os.path.abspath(__file__))

CONFIG_FILE = os.path.join(BASE_PATH, "widget_config.json")

# ================= تبدیل‌گر فارسی =================
FA_DAYS = {"Saturday": "شنبه", "Sunday": "یکشنبه", "Monday": "دوشنبه", "Tuesday": "سه‌شنبه", "Wednesday": "چهارشنبه", "Thursday": "پنج‌شنبه", "Friday": "جمعه"}
FA_MONTHS = {"Farvardin": "فروردین", "Ordibehesht": "اردیبهشت", "Khordad": "خرداد", "Tir": "تیر", "Mordad": "مرداد", "Shahrivar": "شهریور", "Mehr": "مهر", "Aban": "آبان", "Azar": "آذر", "Dey": "دی", "Bahman": "بهمن", "Esfand": "اسفند"}

# جلوگیری از اجرای چندباره
mutex = ctypes.windll.kernel32.CreateMutexW(None, False, "ROOZ_WIDGET_ULTIMATE_V70")
if ctypes.windll.kernel32.GetLastError() == 183: sys.exit(0)

# بارگذاری کانفیگ
default_config = {"x": 100, "y": 100, "font_name": "Tahoma", "font_size": 32, "text_color": "#ffffff", "alpha": 0.9, "locked": False}
if not os.path.exists(CONFIG_FILE):
    with open(CONFIG_FILE, "w", encoding="utf-8") as f: json.dump(default_config, f, ensure_ascii=False, indent=4)
with open(CONFIG_FILE, "r", encoding="utf-8") as f: config = json.load(f)

# ================= پنجره اصلی =================
root = tk.Tk()
root.title("Rooz Widget")
root.overrideredirect(True)
root.configure(bg="black")
root.wm_attributes("-transparentcolor", "black")
root.attributes("-alpha", config["alpha"])

# --- ترفند اصلی برای ضد Win+D ---
# تبدیل پنجره به یک ToolWindow سیستمی که مینی‌مایز نمی‌شود
GWL_EXSTYLE = -20
WS_EX_TOOLWINDOW = 0x00000080
def set_as_tool_window():
    hwnd = ctypes.windll.user32.GetParent(root.winfo_id())
    style = ctypes.windll.user32.GetWindowLongW(hwnd, GWL_EXSTYLE)
    style = style | WS_EX_TOOLWINDOW
    ctypes.windll.user32.SetWindowLongW(hwnd, GWL_EXSTYLE, style)
    # همیشه پایین نگه داشتن نسبت به بقیه پنجره‌ها
    root.lower()

def update_geometry():
    f_size = config["font_size"]
    width, height = int(f_size * 14), int(f_size * 6)
    root.geometry(f"{width}x{height}+{config['x']}+{config['y']}")

update_geometry()

# --- المان‌ها ---
lbl_time = tk.Label(root, text="", font=(config["font_name"], int(config["font_size"]*1.8), "bold"), fg=config["text_color"], bg="black")
lbl_time.place(relx=0.5, rely=0.20, anchor="center")

date_frame = tk.Frame(root, bg="black")
date_frame.place(relx=0.5, rely=0.58, anchor="center")

lbl_year = tk.Label(date_frame, text="", font=(config["font_name"], config["font_size"], "bold"), fg=config["text_color"], bg="black")
lbl_month = tk.Label(date_frame, text="", font=(config["font_name"], config["font_size"], "bold"), fg=config["text_color"], bg="black")
lbl_day = tk.Label(date_frame, text="", font=(config["font_name"], config["font_size"], "bold"), fg=config["text_color"], bg="black")
lbl_day.pack(side="right", padx=5); lbl_month.pack(side="right", padx=5); lbl_year.pack(side="right", padx=5)

lbl_weekday = tk.Label(root, text="", font=(config["font_name"], int(config["font_size"]*0.8), "bold"), fg=config["text_color"], bg="black")
lbl_weekday.place(relx=0.5, rely=0.85, anchor="center")

def update_widget():
    now = jdatetime.datetime.now()
    lbl_time.config(text=now.strftime('%H:%M'))
    lbl_day.config(text=str(now.day))
    lbl_month.config(text=FA_MONTHS.get(now.strftime('%B'), ""))
    lbl_year.config(text=str(now.year))
    lbl_weekday.config(text=FA_DAYS.get(now.strftime('%A'), ""))
    root.after(30000, update_widget)

# ================= درگ و جابه‌جایی روان =================
root.drag_data = {"x": 0, "y": 0}
def start_drag(e):
    if not config["locked"]:
        root.drag_data["x"], root.drag_data["y"] = e.x_root - root.winfo_x(), e.y_root - root.winfo_y()

def on_drag(e):
    if not config["locked"]:
        nx, ny = e.x_root - root.drag_data["x"], e.y_root - root.drag_data["y"]
        root.geometry(f"+{nx}+{ny}")
        config["x"], config["y"] = nx, ny
        with open(CONFIG_FILE, "w", encoding="utf-8") as f: json.dump(config, f, ensure_ascii=False, indent=4)

root.bind("<Button-1>", start_drag); root.bind("<B1-Motion>", on_drag)
for w in [lbl_time, date_frame, lbl_year, lbl_month, lbl_day, lbl_weekday]:
    w.bind("<Button-1>", start_drag); w.bind("<B1-Motion>", on_drag)

# ================= تنظیمات =================
def open_settings():
    win = tk.Toplevel(root)
    win.title("تنظیمات"); win.geometry("340x450"); win.configure(bg="#121212"); win.attributes("-topmost", True)
    tk.Label(win, text="تنظیمات ویجت", font=("Tahoma", 12, "bold"), fg="#00adb5", bg="#121212").pack(pady=20)
    f_cb = ttk.Combobox(win, values=sorted(font.families()), state="readonly"); f_cb.set(config["font_name"]); f_cb.pack(fill="x", padx=40, pady=5)
    sc = tk.Scale(win, from_=20, to=80, orient="horizontal", bg="#121212", fg="white", highlightthickness=0); sc.set(config["font_size"]); sc.pack(fill="x", padx=40)
    
    def apply(*args):
        config["font_name"], config["font_size"] = f_cb.get(), int(sc.get())
        update_geometry()
        lbl_time.config(font=(config["font_name"], int(config["font_size"]*1.8), "bold"))
        for l in [lbl_day, lbl_month, lbl_year]: l.config(font=(config["font_name"], config["font_size"], "bold"))
        lbl_weekday.config(font=(config["font_name"], int(config["font_size"]*0.8), "bold"))
        with open(CONFIG_FILE, "w", encoding="utf-8") as f: json.dump(config, f, ensure_ascii=False, indent=4)
    
    f_cb.bind("<<ComboboxSelected>>", apply); sc.bind("<ButtonRelease-1>", apply)
    tk.Button(win, text="🎨 تغییر رنگ", bg="#00adb5", fg="white", command=lambda: [config.update({"text_color": colorchooser.askcolor(initialcolor=config["text_color"])[1] or config["text_color"]}), [l.config(fg=config["text_color"]) for l in [lbl_time, lbl_day, lbl_month, lbl_year, lbl_weekday]], apply()]).pack(pady=20)

# ================= سیستم تری =================
def tray():
    img = Image.new('RGB', (64, 64), "#00adb5")
    icon = pystray.Icon("rooz", img, "Rooz", menu=pystray.Menu(pystray.MenuItem("تنظیمات", lambda: root.after(0, open_settings)), pystray.MenuItem("خروج", lambda: root.after(0, root.destroy))))
    icon.run()

# اجرای تنظیمات ضد Win+D
root.after(100, set_as_tool_window)
# چک کردن مداوم برای پایین ماندن روی دسکتاپ
def keep_on_bottom():
    root.lower()
    root.after(1000, keep_on_bottom)

keep_on_bottom()
threading.Thread(target=tray, daemon=True).start()
update_widget()
root.mainloop()
