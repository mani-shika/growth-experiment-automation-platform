# Build: Growth Experiment Automation Platform

## Context
This platform automates the entire growth experimentation lifecycle for SaaS companies and digital products. It identifies potential growth opportunities through data analysis, automatically generates and deploys A/B tests, monitors their performance, and applies winning variations—all with minimal human oversight. The target users are growth teams, product managers, and founders who want to accelerate their experimentation velocity without expanding headcount. The idea stems from Anthropic's approach to automating their own growth processes, where AI systems identify opportunities, create experiments, and optimize based on results.

## Tech Stack
- **Next.js 14 with App Router** - Server components for performance and built-in API routes for webhook handlers
- **Convex** - Real-time database perfect for live experiment metrics and automatic state updates
- **Anthropic Claude API** - Analyzes user behavior patterns and generates experiment hypotheses
- **Auth.js with Google OAuth** - Simple team authentication without password management overhead
- **Stripe** - Subscription billing for platform access tiers
- **PostHog** - Analytics API to pull user behavior data for experiment targeting
- **Vercel Analytics** - Track experiment performance metrics
- **shadcn/ui + Tailwind CSS** - Consistent, accessible UI components
- **React Query** - Efficient data fetching with built-in caching for analytics dashboards

## Phase 1: Project Setup & Scaffolding
Create a Next.js 14 app with TypeScript, Tailwind, and shadcn/ui. Install Convex for the database, Auth.js for authentication, and React Query for data fetching. Set up environment variables for Anthropic API, PostHog, Stripe, and OAuth credentials.

Core data model centers around three entities:
- **Experiment**: Contains hypothesis, target metrics, audience criteria, variations, and status
- **Variation**: Holds the specific changes to test (copy, design, flow modifications)
- **Result**: Tracks performance metrics, statistical significance, and conversion data

```typescript
interface Experiment {
  id: string
  hypothesis: string
  targetMetric: string
  status: 'proposed' | 'running' | 'completed' | 'applied'
  variations: Variation[]
  results?: Result
  createdBy: 'ai' | 'human'
}
```

## Phase 2: Core Data Model & Backend
Convex handles all data persistence with real-time subscriptions for live experiment updates. Experiments are the central entity, linking to variations (the different versions being tested) and results (performance data). The system maintains a queue of AI-proposed experiments awaiting approval, active experiments being monitored, and completed experiments with their outcomes.

Background jobs run every hour to analyze user behavior via PostHog API, generate new experiment ideas through Claude, and check running experiments for statistical significance. When experiments conclude, winning variations can be automatically applied or queued for human review based on confidence thresholds.

## Phase 3: UI & Components
**Dashboard** - Grid layout showing active experiments with real-time conversion rates, AI-proposed experiments awaiting review, and recent completed experiments with their impact.

**Experiment Card** - Displays hypothesis, current performance metrics, time remaining, and confidence intervals. Shows live updating charts for conversion rates and includes quick actions to pause, conclude early, or modify parameters.

**AI Proposal Review** - Full-screen view for reviewing AI-generated experiments. Shows the hypothesis rationale, proposed variations with visual previews, estimated impact, and implementation complexity. Includes approve/reject/modify actions.

**Experiment Builder** - Form for manually creating experiments or editing AI proposals. Includes audience targeting rules, success metrics selection, variation editor with live preview, and scheduling options.

**Analytics View** - Time-series charts showing experiment impact on key metrics, cumulative gains from applied experiments, and AI performance metrics (proposal acceptance rate, prediction accuracy).

## Phase 4: Feature Implementation
**AI Experiment Generation** - Every hour, the system pulls recent user behavior data from PostHog, analyzes patterns with Claude to identify drop-off points and optimization opportunities, generates 3-5 experiment hypotheses with implementation details, and adds them to the proposal queue with priority scores.

**Automatic Experiment Deployment** - When experiments are approved, the system generates the necessary code changes or configuration updates, deploys variations through feature flags or dynamic content injection, assigns users to control/treatment groups based on targeting rules, and begins collecting performance data.

**Real-time Monitoring** - Convex subscriptions push live updates to the dashboard showing current conversion rates, confidence intervals, and projected outcomes. The system automatically pauses experiments that show significant negative impact and alerts teams when experiments reach statistical significance.

**Winner Application** - Completed experiments with clear winners can be automatically applied based on confidence thresholds. The system generates pull requests for permanent changes, updates feature flags to serve winning variations to all users, and archives experiment data for future reference.

## Phase 5: Polish & Integration
Implement loading skeletons for all data-fetching states and error boundaries with helpful messages for API failures. Design responsive layouts that work on mobile for checking experiment status on the go, with desktop-optimized views for detailed analysis and configuration.

Add keyboard shortcuts for common actions like approving proposals or pausing experiments. Include a global search to find past experiments by hypothesis or metric. Build a notification system for key events like experiments reaching significance or AI proposing high-impact ideas.

Create an onboarding flow that connects analytics accounts, sets target metrics and guardrails, and configures AI generation preferences. Add role-based access control so team members can have view-only or approval permissions.

## Verification
Run `npm run dev` to start the development server and `npm run build` for production build. Test the full experiment lifecycle: connect PostHog account, wait for AI to generate proposals, approve an experiment, verify it deploys correctly, monitor real-time updates, and confirm winner application works.

Verify edge cases: experiments with no clear winner after maximum duration, API rate limits on analytics data fetching, handling of conflicting experiments targeting same users, and graceful degradation when AI API is unavailable. Test team collaboration by having multiple users review and approve experiments simultaneously.