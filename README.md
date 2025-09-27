<img width="1879" height="1076" alt="Screenshot 2025-09-28 001956" src="https://github.com/user-attachments/assets/a3712525-ee0c-4d7b-8544-e92adcc8cc60" />
<img width="1862" height="1085" alt="Screenshot 2025-09-28 001941" src="https://github.com/user-attachments/assets/a6895ba7-65a5-476e-923f-dfa96ed7f2e8" />
import tkinter as tk
from tkinter import ttk, filedialog, messagebox
import cv2
from PIL import Image, ImageTk, ImageDraw
import numpy as np
import random
import time
import threading
from datetime import datetime
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg


class CropHealthApp:
    def __init__(self, root):
        self.root = root
        self.root.title("🌱 AgriTech Pro - Crop Health Monitor")
        self.root.geometry("1400x900")

        # New Color Scheme - Earthy Green Theme
        self.colors = {
            'primary': '#2E8B57',  # Sea Green
            'secondary': '#8FBC8F',  # Dark Sea Green
            'accent': '#FFD700',  # Gold
            'success': '#32CD32',  # Lime Green
            'warning': '#FF8C00',  # Dark Orange
            'danger': '#DC143C',  # Crimson
            'light': '#F5F5DC',  # Beige
            'dark': '#2F4F4F',  # Dark Slate Gray
            'background': '#F0FFF0'  # Honeydew
        }

        self.root.configure(bg=self.colors['background'])

        self.current_image = None
        self.camera_active = False
        self.cap = None
        self.logged_in = False
        self.user_data = {}

        self.create_auth_screen()

    def create_auth_screen(self):
        # Main auth frame
        self.auth_frame = tk.Frame(self.root, bg=self.colors['background'])
        self.auth_frame.pack(fill='both', expand=True)

        # Animated background elements
        self.canvas = tk.Canvas(self.auth_frame, bg=self.colors['background'], highlightthickness=0)
        self.canvas.pack(fill='both', expand=True)
        self.create_animated_background()

        # Title with better styling
        title_frame = tk.Frame(self.canvas, bg=self.colors['background'])
        title_frame.place(relx=0.5, rely=0.1, anchor='center')

        title_label = tk.Label(title_frame, text="🌱 AgriTech Pro",
                               font=('Arial', 32, 'bold'),
                               bg=self.colors['background'],
                               fg=self.colors['primary'])
        title_label.pack()

        subtitle_label = tk.Label(title_frame, text="AI-Powered Crop Health Monitoring",
                                  font=('Arial', 14),
                                  bg=self.colors['background'],
                                  fg=self.colors['dark'])
        subtitle_label.pack(pady=5)

        # Auth container with glass effect
        auth_container = tk.Frame(self.canvas, bg=self.colors['light'],
                                  relief='ridge', bd=2, highlightthickness=2,
                                  highlightbackground=self.colors['primary'])
        auth_container.place(relx=0.5, rely=0.5, anchor='center', width=400, height=500)

        # Notebook for login/register
        self.auth_notebook = ttk.Notebook(auth_container)
        self.auth_notebook.pack(fill='both', expand=True, padx=10, pady=10)

        # Login tab
        self.login_tab = tk.Frame(self.auth_notebook, bg=self.colors['light'])
        self.auth_notebook.add(self.login_tab, text="🔑 Login")

        # Register tab
        self.register_tab = tk.Frame(self.auth_notebook, bg=self.colors['light'])
        self.auth_notebook.add(self.register_tab, text="📝 Register")

        self.create_login_tab()
        self.create_register_tab()

        # Live date
        self.date_label = tk.Label(self.canvas, text="",
                                   font=('Arial', 12, 'bold'),
                                   bg=self.colors['background'],
                                   fg=self.colors['primary'])
        self.date_label.place(relx=0.5, rely=0.95, anchor='center')
        self.update_date()

    def create_animated_background(self):
        # Create floating leaves animation
        self.leaves = []
        for _ in range(15):
            leaf = {
                'x': random.randint(0, 1400),
                'y': random.randint(0, 900),
                'size': random.randint(20, 40),
                'speed': random.uniform(0.5, 2),
                'symbol': random.choice(['🍃', '🌿', '🍀', '🌱']),
                'color': random.choice([self.colors['primary'], self.colors['success'], self.colors['secondary']])
            }
            self.leaves.append(leaf)

        self.animate_leaves()

    def animate_leaves(self):
        self.canvas.delete('leaves')
        for leaf in self.leaves:
            leaf['y'] += leaf['speed']
            if leaf['y'] > 900:
                leaf['y'] = -50
                leaf['x'] = random.randint(0, 1400)

            self.canvas.create_text(leaf['x'], leaf['y'],
                                    text=leaf['symbol'],
                                    fill=leaf['color'],
                                    font=('Arial', leaf['size']),
                                    tags='leaves')

        self.root.after(50, self.animate_leaves)

    def update_date(self):
        current_date = datetime.now().strftime("%A, %B %d, %Y %I:%M:%S %p")
        self.date_label.config(text=f"📅 {current_date}")
        self.root.after(1000, self.update_date)

    def create_login_tab(self):
        # Login form
        login_frame = tk.Frame(self.login_tab, bg=self.colors['light'])
        login_frame.pack(expand=True, padx=20, pady=20)

        tk.Label(login_frame, text="Email:", bg=self.colors['light'],
                 fg=self.colors['dark'], font=('Arial', 12, 'bold')).pack(pady=10)
        self.login_email = tk.Entry(login_frame, font=('Arial', 12), width=25,
                                    bg='white', relief='solid', bd=1)
        self.login_email.pack(pady=5, ipady=5)

        tk.Label(login_frame, text="Password:", bg=self.colors['light'],
                 fg=self.colors['dark'], font=('Arial', 12, 'bold')).pack(pady=10)
        self.login_password = tk.Entry(login_frame, show="*", font=('Arial', 12), width=25,
                                       bg='white', relief='solid', bd=1)
        self.login_password.pack(pady=5, ipady=5)

        login_btn = tk.Button(login_frame, text="🚀 Login", font=('Arial', 12, 'bold'),
                              bg=self.colors['primary'], fg='white',
                              command=self.login, width=20, height=2,
                              relief='raised', bd=3)
        login_btn.pack(pady=20)

        # Demo credentials
        demo_label = tk.Label(login_frame, text="Demo: admin@agritech.com / password123",
                              bg=self.colors['light'], fg=self.colors['dark'],
                              font=('Arial', 10))
        demo_label.pack()

        # Bind Enter key to login
        self.login_password.bind('<Return>', lambda e: self.login())

    def create_register_tab(self):
        # Register form
        register_frame = tk.Frame(self.register_tab, bg=self.colors['light'])
        register_frame.pack(expand=True, padx=20, pady=20)

        fields = [
            ("👤 Full Name:", "name"),
            ("📧 Email:", "email"),
            ("📱 Mobile:", "mobile"),
            ("🔒 Password:", "password", True),
            ("✅ Confirm Password:", "confirm_password", True)
        ]

        self.register_entries = {}
        for label_text, field_name, *is_password in fields:
            tk.Label(register_frame, text=label_text, bg=self.colors['light'],
                     fg=self.colors['dark'], font=('Arial', 11, 'bold')).pack(pady=8, anchor='w')

            entry = tk.Entry(register_frame,
                             show="*" if is_password else None,
                             font=('Arial', 11), width=25,
                             bg='white', relief='solid', bd=1)
            entry.pack(pady=5, ipady=4, fill='x')
            self.register_entries[field_name] = entry

        register_btn = tk.Button(register_frame, text="📝 Create Account",
                                 font=('Arial', 12, 'bold'),
                                 bg=self.colors['success'], fg='white',
                                 command=self.register, width=20, height=2,
                                 relief='raised', bd=3)
        register_btn.pack(pady=20)

    def login(self):
        email = self.login_email.get()
        password = self.login_password.get()

        if not email or not password:
            messagebox.showerror("Error", "Please fill in all fields")
            return

        if email == "admin@agritech.com" and password == "password123":
            self.user_data = {'name': 'Admin User', 'email': email}
            self.logged_in = True
            self.show_main_interface()
        else:
            messagebox.showerror("Error", "Invalid credentials")

    def register(self):
        # Registration validation
        if not all(entry.get() for entry in self.register_entries.values()):
            messagebox.showerror("Error", "Please fill all fields")
            return

        if self.register_entries['password'].get() != self.register_entries['confirm_password'].get():
            messagebox.showerror("Error", "Passwords don't match")
            return

        if len(self.register_entries['mobile'].get()) != 10 or not self.register_entries['mobile'].get().isdigit():
            messagebox.showerror("Error", "Please enter valid 10-digit mobile number")
            return

        self.user_data = {
            'name': self.register_entries['name'].get(),
            'email': self.register_entries['email'].get(),
            'mobile': self.register_entries['mobile'].get()
        }

        messagebox.showinfo("Success", "Registration successful! Welcome to AgriTech Pro.")
        self.logged_in = True
        self.show_main_interface()

    def show_main_interface(self):
        self.auth_frame.pack_forget()

        # Main interface
        self.main_frame = tk.Frame(self.root, bg=self.colors['background'])
        self.main_frame.pack(fill='both', expand=True)

        # Header
        self.create_header()

        # Content
        self.create_content()

    def create_header(self):
        header = tk.Frame(self.main_frame, bg=self.colors['primary'], height=100)
        header.pack(fill='x', padx=10, pady=5)
        header.pack_propagate(False)

        # Logo and title
        title_frame = tk.Frame(header, bg=self.colors['primary'])
        title_frame.pack(side='left', padx=20, fill='y')

        title = tk.Label(title_frame, text="🌱 AgriTech Pro",
                         font=('Arial', 24, 'bold'),
                         bg=self.colors['primary'], fg='white')
        title.pack(side='left')

        subtitle = tk.Label(title_frame, text="AI Crop Monitoring",
                            font=('Arial', 12),
                            bg=self.colors['primary'], fg=self.colors['light'])
        subtitle.pack(side='left', padx=10)

        # User info
        user_frame = tk.Frame(header, bg=self.colors['primary'])
        user_frame.pack(side='right', padx=20, fill='y')

        user_info = tk.Label(user_frame,
                             text=f"👤 {self.user_data['name']} | 📧 {self.user_data['email']}",
                             font=('Arial', 11),
                             bg=self.colors['primary'], fg='white')
        user_info.pack(side='right')

        # Live date
        self.main_date_label = tk.Label(header, text="", font=('Arial', 12, 'bold'),
                                        bg=self.colors['primary'], fg='white')
        self.main_date_label.place(relx=0.5, rely=0.5, anchor='center')
        self.update_main_date()

        # Navigation
        nav_frame = tk.Frame(header, bg=self.colors['primary'])
        nav_frame.place(relx=0.5, rely=0.8, anchor='center')

        buttons = [
            ("📊 Dashboard", self.show_dashboard, self.colors['secondary']),
            ("🔍 Analyze", self.show_analyze, self.colors['accent']),
            ("📋 Reports", self.show_report, self.colors['success']),
            ("⚙️ Settings", self.show_settings, self.colors['warning']),
            ("🚪 Logout", self.logout, self.colors['danger'])
        ]

        for text, command, color in buttons:
            btn = tk.Button(nav_frame, text=text, font=('Arial', 10, 'bold'),
                            bg=color, fg='white', relief='raised', bd=2,
                            command=command, width=12)
            btn.pack(side='left', padx=3)

    def update_main_date(self):
        current_date = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.main_date_label.config(text=f"📅 {current_date}")
        self.root.after(1000, self.update_main_date)

    def create_content(self):
        # Notebook for main content
        style = ttk.Style()
        style.configure('TNotebook', background=self.colors['background'])
        style.configure('TNotebook.Tab', font=('Arial', 11, 'bold'))

        self.main_notebook = ttk.Notebook(self.main_frame)
        self.main_notebook.pack(fill='both', expand=True, padx=10, pady=10)

        # Create tabs
        self.dashboard_tab = tk.Frame(self.main_notebook, bg=self.colors['background'])
        self.analyze_tab = tk.Frame(self.main_notebook, bg=self.colors['background'])
        self.report_tab = tk.Frame(self.main_notebook, bg=self.colors['background'])
        self.settings_tab = tk.Frame(self.main_notebook, bg=self.colors['background'])

        self.main_notebook.add(self.dashboard_tab, text="📊 Dashboard")
        self.main_notebook.add(self.analyze_tab, text="🔍 Analyze")
        self.main_notebook.add(self.report_tab, text="📋 Reports")
        self.main_notebook.add(self.settings_tab, text="⚙️ Settings")

        self.create_dashboard()
        self.create_analyze_tab()
        self.create_report_tab()
        self.create_settings_tab()

    def create_dashboard(self):
        # Dashboard content
        title = tk.Label(self.dashboard_tab, text="Farm Dashboard",
                         font=('Arial', 28, 'bold'),
                         bg=self.colors['background'], fg=self.colors['primary'])
        title.pack(pady=20)

        # Stats cards
        stats_frame = tk.Frame(self.dashboard_tab, bg=self.colors['background'])
        stats_frame.pack(pady=20)

        stats = [
            ("🌱 Health Score", "87%", self.colors['success'], "↑ 2% from last week"),
            ("🌿 Soil Quality", "Excellent", self.colors['secondary'], "Optimal conditions"),
            ("🐛 Pest Risk", "Low", self.colors['accent'], "No immediate threat"),
            ("💧 Moisture", "85%", self.colors['primary'], "Ideal level"),
            ("🌡️ Temperature", "26°C", self.colors['warning'], "Within range"),
            ("☀️ Sunlight", "Optimal", self.colors['success'], "6-8 hours daily")
        ]

        for i in range(0, len(stats), 3):
            row_frame = tk.Frame(stats_frame, bg=self.colors['background'])
            row_frame.pack(pady=10)

            for text, value, color, subtext in stats[i:i + 3]:
                card = tk.Frame(row_frame, bg=color, relief='ridge', bd=2,
                                width=250, height=120)
                card.pack_propagate(False)
                card.pack(side='left', padx=10)

                tk.Label(card, text=text, font=('Arial', 12, 'bold'),
                         bg=color, fg='white').pack(pady=8)
                tk.Label(card, text=value, font=('Arial', 20, 'bold'),
                         bg=color, fg='white').pack()
                tk.Label(card, text=subtext, font=('Arial', 9),
                         bg=color, fg='white').pack(pady=5)

        # Recent activity
        activity_frame = tk.Frame(self.dashboard_tab, bg=self.colors['light'],
                                  relief='solid', bd=1)
        activity_frame.pack(fill='x', padx=50, pady=20)

        tk.Label(activity_frame, text="📈 Recent Activity",
                 font=('Arial', 16, 'bold'),
                 bg=self.colors['light'], fg=self.colors['dark']).pack(pady=10)

        activities = [
            "✅ Soil analysis completed - Optimal nutrients",
            "🔍 Crop health scan - 87% healthy",
            "💧 Irrigation system checked - Working properly",
            "🌱 New planting scheduled for next week",
            "📊 Monthly report generated"
        ]

        for activity in activities:
            tk.Label(activity_frame, text=f"• {activity}",
                     font=('Arial', 11),
                     bg=self.colors['light'], fg=self.colors['dark'],
                     anchor='w').pack(fill='x', padx=20, pady=2)

    def create_analyze_tab(self):
        # Analysis tab with camera
        main_frame = tk.Frame(self.analyze_tab, bg=self.colors['background'])
        main_frame.pack(fill='both', expand=True, padx=20, pady=20)

        # Left panel - Camera
        left_frame = tk.Frame(main_frame, bg=self.colors['background'])
        left_frame.pack(side='left', fill='both', expand=True)

        # Camera title
        tk.Label(left_frame, text="🌿 Hyperspectral Crop Analysis",
                 font=('Arial', 18, 'bold'),
                 bg=self.colors['background'], fg=self.colors['primary']).pack(pady=10)

        # Large image display
        self.camera_label = tk.Label(left_frame, text="📷 No image selected\n\nClick 'Start Camera' or 'Upload Image'",
                                     bg=self.colors['light'], fg=self.colors['dark'],
                                     font=('Arial', 14), width=50, height=20,
                                     relief='solid', bd=2)
        self.camera_label.pack(pady=10, padx=10, fill='both', expand=True)

        # Camera controls
        control_frame = tk.Frame(left_frame, bg=self.colors['background'])
        control_frame.pack(pady=10)

        self.cam_btn = tk.Button(control_frame, text="📷 Start Camera",
                                 font=('Arial', 11, 'bold'),
                                 bg=self.colors['primary'], fg='white',
                                 command=self.toggle_camera, width=15)
        self.cam_btn.pack(side='left', padx=5)

        self.capture_btn = tk.Button(control_frame, text="📸 Capture",
                                     font=('Arial', 11, 'bold'),
                                     bg=self.colors['secondary'], fg='white',
                                     command=self.capture_image, state='disabled', width=12)
        self.capture_btn.pack(side='left', padx=5)

        upload_btn = tk.Button(control_frame, text="📁 Upload Image",
                               font=('Arial', 11, 'bold'),
                               bg=self.colors['accent'], fg='white',
                               command=self.upload_image, width=15)
        upload_btn.pack(side='left', padx=5)

        analyze_btn = tk.Button(control_frame, text="🧠 Analyze",
                                font=('Arial', 11, 'bold'),
                                bg=self.colors['success'], fg='white',
                                command=self.analyze_image, width=12)
        analyze_btn.pack(side='left', padx=5)

        # Right panel - Results
        right_frame = tk.Frame(main_frame, bg=self.colors['background'])
        right_frame.pack(side='right', fill='both', expand=True)

        tk.Label(right_frame, text="📊 Analysis Results",
                 font=('Arial', 18, 'bold'),
                 bg=self.colors['background'], fg=self.colors['primary']).pack(pady=10)

        # Results display with scrollbar
        results_frame = tk.Frame(right_frame, bg=self.colors['background'])
        results_frame.pack(fill='both', expand=True, padx=10)

        self.results_text = tk.Text(results_frame, width=45, height=25,
                                    font=('Arial', 11), bg=self.colors['light'],
                                    wrap='word')
        scrollbar = tk.Scrollbar(results_frame, command=self.results_text.yview)
        self.results_text.config(yscrollcommand=scrollbar.set)

        self.results_text.pack(side='left', fill='both', expand=True)
        scrollbar.pack(side='right', fill='y')

        self.results_text.insert('1.0', "👨‍🌾 Welcome to AgriTech Pro!\n\n" +
                                 "To get started:\n" +
                                 "1. Start camera or upload a crop image\n" +
                                 "2. Capture or select your image\n" +
                                 "3. Click 'Analyze' for hyperspectral analysis\n\n" +
                                 "🌱 We'll analyze:\n" +
                                 "• Nutrient levels\n• Disease detection\n" +
                                 "• Pest risk assessment\n• Growth recommendations")

    def create_report_tab(self):
        # Report tab
        main_frame = tk.Frame(self.report_tab, bg=self.colors['background'])
        main_frame.pack(fill='both', expand=True, padx=20, pady=20)

        tk.Label(main_frame, text="📋 Detailed Analysis Report",
                 font=('Arial', 24, 'bold'),
                 bg=self.colors['background'], fg=self.colors['primary']).pack(pady=20)

        # Report content
        report_frame = tk.Frame(main_frame, bg=self.colors['light'],
                                relief='solid', bd=1)
        report_frame.pack(fill='both', expand=True, padx=50, pady=20)

        self.report_content = tk.Text(report_frame, font=('Arial', 12),
                                      bg=self.colors['light'], fg=self.colors['dark'],
                                      wrap='word')
        scrollbar = tk.Scrollbar(report_frame, command=self.report_content.yview)
        self.report_content.config(yscrollcommand=scrollbar.set)

        self.report_content.pack(side='left', fill='both', expand=True, padx=10, pady=10)
        scrollbar.pack(side='right', fill='y')

        # Download button
        download_btn = tk.Button(main_frame, text="📥 Download PDF Report",
                                 font=('Arial', 14, 'bold'),
                                 bg=self.colors['success'], fg='white',
                                 command=self.download_report, width=20, height=2)
        download_btn.pack(pady=20)

        self.report_content.insert('1.0', "Analysis reports will appear here after crop analysis.")

    def create_settings_tab(self):
        # Settings tab
        main_frame = tk.Frame(self.settings_tab, bg=self.colors['background'])
        main_frame.pack(fill='both', expand=True, padx=50, pady=30)

        tk.Label(main_frame, text="⚙️ System Settings",
                 font=('Arial', 24, 'bold'),
                 bg=self.colors['background'], fg=self.colors['primary']).pack(pady=20)

        # User info
        user_frame = tk.LabelFrame(main_frame, text="👤 User Information",
                                   font=('Arial', 14, 'bold'),
                                   bg=self.colors['light'], fg=self.colors['dark'])
        user_frame.pack(fill='x', pady=10)

        info_text = f"""
        Name: {self.user_data.get('name', 'N/A')}
        Email: {self.user_data.get('email', 'N/A')}
        Mobile: {self.user_data.get('mobile', 'N/A')}
        Member since: {datetime.now().strftime('%Y-%m-%d')}
        """

        tk.Label(user_frame, text=info_text, font=('Arial', 12),
                 bg=self.colors['light'], fg=self.colors['dark'],
                 justify='left').pack(pady=15, padx=20, anchor='w')

        # App settings
        settings_frame = tk.LabelFrame(main_frame, text="🔧 Application Settings",
                                       font=('Arial', 14, 'bold'),
                                       bg=self.colors['light'], fg=self.colors['dark'])
        settings_frame.pack(fill='x', pady=10)

        settings = [
            ("🔔 Enable push notifications", tk.BooleanVar(value=True)),
            ("💾 Auto-save captured images", tk.BooleanVar(value=True)),
            ("📊 High-resolution analysis", tk.BooleanVar(value=False)),
            ("🌐 Share anonymous data for research", tk.BooleanVar(value=True))
        ]

        for text, var in settings:
            cb = tk.Checkbutton(settings_frame, text=text, variable=var,
                                font=('Arial', 12),
                                bg=self.colors['light'], fg=self.colors['dark'],
                                selectcolor=self.colors['primary'])
            cb.pack(anchor='w', pady=8, padx=20)

    def toggle_camera(self):
        if not self.camera_active:
            self.start_camera()
        else:
            self.stop_camera()

    def start_camera(self):
        try:
            self.cap = cv2.VideoCapture(0)
            self.camera_active = True
            self.cam_btn.config(text="⏹️ Stop Camera", bg=self.colors['danger'])
            self.capture_btn.config(state='normal')

            # Start camera thread
            self.camera_thread = threading.Thread(target=self.update_camera, daemon=True)
            self.camera_thread.start()

        except Exception as e:
            messagebox.showerror("Error", f"Cannot access camera: {str(e)}")

    def stop_camera(self):
        self.camera_active = False
        if self.cap:
            self.cap.release()
        self.cam_btn.config(text="📷 Start Camera", bg=self.colors['primary'])
        self.capture_btn.config(state='disabled')
        self.camera_label.config(text="📷 Camera stopped\n\nClick 'Start Camera' to restart", image='')

    def update_camera(self):
        while self.camera_active:
            ret, frame = self.cap.read()
            if ret:
                # Convert to RGB and resize
                frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                frame_resized = cv2.resize(frame_rgb, (500, 400))

                # Convert to ImageTk
                img = Image.fromarray(frame_resized)
                imgtk = ImageTk.PhotoImage(image=img)

                # Update display
                self.camera_label.config(image=imgtk)
                self.camera_label.image = imgtk

                self.current_image = frame

            time.sleep(0.03)

    def capture_image(self):
        if self.current_image is not None:
            filename = f"crop_analysis_{datetime.now().strftime('%Y%m%d_%H%M%S')}.jpg"
            cv2.imwrite(filename, self.current_image)
            messagebox.showinfo("Success", f"✅ Image saved as {filename}")

    def upload_image(self):
        filename = filedialog.askopenfilename(
            title="Select Crop Image",
            filetypes=[("Image files", "*.jpg *.jpeg *.png *.bmp")]
        )
        if filename:
            try:
                img = Image.open(filename)
                img = img.resize((500, 400))
                imgtk = ImageTk.PhotoImage(image=img)

                self.camera_label.config(image=imgtk)
                self.camera_label.image = imgtk

                # Store for analysis
                self.current_image = cv2.cvtColor(np.array(img), cv2.COLOR_RGB2BGR)
                messagebox.showinfo("Success", "✅ Image uploaded successfully!")
            except Exception as e:
                messagebox.showerror("Error", f"Failed to load image: {str(e)}")

    def analyze_image(self):
        if self.current_image is None:
            messagebox.showwarning("Warning", "Please capture or upload an image first")
            return

        # Simulate hyperspectral analysis
        self.simulate_hyperspectral_analysis()

    def simulate_hyperspectral_analysis(self):
        # Show analyzing animation
        self.results_text.delete('1.0', 'end')
        self.results_text.insert('1.0', "🔬 Analyzing with hyperspectral imaging...\n\nPlease wait...")

        # Simulate processing time
        self.root.after(2000, self.show_analysis_results)

    def show_analysis_results(self):
        # Generate realistic analysis results
        health_score = random.randint(75, 95)
        nutrients = {
            'Nitrogen': random.randint(70, 95),
            'Phosphorus': random.randint(65, 90),
            'Potassium': random.randint(75, 92),
            'Calcium': random.randint(60, 85)
        }

        # Generate results text
        results = f"""
🌱 HYPESPECTRAL ANALYSIS REPORT
Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

📊 OVERALL HEALTH SCORE: {health_score}% {'✅' if health_score > 80 else '⚠️'}

🔬 NUTRIENT ANALYSIS:
   • Nitrogen: {nutrients['Nitrogen']}% {'✅' if nutrients['Nitrogen'] > 75 else '⚠️'}
   • Phosphorus: {nutrients['Phosphorus']}% {'✅' if nutrients['Phosphorus'] > 70 else '⚠️'}
   • Potassium: {nutrients['Potassium']}% {'✅' if nutrients['Potassium'] > 75 else '⚠️'}
   • Calcium: {nutrients['Calcium']}% {'✅' if nutrients['Calcium'] > 65 else '⚠️'}

⚠️ DISEASE DETECTION: {'None detected ✅' if random.random() > 0.3 else 'Early blight detected ⚠️'}

🐛 PEST RISK: {'Low 🟢' if random.random() > 0.4 else 'Moderate 🟡'}

💧 WATER STRESS: {random.randint(10, 30)}% (Optimal)

🌈 SPECTRAL INDICES:
   • NDVI: {random.uniform(0.6, 0.9):.3f}
   • Chlorophyll Index: {random.randint(75, 95)}%
   • Water Index: {random.randint(80, 95)}%

📋 RECOMMENDATIONS:
   • {'Apply balanced fertilizer' if any(v < 75 for v in nutrients.values()) else 'Nutrient levels optimal ✅'}
   • {'Increase irrigation frequency' if random.random() > 0.5 else 'Irrigation schedule optimal ✅'}
   • {'Monitor for pest activity' if random.random() > 0.6 else 'Pest risk low ✅'}

Next scan recommended: 7 days
"""

        self.results_text.delete('1.0', 'end')
        self.results_text.insert('1.0', results)

        # Update report tab
        self.update_report_content(results)

        messagebox.showinfo("Analysis Complete", "✅ Hyperspectral analysis completed!")

    def update_report_content(self, analysis_text):
        self.report_content.delete('1.0', 'end')

        # Enhanced report format
        enhanced_report = f"""
AGRITECH PRO - CROP ANALYSIS REPORT
===================================
Generated for: {self.user_data.get('name', 'User')}
Date: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
Report ID: AT{random.randint(1000, 9999)}

{analysis_text}

TECHNICAL DETAILS:
• Analysis Method: Hyperspectral Imaging
• Spectral Range: 400-1000 nm
• Spatial Resolution: 1cm/pixel
• Analysis Confidence: 92%

CONTACT:
🌐 www.agritech-pro.com
📞 Support: +1-800-AGRITECH
        """

        self.report_content.insert('1.0', enhanced_report)

    def download_report(self):
        content = self.report_content.get('1.0', 'end')
        if len(content.strip()) > 100:
            filename = f"AgriTech_Report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
            try:
                with open(filename, 'w') as f:
                    f.write(content)
                messagebox.showinfo("Download Complete",
                                    f"✅ Report saved as:\n{filename}\n\nIn production, this would be a PDF file.")
            except Exception as e:
                messagebox.showerror("Error", f"Failed to save report: {str(e)}")
        else:
            messagebox.showwarning("Warning", "No analysis data available to download")

    def show_dashboard(self):
        self.main_notebook.select(0)

    def show_analyze(self):
        self.main_notebook.select(1)

    def show_report(self):
        self.main_notebook.select(2)

    def show_settings(self):
        self.main_notebook.select(3)
      <img width="1891" height="1022" alt="Screenshot 2025-09-28 001622" src="https://github.com/user-attachments/assets/43cacfbd-464c-4844-bc1f-3f50305595f1" />

    def logout(self):
        if self.cap:
            self.cap.release()
        self.main_frame.pack_forget()
        self.logged_in = False
        self.create_auth_screen()


if __name__ == "__main__":
    root = tk.Tk()
    app = CropHealthApp(root)
    root.mainloop()
    
