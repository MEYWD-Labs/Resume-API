# Resume-API - Core Backend Service Architecture

This directory contains the core architecture documentation for the Resume-API service, which is the main backend API for resume operations in the MEYWD Labs platform.

## Architecture Documentation

### Core API Service Files

1. **[api-contracts.md](./api-contracts.md)** - API endpoints and contract definitions
2. **[data-flow-diagrams.md](./data-flow-diagrams.md)** - How data flows through the system
3. **[resource-dependencies.md](./resource-dependencies.md)** - Cloudflare resources used by this service
4. **[background-processing-architecture.md](./background-processing-architecture.md)** - Job processing and background task architecture
5. **[real-time-features.md](./real-time-features.md)** - WebSockets, Durable Objects, and real-time functionality

### Supporting Infrastructure Files

6. **[cloudflare-serverless-architecture.md](./cloudflare-serverless-architecture.md)** - Serverless infrastructure on Cloudflare Workers
7. **[performance-scaling-strategy.md](./performance-scaling-strategy.md)** - Caching strategies and edge computing
8. **[security-architecture.md](./security-architecture.md)** - API security patterns and practices
9. **[authentication-flow.md](./authentication-flow.md)** - JWT validation and authentication flows

## About Resume-API

Resume-API is the main backend service responsible for:
- Resume CRUD operations
- Resume parsing and processing
- Resume template management
- Resume export functionality
- Background job processing for resume operations
- Real-time collaboration features

This service integrates with other MEYWD Labs microservices including Payment-Service, Admin-Portal, and mobile applications.
