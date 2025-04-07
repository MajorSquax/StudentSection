# Setting Up Student Section

## Prerequisites
Before getting started, ensure you have the following installed on your machine:
- **Python** (latest version recommended)
- **Node.js & npm** (for the frontend)
- **Virtual Environment** (for Python dependencies)
- **MongoDB:** Ensure IP address is whitelisted in Mongo
- **Redis:** Set up Redis running on port 6379 on the localhost/127.0.0.1

  _note: Redis is Linux native, so this will need to be run in WSL on Windows_
## 🛠Installation & Setup
### 1️⃣ Set Up the Backend
1. **Create & Activate a Virtual Environment** (if not already set up):
   ```sh
   python -m venv venv  # Create a virtual environment
   source venv/bin/activate  # Mac/Linux
   venv\Scripts\activate  # Windows
   ```
2. **Install Python Dependencies**:
   ```sh
   pip install -r requirements.txt
   ```
3. **Generate Redis Secret Key**:
   ```sh
   python -c "import secrets; print(secrets.token_hex())"
   export SECRET_KEY=<TOKEN_FROM_ABOVE>
   ```
   Note: use _set_ if on Windows
4. **Create the mockoon.service file in your /etc/systemd/system/ folder. This will run Mockoon on port 3003**:
   ```sh
   [Unit]
   Description=ROT13 demo service
   After=network.target
   StartLimitIntervalSec=0

   [Service]
   Type=simple
   Restart=always
   RestartSec=1
   User=studentsection
   ExecStart=mockoon-cli start --data /StudentSection/PaciolanMockAPIv1v2.json -p 3003

   [Install]
   WantedBy=multi-user.target
   ```
5. **Start the Backend**:
  Create the following file studentsection.service in your /etc/systemd/system/ folder
   ```sh
   [Unit]
   Description=Gunicorn instance to serve studentsection flask app
   After=network.target
   [Service]
   User=studentsection
   Group=www-data
   WorkingDirectory=/StudentSection
   Environment="PATH=/StudentSection/venv/bin"
   ExecStart=/StudentSection/venv/bin/gunicorn --workers 3 --error-logfile /var/log/studentsection/gunicorn-error.log --access-logfile /var/log/studentsection/gunicorn-access.log --bind unix:studentsection.sock -m 007 wsgi:app
   [Install]
   WantedBy=multi-user.target
   ```
### 2️⃣ Set Up the Frontend
1. **Navigate to the Frontend Directory**:
   ```sh
   cd student-section-frontend
   ```
2. **Install React Dependencies**:
   ```sh
   npm ci
   ```
3. **Build the React Frontend**:
   ```sh
   npm run build
   ```
4. **Create the reactfrontend.service file under the folder /etc/systemd/system **:
   ```sh
   [Unit]
   Description=Start the React Server over port 3000
   After=network.target

   [Service]
   Type=simple
   WorkingDirectory=/StudentSection/student-section-frontend
   ExecStart=serve -s build
   Restart=always
   User=studentsection
   StandardOutput=append:/var/log/student-section-frontend.log
   StandardError=append:/var/log/student-section-frontend.err.log

   [Install]
   WantedBy=multi-user.target
   ```
5. **Start the React Static Server**:
   ```sh
   systemctl start reactfrontend
   ```
## Final Steps
- Your **backend** should now be running on `https://studentsection.xyz/flaskapi`
- Your **frontend** should be accessible at `http://localhost:3000`
- Keep both processes running in **separate terminal windows** or **use a split terminal**
