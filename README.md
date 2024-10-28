# TeamIPTracker
Group assignment repository
import requests
from datetime import datetime
import tkinter as tk
from tkinter import ttk, messagebox
import configparser

# Configuration for user preferences
config = configparser.ConfigParser()
config.read('preferences.ini')

# Define theme colors
themes = {
    'Light': {
        'bg': '#ffffff',
        'fg': '#000000',
        'text_bg': '#ffffff',
        'text_fg': '#000000',
        'frame_bg': '#f0f0f0',
        'button_bg': '#e0e0e0',
        'button_fg': '#000000'
    },
    'Dark': {
        'bg': '#2e2e2e',
        'fg': '#ffffff',
        'text_bg': '#333333',
        'text_fg': '#ffffff',
        'frame_bg': '#1e1e1e',
        'button_bg': '#444444',
        'button_fg': '#ffffff'
    }
}

# List to store history
history = []

def get_ip_info(ip_address=None):
    api_url = f"https://ipapi.co/{ip_address or 'json'}/"
    try:
        response = requests.get(api_url)
        if response.status_code == 200:
            data = response.json()
            current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            ip_info = (
                f"Current Time: {current_time}\n\n"
                f"Public IPv4/IPv6 Address: {data.get('ip')}\n"
                f"City: {data.get('city')}\n"
                f"Region: {data.get('region')}\n"
                f"Country: {data.get('country_name')} ({data.get('country_code')})\n"
                f"Latitude: {data.get('latitude')}\n"
                f"Longitude: {data.get('longitude')}\n"
                f"ISP: {data.get('org')}\n"
                f"ASN: {data.get('asn')}\n"
                f"Timezone: {data.get('timezone')}"
            )
        else:
            ip_info = f"Failed to retrieve IP information. Status Code: {response.status_code}"
    except Exception as e:
        ip_info = f"Error occurred: {str(e)}"
    return ip_info

def refresh_ip_info():
    ip_info = get_ip_info()  # No need to check for a specific IP address
    text.delete(1.0, tk.END)
    text.insert(tk.END, ip_info)
    history.append(ip_info)  # Add to history

def copy_to_clipboard():
    root.clipboard_clear()
    root.clipboard_append(text.get(1.0, tk.END))
    messagebox.showinfo("Copied", "IP information copied to clipboard!")

def save_to_file():
    ip_info = text.get(1.0, tk.END)
    with open("ip_info.txt", "w") as file:
        file.write(ip_info)
    messagebox.showinfo("Saved", "IP information saved to 'ip_info.txt'.")

def show_history():
    history_window = tk.Toplevel(root)
    history_window.title("History")
    history_text = tk.Text(history_window, height=20, width=80, wrap=tk.WORD)
    history_text.pack(padx=10, pady=10)
    history_text.insert(tk.END, "\n\n".join(history))
    history_window.mainloop()

def apply_theme(theme_name):
    theme = themes.get(theme_name, themes['Light'])  # Default to Light theme
    root.tk_setPalette(background=theme['bg'], foreground=theme['fg'])
    text.config(bg=theme['text_bg'], fg=theme['text_fg'])
    frame.config(style=theme_name + '.TFrame')

    # Update styles for ttk widgets
    style = ttk.Style()
    style.configure(f"{theme_name}.TFrame", background=theme['frame_bg'])
    style.configure(f"{theme_name}.TButton", background=theme['button_bg'], foreground=theme['button_fg'])
    style.configure(f"{theme_name}.TLabel", background=theme['bg'], foreground=theme['fg'])
    style.configure(f"{theme_name}.TCheckbutton", background=theme['bg'], foreground=theme['fg'])

def change_theme(*args):
    selected_theme = theme_var.get()
    apply_theme(selected_theme)
    save_preferences()

def save_preferences():
    config['Settings'] = {
        'DarkMode': str(theme_var.get() == 'Dark'),
        'Theme': theme_var.get(),
    }
    with open('preferences.ini', 'w') as configfile:
        config.write(configfile)

def load_preferences():
    if config.read('preferences.ini'):
        theme_name = config.get('Settings', 'Theme', fallback='Light')
        theme_var.set(theme_name)
        apply_theme(theme_name)

# Create the main window
root = tk.Tk()
root.title("IP Information")

# Create a frame for the layout
frame = ttk.Frame(root, padding="10")
frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

# Create a text widget to display IP information
text = tk.Text(frame, height=15, width=80, wrap=tk.WORD)
text.grid(row=0, column=0, columnspan=2, padx=5, pady=5)

# Create buttons with custom styles
style = ttk.Style()
style.configure('TButton', font=('Arial', 12), padding=6)
style.configure('TButton', background=themes['Light']['button_bg'], foreground=themes['Light']['button_fg'])
style.map('TButton', background=[('active', themes['Light']['button_fg'])])

refresh_button = ttk.Button(frame, text="Refresh", command=refresh_ip_info)
refresh_button.grid(row=1, column=0, padx=5, pady=5, sticky=tk.W)
copy_button = ttk.Button(frame, text="Copy to Clipboard", command=copy_to_clipboard)
copy_button.grid(row=1, column=1, padx=5, pady=5, sticky=tk.W)
save_button = ttk.Button(frame, text="Save to File", command=save_to_file)
save_button.grid(row=2, column=0, padx=5, pady=5, sticky=tk.W)
history_button = ttk.Button(frame, text="View History", command=show_history)
history_button.grid(row=2, column=1, padx=5, pady=5, sticky=tk.W)

# Theme selection dropdown
theme_var = tk.StringVar()
theme_var.set('Light')  # Default theme
theme_menu = ttk.OptionMenu(frame, theme_var, 'Light', 'Light', 'Dark', command=change_theme)
theme_menu.grid(row=3, column=0, columnspan=2, padx=5, pady=5, sticky=tk.W)

# Load preferences at startup
load_preferences()

# Run the application
refresh_ip_info()  # Initial data load
root.mainloop()
