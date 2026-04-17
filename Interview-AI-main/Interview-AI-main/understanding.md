# Interview AI - Technical Understanding

## 1. High-Level Overview

### Problem Solved
This project solves the challenge of **personalized interview preparation**. Traditional interview prep is generic - candidates study common questions without knowing what's relevant to their specific role and company. This AI-powered system analyzes a candidate's resume, self-description, and target job description to generate:

- Personalized interview questions (technical + behavioral)
- Skill gap analysis 
- Day-wise preparation plans
- Tailored resume PDFs

### Architecture Choice
**Why this architecture?**
- **React SPA**: Provides responsive, interactive UI for form inputs and real-time feedback
- **Node.js/Express**: Lightweight, fast backend perfect for API-driven AI services
- **MongoDB**: Flexible schema for varied user profiles and interview data
- **Microservice-ready**: Clean separation allows easy scaling of AI processing

### System Design
```
Frontend (React) ↔ Backend (Express) ↔ MongoDB Atlas
                              ↕
                        Google Gemini AI
                              ↕
                        Puppeteer (PDF)
```

## 2. Project Structure Walkthrough

### Frontend Structure
```
Frontend/
├── src/
│   ├── features/           # Feature-based organization
│   │   ├── auth/         # Authentication components
│   │   └── interview/    # Core interview functionality
│   └── style/            # Global styles & components
```

**Why this structure?**
- **Feature folders**: Cohesive code organization - all auth logic together
- **Separation of concerns**: UI components, styles, and logic separated
- **Scalability**: Easy to add new features without refactoring

### Backend Structure
```
Backend/
├── src/
│   ├── config/          # Database connection
│   ├── controllers/     # Request handling logic
│   ├── middlewares/     # Auth, validation
│   ├── models/         # MongoDB schemas
│   ├── routes/         # API endpoints
│   └── services/       # Business logic & external APIs
```

**Clean Architecture Principles:**
- **Controllers**: Handle HTTP requests/responses only
- **Services**: Contain business logic, external API calls
- **Models**: Define data structure and validation
- **Middlewares**: Cross-cutting concerns (auth, validation)

## 3. Frontend Architecture

### Tech Stack & Why
- **React 19**: Latest features, component-based UI
- **React Router**: Client-side routing for SPA experience
- **Vite**: Fast development server and builds
- **Sass**: CSS preprocessing for maintainable styles
- **Axios**: HTTP client with better error handling than fetch

### Component Structure
```
App
├── Router
├── Auth Features
│   ├── Login
│   └── Register
└── Interview Features
    ├── Home (form inputs)
    └── Interview (results display)
```

### State Management
**Current approach**: Component state + props
- **Why**: Simple for current scope, no complex state sharing needed
- **Future**: Could add Redux/Zustand for global state

### API Communication
```javascript
// Axios interceptors handle auth tokens automatically
axios.post('/api/interview/generate-report', data)
  .then(response => setResults(response.data))
```

### Authentication Flow
1. User enters credentials → POST `/api/auth/login`
2. Backend validates → returns JWT in HTTP-only cookie
3. Frontend redirects to dashboard
4. Subsequent requests include cookie automatically

## 4. Backend Architecture

### Entry Point
**server.js**: 
- Loads environment variables
- Connects to MongoDB
- Starts Express server on port 3000

### Routing Layer
```javascript
// Clean route organization
app.use('/api/auth', authRoutes)      // Login, register, logout
app.use('/api/interview', interviewRoutes)  // Generate reports, PDFs
```

### Controllers
**Responsibilities:**
- Parse request bodies
- Call appropriate services
- Handle success/error responses
- **DON'T**: Direct database queries, business logic

Example:
```javascript
const registerUser = async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await authService.register(email, password);
    res.status(201).json({ user });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};
```

### Services
**Business Logic Layer:**
- AI API integration (Google Gemini)
- PDF generation (Puppeteer)
- Data processing and validation
- **Why separated**: Testable, reusable, single responsibility

### Middlewares
```javascript
// auth.middleware.js - JWT verification
const authUser = async (req, res, next) => {
  const token = req.cookies.token;
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  req.user = decoded;
  next();
};
```

## 5. API Flow (Request → Response Lifecycle)

### Step-by-Step Flow:
1. **Client Request**: Axios sends HTTP request with auth cookie
2. **CORS Middleware**: Validates origin, sets headers
3. **Cookie Parser**: Extracts JWT from request cookies
4. **Auth Middleware**: Verifies JWT, attaches user to request
5. **Route Handler**: Matches URL pattern, calls controller
6. **Controller**: Validates input, calls service
7. **Service**: Executes business logic, calls external APIs
8. **Database**: Mongoose performs CRUD operations
9. **Response**: Data flows back through same layers
10. **Client**: Receives response, updates UI state

### Example: Generate Interview Report
```
POST /api/interview/generate-report
{
  "resume": "...",
  "selfDescription": "...", 
  "jobDescription": "..."
}
```

## 6. Database Layer

### Why MongoDB?
- **Flexible schema**: User profiles vary greatly
- **Document-based**: Natural fit for resume/job data
- **Scalable**: MongoDB Atlas handles growth
- **JSON-like**: Direct mapping to JavaScript objects

### Key Models:
```javascript
// User Model
{
  email: String (unique),
  password: String (hashed),
  createdAt: Date
}

// Interview Report Model  
{
  userId: ObjectId,
  resume: String,
  jobDescription: String,
  report: {
    matchScore: Number,
    technicalQuestions: Array,
    behavioralQuestions: Array,
    skillGaps: Array,
    preparationPlan: Array
  },
  createdAt: Date
}
```

### Relationships:
- **One-to-Many**: One user → Many interview reports
- **Referenced**: Reports store `userId` reference

### Query Patterns:
```javascript
// Find user's reports
InterviewReport.find({ userId: req.user.id })
  .sort({ createdAt: -1 })
  .limit(10);
```

## 7. Authentication & Authorization

### End-to-End Flow:
1. **Registration**: Password → bcrypt hash → store in MongoDB
2. **Login**: Email lookup → bcrypt compare → JWT creation → HTTP-only cookie
3. **Protected Routes**: JWT from cookie → verify → attach user to request

### Password Hashing (bcrypt)
```javascript
const saltRounds = 12;
const hashedPassword = await bcrypt.hash(password, saltRounds);
```

**Why bcrypt?**
- **Slow**: Resists brute force attacks
- **Salt**: Each password gets unique salt → prevents rainbow tables
- **Adaptive**: Can increase work factor as hardware improves

**How salting works:**
1. Generate random salt for each password
2. Combine password + salt
3. Hash the combination
4. Store salt + hash together
5. Verification: Extract salt, hash input, compare

### JWT Implementation
```javascript
// Token creation
const token = jwt.sign(
  { userId: user._id, email: user.email },
  process.env.JWT_SECRET,
  { expiresIn: '7d' }
);

// Token verification
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

**Why HTTP-only cookies?**
- Prevents XSS attacks accessing tokens
- Automatic inclusion in requests
- No localStorage vulnerabilities

## 8. Security Considerations

### Implemented Protections:
1. **Password Hashing**: bcrypt with salt rounds
2. **JWT Security**: HTTP-only cookies, expiration
3. **Input Validation**: Zod schemas for API inputs
4. **CORS**: Restricted to frontend origin
5. **Environment Variables**: Sensitive data not in code

### Attack Vectors Mitigated:
- **SQL Injection**: MongoDB ORM prevents injection
- **XSS**: HTTP-only cookies, input sanitization
- **CSRF**: SameSite cookies, CORS validation
- **Brute Force**: bcrypt slow hashing

## 9. Error Handling & Validation

### Input Validation Strategy
```javascript
// Zod schemas for type-safe validation
const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
});
```

### Error Handling Flow
1. **Zod Validation**: Catches invalid input early
2. **Try-Catch**: Service layer errors caught
3. **Consistent Responses**: Standard error format
4. **Client Handling**: Axios interceptors display user-friendly messages

## 10. Scalability & Performance

### Current Limitations:
- **Synchronous AI calls**: Blocks server during processing
- **Single instance**: No horizontal scaling
- **Database queries**: No caching layer

### Production Improvements:
1. **Queue System**: Redis queue for AI processing
2. **Load Balancer**: Multiple server instances
3. **CDN**: Static assets delivery
4. **Database Indexing**: Optimize query performance
5. **Caching**: Redis for frequent queries

### FAANG-Scale Architecture:
```
Load Balancer → Multiple API Servers → Redis Cache → MongoDB Cluster
                    ↓
              Message Queue → AI Processing Workers
```

## 11. Interview-Ready Explanations

### "Explain this project in 2 minutes"
"I built an AI-powered interview preparation platform that helps candidates prepare for job interviews. Users upload their resume and job description, and our system generates personalized interview questions, skill gap analysis, and preparation plans using Google's Gemini AI. The frontend is React with a clean UI, backend is Node.js/Express with MongoDB for data storage, and we use JWT for authentication. The key innovation is personalization - instead of generic questions, candidates get role-specific preparation based on their actual profile."

### "Explain backend design decisions"
"I chose Express.js for its simplicity and middleware ecosystem. MongoDB because interview data is unstructured - different resumes and job descriptions have varying fields. I separated concerns with controllers handling HTTP logic, services for business logic, and models for data structure. For authentication, I use JWT in HTTP-only cookies for security. The AI service is abstracted into its own layer, making it easy to swap providers or add caching."

### "Explain authentication in this project"
"I implemented JWT-based authentication with security best practices. When users register, passwords are hashed using bcrypt with 12 salt rounds. On login, we verify credentials and create a JWT signed with a secret key, stored as an HTTP-only cookie. For protected routes, middleware extracts the token, verifies it, and attaches user data to the request. This prevents XSS attacks since JavaScript can't access the cookie, and the token expires after 7 days."

### "If you had more time, what would you improve?"
"I'd add a Redis caching layer for AI responses to reduce API costs and improve response times. Implement a job queue system for handling multiple AI requests asynchronously. Add comprehensive logging and monitoring. Implement rate limiting to prevent abuse. Add more sophisticated resume parsing with NLP. And create a dashboard for tracking user progress and analytics."

---

**Key Technical Takeaways:**
- Clean architecture with separated concerns
- Security-first authentication design
- Scalable microservice-ready structure
- Modern React with optimized build tools
- Production-ready error handling and validation

## 12. Complete File-by-File Breakdown

### Frontend Files

#### Root Level
- **package.json**: Frontend dependencies and scripts (Vite, React, Sass, Axios)
- **vite.config.js**: Vite configuration with React plugin
- **index.html**: Entry point HTML file for the SPA

#### src/
- **style.scss**: Global styles, CSS reset, typography, base colors
- **App.jsx**: Root React component with router setup

#### src/features/auth/
- **auth.form.scss**: Authentication form styling (login/register inputs, buttons)
- **pages/Login.jsx**: Login page component with form handling
- **pages/Register.jsx**: Registration page component
- **components/Protected.jsx**: Route protection wrapper component

#### src/features/interview/
- **style/home.scss**: Home page styles, variables, component styling
- **style/interview.scss**: Interview results page styles, layout, cards
- **pages/Home.jsx**: Main form page (resume upload, job description inputs)
- **pages/Interview.jsx**: Results display page (questions, skill gaps, prep plan)

#### src/style/
- **button.scss**: Reusable button component styles

### Backend Files

#### Root Level
- **package.json**: Backend dependencies (Express, MongoDB, JWT, AI services)
- **server.js**: Application entry point, server setup, database connection
- **.env**: Environment variables (database URI, JWT secret, API keys)

#### src/
- **app.js**: Express app configuration, middleware setup, route mounting

#### src/config/
- **database.js**: MongoDB connection logic with error handling

#### src/controllers/
- **auth.controller.js**: Authentication request handlers (register, login, logout, get-me)
- **interview.controller.js**: Interview report generation, PDF creation endpoints

#### src/middlewares/
- **auth.middleware.js**: JWT verification middleware for protected routes
- **error.middleware.js**: Global error handling middleware

#### src/models/
- **User.js**: User schema with email, password, timestamps
- **InterviewReport.js**: Interview report schema with user reference, report data

#### src/routes/
- **auth.routes.js**: Authentication route definitions (/register, /login, /logout, /get-me)
- **interview.routes.js**: Interview route definitions (/generate-report, /generate-resume)

#### src/services/
- **ai.service.js**: Google Gemini AI integration, report generation logic
- **pdf.service.js**: Puppeteer PDF generation from HTML content

### Configuration Files

#### Frontend
- **.gitignore**: Node modules, build artifacts, environment files
- **README.md**: Project documentation (created)

#### Backend
- **.gitignore**: Node modules, environment files

### Documentation Files
- **understanding.md**: Comprehensive technical documentation (created)
- **README.md**: Project setup and usage instructions (created)

### File Purpose Summary

**Frontend Flow:**
1. **index.html** → **App.jsx** → **Router** → **Page Components**
2. **Styles**: Global (style.scss) → Feature-specific (auth.form.scss, home.scss, interview.scss)
3. **API Communication**: Axios calls to backend endpoints

**Backend Flow:**
1. **server.js** → **app.js** → **Routes** → **Controllers** → **Services** → **Models**
2. **Authentication**: auth.middleware.js protects routes
3. **Database**: database.js manages MongoDB connection
4. **External Services**: ai.service.js handles Google AI integration

**Key Responsibilities:**
- **Controllers**: HTTP request/response handling only
- **Services**: Business logic, external API calls, data processing
- **Models**: Data structure definition and validation
- **Middlewares**: Cross-cutting concerns (auth, errors)
- **Routes**: API endpoint definitions and mapping

## 13. File Communication & Data Flow

### Frontend File Communication

#### Component Hierarchy & Data Flow
```
index.html
    ↓ (loads)
App.jsx
    ↓ (provides Router)
├── Login.jsx ← auth.form.scss
├── Register.jsx ← auth.form.scss
└── Home.jsx ← home.scss
    ↓ (navigation)
Interview.jsx ← interview.scss
```

#### Style Communication
- **style.scss**: Global variables, CSS reset, typography
- **auth.form.scss**: Inherits global styles, adds form-specific styling
- **home.scss**: Defines color variables, imports global styles
- **interview.scss**: Mirrors home.scss variables for consistency
- **button.scss**: Reusable button styles imported by components

#### API Communication Flow
```
Component (Login.jsx)
    ↓ (Axios POST)
Backend Route (/api/auth/login)
    ↓ (Response)
Component updates state
    ↓ (React Router)
Redirect to Home.jsx
```

### Backend File Communication

#### Request Processing Chain
```
server.js
    ↓ (starts)
app.js
    ↓ (mounts routes)
auth.routes.js
    ↓ (matches POST /login)
auth.controller.js
    ↓ (calls service)
auth.service.js
    ↓ (queries database)
User.js (Model)
    ↓ (returns data)
Response flows back through same chain
```

#### Inter-File Dependencies

**server.js:**
```javascript
require("dotenv").config()           // ← .env file
const app = require("./src/app")      // ← src/app.js
const connectToDB = require("./src/config/database")  // ← src/config/database.js
```

**src/app.js:**
```javascript
const authRouter = require("./routes/auth.routes")      // ← src/routes/auth.routes.js
const interviewRouter = require("./routes/interview.routes")  // ← src/routes/interview.routes.js
```

**src/routes/auth.routes.js:**
```javascript
const authController = require("../controllers/auth.controller")     // ← src/controllers/auth.controller.js
const authMiddleware = require("../middlewares/auth.middleware")     // ← src/middlewares/auth.middleware.js
```

**src/controllers/auth.controller.js:**
```javascript
const User = require("../models/User")                    // ← src/models/User.js
const authService = require("../services/auth.service")   // ← src/services/auth.service.js
```

### Cross-Layer Communication

#### Frontend ↔ Backend Communication
```
Frontend Component (Home.jsx)
    ↓ (Axios POST with FormData)
Backend Route (/api/interview/generate-report)
    ↓ (auth.middleware.js)
JWT Verification
    ↓ (interview.controller.js)
Request Validation
    ↓ (ai.service.js)
Google Gemini API Call
    ↓ (InterviewReport.js Model)
MongoDB Save
    ↓ (Response Chain)
Frontend Component (Interview.jsx)
```

#### Database Communication
```
database.js
    ↓ (mongoose.connect)
MongoDB Atlas
    ↓ (connection)
Models (User.js, InterviewReport.js)
    ↓ (queries)
Controllers
    ↓ (data)
Services
```

#### External API Communication
```
ai.service.js
    ↓ (GoogleGenAI)
Google Gemini API
    ↓ (AI response)
Report Generation
    ↓ (pdf.service.js)
Puppeteer
    ↓ (PDF Buffer)
Frontend Download
```

### Data Flow Examples

#### User Registration Flow
```
1. Register.jsx (form submission)
   ↓ Axios POST /api/auth/register
2. auth.routes.js (route matching)
   ↓ authController.registerUserController
3. auth.controller.js (request handling)
   ↓ authService.register
4. auth.service.js (business logic)
   ↓ User.create (MongoDB)
5. User.js (model validation)
   ↓ Database save
6. Response chain back to frontend
   ↓ Router redirect to Login.jsx
```

#### Interview Report Generation Flow
```
1. Home.jsx (form submission)
   ↓ Axios POST /api/interview/generate-report
2. auth.middleware.js (JWT verification)
   ↓ req.user attachment
3. interview.controller.js (validation)
   ↓ ai.service.generateInterviewReport
4. ai.service.js (AI API call)
   ↓ Google Gemini API
5. InterviewReport.js (save results)
   ↓ MongoDB store
6. Response chain back to frontend
   ↓ Router navigation to Interview.jsx
```

#### Authentication Flow
```
1. Login.jsx (credentials)
   ↓ Axios POST /api/auth/login
2. auth.controller.js (verification)
   ↓ bcrypt.compare
3. JWT creation
   ↓ HTTP-only cookie
4. Frontend receives response
   ↓ Router navigation
5. Protected routes check auth.middleware.js
   ↓ JWT verification
   ↓ Grant/deny access
```

### Import/Export Patterns

#### Frontend Imports
```javascript
// Component imports
import { Router } from "react-router"
import Login from "./features/auth/pages/Login"
import Protected from "./features/auth/components/Protected"

// Style imports
import "./style.scss"
import "./features/auth/auth.form.scss"
```

#### Backend Imports
```javascript
// Module imports
const express = require("express")
const { Router } = require('express')

// Local imports
const authController = require("../controllers/auth.controller")
const User = require("../models/User")
```

### Event-Driven Communication

#### Frontend Events
- **Form submissions**: onClick → API calls → state updates
- **Navigation**: Router redirects → component mounting
- **File uploads**: onChange → FormData → multipart requests

#### Backend Events
- **Database events**: Model hooks → data validation
- **Middleware chain**: request → auth → validation → controller
- **Error events**: try/catch → error middleware → client response

This communication pattern ensures:
- **Separation of concerns**: Each file has single responsibility
- **Loose coupling**: Files communicate through well-defined interfaces
- **Data integrity**: Validation at multiple layers
- **Security**: Authentication checks at route level
- **Error handling**: Consistent error propagation
