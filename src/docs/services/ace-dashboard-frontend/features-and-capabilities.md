# Features and capabilities

## 1. Routing and access control

- **Public routes**: `/login`, `/login/success`, `/settings/provider-callback`, `/admin-signup`, `/verify`, `/request-recover-password`, `/reset-password`, `/onboarding`. Temporary `/tmp-*` routes exist for direct access to some management pages without the full admin layout.
- **Private routes** (require authenticated user): `/`, `/data-sources`, `/assistant`, `/documentation`, `/secrets`, `/commands`, `/integrations`, `/settings`; `/ace-aws`, `/ace-aws/knowledge-base`, `/ace-aws/kb-deletion-blacklist`; `/ace-jira`, `/ace-jira/project-links`; `/client-management`, `/user-management`, `/project-management`, `/dashboard-management`, `/user-project-link`, `/channel-management`.
- **Admin panel** (under `/admin-panel`): Dashboard, user/client/project management, user-project link, external linkages, add external users, channel management, permission management, user type management, user type permissions, projects configurations, monitored resources (when feature-flagged). Client management under admin is wrapped in **SuperUserRoute**.
- **SuperUserRoute**: Restricts ACE AWS, ACE Jira, and admin-panel client-management to super users only.
- **404**: Any other path renders the NotFound page.

## 2. Authentication

- Login (and optional OAuth provider callback), admin sign-up, verify, request recover password, reset password.
- JWT stored (e.g. in localStorage); sent as `Authorization: Bearer <token>` on API requests.
- **AuthContext** provides user state and logout; **PrivateRoute** redirects unauthenticated users to login; axios interceptor in useApi redirects to login on 401.

## 3. API clients

- **VITE_API_URL** (useApi / axios): Admin and user APIs—users, clients, projects, channels, permissions, user types, configurations, external linkages, etc. Base URL must be set at build time.
- **VITE_BACKEND_URL** (DashboardBackend and some hooks): Secrets, documentations, chat, integrations, user projects. Used by fetch and by hooks such as useConversationApi, use-omni-socket.
- **VITE_LLM_URL**: Used by LLM-related features when configured.
- **VITE_FRONTEND_URL**: Used for redirects and links (e.g. login success, provider callback, profile settings).
- **VITE_AWS_REGION**: Optional; used in ACE AWS Knowledge Base (e.g. default region).

## 4. Main feature areas

- **Home / Index**: Landing after login.
- **Assistant**: Chat interface; real-time via socket and backend chat API.
- **Documentation, Commands, Secrets, Data sources, Integrations, Settings**: Project-scoped or user-scoped flows; data from backend.
- **Admin panel**: Central place for user, client, project, channel, permission, user type, and configuration management; external linkages; Slack channel management; monitored resources (if `FEATURE_MONITORED_RESOURCES_ENABLED` is true).
- **ACE AWS**: Knowledge Base and KB deletion blacklist (super user).
- **ACE Jira**: Jira project links (super user).

## 5. Feature flags

- **FEATURE_MONITORED_RESOURCES_ENABLED** (`src/lib/features.ts`): When `false`, the monitored resources menu item is hidden and navigation to `/admin-panel/monitored-resources` redirects to `/admin-panel`. Set to `true` to enable without removing code.

## 6. UI and tooling

- **Components**: Radix-based UI (shadcn/ui style), Tailwind; modals, forms, tables, toasts (sonner).
- **Forms**: react-hook-form with zod validation where used.
- **Markdown**: react-markdown, remark-gfm, mermaid for content and diagrams.
- **Charts**: recharts where used.
- **Theme**: next-themes for dark/light.

## 7. Development and quality

- **Lint**: `yarn lint` (ESLint).
- **Tests**: Playwright E2E (`yarn test`); optional `yarn test:user-management` for user-management flows. See repo `docs/development/testing.md`.
- **Path alias**: `@/` resolves to `src/` (Vite and TypeScript).
