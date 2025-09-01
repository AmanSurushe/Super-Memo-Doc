# Quick Start Guide

Get up and running with the SM-15 Spaced Repetition Application in minutes.

## 🎯 Prerequisites

- Node.js 18+ and npm/yarn
- PostgreSQL 14+
- Docker (optional, for containerized deployment)
- Git for version control

## ⚡ 5-Minute Setup

### 1. Clone and Install Dependencies

```bash
# Clone the repository
git clone <repository-url>
cd sm15-spaced-repetition

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies  
cd ../frontend
npm install
```

### 2. Database Setup

```bash
# Create PostgreSQL database
createdb spaced_repetition

# Configure environment variables
cp .env.example .env
# Edit .env with your database credentials
```

### 3. Initialize Database

```bash
# Run database migrations
cd backend
npx prisma migrate dev
npx prisma generate
```

### 4. Start Development Servers

```bash
# Terminal 1: Start backend (runs on http://localhost:3001)
cd backend
npm run start:dev

# Terminal 2: Start frontend (runs on http://localhost:3000)
cd frontend
npm run dev
```

### 5. Verify Setup

- Open http://localhost:3000 in your browser
- Create a new account or log in
- Add your first flashcard
- Start learning!

## 🎮 First Steps

### Create Your First Card

1. Navigate to "Create Card" or use the quick action on dashboard
2. Add a question (front) and answer (back)
3. Optionally add a label for organization
4. Save the card

### Start Learning

1. Go to the dashboard
2. Click "Start Review" if cards are due
3. Read the question, think of the answer
4. Click "Show Answer" to reveal the correct answer
5. Grade yourself honestly (1=Failed, 5=Perfect)
6. Watch the SM-15 algorithm optimize your learning!

## 🔧 Configuration

### Environment Variables

```bash
# Backend (.env)
DATABASE_URL=postgresql://user:password@localhost:5432/spaced_repetition
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# AI API Keys (optional, for card generation)
GROQ_API_KEY=your-groq-api-key
COHERE_API_KEY=your-cohere-api-key
HUGGINGFACE_API_KEY=your-huggingface-api-key

# Frontend (.env.local)
NEXT_PUBLIC_API_URL=http://localhost:3001
```

### AI Integration Setup (Optional)

Get free API keys for automated card generation:

1. **Groq**: Sign up at https://console.groq.com/
2. **Cohere**: Register at https://dashboard.cohere.ai/
3. **HuggingFace**: Create account at https://huggingface.co/

Add the API keys to your `.env` file to enable AI-powered card generation from text.

## 📖 Understanding the SM-15 Algorithm

### Key Concepts (5-minute read)

1. **A-Factor**: Measures card difficulty (1.1 = easy, 2.5 = hard)
2. **Intervals**: Time between reviews (grows with successful recalls)
3. **OF Matrix**: Optimal factors for interval calculation  
4. **Grades**: Your performance rating (1-5 scale)

### Real Example

```
New Spanish card: "Hola" = "Hello"
├── Day 1: Learn card (A-Factor: 2.5, Interval: 1 day)
├── Day 2: Grade 4 (Easy) → A-Factor: 2.3, Next review: 3 days
├── Day 5: Grade 3 (Good) → A-Factor: 2.35, Next review: 6 days  
├── Day 11: Grade 1 (Failed) → A-Factor: 2.7, Next review: 1 day
└── Day 12: Grade 5 (Perfect) → A-Factor: 2.4, Next review: 2 days
```

The algorithm learns your memory patterns and optimizes review timing.

## 🚀 Next Steps

### For Learning
- [Understand the SM-15 Algorithm](./modules/sm15-algorithm/README.md)
- [Explore Performance Analytics](./modules/frontend/README.md#analytics)
- [Try AI Card Generation](./modules/ai-integration/README.md)

### For Development
- [Database Design Deep Dive](./modules/database/README.md)
- [Backend API Reference](./modules/backend/README.md)
- [Frontend Architecture](./modules/frontend/README.md)

### For Production
- [Docker Deployment](./modules/deployment/README.md#docker-setup)
- [CI/CD Pipeline](./modules/deployment/README.md#ci-cd-pipeline)
- [Monitoring Setup](./modules/deployment/README.md#monitoring)

## ❗ Common Issues

### Database Connection Issues

```bash
# Check PostgreSQL is running
sudo service postgresql status

# Verify database exists
psql -l | grep spaced_repetition

# Reset database if needed
npx prisma migrate reset
```

### Port Conflicts

```bash
# Check what's using port 3000/3001
lsof -i :3000
lsof -i :3001

# Kill process if needed
kill -9 <PID>
```

### NPM/Node Version Issues

```bash
# Check versions
node --version  # Should be 18+
npm --version   # Should be 8+

# Update if needed using nvm
nvm install 18
nvm use 18
```

## 🆘 Need Help?

1. **Check the logs** in your terminal for error messages
2. **Review the module docs** for specific component issues
3. **Verify environment variables** are set correctly
4. **Check database connectivity** and permissions

## 📋 Development Checklist

- [ ] Database connected and migrated
- [ ] Backend server running without errors
- [ ] Frontend loading at localhost:3000
- [ ] Can create and log in to user account
- [ ] Can create, review, and grade cards
- [ ] Performance statistics updating
- [ ] (Optional) AI card generation working

---

**🎉 You're ready to build and learn!** 

Continue with the [SM-15 Algorithm documentation](./modules/sm15-algorithm/README.md) to understand the core concepts, or dive into [Database Design](./modules/database/README.md) to see how everything is stored and connected.