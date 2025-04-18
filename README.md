# RooStats: Analytics Platform for Roobet

![RooStats Logo](/images/icon.ico)

## Overview

RooStats is a comprehensive analytics platform designed to help Roobet users gain deeper insights into their gambling patterns, profitability, and gameplay statistics. Similar to StakeStats.net, RooStats provides detailed visualizations, performance metrics, and personalized insights to enhance your gaming experience.

## Features

- **Comprehensive Bet History**: Track all your bets, wins, losses, and profitability metrics
- **Game-Specific Analytics**: Analyze performance across different games and providers
- **Session Analysis**: Review gameplay sessions with detailed breakdowns
- **Data Export**: Download your stats in various formats (PDF, JSON, Excel)
- **Personalized Insights**: Get tailored recommendations based on your playing patterns
- **Secure Authentication**: Discord OAuth integration for easy and secure login

## Project Structure

```
roostats/
├── docs/
│   ├── roo-integration.md       # Integration guide for Roobet API
│   └── roo-history.yaml         # OpenAPI specifications for Roobet History API
├── frontend/                    # React/Next.js application
├── backend/                     # Node.js API server
└── scripts/                     # Utility scripts
```

## Documentation

### Roo-Integration Guide

The [roo-integration.md](docs/roo-integration.md) document outlines the implementation details for integrating with Roobet's API endpoints. It covers:

- Authentication methods and security considerations
- Data retrieval techniques from Roobet's undocumented APIs
- User account linking methodology
- Few proposals to the roobet-dev team for implementing a history retrieval functionallity, used by the roobet's end user for downloading bet history
- Rate limiting and performance optimization strategies(TBD)
- Error handling and fallback procedures(TBD)

This guide provides some variants, for the roobet devs,

### Roobet API Documentation

[roo-history.yaml](docs/roo-history.yaml) is an OpenAPI 3.1.0 specification document that provides a comprehensive definition of Roobet's betting history API endpoints(at this point). It includes:

- Detailed endpoint descriptions for `/_api/history` and `/_api/history/bets`
- Request parameters and pagination options
- Complete schema definitions for API responses
- Example responses and status codes
- Security scheme information

This specification provides the API endpoints available. It will get extended, as Roobet dev team provide us with more information.


## Getting Started

### Prerequisites

- Node.js 18.x or higher
- NPM or Yarn
- MongoDB
- Redis (for caching)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/roostats.git
   cd roostats
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up environment variables:
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. Start development server:
   ```bash
   npm run dev
   ```

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## Security

RooStats takes security seriously. We never store Roobet login credentials and use secure OAuth flows for authentication. All sensitive data is encrypted, and we follow industry best practices for data protection.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE.txt) file for details.

## Acknowledgments

- Inspired by StakeStats.net
- Built with support from the Roobet community
- Special thanks to all contributors

---

**Disclaimer**: RooStats is an independent project and is not officially affiliated with or endorsed by Roobet.
