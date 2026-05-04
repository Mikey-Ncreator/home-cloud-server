from flask import Flask, request, render_template_string, send_from_directory, redirect, url_for, session
import os
import sqlite3
import hashlib
from functools import wraps
from datetime import datetime

app = Flask(__name__)
app.secret_key = "change-this-to-something-secret"

os.makedirs("files", exist_ok=True)

# ========== DATABASE ==========
def init_db():
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users (
        username TEXT PRIMARY KEY,
        password TEXT,
        banned INTEGER DEFAULT 0,
        is_admin INTEGER DEFAULT 0,
        created_at TEXT
    )''')
    c.execute('''CREATE TABLE IF NOT EXISTS shares (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        owner TEXT,
        filename TEXT,
        shared_with TEXT,
        shared_at TEXT
    )''')
    
    # Create admin user (password: admin123)
    admin_pass = hashlib.sha256("admin123".encode()).hexdigest()
    c.execute("INSERT OR IGNORE INTO users VALUES ('admin', ?, 0, 1, ?)", 
              (admin_pass, datetime.now().strftime("%Y-%m-%d %H:%M")))
    
    conn.commit()
    conn.close()
    print("✅ Database initialized")

init_db()

def login_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if 'username' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated

# ========== LOGIN (with ban check) ==========
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = hashlib.sha256(request.form['password'].encode()).hexdigest()
        
        conn = sqlite3.connect("users.db")
        c = conn.cursor()
        c.execute("SELECT banned, is_admin FROM users WHERE username=? AND password=?", (username, password))
        row = c.fetchone()
        conn.close()
        
        if not row:
            return "<h2>❌ Invalid credentials</h2><a href='/login'>Try again</a>"
        
        if row[0] == 1:
            return "<h2>🚫 Your account is BANNED</h2><a href='/login'>Contact admin</a>"
        
        session['username'] = username
        session['is_admin'] = row[1]
        return redirect(url_for('index'))
    
    return '''
    <!DOCTYPE html>
    <html>
    <head><meta name="viewport" content="width=device-width, initial-scale=1"><title>Login</title>
    <style>body{font-family:Arial;display:flex;justify-content:center;align-items:center;height:100vh;background:linear-gradient(135deg,#667eea,#764ba2);margin:0}.card{background:white;padding:40px;border-radius:15px;width:320px;text-align:center}h2{margin:0 0 20px 0}input{width:100%;padding:12px;margin:10px 0;border:1px solid #ddd;border-radius:8px}button{width:100%;padding:12px;background:#667eea;color:white;border:none;border-radius:8px;cursor:pointer}</style></head>
    <body><div class="card"><h2>☁️ Home Cloud</h2><form method="post"><input name="username" placeholder="Username" required><input name="password" type="password" placeholder="Password" required><button type="submit">Login</button></form><p style="margin-top:15px;font-size:12px;color:#999;">admin / admin123</p></div></body></html>
    '''

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('login'))

# ========== MAIN DASHBOARD ==========
@app.route('/')
@login_required
def index():
    username = session['username']
    is_admin = session['is_admin']
    
    # Get user's files
    all_files = os.listdir("files") if os.path.exists("files") else []
    my_files = []
    for f in all_files:
        if f.startswith(username + "_"):
            filepath = os.path.join("files", f)
            my_files.append({
                'name': f.replace(username + "_", ""),
                'fullname': f,
                'size': os.path.getsize(filepath),
                'modified': datetime.fromtimestamp(os.path.getmtime(filepath)).strftime("%Y-%m-%d %H:%M")
            })
    my_files.sort(key=lambda x: x['modified'], reverse=True)
    
    # Get files shared with me
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("SELECT owner, filename, shared_at FROM shares WHERE shared_with=?", (username,))
    shared_with_me = c.fetchall()
    
    # Get all users for share dropdown
    c.execute("SELECT username FROM users WHERE banned=0")
    all_users = c.fetchall()
    
    # Get all users for admin panel
    c.execute("SELECT username, banned, is_admin, created_at FROM users")
    all_users_list = c.fetchall()
    conn.close()
    
    return render_template_string(MAIN_TEMPLATE, 
                                  username=username, 
                                  is_admin=is_admin,
                                  my_files=my_files,
                                  shared_with_me=shared_with_me,
                                  all_users=all_users,
                                  all_users_list=all_users_list)

MAIN_TEMPLATE = '''
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Home Cloud - {{ username }}</title>
<style>
*{box-sizing:border-box}
body{font-family:Arial;margin:0;padding:20px;background:#f0f2f5}
.container{max-width:1200px;margin:0 auto}
.header{background:linear-gradient(135deg,#667eea,#764ba2);color:white;padding:20px;border-radius:15px;margin-bottom:20px}
.header h1{margin:0;display:inline-block}
.header-right{float:right;display:flex;gap:10px}
.header-right a,.header-right button{background:rgba(255,255,255,0.2);padding:8px 16px;border-radius:8px;color:white;text-decoration:none;border:none;cursor:pointer}
.tabs{display:flex;gap:10px;margin-bottom:20px;flex-wrap:wrap}
.tab{background:white;padding:12px 24px;border-radius:10px;cursor:pointer;border:none;font-size:14px}
.tab.active{background:#667eea;color:white}
.tab-content{display:none}
.tab-content.active{display:block}
.card{background:white;border-radius:15px;padding:20px;margin-bottom:20px;box-shadow:0 2px 5px rgba(0,0,0,0.1)}
.file-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:15px}
.file-card{background:#f9f9f9;border-radius:10px;padding:12px;border:1px solid #ddd}
.file-actions{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
.btn{padding:5px 12px;border-radius:5px;text-decoration:none;font-size:12px;border:none;cursor:pointer}
.btn-view{background:#2196F3;color:white}
.btn-download{background:#4caf50;color:white}
.btn-delete{background:#f44336;color:white}
.btn-share{background:#ff9800;color:white}
.btn-reset{background:#9c27b0;color:white}
.upload-area{border:2px dashed #ddd;border-radius:10px;padding:40px;text-align:center;cursor:pointer;background:#fafafa}
.upload-area:hover{border-color:#667eea}
input[type="file"]{display:none}
table{width:100%;border-collapse:collapse}
th,td{padding:10px;text-align:left;border-bottom:1px solid #ddd}
input,select{padding:8px;margin:5px;border:1px solid #ddd;border-radius:5px}
button{cursor:pointer}
.modal{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);justify-content:center;align-items:center;z-index:1000}
.modal-content{background:white;padding:20px;border-radius:15px;width:400px}
.warning{background:#ff9800;color:white;padding:5px 10px;border-radius:5px;font-size:12px}
.reset-modal input{width:100%}
@media (max-width:600px){.file-grid{grid-template-columns:1fr}}
</style>
</head>
<body>
<div class="container">
<div class="header">
<h1>☁️ {{ username }}'s Cloud</h1>
<div class="header-right">
<a href="/profile">👤 Profile</a>
<a href="/logout">🚪 Logout</a>
</div>
</div>

<div class="tabs">
<button class="tab active" onclick="showTab('myfiles')">📁 My Files</button>
<button class="tab" onclick="showTab('shared')">🔗 Shared With Me</button>
<button class="tab" onclick="showTab('upload')">📤 Upload</button>
{% if is_admin %}<button class="tab" onclick="showTab('admin')">👑 Admin Panel</button>{% endif %}
</div>

<!-- My Files Tab -->
<div id="myfiles" class="tab-content active">
<div class="card">
<h3>Your Files</h3>
<div class="file-grid">
{% for f in my_files %}
<div class="file-card">
<div style="font-size:40px;text-align:center">📄</div>
<div><strong>{{ f.name[:35] }}</strong><br><small>{{ (f.size/1024)|round(1) }} KB • {{ f.modified }}</small></div>
<div class="file-actions">
<a href="/file/{{ f.fullname }}" target="_blank" class="btn btn-view">View</a>
<a href="/download/{{ f.fullname }}" class="btn btn-download">Download</a>
<button onclick="shareFile('{{ f.fullname }}','{{ f.name }}')" class="btn btn-share">Share</button>
<a href="/delete/{{ f.fullname }}" onclick="return confirm('Delete this file?')" class="btn btn-delete">Delete</a>
</div>
</div>
{% endfor %}
</div>
{% if not my_files %}<p style="text-align:center;color:#999;">No files yet. Upload something!</p>{% endif %}
</div>
</div>

<!-- Shared Tab -->
<div id="shared" class="tab-content">
<div class="card">
<h3>Files Shared With You</h3>
<div class="file-grid">
{% for owner, filename, shared_at in shared_with_me %}
<div class="file-card">
<div style="font-size:40px;text-align:center">🔗</div>
<div><strong>{{ filename }}</strong><br><small>From: {{ owner }} • {{ shared_at }}</small></div>
<div class="file-actions">
<a href="/shared_file/{{ owner }}/{{ filename }}" target="_blank" class="btn btn-view">View</a>
<a href="/download_shared/{{ owner }}/{{ filename }}" class="btn btn-download">Download</a>
</div>
</div>
{% endfor %}
</div>
{% if not shared_with_me %}<p style="text-align:center;color:#999;">No shared files yet</p>{% endif %}
</div>
</div>

<!-- Upload Tab -->
<div id="upload" class="tab-content">
<div class="card">
<h3>Upload Files</h3>
<div class="upload-area" onclick="document.getElementById('fileInput').click()">
<p>📁 Click or drag files here</p>
</div>
<form id="uploadForm" method="post" action="/upload" enctype="multipart/form-data">
<input type="file" name="file" id="fileInput" multiple style="display:none" onchange="this.form.submit()">
</form>
</div>
</div>

<!-- Admin Tab -->
{% if is_admin %}
<div id="admin" class="tab-content">
<div class="card">
<h3>👑 User Management</h3>
<table style="width:100%; overflow-x:auto; display:block">
<thead>
<tr><th>User</th><th>Status</th><th>Role</th><th>Created</th><th>Actions</th></tr>
</thead>
<tbody>
{% for u in all_users_list %}
<tr>
<td><strong>{{ u[0] }}</strong></td>
<td>{% if u[1] %}🔴 BANNED{% else %}🟢 Active{% endif %}</td>
<td>{% if u[2] %}👑 Admin{% else %}📁 User{% endif %}</td>
<td>{{ u[3] or 'Unknown' }}</td>
<td>
{% if u[0] != 'admin' %}
    {% if u[1] %}
        <a href="/unban/{{ u[0] }}" class="btn btn-view" style="background:#4caf50">Unban</a>
    {% else %}
        <a href="/ban/{{ u[0] }}" class="btn btn-delete">Ban</a>
    {% endif %}
    <button onclick="openResetModal('{{ u[0] }}')" class="btn btn-reset">Reset PW</button>
    <a href="/delete_user/{{ u[0] }}" class="btn btn-delete" onclick="return confirm('Delete user {{ u[0] }} and ALL their files? This cannot be undone!')">Delete User</a>
{% else %}
    <span style="color:#999">Protected</span>
{% endif %}
</td>
</tr>
{% endfor %}
</tbody>
</table>

<h4 style="margin-top:20px">➕ Create New User</h4>
<form method="post" action="/create_user" style="display:flex; gap:10px; flex-wrap:wrap; align-items:center">
<input name="username" placeholder="Username" required>
<input name="password" type="password" placeholder="Password" required>
<button type="submit" class="btn btn-view">Create User</button>
</form>

<hr>
<h4>📊 Database Stats</h4>
<p>Total users: {{ all_users_list|length }}</p>
</div>
</div>
{% endif %}
</div>

<!-- Share Modal -->
<div id="shareModal" class="modal">
<div class="modal-content">
<h3>Share File</h3>
<p id="shareFileName"></p>
<select id="shareUser" style="width:100%">
{% for u in all_users %}
<option value="{{ u[0] }}">{{ u[0] }}</option>
{% endfor %}
</select>
<button onclick="submitShare()" style="width:100%; margin-top:10px" class="btn btn-view">Share</button>
<button onclick="closeShareModal()" style="width:100%; margin-top:5px" class="btn btn-delete">Cancel</button>
</div>
</div>

<!-- Reset Password Modal -->
<div id="resetModal" class="modal">
<div class="modal-content">
<h3>Reset Password</h3>
<p>User: <strong id="resetUsername"></strong></p>
<input type="password" id="newPassword" placeholder="Enter new password" style="width:100%; padding:10px; margin:10px 0">
<button onclick="submitReset()" style="width:100%; margin-top:10px" class="btn btn-view">Set Password</button>
<button onclick="closeResetModal()" style="width:100%; margin-top:5px" class="btn btn-delete">Cancel</button>
</div>
</div>

<script>
let currentShareFile = null;
let currentResetUser = null;

function showTab(tabId) {
    document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    document.getElementById(tabId).classList.add('active');
    event.target.classList.add('active');
}

function shareFile(fullname, name) {
    currentShareFile = fullname;
    document.getElementById('shareFileName').innerText = name;
    document.getElementById('shareModal').style.display = 'flex';
}

function closeShareModal() {
    document.getElementById('shareModal').style.display = 'none';
}

function submitShare() {
    const user = document.getElementById('shareUser').value;
    if (!user) {
        alert('Select a user');
        return;
    }
    window.location.href = '/share/' + encodeURIComponent(currentShareFile) + '/' + encodeURIComponent(user);
}

function openResetModal(username) {
    currentResetUser = username;
    document.getElementById('resetUsername').innerText = username;
    document.getElementById('newPassword').value = '';
    document.getElementById('resetModal').style.display = 'flex';
}

function closeResetModal() {
    document.getElementById('resetModal').style.display = 'none';
}

function submitReset() {
    const newPassword = document.getElementById('newPassword').value;
    if (!newPassword || newPassword.length < 3) {
        alert('Password must be at least 3 characters');
        return;
    }
    window.location.href = '/reset_password/' + encodeURIComponent(currentResetUser) + '/' + encodeURIComponent(newPassword);
}

const dropzone = document.querySelector('.upload-area');
if (dropzone) {
    dropzone.addEventListener('dragover', (e) => {
        e.preventDefault();
        dropzone.style.borderColor = '#667eea';
    });
    dropzone.addEventListener('dragleave', () => {
        dropzone.style.borderColor = '#ddd';
    });
    dropzone.addEventListener('drop', (e) => {
        e.preventDefault();
        dropzone.style.borderColor = '#ddd';
        const files = e.dataTransfer.files;
        const input = document.getElementById('fileInput');
        input.files = files;
        document.getElementById('uploadForm').submit();
    });
}
</script>
</body>
</html>
'''

# ========== FILE OPERATIONS ==========
@app.route('/upload', methods=['POST'])
@login_required
def upload():
    username = session['username']
    files = request.files.getlist('file')
    for file in files:
        if file and file.filename:
            file.save(os.path.join("files", f"{username}_{file.filename}"))
    return redirect(url_for('index'))

@app.route('/file/<filename>')
@login_required
def view_file(filename):
    username = session['username']
    if filename.startswith(username + "_") or session.get('is_admin'):
        return send_from_directory("files", filename)
    return "Access denied", 403

@app.route('/download/<filename>')
@login_required
def download_file(filename):
    username = session['username']
    if filename.startswith(username + "_") or session.get('is_admin'):
        return send_from_directory("files", filename, as_attachment=True)
    return "Access denied", 403

@app.route('/delete/<filename>')
@login_required
def delete_file(filename):
    username = session['username']
    if filename.startswith(username + "_") or session.get('is_admin'):
        filepath = os.path.join("files", filename)
        if os.path.exists(filepath):
            os.remove(filepath)
    return redirect(url_for('index'))

# ========== SHARE OPERATIONS ==========
@app.route('/share/<filename>/<target_user>')
@login_required
def share_file(filename, target_user):
    username = session['username']
    clean_name = filename.replace(username + "_", "")
    
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("INSERT INTO shares (owner, filename, shared_with, shared_at) VALUES (?, ?, ?, ?)",
              (username, clean_name, target_user, datetime.now().strftime("%Y-%m-%d %H:%M")))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

@app.route('/shared_file/<owner>/<filename>')
@login_required
def shared_file(owner, filename):
    username = session['username']
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("SELECT 1 FROM shares WHERE owner=? AND filename=? AND shared_with=?", (owner, filename, username))
    row = c.fetchone()
    conn.close()
    if row or session.get('is_admin'):
        return send_from_directory("files", f"{owner}_{filename}")
    return "Access denied", 403

@app.route('/download_shared/<owner>/<filename>')
@login_required
def download_shared(owner, filename):
    username = session['username']
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("SELECT 1 FROM shares WHERE owner=? AND filename=? AND shared_with=?", (owner, filename, username))
    row = c.fetchone()
    conn.close()
    if row or session.get('is_admin'):
        return send_from_directory("files", f"{owner}_{filename}", as_attachment=True)
    return "Access denied", 403

# ========== ADMIN OPERATIONS ==========
@app.route('/create_user', methods=['POST'])
@login_required
def create_user():
    if not session.get('is_admin'):
        return "Access denied", 403
    
    new_username = request.form['username']
    new_password = hashlib.sha256(request.form['password'].encode()).hexdigest()
    
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    try:
        c.execute("INSERT INTO users (username, password, created_at) VALUES (?, ?, ?)",
                  (new_username, new_password, datetime.now().strftime("%Y-%m-%d %H:%M")))
        conn.commit()
        print(f"✅ User created: {new_username}")
    except sqlite3.IntegrityError:
        print(f"❌ User exists: {new_username}")
    conn.close()
    return redirect(url_for('index'))

@app.route('/ban/<user>')
@login_required
def ban_user(user):
    if not session.get('is_admin'):
        return "Access denied", 403
    if user == 'admin':
        return "Cannot ban admin", 403
    
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("UPDATE users SET banned=1 WHERE username=?", (user,))
    conn.commit()
    
    # Verify it worked
    c.execute("SELECT banned FROM users WHERE username=?", (user,))
    result = c.fetchone()
    conn.close()
    print(f"✅ Banned {user}: banned={result[0] if result else 'not found'}")
    return redirect(url_for('index'))

@app.route('/unban/<user>')
@login_required
def unban_user(user):
    if not session.get('is_admin'):
        return "Access denied", 403
    
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("UPDATE users SET banned=0 WHERE username=?", (user,))
    conn.commit()
    conn.close()
    print(f"✅ Unbanned {user}")
    return redirect(url_for('index'))

@app.route('/delete_user/<user>')
@login_required
def delete_user(user):
    if not session.get('is_admin'):
        return "Access denied", 403
    if user == 'admin':
        return "Cannot delete admin", 403
    
    # Delete user's files
    for f in os.listdir("files"):
        if f.startswith(user + "_"):
            os.remove(os.path.join("files", f))
    
    # Delete from database
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("DELETE FROM users WHERE username=?", (user,))
    c.execute("DELETE FROM shares WHERE owner=? OR shared_with=?", (user, user))
    conn.commit()
    conn.close()
    print(f"✅ Deleted user: {user}")
    return redirect(url_for('index'))

@app.route('/reset_password/<user>/<new_password>')
@login_required
def reset_password(user, new_password):
    if not session.get('is_admin'):
        return "Access denied", 403
    if user == 'admin':
        return "Cannot reset admin password", 403
    
    new_hash = hashlib.sha256(new_password.encode()).hexdigest()
    conn = sqlite3.connect("users.db")
    c = conn.cursor()
    c.execute("UPDATE users SET password=? WHERE username=?", (new_hash, user))
    conn.commit()
    conn.close()
    print(f"✅ Reset password for {user} to: {new_password}")
    return redirect(url_for('index'))

# ========== PROFILE ==========
@app.route('/profile', methods=['GET', 'POST'])
@login_required
def profile():
    username = session['username']
    message = None
    
    if request.method == 'POST':
        if 'new_username' in request.form:
            new_name = request.form['new_username']
            if new_name and new_name != username:
                conn = sqlite3.connect("users.db")
                c = conn.cursor()
                c.execute("SELECT 1 FROM users WHERE username=?", (new_name,))
                if c.fetchone():
                    message = "Username taken"
                else:
                    c.execute("UPDATE users SET username=? WHERE username=?", (new_name, username))
                    for f in os.listdir("files"):
                        if f.startswith(username + "_"):
                            os.rename(os.path.join("files", f), os.path.join("files", f.replace(username, new_name, 1)))
                    c.execute("UPDATE shares SET owner=? WHERE owner=?", (new_name, username))
                    c.execute("UPDATE shares SET shared_with=? WHERE shared_with=?", (new_name, username))
                    conn.commit()
                    session['username'] = new_name
                    message = f"Username changed to {new_name}"
                conn.close()
        
        elif 'old_password' in request.form:
            old_hash = hashlib.sha256(request.form['old_password'].encode()).hexdigest()
            new_pass = request.form['new_password']
            confirm = request.form['confirm_password']
            
            conn = sqlite3.connect("users.db")
            c = conn.cursor()
            c.execute("SELECT password FROM users WHERE username=?", (username,))
            current = c.fetchone()
            
            if current[0] != old_hash:
                message = "Wrong current password"
            elif new_pass != confirm:
                message = "Passwords don't match"
            elif len(new_pass) < 4:
                message = "Password too short"
            else:
                new_hash = hashlib.sha256(new_pass.encode()).hexdigest()
                c.execute("UPDATE users SET password=? WHERE username=?", (new_hash, username))
                conn.commit()
                message = "Password changed"
            conn.close()
    
    return render_template_string('''
    <!DOCTYPE html>
    <html>
    <head><meta name="viewport" content="width=device-width, initial-scale=1"><title>Profile</title>
    <style>body{font-family:Arial;padding:20px;background:#f0f2f5}.container{max-width:500px;margin:0 auto}.card{background:white;border-radius:15px;padding:20px;margin-bottom:20px}input{width:100%;padding:10px;margin:10px 0;border:1px solid #ddd;border-radius:8px}button{background:#667eea;color:white;border:none;padding:10px;border-radius:8px;cursor:pointer}.msg{background:#4caf50;color:white;padding:10px;border-radius:8px;margin-bottom:10px}</style></head>
    <body><div class="container"><div class="card"><h2>👤 Profile: {{ username }}</h2><a href="/">← Back</a></div>
    {% if message %}<div class="msg">{{ message }}</div>{% endif %}
    <div class="card"><h3>Change Username</h3><form method="post"><input name="new_username" placeholder="New username" required><button type="submit">Change</button></form></div>
    <div class="card"><h3>Change Password</h3><form method="post"><input name="old_password" type="password" placeholder="Current password" required><input name="new_password" type="password" placeholder="New password (min 4 chars)" required><input name="confirm_password" type="password" placeholder="Confirm" required><button type="submit">Change</button></form></div>
    </div></body></html>
    ''', username=username, message=message)

# ========== RUN ==========
if __name__ == '__main__':
    print("\n" + "="*50)
    print("🚀 Home Cloud Server v5.0")
    print("📍 Access at: http://192.168.1.100:5000")
    print("👑 Admin: admin / admin123")
    print("="*50 + "\n")
    app.run(host='0.0.0.0', port=5000, debug=False)
