# Expertise Marketplace

A professional employee expertise directory built with React, Azure Static Web Apps, and Azure Cosmos DB.

## Features

- Search and discover colleagues by skills and expertise
- Clean, professional UI
- Azure AD authentication
- Serverless Azure Functions API
- Azure Cosmos DB backend

## Getting Started

### Prerequisites

- Node.js 18 or later
- Azure account
- Azure Static Web Apps CLI (optional for local development)

### Local Development

1. Install dependencies:
```bash
npm install
```

2. Start the development server:
```bash
npm run dev
```

3. Open [http://localhost:5173](http://localhost:5173) in your browser

### Azure Functions API (Optional for local testing)

1. Navigate to the API directory:
```bash
cd api
```

2. Install API dependencies:
```bash
npm install
```

3. Start the Functions runtime:
```bash
npm start
```

## Project Structure

```
├── src/
│   ├── components/          # React components
│   │   ├── Header.tsx       # App header
│   │   ├── SearchBar.tsx    # Search functionality
│   │   ├── ExpertDirectory.tsx  # Expert list
│   │   └── ExpertCard.tsx   # Individual expert card
│   ├── App.tsx              # Main app component
│   └── main.tsx             # Entry point
├── api/
│   └── src/functions/       # Azure Functions
│       ├── getExperts.ts    # Get all experts
│       └── searchExperts.ts # Search experts
├── staticwebapp.config.json # Azure SWA configuration
└── vite.config.ts           # Vite configuration
```

## Deployment to Azure

This app is designed to be deployed to Azure Static Web Apps. The deployment will automatically:
- Build the React frontend
- Deploy Azure Functions API
- Configure authentication with Azure AD
- Set up Cosmos DB integration

### Deploy using Azure Portal or Azure CLI

See [Azure Static Web Apps documentation](https://learn.microsoft.com/azure/static-web-apps/) for deployment instructions.

## Technology Stack

- **Frontend**: React 18 with TypeScript
- **Build Tool**: Vite
- **Backend**: Azure Functions (Node.js)
- **Database**: Azure Cosmos DB (to be configured)
- **Authentication**: Microsoft Entra ID (Azure AD)
- **Hosting**: Azure Static Web Apps

## Next Steps

1. Configure Azure Cosmos DB connection
2. Implement authentication
3. Add user profile management
4. Enhance search with filters
5. Add messaging/contact features

## License

Internal use only - VOA
