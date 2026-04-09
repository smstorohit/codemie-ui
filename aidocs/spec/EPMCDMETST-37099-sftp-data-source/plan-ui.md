# Implementation Plan: Add SFTP Data Source to CodeMie Platform (Frontend)

## Overview
This plan covers the frontend implementation for adding SFTP (Secure File Transfer Protocol) data source configuration UI to the CodeMie platform. Users will be able to configure SFTP connections through a form-based interface with real-time validation, test connections before saving, and manage SFTP data sources through the existing data sources management interface.

**Repository**: codemie-ui (https://github.com/smstorohit/codemie-ui)

## User Story
**As a** CodeMie administrator,  
**I want to** configure an SFTP data source in the CodeMie setup wizard,  
**so that** I can securely pull and index CSV files from legacy systems for use with CodeMie assistants.

### Acceptance Criteria
1. SFTP Data Source option available in data sources creation UI
2. Configuration form with fields: Project, Alias, Hostname, Port (default 22), Username, Password, Directory Path, File Pattern (default *.csv)
3. All required fields display validation errors when empty or invalid
4. "Test Connection" button validates credentials and connectivity before saving
5. Form follows existing CodeMie data source UI patterns (similar to SharePoint/Confluence)
6. SFTP data sources appear in Data Sources list with status indicators
7. Users can edit, pause, resume, or delete SFTP data sources
8. UI displays connection status, file retrieval logs, and indexing progress

### Assumptions
- Backend API endpoints are implemented as specified in backend plan
- Form validation follows existing CodeMie patterns (Zod schemas, react-hook-form)
- "Test Connection" uses existing health check API pattern
- SFTP is added to existing INDEX_TYPES constant
- UI components leverage existing form components (Input, Switch, etc.)

## Research Findings

### Existing Codebase Patterns
- **Similar Features**: SharePoint, Confluence, Jira, Azure DevOps Wiki data source forms
- **Reusable Components**:
  - `DataSourceForm` - Main form component for data source configuration
  - `IndexTypeField` components - Type-specific field renderers (e.g., IndexTypeSharePoint)
  - `HealthCheckMessage` - Component for displaying test connection results
  - `Input`, `Switch`, `Textarea` - Form field components with validation
  - `ProjectSelector` - Project dropdown component
- **Established Patterns**:
  - Form state management: react-hook-form with Zod validation
  - Test connection: Health check API call with loading/success/error states
  - Data source list: Status indicators (active, indexing, failed, paused)
  - Edit/Delete: Modal-based workflows

### CodeMie User Guide References
- **Relevant Documentation**: Data source configuration patterns, integration setup flows
- **User Flows**: Data Sources → + Create → Select Type → Configure → Test Connection → Save
- **Platform Conventions**: Required field indicators (*), inline validation, tooltip help text

## Technical Context
- **Repository**: codemie-ui
- **Tech Stack**: React 18, TypeScript, Vite, react-hook-form, Zod, Valtio (state), TailwindCSS
- **Dependencies**: 
  - Existing: `react-hook-form`, `zod`, `valtio`, `@tanstack/react-query`
  - No new dependencies required
- **Integrations**: 
  - Backend API: `/v1/index/knowledge_base/sftp` (POST), `/v1/index/{id}/sftp` (PUT)
  - Health check: `/v1/index/datasource/health-check` (POST)

## Architecture & Design

### Component Overview
1. **IndexTypeSFTP** (`src/pages/dataSources/components/DataSourceForm/IndexTypeField/IndexTypeSFTP.tsx`): New component rendering SFTP-specific form fields
2. **DataSourceForm Updates** (`src/pages/dataSources/components/DataSourceForm/DataSourceForm.tsx`): Add SFTP to index type switch/router
3. **Constants Updates** (`src/constants/dataSources.ts`): Add SFTP to INDEX_TYPES enum
4. **Type Definitions** (`src/types/entity/dataSource.ts`): Add SFTPDataSourceConfig interface
5. **API Integration** (`src/api/dataSources.ts` or inline): Add createSFTPDataSource, updateSFTPDataSource functions

### UI/UX Design

**Form Layout** (similar to existing data source forms):

```
+----------------------------------------------+
| Data Source Configuration                     |
+----------------------------------------------+
| Project *          [Dropdown: My Project  ▼] |
| Alias *            [Input: my-sftp-source   ] |
| Description *      [Textarea: ...           ] |
+----------------------------------------------+
| Index Type *       [Dropdown: SFTP        ▼] |
+----------------------------------------------+
| SFTP Configuration                            |
+----------------------------------------------+
| Hostname *         [Input: sftp.example.com ] |
|                    ⓘ SFTP server address     |
| Port               [Input: 22               ] |
|                    ⓘ Default: 22 (SFTP)      |
| Username *         [Input: admin            ] |
| Password *         [Input: ••••••••••••      ] |
| Directory Path *   [Input: /data/exports    ] |
|                    ⓘ Remote directory path   |
| File Pattern       [Input: *.csv            ] |
|                    ⓘ Glob pattern (default: *.csv) |
+----------------------------------------------+
| Cron Expression    [Input: */15 * * * *    ] |
| (optional)         ⓘ Leave blank for default  |
+----------------------------------------------+
| [Test Connection]           [Cancel] [Save] |
+----------------------------------------------+
```

**Test Connection States**:
- **Loading**: Button shows spinner, disabled
- **Success**: Green checkmark with "Connection successful! Found X files."
- **Error**: Red X with error message (e.g., "Authentication failed", "Directory not found")

### Data Model

```typescript
// src/types/entity/dataSource.ts
export interface SFTPDataSourceConfig {
  hostname: string
  port?: number // Default: 22
  username: string
  password: string
  directory_path: string
  file_pattern?: string // Default: *.csv
}

export interface SFTPDataSourceFormValues {
  name: string
  project_name: string
  description: string
  project_space_visible?: boolean
  index_type: 'knowledge_base_sftp'
  sftp: SFTPDataSourceConfig
  cron_expression?: string
  embedding_model?: string
  guardrail_assignments?: GuardrailAssignmentItem[]
}
```

### Security Considerations
- **Password Masking**: Use `type="password"` input field; never display password after saving
- **HTTPS Only**: All API calls use HTTPS (enforced by platform)
- **Client-Side Validation**: Validate hostname format, port range (1-65535), required fields
- **No Credential Storage**: Form state cleared after submission; passwords not persisted in browser storage

### Error Handling
- **Form Validation Errors**: Display inline below each field (Zod validation)
- **API Errors**: Display toast notifications for save/update failures
- **Test Connection Errors**: Display error message in HealthCheckMessage component with actionable help text
- **Network Errors**: Retry with exponential backoff (react-query default behavior)

## Implementation Phases

### Phase 0: Research & Discovery ✅
- [x] Review existing SharePoint/Confluence data source form implementations *(completed during planning)*
- [x] Analyze form patterns and validation strategies *(completed during planning)*
- [x] Review CodeMie User Guide for UI conventions *(completed during planning)*
- [ ] Confirm backend API contracts with backend team

### Phase 1: Constants & Type Definitions
**Goal**: Add SFTP to platform constants and TypeScript type definitions

**Tasks**:
1. Update `src/constants/dataSources.ts`:
   - Add `knowledge_base_sftp` to `INDEX_TYPES` array
   - Add SFTP to `INDEX_TYPE_LABELS` mapping
   - (Optional) Add SFTP icon to `INDEX_TYPE_ICONS`
2. Update `src/types/entity/dataSource.ts`:
   - Add `SFTPDataSourceConfig` interface
   - Add `SFTPDataSourceFormValues` interface
   - Update `DataSourceDetailsResponse` union type to include SFTP
3. Create Zod validation schema for SFTP form fields

**Files Created/Modified**:
- `src/constants/dataSources.ts` (add SFTP constants)
- `src/types/entity/dataSource.ts` (add SFTP types)
- `src/schemas/sftpDataSource.ts` (new, Zod validation schema)

**Acceptance Criteria**:
- SFTP appears in INDEX_TYPES dropdown in data source creation UI
- TypeScript types compile without errors
- Zod schema validates all required/optional fields correctly

---

### Phase 2: IndexTypeSFTP Component
**Goal**: Create form fields component for SFTP-specific configuration

**Tasks**:
1. Create `src/pages/dataSources/components/DataSourceForm/IndexTypeField/IndexTypeSFTP.tsx`
2. Implement IndexTypeSFTP component:
   - Accept `control` prop from react-hook-form
   - Render form fields: Hostname, Port, Username, Password, Directory Path, File Pattern
   - Add default value for Port (22) and File Pattern (*.csv)
   - Add tooltip help text for each field (ⓘ icon)
   - Follow existing field component patterns (use `<Controller>` from react-hook-form)
3. Add inline validation error display for each field
4. Style using TailwindCSS (match existing form field styles)

**Files Created/Modified**:
- `src/pages/dataSources/components/DataSourceForm/IndexTypeField/IndexTypeSFTP.tsx` (new)

**Acceptance Criteria**:
- Component renders all SFTP fields correctly
- Port defaults to 22, File Pattern defaults to *.csv
- Validation errors display inline below fields
- Tooltips provide helpful context for each field
- Styling matches existing data source forms

---

### Phase 3: DataSourceForm Integration
**Goal**: Integrate IndexTypeSFTP into main DataSourceForm component

**Tasks**:
1. Update `src/pages/dataSources/components/DataSourceForm/DataSourceForm.tsx`:
   - Import IndexTypeSFTP component
   - Add SFTP case to index type conditional rendering (switch/if-else)
   - Pass form control to IndexTypeSFTP
2. Update form submission logic:
   - Map SFTP form values to API request format
   - Handle SFTP-specific fields in `onSubmit` handler
3. Update DataSourceForm props/types to support SFTP config

**Files Created/Modified**:
- `src/pages/dataSources/components/DataSourceForm/DataSourceForm.tsx` (update)

**Acceptance Criteria**:
- Selecting "SFTP" in Index Type dropdown shows IndexTypeSFTP fields
- Form submission correctly sends SFTP config to backend API
- Edit mode correctly populates SFTP fields from existing data source

---

### Phase 4: Test Connection Feature
**Goal**: Implement "Test Connection" button with health check API integration

**Tasks**:
1. Add "Test Connection" button to IndexTypeSFTP component (conditionally shown)
2. Implement health check API call:
   - Use existing `/v1/index/datasource/health-check` endpoint
   - Send current form values (hostname, port, username, password, directory_path)
   - Handle loading/success/error states
3. Display HealthCheckMessage component with results:
   - Success: "Connection successful! Found X files."
   - Error: Display error.message with help text (e.g., "Check hostname", "Verify credentials")
4. Disable "Test Connection" button while form fields are invalid

**Files Created/Modified**:
- `src/pages/dataSources/components/DataSourceForm/IndexTypeField/IndexTypeSFTP.tsx` (add Test Connection logic)
- (Optional) `src/api/dataSources.ts` (add testSFTPConnection function)

**Acceptance Criteria**:
- "Test Connection" button appears below SFTP form fields
- Button disabled when required fields are empty
- Clicking button triggers health check API call
- Success state shows file count from API response
- Error state displays actionable error messages
- Health check does not save data source (validation only)

---

### Phase 5: Data Source List & Management
**Goal**: Display SFTP data sources in list with status indicators and management actions

**Tasks**:
1. Verify SFTP data sources appear in Data Sources list (likely works automatically if backend returns correct format)
2. Add SFTP icon/label to data source type column
3. Ensure Edit/Delete/Pause/Resume actions work for SFTP sources
4. Test status indicators: Active, Indexing, Failed, Paused
5. Add SFTP-specific details to data source detail view (if exists)

**Files Created/Modified**:
- (Likely no changes needed if backend returns standard IndexInfo format)
- `src/pages/dataSources/DataSourcesList.tsx` (verify SFTP display)
- (Optional) Add SFTP icon asset

**Acceptance Criteria**:
- SFTP data sources display in list with correct type label/icon
- Status indicators update correctly during indexing
- Edit modal opens with pre-filled SFTP fields
- Delete action works correctly
- Pause/Resume actions work (if applicable to SFTP)

---

### Phase 6: Testing & Documentation
**Goal**: Comprehensive testing and user documentation

**Tasks**:
1. **Unit Tests** (if testing infrastructure exists):
   - IndexTypeSFTP component: Field rendering, validation
   - Form submission: Correct API payload format
2. **Integration Tests** (if E2E framework exists):
   - Create SFTP data source flow: Fill form → Test connection → Save
   - Edit SFTP data source flow
   - Delete SFTP data source flow
3. **Manual Testing**:
   - Test all form validation scenarios (empty fields, invalid port, etc.)
   - Test "Test Connection" with valid/invalid credentials
   - Test create/edit/delete workflows
   - Test with different screen sizes (responsive design)
4. **Documentation**:
   - Update CodeMie User Guide with SFTP setup walkthrough (screenshots)
   - Add troubleshooting section for common UI errors
   - Document field requirements and validation rules
5. **Accessibility**:
   - Verify keyboard navigation works
   - Check screen reader compatibility
   - Ensure color contrast meets WCAG standards

**Files Created/Modified**:
- `tests/dataSources/IndexTypeSFTP.test.tsx` (new, unit tests)
- `e2e/dataSources/sftpDataSource.spec.ts` (new, E2E tests)
- User Guide updates (external documentation)

**Acceptance Criteria**:
- All validation scenarios covered by tests
- E2E tests pass for create/edit/delete flows
- Manual testing checklist completed
- User Guide includes SFTP setup instructions with screenshots
- Accessibility audit passes (keyboard nav, screen reader, contrast)

## Expected Artifacts
- `plan-ui.md` (this file)
- IndexTypeSFTP component (`IndexTypeSFTP.tsx`)
- Updated DataSourceForm integration
- SFTP type definitions and Zod schemas
- Updated constants (INDEX_TYPES, labels)
- Unit and integration tests
- Updated CodeMie User Guide sections

## Dependencies
- **Backend API**: Endpoints must be implemented per backend plan OpenAPI spec
- **Design Assets**: SFTP icon (optional, can use default datasource icon)
- **Documentation**: Screenshots for User Guide (created during testing phase)

## Notes
- **Demo Timeline**: Frontend implementation estimated at 3-5 days; demo scheduled July 15th, 2025
- **Responsive Design**: Form should work on desktop and tablet (mobile may have constrained layout)
- **Browser Support**: Chrome, Firefox, Edge, Safari (latest versions)
- **Future Enhancements** (out of scope for v1):
  - SSH key-based authentication UI (file upload for private key)
  - User-configurable polling interval UI (slider/dropdown)
  - File format selector (CSV, JSON, XML checkboxes)
  - Advanced settings accordion (connection timeout, retry settings)
  - Real-time file preview after test connection
