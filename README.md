# ☁️ Home Cloud Server

A **production-ready, self-hosted cloud storage system** running on Intel Atom 2GB RAM + 300GB HDD.

![Python](https://img.shields.io/badge/Python-3.11-blue.svg)
![Flask](https://img.shields.io/badge/Flask-2.3+-green.svg)
![Status](https://img.shields.io/badge/Status-Production-brightgreen.svg)

---

## 🖥️ Hardware Specs

| Component | Specification |
|-----------|---------------|
| **CPU** | Intel Atom |
| **RAM** | 2GB |
| **Storage** | 300GB HDD |
| **OS** | Debian Linux |
| **RAM Usage** | ~50MB |

---

## 🚀 Features

### For Users
- ✅ Secure login with password hashing (SHA256)
- ✅ Upload/download any file type
- ✅ File preview (images, PDFs, videos, audio)
- ✅ Share files with other users
- ✅ Change username/password
- ✅ Mobile responsive design

### For Admins
- ✅ Create/delete users
- ✅ Ban/unban users (blocks login)
- ✅ Reset any user's password
- ✅ View all users and their status

---

## 📋 Complete Installation Commands

```bash
# Update system
sudo apt update
sudo apt install python3 python3-pip python3-full -y

# Create project
mkdir ~/cloud_app
cd ~/cloud_app
python3 -m venv venv
source venv/bin/activate
pip install flask

# Create app.py (copy from this repository)
nano app.py

# Run server
python app.py



#Access at: http://192.168.1.100:5000

#Default login: admin / admin123
