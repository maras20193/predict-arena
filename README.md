# ⚽ PredictArena

A web application for private match prediction tournaments among friends. PredictArena provides a secure, admin-moderated platform for organizing predictions with configurable scoring and transparent dashboards.

## Table of Contents

- [Project Description](#project-description)
- [Tech Stack](#tech-stack)
- [Getting Started Locally](#getting-started-locally)
- [Available Scripts](#available-scripts)
- [Project Scope](#project-scope)
- [Project Status](#project-status)
- [License](#license)

## Project Description

PredictArena is a web application designed to solve the chaos of organizing private prediction tournaments. It replaces error-prone spreadsheets and manual scoring with a centralized, secure platform that enforces time-based locks, automatic point calculation, and clear visualizations.

### Key Features (MVP)

- **Google Authentication**: One-click login via Supabase Auth with user profiles and avatars
- **Tournament Management**: Single tournament with phases containing matches
- **Time-based Controls**:
  - `lock_at`: Deadline for editing predictions (enforced on backend)
  - `start_at`: When predictions become visible to all participants
  - Default mode: `start_at = lock_at` (simple mode)
  - Advanced mode: Separate times supported
- **Admin Moderation**: Join requests require admin approval; access only before tournament start
- **Configurable Scoring**: Default 5 points for exact score, 3 points for correct winner/side (90-minute result only)
- **Dashboards**:
  - **Leaderboard**: Bar chart with avatars showing total points (no separate ranking table)
  - **Predictions Matrix**: Table with matches as rows, users as columns, color-coded cells (5 pts = green, 3 pts = blue)
  - **My Predictions**: Overview of submitted and open predictions with access to previous phase history
- **Privacy**: Predictions are private until phase `start_at`, then visible to all participants
- **No Realtime**: Data refreshes on page reload (no streaming)

## Tech Stack

### Frontend

- **Astro 5**: Fast, efficient static site generation with minimal JavaScript
- **React 19**: Interactive components where needed
- **TypeScript 5**: Static typing for better IDE support and code quality
- **Tailwind 4**: Utility-first CSS framework for styling
- **Shadcn/ui**: Accessible React component library
- **Recharts 3**: Bar charts for leaderboard visualization
- **Tanstack Query**: Efficient data fetching, caching, and state synchronization
- **Tanstack Table**: Virtualized, performant predictions matrix with grouping and sticky headers
- **react-hook-form + zod**: Form handling with strong client-side validation

### Backend

- **Supabase**: Backend-as-a-Service providing:
  - PostgreSQL database
  - Authentication (Google OAuth)
  - Row Level Security (RLS) for data access control
  - TypeScript SDK

### CI/CD & Hosting

- **GitHub Actions**: CI/CD pipelines
- **DigitalOcean**: Application hosting via Docker

## Getting Started Locally

### Prerequisites

- **Node.js**: Version 22.14.0 (see `.nvmrc`)
- **npm** or **yarn**: Package manager
- **Supabase Account**: For backend services (database, auth)

### Installation

1. **Clone the repository**:

   ```bash
   git clone <repository-url>
   cd predict-arena
   ```

2. **Install Node.js version** (if using nvm):

   ```bash
   nvm install
   nvm use
   ```

3. **Install dependencies**:

   ```bash
   npm install
   ```

4. **Configure environment variables**:

   Create a `.env` file in the root directory with your Supabase credentials:

   ```env
   PUBLIC_SUPABASE_URL=your_supabase_url
   PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
   ```

   > **Note**: You'll need to set up a Supabase project and configure Google OAuth provider. Refer to [Supabase documentation](https://supabase.com/docs) for detailed setup instructions.

5. **Start the development server**:

   ```bash
   npm run dev
   ```

6. **Open your browser**:

   Navigate to `http://localhost:4321` (or the port shown in your terminal)

### Additional Setup

- Configure Google OAuth in your Supabase project dashboard
- Set up database schema and RLS policies (see project documentation)
- Configure timezone settings (backend uses UTC, UI displays Europe/Warsaw)

## Available Scripts

| Script             | Description                                                    |
| ------------------ | -------------------------------------------------------------- |
| `npm run dev`      | Start the Astro development server with hot module replacement |
| `npm run build`    | Build the production-ready site to `./dist/`                   |
| `npm run preview`  | Preview the production build locally                           |
| `npm run astro`    | Run Astro CLI commands                                         |
| `npm run lint`     | Run ESLint to check for code issues                            |
| `npm run lint:fix` | Run ESLint and automatically fix issues                        |
| `npm run format`   | Format code using Prettier                                     |

## Project Scope

### In Scope (MVP)

- Single tournament with phases and matches
- Google OAuth authentication via Supabase
- Admin-moderated join requests (approval required)
- Time-based prediction locks (`lock_at`) and visibility (`start_at`)
- Configurable scoring system (default: 5/3 points)
- Leaderboard visualization (bar chart with avatars)
- Predictions matrix (virtualized table)
- Automatic point calculation
- Privacy controls (predictions hidden until `start_at`)
- Manual result entry by admin
- Phase history access
- Team flags (ISO-compliant, CDN with fallback)
- Basic analytics (page views, key events, TTFB/LCP metrics)

### Out of Scope (MVP)

- Realtime/streaming updates
- Multiple OAuth providers (Google only)
- External API integrations for schedules/results
- Advanced scoring rules (bonuses, extra time/penalties, multipliers)
- Public tournament catalogs or multi-tournament UI
- Email/push notifications
- Mobile applications (web only)
- CSV import/export

### Limitations

- One admin per tournament
- Scoring based on 90-minute result only
- Join requests only before tournament start
- Ranking without tie-breakers (ex aequo with gaps, e.g., 1,1,3)
- No realtime updates (data refreshes on page reload)

## Project Status

**Status**: 🚧 MVP in Development

This project is currently in the MVP (Minimum Viable Product) phase. The focus is on delivering core functionality for organizing private prediction tournaments with:

- Secure authentication and access control
- Time-based prediction management
- Automatic scoring
- Clear visualizations

### Success Metrics (Target)

- 20-30 active players join and are accepted in tournament
- At least 85% of players submit predictions for full phase before `lock_at`
- Zero ability to edit predictions after `lock_at` (RLS enforced)
- Dashboards load under 2 seconds with 30 players and dozens of matches
- Admin creates phase and adds matches within 15 minutes
- Admin enters single match result within 15 seconds
- 100% accuracy in automatic point calculation

## License

This project is currently unlicensed. All rights reserved.

---

For detailed product requirements and user stories, see [`.ai/prd.md`](.ai/prd.md).
