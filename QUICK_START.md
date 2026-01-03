# GuruNirman - Quick Start Guide

This guide will help you set up and run the GuruNirman platform locally in under 10 minutes.

---

## Prerequisites

Ensure you have the following installed:

- **Go** >= 1.21.0 ([Download](https://go.dev/dl/))
- **MongoDB** >= 6.0 ([Download](https://www.mongodb.com/try/download/community) or use MongoDB Atlas)
- **Git** ([Download](https://git-scm.com/downloads))
- **Python** >= 3.9 (for AI engine, optional for basic setup)
- **Node.js** >= 18.0.0 (for frontend only) ([Download](https://nodejs.org/))

Optional but recommended:
- **Docker** & **Docker Compose** (for containerized setup)
- **Redis** (for caching, can start with in-memory alternative)

---

## Quick Setup (Without Docker)

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/gurunirman.git
cd gurunirman
```

### 2. Backend Setup

```bash
cd backend

# Download Go dependencies
go mod download

# Create environment file
cp .env.example .env

# Edit .env file with your configuration
nano .env  # or use your favorite editor
```

**Minimal `.env` configuration**:
```env
# Server
PORT=5000
NODE_ENV=development

# Database
MONGODB_URI=mongodb://localhost:27017/gurunirman
# Or use MongoDB Atlas:
# MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/gurunirman

# JWT
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRE=1h
JWT_REFRESH_EXPIRE=7d

# Optional (can be added later)
# OPENAI_API_KEY=your-openai-key
# REDIS_URL=redis://localhost:6379
# AWS_S3_BUCKET=your-bucket-name
```

```bash
# Start the backend server
go run cmd/server/main.go

# Or build and run
go build -o bin/server cmd/server/main.go
./bin/server
```

Backend will run at: `http://localhost:5000`

### 3. Frontend Setup

Open a new terminal:

```bash
cd frontend

# Install dependencies
npm install

# Create environment file
cp .env.example .env

# Edit .env file
nano .env
```

**Frontend `.env` configuration**:
```env
VITE_API_URL=http://localhost:5000/api
VITE_APP_NAME=GuruNirman
```

```bash
# Start the frontend dev server
npm run dev
```

Frontend will run at: `http://localhost:3000`

### 4. Access the Application

Open your browser and navigate to:
- **Frontend**: http://localhost:3000
- **API Docs**: http://localhost:5000/api-docs (if Swagger is configured)

### 5. Create First Admin User

You can seed the database with a test admin user:

```bash
cd backend
npm run seed:admin
```

Default admin credentials:
- **Email**: admin@gurunirman.edu.in
- **Password**: Admin@123

**⚠️ Change these credentials immediately after first login!**

---

## Docker Setup (Recommended for Production-like Environment)

### 1. Prerequisites
- Docker Desktop installed and running

### 2. One-Command Setup

```bash
# Clone repository
git clone https://github.com/your-org/gurunirman.git
cd gurunirman

# Create environment files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Edit environment files as needed
# (Use docker service names for connections, e.g., mongodb://mongo:27017)

# Start all services
docker-compose up -d
```

### 3. Services Running

After `docker-compose up`, the following will be available:

| Service | URL | Description |
|---------|-----|-------------|
| Frontend | http://localhost:3000 | React application |
| Backend API | http://localhost:5000 | Express API |
| MongoDB | localhost:27017 | Database |
| Redis | localhost:6379 | Cache |
| Mongo Express | http://localhost:8081 | Database admin UI |

### 4. View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
```

### 5. Stop Services

```bash
docker-compose down

# To also remove volumes (database data)
docker-compose down -v
```

---

## Database Seeding

### Seed Sample Data

```bash
cd backend
go run cmd/seed/main.go

# Or if you built the binary
go build -o bin/seed cmd/seed/main.go
./bin/seed
```

This will create:
- 3 clusters (Delhi North, Mumbai Central, Bangalore South)
- 10 sample teachers
- 5 training modules
- Sample feedback entries

### Clear Database

```bash
go run cmd/seed/main.go --reset  # ⚠️ This will delete all data!
```

---

## Testing the Setup

### 1. Test Backend API

```bash
# Check health endpoint
curl http://localhost:5000/api/health

# Expected response:
# {"status":"OK","timestamp":1234567890}
```

### 2. Test Frontend

Navigate to http://localhost:3000 in your browser. You should see the landing page.

### 3. Login Flow

1. Click "Login" button
2. Use credentials:
   - Teacher: `teacher@example.com` / `Teacher@123`
   - Admin: `admin@gurunirman.edu.in` / `Admin@123`
3. Explore the dashboard

---

## AI Engine Setup (Optional)

The AI features require additional setup:

### 1. Install Python Dependencies

```bash
cd ai-engine

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure AI Environment

```bash
cp .env.example .env
nano .env
```

**AI Engine `.env`**:
```env
OPENAI_API_KEY=your-openai-api-key
PINECONE_API_KEY=your-pinecone-api-key
PINECONE_ENV=us-west1-gcp

# Or use open-source models
USE_OPEN_SOURCE=true
HF_MODEL=mistralai/Mistral-7B-Instruct-v0.1
```

### 3. Start AI Service

```bash
python app.py
```

AI Engine will run at: `http://localhost:8000`

---

## Development Workflow

### Backend Development

```bash
cd backend

# Run in development mode (with hot-reload using air)
air
# Or install air first: go install github.com/cosmtrek/air@latest

# Run without hot-reload
go run cmd/server/main.go

# Run tests
go test ./...

# Run tests with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Lint code (install golangci-lint first)
golangci-lint run

# Format code
go fmt ./...
goimports -w .
```

### Frontend Development

```bash
cd frontend

# Run dev server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run tests
npm test

# Lint code
npm run lint
```

---

## Troubleshooting

### Common Issues

#### 1. Port Already in Use

```bash
# Error: Port 5000 is already in use

# Solution: Kill the process using the port
# On macOS/Linux:
lsof -ti:5000 | xargs kill -9

# On Windows:
netstat -ano | findstr :5000
taskkill /PID <PID> /F
```

#### 2. MongoDB Connection Failed

```bash
# Error: MongoNetworkError: connect ECONNREFUSED

# Solution 1: Ensure MongoDB is running
# Start MongoDB service:
# macOS:
brew services start mongodb-community
# Linux:
sudo systemctl start mongod
# Windows: Start MongoDB service from Services app

# Solution 2: Use MongoDB Atlas
# Update MONGODB_URI in .env with Atlas connection string
```

#### 3. Go Module Issues

```bash
# Error: cannot find module

# Solution: Update and tidy Go modules
go mod tidy
go mod download
go clean -modcache  # If needed
```

#### 4. Environment Variables Not Loading

```bash
# Error: JWT_SECRET is not defined

# Solution: Ensure .env file exists and is properly formatted
# No spaces around =
# Use quotes for values with spaces
JWT_SECRET="my secret key"
```

#### 5. CORS Errors in Browser

```go
// Error: Access to fetch has been blocked by CORS policy

// Solution: Ensure backend CORS is configured correctly
// In backend/cmd/server/main.go:
import "github.com/gin-contrib/cors"

router := gin.Default()
router.Use(cors.New(cors.Config{
    AllowOrigins:     []string{"http://localhost:3000"},
    AllowMethods:     []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
    AllowHeaders:     []string{"Origin", "Content-Type", "Authorization"},
    AllowCredentials: true,
}))
```

---

## Next Steps

### For Developers

1. **Read the Architecture Docs**: [docs/architecture/SYSTEM_ARCHITECTURE.md](docs/architecture/SYSTEM_ARCHITECTURE.md)
2. **Review API Specs**: [docs/api/API_SPECIFICATION.md](docs/api/API_SPECIFICATION.md)
3. **Understand Database Schema**: [docs/database/DATABASE_SCHEMA.md](docs/database/DATABASE_SCHEMA.md)
4. **Check AI System Design**: [docs/architecture/AI_SYSTEM_DESIGN.md](docs/architecture/AI_SYSTEM_DESIGN.md)

### For Admins

1. **Upload Training Content**: Admin Dashboard → Upload Module
2. **Create Clusters**: Admin Dashboard → Cluster Management
3. **Monitor Analytics**: Analytics Dashboard

### For Teachers

1. **Submit Feedback**: Teacher Panel → Submit Feedback
2. **Browse Modules**: Teacher Panel → Training Modules
3. **Track Progress**: Teacher Panel → My Progress

---

## Getting Help

- **Documentation**: [docs/](docs/)
- **Issues**: [GitHub Issues](https://github.com/your-org/gurunirman/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-org/gurunirman/discussions)
- **Email**: support@gurunirman.edu.in

---

## Production Deployment

For production deployment guides, see:
- [Deployment Guide](docs/guides/DEPLOYMENT.md)
- [Security Best Practices](docs/guides/SECURITY.md)
- [Monitoring Setup](docs/guides/MONITORING.md)

---

**Happy Coding! 🚀**

*Building the future of teacher training in India* ❤️
