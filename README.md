# Authentication as a Service

A production-ready centralized authentication and user management service designed for multi-product companies. Provides Single Sign-On (SSO) capabilities across multiple domains and subdomains with comprehensive scope-based authorization.

## Features

### Authentication & Authorization
- **JWT Authentication**: Secure token-based auth with automatic refresh token rotation
- **OAuth Integration**: Complete Google, GitHub, and Microsoft Sign-In with account linking
- **Scope-Based Authorization**: Fine-grained permissions (`users:read`, `products:write`)
- **Role-Based Access Control**: User roles (user, moderator, admin) with inherited permissions

### Cross-Domain Support
- **Single Sign-On**: Login once, access all company products
- **Subdomain Authentication**: Works across `app.company.com`, `admin.company.com`
- **Return URL Handling**: Seamless redirects after authentication
- **Flexible Integration**: Multiple integration methods for different tech stacks

### Security
- **Production Ready**: Rate limiting, CORS, bcrypt password hashing
- **Token Security**: JWT with RS256 signing and secure storage
- **Input Validation**: Comprehensive request validation and sanitization
- **OAuth Security**: CSRF protection, account linking, email verification

## Quick Start

### Prerequisites
- **Backend**: Go 1.23+, PostgreSQL 15+
- **Frontend**: Node.js 18+, npm
- **Optional**: Docker & Docker Compose

### 1. Clone Repository with Submodules
```bash
git clone --recursive <repository-url>
cd authentication-as-a-service

# If you already cloned without --recursive, initialize submodules:
git submodule update --init --recursive
```

### 2. Backend Setup
```bash
cd identity-backend/

# Copy environment configuration
cp .env.example .env

# Start PostgreSQL
docker-compose up postgres -d

# Install Go dependencies
go mod tidy

# Run the backend service
go run cmd/server/main.go
```

### 3. Frontend Setup
```bash
cd ../identity-frontend/

# Install dependencies
npm install

# Create environment configuration (no .env.local.example exists, create manually)
cat > .env.local << EOF
NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1
NEXT_PUBLIC_COMPANY_DOMAIN=localhost
EOF

# Run the frontend application
npm run dev
```

### 4. Verify Setup
```bash
# Check backend health
curl http://localhost:8080/health

# Visit frontend
open http://localhost:3000
```

## Architecture

```
╔═══════════════════════════════════════════════════════════════════╗
║                    AUTHENTICATION SERVICE                         ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║   ┌───────────────────────────────────────────────────────────┐   ║
║   │                    FRONTEND LAYER                         │   ║
║   │                       (Next.js)                           │   ║
║   │                                                           │   ║
║   │ • Login / Signup UI                                       │   ║
║   │ • OAuth Handler                                           │   ║
║   │ • Profile Management                                      │   ║
║   │ • Redirects                                               │   ║
║   └───────────────────────────────────────────────────────────┘   ║
║                              │                                    ║
║                              ▼ HTTP                               ║
║   ┌───────────────────────────────────────────────────────────┐   ║
║   │                    BACKEND LAYER                          │   ║
║   │                      (Go + Gin)                           │   ║
║   │                                                           │   ║
║   │ • JWT Manager                                             │   ║
║   │ • User Service                                            │   ║
║   │ • Auth Handler                                            │   ║
║   │ • Input Validator                                         │   ║
║   └───────────────────────────────────────────────────────────┘   ║
║                              │                                    ║
║                              ▼ SQL                                ║
║   ┌───────────────────────────────────────────────────────────┐   ║
║   │                   DATABASE LAYER                          │   ║
║   │                    (PostgreSQL)                           │   ║
║   │                                                           │   ║ 
║   │ • users                                                   │   ║
║   │ • user_profiles                                           │   ║
║   │ • roles                                                   │   ║
║   │ • permissions                                             │   ║
║   │ • oauth_accounts                                          │   ║
║   │ • refresh_tokens                                          │   ║
║   └───────────────────────────────────────────────────────────┘   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Components
- **Frontend (`identity-frontend/`)**: Next.js authentication portal serving as `accounts.company.com`
- **Backend (`identity-backend/`)**: Go API service providing JWT authentication and user management
- **Database**: PostgreSQL with automatic migrations and seeding

## Repository Structure

This repository uses Git submodules to organize the frontend and backend components:

```
authentication-as-a-service/
├── identity-backend/           # Go API service (submodule)
│   ├── cmd/server/            # Application entry point
│   ├── internal/              # Internal packages
│   ├── pkg/                   # Public packages
│   ├── .env.example           # Backend environment template
│   └── docker-compose.yml     # Development environment
├── identity-frontend/          # Next.js app (submodule)
│   ├── app/                   # Next.js App Router
│   ├── components/            # React components
│   ├── lib/                   # Utility functions
│   └── package.json           # Frontend dependencies
└── .gitmodules                # Submodule configuration
```

## Configuration

### Backend Configuration (`.env`)
```bash
# Server
PORT=8080
ENVIRONMENT=development
JWT_SECRET=your-super-secret-jwt-key

# Database
DATABASE_URL=postgres://username:password@localhost:5432/identity_service

# OAuth Providers
GOOGLE_CLIENT_ID=your-google-client-id
GITHUB_CLIENT_ID=your-github-client-id
MICROSOFT_CLIENT_ID=your-microsoft-client-id

# Cross-Domain
ALLOWED_ORIGINS=https://mycompany.com,https://app.mycompany.com
COOKIE_DOMAIN=.mycompany.com
```

### Frontend Configuration (`.env.local`)
```bash
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1

# Domain Configuration
NEXT_PUBLIC_COMPANY_DOMAIN=mycompany.com
```

## Integration Examples

### Redirect to Authentication
```javascript
// From your product application
const authURL = 'https://accounts.mycompany.com/signin?return_url=' + 
               encodeURIComponent('https://app.mycompany.com/dashboard');
window.location.href = authURL;
```

### Using Go Client Library
```go
import "github.com/sharan-industries/identity-service/pkg/auth"

// Create protected middleware
authClient := auth.NewClient("https://accounts.mycompany.com")
protectedHandler := authClient.Middleware("users:read")(yourHandler)
```

### Manual Token Validation
```bash
curl -X POST https://accounts.mycompany.com/api/v1/auth/validate \
  -H "Content-Type: application/json" \
  -d '{"token":"YOUR_JWT_TOKEN"}'
```

## API Documentation

### Authentication Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/auth/register` | User registration |
| `POST` | `/api/v1/auth/login` | User login |
| `POST` | `/api/v1/auth/refresh` | Refresh access token |
| `POST` | `/api/v1/auth/logout` | User logout |
| `POST` | `/api/v1/auth/validate` | Token validation |

### OAuth Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/auth/oauth/google` | Google OAuth initiation |
| `GET` | `/api/v1/auth/oauth/google/callback` | Google OAuth callback |
| `GET` | `/api/v1/auth/oauth/github` | GitHub OAuth initiation |
| `GET` | `/api/v1/auth/oauth/github/callback` | GitHub OAuth callback |
| `GET` | `/api/v1/auth/oauth/microsoft` | Microsoft OAuth initiation |
| `GET` | `/api/v1/auth/oauth/microsoft/callback` | Microsoft OAuth callback |

### User Management (Protected)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/users/profile` | Get user profile |
| `PUT` | `/api/v1/users/profile` | Update user profile |
| `DELETE` | `/api/v1/users/account` | Delete user account |

## OAuth Provider Setup

### Google OAuth
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create OAuth 2.0 credentials
3. Set redirect URI: `{your-domain}/api/v1/auth/oauth/google/callback`

### GitHub OAuth  
1. Go to GitHub Settings > Developer settings > OAuth Apps
2. Create new OAuth App
3. Set callback URL: `{your-domain}/api/v1/auth/oauth/github/callback`

### Microsoft OAuth
1. Go to [Azure Portal](https://portal.azure.com/)
2. Register new application
3. Set redirect URI: `{your-domain}/api/v1/auth/oauth/microsoft/callback`

## Development Commands

### Backend (Go Service)
```bash
cd identity-backend/

# Run the application locally
go run cmd/server/main.go

# Run tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Format code
go fmt ./...

# Lint code (requires golangci-lint)
golangci-lint run

# Build binary
go build -o bin/identity-service cmd/server/main.go
```

### Frontend (Next.js App)
```bash
cd identity-frontend/

# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run linter
npm run lint
```

### Docker Commands (Backend)
```bash
cd identity-backend/

# Start full stack
docker-compose up -d

# Start only PostgreSQL
docker-compose up postgres -d

# View logs
docker-compose logs -f identity-service

# Stop services
docker-compose down
```

## Testing

### Backend Testing
```bash
cd identity-backend/

# Run all tests
go test ./...

# Run with coverage
go test -cover ./...

# Run specific package
go test ./internal/services -v
```

### Example API Calls
```bash
# Register user
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123","first_name":"John","last_name":"Doe"}'

# Login user
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'

# Get profile (requires JWT token)
curl -X GET http://localhost:8080/api/v1/users/profile \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Default Roles & Permissions

The service automatically creates default roles with inherited permissions:

| Role | Permissions |
|------|-------------|
| **user** | `users:read`, `products:read`, `orders:read` |
| **moderator** | User permissions + `users:write`, `products:write`, `orders:write` |
| **admin** | All permissions including `users:admin`, `products:admin`, `orders:admin` |

## Production Deployment

### Docker Deployment
```bash
cd identity-backend/
docker-compose up -d
```

### Environment Checklist
- [ ] Set strong `JWT_SECRET` (32+ characters)
- [ ] Configure `DATABASE_URL` for production database
- [ ] Set `ENVIRONMENT=production`
- [ ] Configure `ALLOWED_ORIGINS` and `CORS_ALLOWED_ORIGINS`
- [ ] Set OAuth provider credentials
- [ ] Configure `COOKIE_DOMAIN` for subdomain sharing
- [ ] Enable HTTPS everywhere

## Submodule Management

This project uses Git submodules for the frontend and backend components:

```bash
# Clone with submodules
git clone --recursive <repository-url>

# Initialize submodules if already cloned
git submodule update --init --recursive

# Update submodules to latest commits
git submodule update --remote

# Pull latest changes including submodules
git pull --recurse-submodules
```

## Documentation

- **[Backend README](./identity-backend/README.md)**: Detailed backend documentation
- **[Frontend README](./identity-frontend/README.md)**: Frontend-specific documentation  
- **[Cross-Domain Setup](./identity-backend/CROSS_DOMAIN_SETUP.md)**: Integration guide for multiple domains

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes and test thoroughly
4. Format code: `go fmt ./...` (backend) and `npm run lint` (frontend)
5. Commit changes: `git commit -m 'Add amazing feature'`
6. Push to branch: `git push origin feature/amazing-feature`
7. Open a Pull Request

### Development Guidelines
- Follow Go conventions and best practices
- Use meaningful commit messages
- Ensure all tests pass before committing
- Add proper authorization middleware to new endpoints
- Update documentation for significant changes

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Issues**: Report bugs and request features via GitHub Issues
- **Documentation**: Check existing documentation files for detailed information
- **API Testing**: Use the provided Postman collection in `identity-backend/Identity-Service-API.postman_collection.json`

## Status

**Production Ready** - This authentication service is fully functional with complete authentication logic, user management, OAuth integration, and security features implemented.

---

Built with care for secure, scalable authentication across your entire product ecosystem.