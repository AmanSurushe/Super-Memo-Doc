# SM-15 Spaced Repetition Application

A comprehensive implementation of the SM-15 spaced repetition algorithm with modern web technologies, designed for optimal learning efficiency and user experience.

## 🚀 Quick Start

- **New Developer?** Start with [Quick Start Guide](./QUICK_START.md)
- **Understanding SM-15?** Check out [SM-15 Algorithm Documentation](./modules/sm15-algorithm/README.md)
- **Database Design?** See [Database Documentation](./modules/database/README.md)
- **API Development?** Visit [Backend Documentation](./modules/backend/README.md)

## 📋 Project Overview

This application implements the advanced SM-15 spaced repetition algorithm with:
- **Personalized Learning**: Adaptive difficulty assessment with A-Factors
- **Optimal Scheduling**: Dynamic interval calculation using OF Matrix
- **Performance Analytics**: Comprehensive user statistics and progress tracking
- **AI-Powered Content**: Automated flashcard generation from text
- **Modern Stack**: NestJS backend, Next.js frontend, PostgreSQL database

## 🏗️ Architecture

### Tech Stack
- **Backend**: NestJS with TypeScript
- **Frontend**: Next.js with TypeScript  
- **Database**: PostgreSQL with Prisma ORM
- **AI Integration**: Free APIs (Groq, Cohere, HuggingFace)
- **Authentication**: JWT-based sessions
- **Deployment**: Docker containers with CI/CD

### Core Components
1. **SM-15 Engine**: Advanced spaced repetition algorithm
2. **Card Management**: CRUD operations with intelligent scheduling
3. **Review System**: Interactive learning sessions with performance tracking
4. **Analytics Dashboard**: Progress visualization and learning insights
5. **AI Content Generator**: Text-to-flashcards conversion

## 📚 Documentation Modules

### Algorithm & Core Logic
- [**SM-15 Algorithm**](./modules/sm15-algorithm/README.md) - Mathematical framework and implementation
  - Algorithm specifications and formulas
  - Real-world examples and learning journeys  
  - Implementation pitfalls and best practices
  - Constants and configuration parameters

### Data & Storage
- [**Database Design**](./modules/database/README.md) - Complete data architecture
  - Prisma schema with field explanations
  - Table relationships and data flow
  - Performance optimization strategies
  - Sample queries and indexes

### Backend Development
- [**Backend Implementation**](./modules/backend/README.md) - NestJS server architecture
  - Project structure and modules
  - SM-15 service implementation
  - API controllers and endpoints
  - Statistics and analytics services

### Frontend Development  
- [**Frontend Implementation**](./modules/frontend/README.md) - Next.js client application
  - Component architecture and design
  - Custom hooks for SM-15 integration
  - Page implementations and routing
  - State management patterns

### AI & Content Generation
- [**AI Integration**](./modules/ai-integration/README.md) - Automated content creation
  - Service architecture and providers
  - Free AI API implementations
  - Text-to-cards generation logic
  - Configuration and API setup

### Quality Assurance
- [**Testing Strategy**](./modules/testing/README.md) - Comprehensive testing approach
  - Unit testing for SM-15 algorithm
  - Integration testing for APIs
  - Frontend component testing
  - Performance and load testing

### Production Deployment
- [**Deployment**](./modules/deployment/README.md) - Production setup and operations
  - Docker containerization
  - CI/CD pipeline configuration
  - Environment management
  - Monitoring and logging

## 🛠️ Development Information

- [**Development Phases**](./development/phases.md) - Implementation timeline and milestones
- [**Tech Stack Decisions**](./development/tech-stack.md) - Technology choices and rationale  
- [**Contributing Guidelines**](./development/contributing.md) - Development workflow and standards

## 🎯 Key Features

### For Learners
- **Intelligent Scheduling**: Cards appear at optimal intervals for maximum retention
- **Progress Tracking**: Detailed analytics on learning performance and trends
- **Streak Gamification**: Daily study streaks and achievement systems
- **Adaptive Difficulty**: Algorithm learns your strengths and weaknesses

### For Developers
- **Complete SM-15 Implementation**: Research-backed algorithm with real-world optimizations
- **Comprehensive Documentation**: Every component explained with examples
- **Modern Architecture**: Scalable, maintainable code with TypeScript
- **Free AI Integration**: Cost-effective content generation

## 📈 Performance Statistics

The application tracks comprehensive user performance metrics:
- **Real-time Analytics**: Accuracy rates, response times, retention analysis  
- **Learning Insights**: Grade distributions, difficulty progression, mastery tracking
- **Algorithm Feedback**: Performance data improves SM-15 effectiveness over time

## 🔄 SM-15 Algorithm Benefits

- **Advanced Memory Modeling**: Two-component system (Stability + Retrievability)
- **Personalized Optimization**: User-specific OF Matrix adaptation
- **Forgetting Curve Integration**: Scientific approach to memory retention
- **Real-time Refinement**: Continuous algorithm improvement based on user data

## 🚀 Getting Started

1. **Read the [Quick Start Guide](./QUICK_START.md)** for setup instructions
2. **Explore the [SM-15 Algorithm](./modules/sm15-algorithm/README.md)** to understand the core concepts
3. **Review the [Database Design](./modules/database/README.md)** for data architecture
4. **Follow the [Development Phases](./development/phases.md)** for implementation order

## 📞 Support & Contributing

- **Issues**: Report bugs and feature requests in the project repository
- **Contributing**: See [Contributing Guidelines](./development/contributing.md)
- **Documentation**: Each module has detailed README files for specific areas

---

**Built with ❤️ for effective learning and knowledge retention**

> This documentation is organized for developers at all levels. Start with the Quick Start guide if you're new, or dive directly into specific modules based on your needs.