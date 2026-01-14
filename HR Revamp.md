---
layout: default
title: HR Command
---

# HR Command

Modern HR workflow orchestration platform with Personnel Action Form (PAF) management, electronic signatures, and real-time approvals.

## Overview

HR Command streamlines personnel action workflows through a Chrome extension interface, providing:
- **Personnel Action Forms**: Create, submit, and track PAFs with automated routing
- **Electronic Signatures**: Digital signature capture and consent management
- **Multi-level Approvals**: Supervisor → HR → VP approval chains
- **Real-time Updates**: Live status tracking and notifications
- **Paycor Integration**: Automatic employee data synchronization
- **Role-based Access**: Supervisor, HR, and VP permission levels

## Roadmap

### ✅ Phase 1: Core HR Workflows (Complete)
- Personnel Action Forms with approval chains
- Electronic signatures and consent management
- Chrome extension interface
- Azure Entra ID authentication
- Production deployment

### 🚧 Phase 2: Onboarding Portal (Planned)
**Onboarding Web App** - Public-facing Static Web App for new hire onboarding
- Pre-employment form completion (I-9, W-4, direct deposit, etc.)
- Document upload and secure storage
- Progress tracking for new hires
- HR review and approval workflows
- Automatic conversion to employee record on hire date
- Architecture: React SWA → Shared backend API → Azure Blob Storage

## Key Features

- ✅ **Azure Entra ID Authentication**: Secure OAuth-based login
- ✅ **Chrome Extension**: Seamless browser-integrated experience
- ✅ **Automated Workflows**: Smart routing based on action types
- ✅ **Audit Trails**: Complete history of all actions and approvals
- ✅ **Production Ready**: Deployed on Azure Container Apps

## 🏗️ Architecture

```
hr-command/
├── backend/          # Node.js + TypeScript API (Express, Prisma)
│   ├── src/          # Controllers, services, routes, middleware
│   ├── prisma/       # Database schema and migrations
│   └── package.json
├── extension/        # Chrome Extension (React + Vite + TypeScript)
│   ├── src/          # Components, services, OAuth integration
│   ├── manifest.json # Extension manifest (v3)
│   └── package.json
├── onboarding-web/   # 🚧 PLANNED: Static Web App (React SWA)
│   └── ...           # Public-facing onboarding portal for new hires
└── docs/             # Documentation
```

**Note:** All frontend applications (extension + onboarding-web) share the same backend API.

## 📚 Documentation

- **[Setup Guide](docs/setup/SETUP.md)** - Complete installation instructions
- **[Authentication](docs/setup/AUTHENTICATION.md)** - Azure Entra ID configuration
- **[Azure Deployment](docs/deployment/AZURE_DEPLOYMENT.md)** - Production deployment
- **[Privacy Policy](PRIVACY_POLICY.md)** - Data handling and privacy information

## 🚀 Quick Start

### Prerequisites

- Node.js 20+
- Azure SQL Database (or local SQL Server)
- Azure account (for Entra ID auth)
- Chrome browser

### 1. Backend Setup

```bash
cd backend
npm install
cp .env.example .env
# Edit .env with your credentials
npx prisma generate
npx prisma migrate deploy
npm run dev
```

### 2. Extension Setup

```bash
cd extension
npm install
cp .env.example .env
# Edit .env with Azure Entra ID credentials
npm run build
```

### 3. Load Extension in Chrome

1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" → select `extension/dist`
4. Note your extension ID for Azure redirect URI

### 4. Configure Azure Entra ID

See [Complete Setup Guide](docs/setup/SETUP.md#authentication-setup) for detailed instructions.

## 📋 Features

- **Personnel Action Forms (PAFs)**: Multi-step workflow with supervisor, VP, and HR approvals
- **Digital Signatures**: Secure signature capture with Azure Entra ID authentication
- **Real-time Updates**: Auto-refresh polling for status changes
- **PDF Generation**: Automated PDF creation with signatures
- **Role-based Access**: Different views for supervisors, VPs, and HR
- **Azure Integration**: Entra ID authentication, Azure SQL, Container Apps deployment

## 🛠️ Tech Stack

**Backend:**
- Node.js 20 + TypeScript
- Express.js + Prisma ORM
- Azure SQL Database
- Winston logging
- JWT authentication

**Frontend:**
- Chrome Extension (Manifest V3)
- React + TypeScript
- Vite build tool
- Azure Entra ID authentication

**Infrastructure:**
- Azure Container Apps (backend)
- Azure SQL Server (database)
- GitHub Actions (CI/CD)

## 🔒 Security

- Azure Entra ID (OAuth 2.0) authentication
- JWT tokens (8hr access, 30d refresh)
- SQL injection prevention (Prisma)
- XSS protection (Content Security Policy)
- Secure password hashing (bcrypt)
- Rate limiting and request validation

See [Security Audit](docs/security/SECURITY_AUDIT.md) for complete security documentation.

## 🚢 Deployment

Production deployment to Azure:

```bash
cd backend
docker build -t hrcommandacr.azurecr.io/hr-command-backend:latest .
docker push hrcommandacr.azurecr.io/hr-command-backend:latest
```

See [Azure Deployment Guide](docs/deployment/AZURE_DEPLOYMENT.md) for complete deployment instructions.

## 📖 API Documentation

API is versioned at `/api/v1/` with standardized responses:

```typescript
{
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
  meta?: { timestamp: string; /* pagination, etc */ }
}
```

Key endpoints:
- `POST /api/v1/auth/login` - User authentication
- `GET /api/v1/personnel-actions` - List PAFs
- `POST /api/v1/personnel-actions` - Create PAF
- `PUT /api/v1/personnel-actions/:id/sign` - Sign PAF
- `GET /api/v1/employees` - List employees

## 🧪 Development

### Backend Development

```bash
cd backend
npm run dev          # Start dev server with hot reload
npm run build        # Build TypeScript
npm run test         # Run tests (when available)
npx prisma studio    # Open Prisma GUI
```

### Extension Development

```bash
cd extension
npm run dev          # Build with watch mode
npm run build        # Production build
```

### Code Quality

- Winston structured logging (no console.log)
- Prisma singleton pattern for database connections
- Centralized configuration in `app.config.ts`
- TypeScript strict mode enabled
- ESLint + Prettier formatting

## 📝 License

Proprietary - VOA North Louisiana

## 👥 Contributors

- Russell Irby - Lead Developer

## 🆘 Support

For setup issues or questions:
1. Check [Complete Setup Guide](docs/setup/SETUP.md)
2. Review [Troubleshooting](docs/setup/SETUP.md#troubleshooting)
3. Check [Security Documentation](docs/security/SECURITY_AUDIT.md)
4. Contact development team
4. Explore Operational and Administrative views

## 🔧 Development Workflow

### Backend Development

```powershell
cd backend

# Run with hot reload
npm run dev

# Run tests
npm test

# Lint code
npm run lint

# Build for production
npm run build
```

### Extension Development

```powershell
cd extension

# Development mode with auto-reload
npm run dev

# Build for production
npm run build

# Type checking
npm run type-check
```

## 📊 Database Setup

### Azure SQL Database

```sql
-- Create database
CREATE DATABASE hrcommand;

-- Run Prisma migrations
-- This is handled by: npm run prisma:migrate

-- Seed initial data
-- Execute backend/prisma/seed.sql
```

### Local SQL Server (Development)

```powershell
# Using Docker
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong@Password" `
  -p 1433:1433 --name sqlserver `
  -d mcr.microsoft.com/mssql/server:2022-latest

# Update DATABASE_URL in backend/.env
# DATABASE_URL="sqlserver://localhost:1433;database=hrcommand;user=sa;password=YourStrong@Password;encrypt=true;trustServerCertificate=true"
```

## 🔐 Security Configuration

### JWT Secrets

Generate secure secrets for production:

```powershell
# Generate random secrets
-join ((48..57) + (65..90) + (97..122) | Get-Random -Count 64 | % {[char]$_})
```

Update in `backend/.env`:
- `JWT_SECRET`
- `REFRESH_TOKEN_SECRET`

### CORS Configuration

Update `backend/.env` with your extension ID:

```env
CORS_ORIGIN=chrome-extension://YOUR_EXTENSION_ID,http://localhost:5173
```

Find your extension ID at `chrome://extensions/`

## 🚢 Production Deployment

### Backend (Azure App Service)

```powershell
cd backend

# Build Docker image
docker build -t hrcommand-api .

# Tag for Azure Container Registry
docker tag hrcommand-api yourregistry.azurecr.io/hrcommand-api:latest

# Push to registry
docker push yourregistry.azurecr.io/hrcommand-api:latest

# Deploy to Azure App Service
az webapp create --resource-group hrcommand-rg `
  --plan hrcommand-plan `
  --name hrcommand-api `
  --deployment-container-image-name yourregistry.azurecr.io/hrcommand-api:latest
```

### Extension (Chrome Web Store)

```powershell
cd extension

# Build production bundle
npm run build

# Create distribution package
Compress-Archive -Path dist/* -DestinationPath hr-command-v1.0.0.zip

# Upload to Chrome Web Store Developer Dashboard
# https://chrome.google.com/webstore/devconsole
```

## 📖 Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Complete technical specification
- **[backend/README.md](./backend/README.md)** - Backend API documentation
- **[extension/README.md](./extension/README.md)** - Extension development guide

## 🧪 Testing

### Backend Tests

```powershell
cd backend
npm test
```

### Extension Testing

1. Load unpacked extension in Chrome
2. Test authentication flow
3. Verify Operational view displays moments
4. Verify Administrative view shows workflows
5. Test WebSocket notifications (if enabled)

## 🐛 Troubleshooting

### Extension not connecting to backend

1. Check CORS configuration in backend `.env`
2. Verify extension has correct `host_permissions` in `manifest.json`
3. Check browser console for errors (F12)

### Database connection errors

1. Verify SQL Server is running
2. Check `DATABASE_URL` format in `.env`
3. Ensure firewall allows connections
4. For Azure SQL, add your IP to firewall rules

### Build errors

```powershell
# Clear node_modules and reinstall
Remove-Item -Recurse -Force node_modules
npm install

# Clear build cache
Remove-Item -Recurse -Force dist
npm run build
```

## 📞 Support

For internal support, contact the development team.

## 🗺️ Product Development Roadmap

### What We've Built So Far ✅

The HR Command system is now working with these features:
- **Digital Forms:** Employees can submit personnel action requests online (promotions, raises, transfers, etc.)
- **Approval Workflow:** Forms automatically route to HR, then Supervisor, then VP for approval
- **Digital Signatures:** Each approver signs electronically using a drawing pad
- **PDF Generation:** Completed forms automatically generate professional PDFs with all signatures
- **Dashboard Views:** HR can see all pending approvals, forms waiting for signatures, and completed forms
- **Real-time Updates:** Everyone sees the current status of their requests

**Bottom Line:** The core system works and people can use it today. What follows are improvements to make it production-ready and easier to use.

---

### Phase 1: Critical Fixes Needed for Launch (Week 1)

**Why This Matters:** These are must-have features before we can launch to all staff. They prevent errors and handle real-world scenarios.

#### 1.1 Require Signatures Before Signing
**The Problem:** Users can attempt to sign forms without having a signature on file, causing errors.

**The Solution:** System validates signature existence before allowing form signing. Users are prompted to create a signature if one doesn't exist.

**Time Needed:** 2-3 hours  
**Impact:** Prevents errors and improves user experience

---

#### 1.2 Ability to Reject Forms
**The Problem:** No formal rejection workflow exists for forms that cannot be approved.

**The Solution:** Add rejection capability with required feedback. Forms return to submitter with explanation.

**What Changes:**
- New "Rejected" status for forms
- Required text box for rejection reason
- Rejected forms appear in a separate section
- Submitters can see rejection reasoning

**Time Needed:** 4-5 hours  
**Impact:** Handles real-world denials professionally and transparently

---

#### 1.3 Request Changes Instead of Rejecting
**The Problem:** Minor errors require full rejection when simple corrections would suffice.

**The Solution:** Add "Request Corrections" workflow that returns forms for editing.

**How It Works:**
1. Reviewer identifies needed corrections
2. Submits correction request with specific feedback
3. Form returns to submitter for edits
4. Submitter makes changes and resubmits
5. Form continues through approval process

**Time Needed:** 5-6 hours  
**Impact:** Faster corrections, better collaboration, less frustration

---

#### 1.4 Form Validation (Preventing Mistakes)
**The Problem:** Forms can be submitted with incomplete or invalid data.

**The Solution:** Real-time validation with clear error messages.

**What Gets Checked:**
- All required fields are complete
- Date fields contain valid dates
- Numeric fields contain appropriate values
- Business logic rules are enforced
- Data consistency across related fields

**What Users See:**
- Visual indicators for invalid fields
- Clear, actionable error messages
- Submit button disabled until validation passes
- Summary of remaining validation issues

**Time Needed:** 4-5 hours  
**Impact:** Dramatically reduces errors and back-and-forth

---

**Phase 1 Total Time:** 2-3 days of focused development  
**Phase 1 Result:** System is ready for organization-wide launch

---

### Phase 2: Making It Easier to Use (Week 2)

**Why This Matters:** These improvements make the system faster and more transparent for daily use.

#### 2.1 Activity History for Each Form
**What It Does:** Shows a timeline of everything that happened to a form
- "Submitted by John Smith on Dec 1 at 2:30pm"
- "Approved by HR on Dec 2 at 9:15am"
- "Signed by Supervisor on Dec 3 at 11:00am"

**Why It Helps:** Answers "where is my form stuck?" and provides transparency  
**Time:** 4-5 hours

---

#### 2.2 Search and Filter
**What It Does:** Add a search box where HR can quickly find forms by typing an employee name, date, or action type

**Example:** HR types "promotion" and instantly sees all promotion requests, or types "Johnson" to find all of Jane Johnson's forms

**Why It Helps:** When you have 100+ forms, finding specific ones is critical  
**Time:** 3-4 hours

---

#### 2.3 Cancel a Request
**What It Does:** Employee can cancel their own request if circumstances change (like they decide not to apply for the promotion)

**How It Works:** "Cancel Request" button (only available before final approval), requires confirmation and a reason

**Why It Helps:** Reduces clutter and prevents wasted approval time  
**Time:** 3-4 hours

---

#### 2.4 Duplicate Warning
**What It Does:** If someone tries to submit a second request for the same employee while one is already pending, show a warning

**Example:** "An active promotion request for this employee already exists. Are you sure you want to create another?"

**Why It Helps:** Prevents accidental duplicates and confusion  
**Time:** 2-3 hours

**Phase 2 Total Time:** 2 days

---

### Phase 3: Notifications (Week 3)
**What It Adds:** Email and in-app notifications so people know when action is needed
- "You have a form to approve"
- "Your request was approved"
- "A form needs your signature"

**Why It Helps:** People don't have to constantly check the system  
**Time:** 2-3 days

---

### Phase 4: Better Data Quality (Week 4)
**What It Adds:** 
- Valid position titles and compensation ranges in the system
- Smarter date validation with helpful suggestions
- Action-specific validation requirements

**Why It Helps:** Ensures data is accurate and consistent  
**Time:** 2 days

---

### Phase 5: Reports and Analytics (Week 5)
**What It Adds:**
- Dashboard showing metrics (how many forms pending, average approval time, etc.)
- Export to Excel for HR reports
- Custom report builder

**Why It Helps:** HR can analyze trends and generate reports for leadership  
**Time:** 3 days

---

### Phase 6: Advanced Features (Week 6)
**What It Adds:**
- Delegation: VP can delegate approval authority when on vacation
- Bulk approve: HR can approve multiple forms at once
- Form templates: Save common requests as templates
- Comments: Internal discussion threads on forms

**Why It Helps:** Power user features for heavy usage  
**Time:** 3-4 days

---

### Phase 7: Security and Compliance (Week 7)
**What It Adds:**
- Detailed permission controls (HR Admin vs HR Staff)
- Automatic archiving of old records (compliance)
- Enhanced signature security (tamper-proof)
- Complete audit logs (who did what when)

**Why It Helps:** Meets regulatory requirements and security best practices  
**Time:** 3 days

---

### Phase 8: Production Launch (Week 8)
**What It Adds:**
- Integration with company's Azure Active Directory (real login)
- Production database setup with backups
- Monitoring and error tracking
- User training materials and documentation

**Why It Helps:** Transitions from test environment to live system  
**Time:** 2-3 days

---

## Summary Timeline

| Phase | What You Get | Time | Can Launch After? |
|-------|--------------|------|-------------------|
| **Phase 1** | Core fixes (reject, corrections, validation) | 2-3 days | ✅ **YES - Soft Launch** |
| **Phase 2** | Easier to use (search, history, cancel) | 2 days | ✅ **YES - Full Launch** |
| **Phase 3** | Notifications | 2-3 days | Optional |
| **Phase 4** | Data quality | 2 days | Optional |
| **Phase 5** | Reports | 3 days | Optional |
| **Phase 6** | Power features | 3-4 days | Optional |
| **Phase 7** | Security | 3 days | Recommended |
| **Phase 8** | Production | 2-3 days | **REQUIRED** |

**Recommended Launch Strategy:**
1. Complete Phase 1 → Launch to pilot group (2-3 weeks)
2. Complete Phase 2 → Launch to full organization (1 week)
3. Add Phases 3-6 based on user feedback (4-6 weeks)
4. Complete Phases 7-8 → Full production (2 weeks)

**Total Time to First Launch:** 1-2 weeks  
**Total Time to Full Production:** 6-8 weeks

---

## What Happens Next?

**Immediate Next Steps:**
1. Review and approve Phase 1 priorities
2. Development team begins implementation
3. Weekly progress updates
4. Demo sessions as features complete

**Questions?** Contact the development team for clarification on any feature or timeline.

## 📝 License

UNLICENSED - Internal use only
