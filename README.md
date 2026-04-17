# 🚀 Interview AI

**Interview AI** is an AI-powered interview preparation platform that helps candidates prepare effectively for job interviews by generating **personalized interview reports, targeted questions, and structured preparation plans** based on their resume and job description.

---

## ✨ Features

- **AI-Generated Interview Reports**  
  Detailed interview analysis tailored to your resume and target role.

- **Technical & Behavioral Questions**  
  Curated questions with interviewer intent and answer guidelines.

- **Skill Gap Analysis**  
  Identifies missing or weak skills relevant to the applied role.

- **Personalized Preparation Plan**  
  Day-wise preparation roadmap to maximize interview readiness.

- **Resume PDF Generation**  
  Generate role-specific resumes in downloadable PDF format.

---

## 🛠️ Tech Stack

### Frontend
- React `19.2.0`
- React Router `7.13.0`
- Vite `7.3.1`
- Sass `1.97.3`
- Axios `1.13.5`

### Backend
- Node.js
- Express `5.2.1`
- MongoDB with Mongoose `9.2.1`
- Google Generative AI
- Puppeteer (PDF generation)
- JWT Authentication
- bcryptjs (Password hashing)

---

## 📋 Prerequisites

- Node.js `v18+`
- MongoDB (Local or MongoDB Atlas)
- Google AI API key  
  👉 https://makersuite.google.com/app/apikey

---

## ⚙️ Installation

### 1️⃣ Clone the repository
```bash
git clone https://github.com/ShridhiGupta/Interview-AI.git
cd Interview-AI
```

### 2️⃣ Install dependencies
```bash
# Backend dependencies
cd Backend
npm install

# Frontend dependencies  
cd ../Frontend
npm install
```

### 3️⃣ Set up environment variables
Create `Backend/.env` file:
```env
MONGO_URI=mongodb+srv://your-connection-string
JWT_SECRET=your-jwt-secret-key
GOOGLE_GENAI_API_KEY=your-google-ai-api-key
PORT=3000
```

### 4️⃣ Start the application
```bash
# Terminal 1 - Backend (in Backend directory)
npm run dev

# Terminal 2 - Frontend (in Frontend directory)  
npm run dev
```

---

## 🌐 Access

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:3000

---

## 📖 Usage Guide

### 1️⃣ Create Account
- Register with email and password
- Login to access the dashboard

### 2️⃣ Prepare Your Profile
- Upload your resume (PDF/TXT format)
- Add a brief self-description
- Enter the target job description

### 3️⃣ Generate Interview Report
- Click "Generate Report" button
- AI analyzes your profile against job requirements
- Receive comprehensive interview preparation guide

### 4️⃣ Review Results
- **Technical Questions**: Role-specific technical interview questions
- **Behavioral Questions**: Situational and behavioral questions
- **Skill Gaps**: Areas needing improvement
- **Preparation Plan**: Day-wise study schedule

### 5️⃣ Download Resume
- Generate tailored resume PDF
- Optimized for the specific job description
- Professional formatting ready for applications

---

## 🔗 API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user
- `GET /api/auth/logout` - Logout user
- `GET /api/auth/get-me` - Get current user info

### Interview
- `POST /api/interview/generate-report` - Generate interview report
- `POST /api/interview/generate-resume` - Generate resume PDF

---

## 🛡️ Security Features

- **JWT Authentication**: Secure token-based authentication
- **Password Hashing**: bcrypt with salt rounds for security
- **Input Validation**: Zod schemas for data validation
- **CORS Protection**: Cross-origin request security
- **Environment Variables**: Sensitive data protection

---

## 🚀 Project Structure

```
Interview-AI/
├── Backend/                 # Node.js API server
│   ├── src/
│   │   ├── controllers/     # Request handlers
│   │   ├── models/          # MongoDB schemas
│   │   ├── routes/          # API endpoints
│   │   ├── services/        # Business logic
│   │   ├── middlewares/     # Auth & validation
│   │   └── config/          # Database connection
│   └── package.json
├── Frontend/                # React application
│   ├── src/
│   │   ├── features/       # Feature-based components
│   │   │   ├── auth/     # Authentication components
│   │   │   └── interview/ # Interview functionality
│   │   └── style/        # Global styles
│   └── package.json
├── README.md              # Project documentation
├── understanding.md       # Technical deep-dive
└── .gitignore           # Git ignore rules
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

---

## 📝 License

This project is licensed under the ISC License.

---

## 👨‍💻 Author

Created by **Shridhi Gupta**  
🔗 GitHub: [ShridhiGupta](https://github.com/ShridhiGupta)

---

## 🙏 Acknowledgments

- **Google Generative AI** - For powerful AI capabilities
- **React Community** - Excellent documentation and tools
- **Open Source Contributors** - For amazing libraries and tools