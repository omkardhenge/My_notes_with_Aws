# My Notes App 📃 (Full Stack with AWS)

A full-stack cloud-native Notes application using:

- ✨ **Kotlin (Jetpack Compose)** for Android frontend  
- ⚡ **Node.js + Express** for backend (REST API)  
- ☁️ **AWS DynamoDB** for NoSQL storage  
- 📦 **Docker** for containerization  
- ⛅ **AWS EC2** for deployment  

---

## 📂 Folder Structure for backend

```
├── aws-server-code/             # Node.js backend with Express & AWS SDK
│   ├── routes/notes.js          # Express routers
│   ├── index.js                 # Main server code
│   ├── ddbClient.js             # Client file for DynamoDB
│   ├── Dockerfile               # Docker build file
│   ├── .dockerignore            # To ignore files while docker build
│   ├── package.json             # Node dependencies
├── .env                         # Environment variables (not committed)
```

---

## 📄 Features

### ✅ Frontend (Kotlin Jetpack Compose)
- Clean architecture with **data**, **domain**, **presentation**, **di** layers
- Login/Register screen
- Compose UI for listing & saving notes
- Retrofit client for API calls to EC2 backend

### ⚙️ Backend (Node.js + Express)
- `GET /notes` - Fetch all notes for user
- `POST /save-notes` - Save a new note
- Uses **AWS SDK v3** for DynamoDB
- Environment-driven configuration

### 🗃️ Database (DynamoDB)
- Table: `NotesTable`
- Partition Key: `userId`
- Sort Key: `noteId`

---

## 📆 Setup Instructions

### 1. Android Frontend (Kotlin Jetpack Compose)

#### 📥 Clone Kotlin App from GitHub
```bash
git clone https://github.com/omkardhenge/notes_app-master.git android-app
cd android-app
```

#### ⚙️ Configure API base URL
Update your Retrofit base URL to:
```
http://<ec2-ip>:3000
```

#### 🧱 Build the App
Open the project in Android Studio  
Use Retrofit to call backend APIs

---

### 2. Node.js Backend

#### ✅ Environment Variables (`.env`)
```
PORT=3000
AWS_REGION=ap-south-1
NOTES_TABLE=NotesTable
```

#### 📝 `index.js` Highlights
```js
require('dotenv').config();
const PORT = process.env.PORT || 3000;

app.listen(PORT, '0.0.0.0', () => {
  console.log(`🚀 Server running at http://0.0.0.0:${PORT}`);
});
```

---

## 🛠️ Docker

### 📄 Dockerfile
```Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "index.js"]
```

### 🔨 Build Image Locally
```bash
docker build -t aws-server-code .
```

### ▶️ Run Container Locally
```bash
docker run -d \
  -p 3000:3000 \
  -e PORT=3000 \
  -e AWS_REGION=ap-south-1 \
  -e NOTES_TABLE=NotesTable \
  --name mynotes-container \
  aws-server-code
```

---

## ✨ AWS EC2 Deployment

### 1. Launch EC2 Instance
- OS: **Amazon Linux 2**
- Security Group: **Allow inbound port 3000**

### 2. Connect via SSH
```bash
ssh -i your-key.pem ec2-user@<ec2-ip>
```

### 3. Setup Script (Full Bash)
```bash
#!/bin/bash

# Install Docker
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user
sudo chkconfig docker on

# Pull backend image
docker pull omkardhenge/aws-server-code

# Create .env
cat <<EOF > /home/ec2-user/.env
AWS_REGION=ap-south-1
NOTES_TABLE=NotesTable
EOF

# Run container
docker rm -f mynotes-container || true
docker run -d \
  --restart=always \
  --env-file /home/ec2-user/.env \
  -e PORT=3000 \
  -p 3000:3000 \
  --name mynotes-container \
  omkardhenge/aws-server-code
```

### 4. Check Logs
```bash
docker logs mynotes-container
```

### 5. Test API on EC2
```bash
curl http://localhost:3000/ping
```

#### Test from Postman
```
http://<ec2-public-ip>:3000/ping
```

---

## 🚀 DynamoDB Setup

1. Go to **AWS Console > DynamoDB**
2. Create Table:
   - Table name: `NotesTable`
   - Partition key: `userId` (String)
   - Sort key: `noteId` (String)

3. Add permissions to your EC2 IAM role  
   *(Or configure AWS credentials locally)*

---

## 📲 Android API Usage Example

```kotlin
@POST("/save-notes")
suspend fun saveNote(@Body note: NoteRequest): Response<Unit>

@GET("/notes")
suspend fun getNotes(@Query("userId") userId: String): Response<List<Note>>
```

---

## 🎓 Credits

Built by **Omkar Dhenge** 👨‍💼  
Hosted on **AWS EC2 + DynamoDB**  
Docker Image: [DockerHub - omkardhenge/aws-server-code](https://hub.docker.com/r/omkardhenge/aws-server-code)

---

## 🚫 Disclaimer

Do **not** commit `.env` file or AWS credentials to GitHub.  
Always use `.gitignore` to ignore sensitive files.

---

## 🌟 If this helped, give it a ⭐ or fork !
