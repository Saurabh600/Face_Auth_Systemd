# 🔐 AI-Based Face Authentication Web Login System
### Final Year Project | Computer Science | Python Flask + OpenCV + MySQL

---

## 📋 Project Overview

This project implements a **secure web-based login system** where users authenticate
using their **face** instead of traditional passwords.

### Problem Statement
Traditional password-based authentication suffers from:
- Password theft via phishing or data breaches
- Weak/reused passwords by users
- Brute force and credential stuffing attacks
- Poor user experience (forgotten passwords)

### Solution — Biometric Face Authentication
Our system replaces passwords with **facial recognition**:
1. Users register once — their face is captured (30 images) and stored
2. At login, the webcam captures their live face
3. AI compares the face to stored profiles using **128-dimensional embeddings**
4. Match found → access granted | Unknown face → access denied

### Security Advantages
| Feature | Password | Face Auth |
|---|---|---|
| Can be stolen | ✅ Yes | ❌ No |
| Can be guessed | ✅ Yes | ❌ No |
| Can be phished | ✅ Yes | ❌ No |
| Biometric binding | ❌ No | ✅ Yes |
| Audit trail | Optional | ✅ Built-in |

---

## 🏗️ Project Folder Structure

```
face-auth-system/
│
├── app.py                  ← Flask web server (all routes)
├── database.py             ← MySQL connection & query functions
├── capture_faces.py        ← Standalone webcam face capture script
├── recognize_face.py       ← Standalone face recognition test script
├── requirements.txt        ← Python package dependencies
├── schema.sql              ← MySQL database schema (run in phpMyAdmin)
│
├── dataset/
│   └── user_faces/
│       ├── alice/          ← Face images for user "alice"
│       │   ├── face_001.jpg
│       │   └── face_002.jpg ...
│       └── bob/            ← Face images for user "bob"
│
├── templates/              ← Jinja2 HTML templates (rendered by Flask)
│   ├── index.html          ← Landing / home page
│   ├── register.html       ← Registration + face capture
│   ├── login.html          ← Face login page
│   ├── dashboard.html      ← User dashboard (protected)
│   └── admin.html          ← Admin panel (admin role only)
│
└── static/
    ├── css/
    │   └── style.css       ← All page styles
    └── js/
        └── main.js         ← Global JS utilities
```

---

## ⚙️ STEP-BY-STEP SETUP GUIDE

### Step 1 — Prerequisites

Install these before starting:
- **Python 3.8–3.11** → https://python.org/downloads
- **XAMPP** (Apache + MySQL) → https://apachefriends.org
- **Git** (optional) → https://git-scm.com

---

### Step 2 — Install Python Dependencies

Open Command Prompt / Terminal in the project folder:

```bash
# Navigate to project folder
cd face-auth-system

# (Recommended) Create a virtual environment
python -m venv venv

# Activate it:
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# Install all required packages
pip install -r requirements.txt
```

#### What each package does:
| Package | Purpose |
|---|---|
| `flask` | Web framework — routes, templates, sessions |
| `opencv-python` | Webcam access, image processing, Haar cascade detection |
| `face-recognition` | dlib-powered 128D face encoding & matching |
| `numpy` | Numerical arrays for face distance calculations |
| `pymysql` | Connect Python to MySQL (pure Python, no C libs needed) |
| `Pillow` | Image file operations |

#### ⚠️ Windows Note — Installing face-recognition:
`face-recognition` requires `dlib` which requires C++ build tools.

**Option A** (install build tools):
1. Download Visual Studio Build Tools from https://visualstudio.microsoft.com/visual-cpp-build-tools/
2. Install "C++ build tools" workload
3. Then run: `pip install dlib face-recognition`

**Option B** (use pre-built wheel — easier):
1. Download dlib wheel for your Python version from:
   https://github.com/jloh02/dlib/releases
2. `pip install dlib-19.24.0-cp310-cp310-win_amd64.whl`
3. `pip install face-recognition`

---

### Step 3 — Setup MySQL Database

1. **Start XAMPP** → Open XAMPP Control Panel
2. Click **Start** next to **MySQL**
3. Click **Start** next to **Apache** (optional, for phpMyAdmin)
4. Open browser → go to http://localhost/phpmyadmin
5. Click the **SQL** tab at the top
6. **Copy the contents of `schema.sql`** and paste it into the SQL box
7. Click **Go**

This creates:
- Database: `face_auth_system`
- Tables: `users`, `login_history`, `admin_logs`
- Default admin account: username=`admin`, password=`admin123`

---

### Step 4 — Run the Flask Application

```bash
# Make sure your virtual environment is active!
# Make sure XAMPP MySQL is running!

python app.py
```

You should see:
```
* Running on http://0.0.0.0:5000
* Debug mode: on
```

Open browser → http://localhost:5000

---

### Step 5 — Register a User

1. Go to http://localhost:5000/register
2. Fill in: Full Name, Username, Email, Password
3. Click **"Start Camera"** → allow browser camera access
4. Click **"Start Auto-Capture"**
5. Look at the camera — 30 images will be captured automatically
6. Click **"Complete Registration"**

Your face images are now saved in `dataset/user_faces/<username>/`

---

### Step 6 — Test Face Login

1. Go to http://localhost:5000/login
2. Click **"Start Camera & Authenticate"**
3. Allow camera access, look at the camera
4. After 3 seconds, the system captures and analyzes your face
5. If recognized → redirected to dashboard
6. If unknown → "Access Denied" message shown

---

## 🔬 How the Face Recognition Works (Technical)

```
Registration Flow:
  Webcam → 30 JPEG frames → Base64 → Flask → Saved to disk

Login Flow:
  Webcam → 1 JPEG frame → Base64 → Flask
    ↓
  face_recognition.face_locations(frame)  ← Find face bounding boxes
    ↓
  face_recognition.face_encodings(frame)  ← Extract 128D embedding
    ↓
  face_recognition.face_distance(known_encodings, detected)
    ↓
  best_distance < 0.50?
    YES → Identity verified → Create session → Redirect dashboard
    NO  → Unknown face → Log attempt → Show "Access Denied"
```

### What is a "face encoding"?
dlib's ResNet-34 neural network maps each face to a **128-dimensional vector**
(128 floating-point numbers). Two photos of the same person produce vectors
very close to each other (distance ≈ 0.3). Photos of different people have
much larger distances (≈ 0.7+).

We use **Euclidean distance** as the similarity metric:
- Distance < 0.50 → Same person (our threshold)
- Distance > 0.50 → Different person

---

## 🛡️ Security Features

| Feature | Implementation |
|---|---|
| Unknown face detection | `face_distance > 0.50` → denied |
| Login attempt logging | Every attempt stored in `login_history` table |
| Session management | Flask server-side sessions with secret key |
| Password hashing | SHA-256 hash stored (never plaintext) |
| Input validation | Server-side + client-side validation |
| Admin access control | Role-based route protection |
| IP address logging | `request.remote_addr` captured per attempt |

---

## 🎓 VIVA PRESENTATION GUIDE

### Problem Statement (30 seconds)
> "Traditional password-based authentication has fundamental weaknesses — passwords
> can be stolen, guessed, or phished. My project replaces passwords with facial
> biometrics, which cannot be replicated or stolen in the same way."

### Technology Explanation (2 minutes)
- **Flask** — Python web framework. Handles HTTP routes and renders HTML pages.
- **OpenCV** — Computer vision library. Opens webcam, decodes images.
- **face_recognition + dlib** — AI library using ResNet-34 neural network.
  Converts a face photo into a unique 128-number "fingerprint" vector.
- **MySQL** — Stores user records and login history. Accessed via PyMySQL.
- **Bootstrap** — CSS framework for responsive, professional UI.

### System Architecture (show the diagram)
```
Browser (HTML/CSS/JS)
    ↕ HTTP (JSON + Form data)
Flask Backend (app.py)
    ├── Face Recognition Engine (face_recognition library)
    ├── Image Processing (OpenCV)
    └── Database Layer (database.py)
           ↕ SQL queries
         MySQL (XAMPP)
           ├── users table
           └── login_history table
```

### Demo Script
1. Show the home page and explain the concept
2. Register a new user — show webcam capturing face
3. Show dataset folder — point out the saved images
4. Login as that user — show face detection working
5. Try with another person's face — show "Access Denied"
6. Show the dashboard and login history
7. Show admin panel with full audit log

### Expected Viva Questions & Answers

**Q: What is face_distance and what threshold do you use?**
> face_distance returns the Euclidean distance between two 128D face encoding
> vectors. I use 0.50 — values below this indicate the same person.

**Q: How many images do you capture and why?**
> 30 images, to capture slight variations in angle, lighting, and expression.
> More images = more robust matching under varying conditions.

**Q: What happens if someone shows a photo of the user?**
> This is called a "spoof attack." Currently the system is vulnerable to it.
> An advanced version would add liveness detection (eye blink, head movement).
> This is one of my suggested future enhancements.

**Q: How do you store the password?**
> Using SHA-256 hashing. The plaintext password is never stored. However, in
> production I would use bcrypt or argon2 which include salting.

**Q: Is the face data secure?**
> Face images are stored on the local server filesystem, not in the database.
> Only the file path is stored in the database. For production, they should
> be stored encrypted or as embeddings only.

---

## 🚀 3 ADVANCED FEATURES TO ADD (Extra Marks!)

### Feature 1: Liveness Detection (Anti-Spoofing)
**Problem it solves:** Someone could hold a photo of your face to fool the camera.

**Implementation:**
```python
# Add eye blink detection using facial landmarks
import dlib
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor("shape_predictor_68_face_landmarks.dat")

# Calculate Eye Aspect Ratio (EAR)
# If EAR drops below threshold = blink detected
# Require 2+ blinks during login = proves live person
def eye_aspect_ratio(eye_landmarks):
    A = dist(eye_landmarks[1], eye_landmarks[5])
    B = dist(eye_landmarks[2], eye_landmarks[4])
    C = dist(eye_landmarks[0], eye_landmarks[3])
    return (A + B) / (2.0 * C)
```

**Why it matters for cybersecurity:**
Liveness detection is a mandatory requirement in biometric standards like
ISO/IEC 30107 (Presentation Attack Detection). Banks and government systems
require it.

---

### Feature 2: Rate Limiting & Account Lockout
**Problem it solves:** Brute force attempts using many face images.

**Implementation:**
```python
# In database.py — check recent failed attempts
def get_recent_failures(ip_address, minutes=10):
    sql = """
        SELECT COUNT(*) as cnt FROM login_history
        WHERE ip_address = %s
          AND status = 'failed'
          AND attempted_at > NOW() - INTERVAL %s MINUTE
    """

# In app.py — block after 5 failures in 10 minutes
failures = get_recent_failures(request.remote_addr, 10)
if failures >= 5:
    return jsonify({
        "success": False,
        "message": "Too many failed attempts. Try again in 10 minutes."
    })
```

**Why it matters:**
Rate limiting is a OWASP Top 10 recommendation. It prevents automated
attack tools from continuously trying to break in.

---

### Feature 3: Email OTP as Second Factor (2FA)
**Problem it solves:** Adds a second layer if face recognition is fooled.

**Implementation:**
```python
import smtplib, random
from email.mime.text import MIMEText

def send_otp_email(email, otp):
    msg = MIMEText(f"Your FaceAuth OTP: {otp}")
    msg['Subject'] = 'FaceAuth Login Verification'
    # Use smtplib to send via Gmail SMTP

# In login flow:
# 1. Face matches? → Generate 6-digit OTP
# 2. Email OTP to registered email
# 3. Show OTP input form
# 4. Only grant session after OTP verified
```

**Why it matters:**
Two-factor authentication is industry standard for any system holding
sensitive data. NIST recommends it for all authentication systems.

---

## 📊 Project Metrics

| Metric | Value |
|---|---|
| Face encoding dimensions | 128D |
| Match threshold | 0.50 |
| Recognition accuracy | ~99.38% (LFW benchmark) |
| Avg recognition speed | ~500ms |
| Images captured per user | 30 |
| Tables in database | 3 |
| Flask routes | 8 |
| Lines of code (approx) | ~1,500 |

---

*Built with ❤️ using Python Flask, OpenCV, face-recognition, MySQL*
