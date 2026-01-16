# Nexus PM - Elite Project Management

<div align="center">
  <h3>Enterprise-Grade Project Management System</h3>
  <p>Built with Clean Architecture, TypeScript, and React</p>
</div>

## Overview

Nexus PM is a sophisticated project management web application designed for enterprise use. It provides comprehensive project tracking, task management, team collaboration, and client portal capabilities.

### Key Features

- ✅ **Project Management**: Create, update, archive, and remove projects with full budget tracking
- ✅ **Task Management**: List view, Kanban board, and Gantt chart with dependencies
- ✅ **Team Management**: Create teams, assign to projects and tasks with safe deletion
- ✅ **Client Portal**: Read-only access for clients to view progress and submit tickets
- ✅ **Ticket System**: Unified ticketing for inquiries, issues, and feature requests
- ✅ **Configuration-Driven**: Feature flags and workflow configuration
- ✅ **Clean Architecture**: Separation of concerns, testable, maintainable

## Architecture

This application follows **Clean Architecture** principles with clear separation of concerns:

```
UI Layer (Components)
    ↓
Business Logic Layer (Services)
    ↓
Domain Models Layer (Core)
    ↓
Data Access Layer (Repositories)
```

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed architecture documentation.

## Quick Start

### Prerequisites

- Node.js 18+ 
- npm or yarn

### Installation

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Run the development server:**
   ```bash
   npm run dev
   ```

3. **Open your browser:**
   Navigate to `http://localhost:5173` (or the port shown in terminal)

### Build for Production

```bash
npm run build
npm run preview
```

## Project Structure

```
nexus-pm/
├── core/                    # Core business logic
│   ├── models/              # Domain models (Project, Task, Team, Ticket, Client)
│   └── services/            # Business logic services
├── data/                    # Data access layer
│   └── repositories/        # Data repositories
├── config/                  # Configuration
│   └── AppConfig.ts         # Feature flags and settings
├── components/              # React UI components
├── utils/                   # Utility functions
└── types.ts                 # Type definitions (backward compatibility)
```

## Core Concepts

### Projects

Projects are the top-level organizational unit. Each project has:

- **Timeline**: Start and end dates
- **Budget**: Allocated budget, expenses, and resource allocations
- **Deliverables**: Expected outputs from the project
- **Teams**: Assigned teams working on the project
- **Status**: Draft → Active → Archived

**Example:**
```typescript
import { projectService } from './core/services/ProjectService';

const project = projectService.createProject({
  name: 'Website Redesign',
  description: 'Complete website overhaul',
  startDate: '2024-01-01',
  endDate: '2024-06-30',
  budgetAllocated: 150000,
  currency: 'USD',
  clientId: 'client-1',
});
```

### Tasks

Tasks represent work items within a project. Features:

- **Workflow**: Configurable status workflow (todo → in-progress → review → done)
- **Dependencies**: Tasks can depend on other tasks (no circular dependencies)
- **Assignment**: Assigned to teams or individual members
- **Grouping**: Can be grouped into task groups/epics

**Example:**
```typescript
import { taskService } from './core/services/TaskService';

const task = taskService.createTask({
  projectId: 'proj-1',
  title: 'Design homepage',
  description: 'Create homepage mockups',
  priority: Priority.HIGH,
  startDate: '2024-01-05',
  endDate: '2024-01-20',
  dependencies: ['task-1'], // Depends on task-1
});
```

### Teams

Teams are collections of team members that can be assigned to projects and tasks.

**Business Rules:**
- Teams must have at least one member
- Teams cannot be deleted if assigned to projects or tasks

**Example:**
```typescript
import { teamService } from './core/services/TeamService';

const team = teamService.createTeam({
  name: 'Frontend Squad',
  members: [
    { name: 'Alice', email: 'alice@example.com', role: 'Developer' },
    { name: 'Bob', email: 'bob@example.com', role: 'Designer' },
  ],
});
```

### Tickets

Tickets are used for client inquiries, bug reports, and feature requests.

**Types:**
- `inquiry`: General question
- `issue`: Bug report
- `feature_request`: Request for new functionality
- `backlog_item`: Work item for future consideration

**Example:**
```typescript
import { ticketService } from './core/services/TicketService';

const ticket = ticketService.createTicket({
  type: TicketType.ISSUE,
  title: 'Login button not working',
  description: 'The login button does not respond when clicked',
  priority: Priority.HIGH,
  projectId: 'proj-1',
  reporterName: 'John Doe',
  reporterEmail: 'john@example.com',
});
```

## Configuration

The application uses a centralized configuration system for feature flags and workflows.

### Feature Flags

Enable/disable features via `config/AppConfig.ts`:

```typescript
import { getAppConfig } from './config/AppConfig';

const config = getAppConfig();

if (config.features.ganttChartEnabled) {
  // Show Gantt chart
}

if (config.features.clientTicketingEnabled) {
  // Allow ticket creation
}
```

### Workflow Configuration

Task and ticket workflows are configurable:

```typescript
const config = getAppConfig();

// Task workflow
config.taskWorkflow.statuses; // ['todo', 'in-progress', 'review', 'done', ...]
config.taskWorkflow.transitions; // Allowed status transitions

// Ticket workflow
config.ticketWorkflow.statuses; // ['open', 'in-progress', 'resolved', ...]
config.ticketWorkflow.transitions; // Allowed status transitions
```

## Usage Examples

### Creating a Project with Tasks

```typescript
import { projectService } from './core/services/ProjectService';
import { taskService } from './core/services/TaskService';
import { ProjectRepository } from './data/repositories/ProjectRepository';
import { TaskRepository } from './data/repositories/TaskRepository';

// Create project
const project = projectService.createProject({
  name: 'New Project',
  description: 'Project description',
  startDate: '2024-01-01',
  endDate: '2024-12-31',
  budgetAllocated: 100000,
  currency: 'USD',
  clientId: 'client-1',
});

// Save project
const projectRepo = new ProjectRepository(projectService);
projectRepo.create(project);

// Create tasks
const task1 = taskService.createTask({
  projectId: project.id,
  title: 'Task 1',
  description: 'First task',
  priority: Priority.HIGH,
  startDate: '2024-01-01',
  endDate: '2024-01-15',
});

const task2 = taskService.createTask({
  projectId: project.id,
  title: 'Task 2',
  description: 'Second task',
  priority: Priority.MEDIUM,
  startDate: '2024-01-16',
  endDate: '2024-01-30',
  dependencies: [task1.id], // Depends on task1
});

// Save tasks
const taskRepo = new TaskRepository(taskService);
taskRepo.create(task1);
taskRepo.create(task2);
```

### Updating Task Status

```typescript
import { taskService } from './core/services/TaskService';
import { TaskStatus } from './core/models/Task';

// Update task status (validates workflow transitions)
const updatedTask = taskService.updateTaskStatus(
  task,
  TaskStatus.IN_PROGRESS,
  allTasks
);
```

### Safe Team Deletion

```typescript
import { teamService } from './core/services/TeamService';

// Check if team can be deleted
const canDelete = teamService.canDeleteTeam(teamId, projects, tasks);

if (canDelete.canDelete) {
  // Safe to delete
  teamService.deleteTeam(teamId, projects, tasks);
} else {
  // Show error: canDelete.reason
  console.error(canDelete.reason);
}
```

## Client Portal

The client portal provides read-only access for clients to:

- View project progress and timelines
- See milestones and deliverables
- Submit tickets, inquiries, and backlog requests
- View project financials (spend to date)

Clients have **read-only access** except for ticket submission.

## Development Guidelines

### Code Style

- **TypeScript**: Strict mode enabled
- **Naming**: Clear, descriptive names
- **Comments**: Explain "why", not "what"
- **Documentation**: JSDoc for all public methods

### Architecture Principles

1. **Separation of Concerns**: UI, business logic, and data access are separate
2. **Single Responsibility**: Each class has one reason to change
3. **Dependency Inversion**: Dependencies flow inward (UI → Services → Models)
4. **Open/Closed**: Open for extension, closed for modification

### Adding New Features

1. Define domain model in `core/models/`
2. Create service in `core/services/`
3. Create repository in `data/repositories/`
4. Add feature flag in `config/AppConfig.ts`
5. Build UI components
6. Wire up in App.tsx

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed guidelines.

## Testing

### Unit Tests

Test business logic in isolation:

```typescript
import { projectService } from './core/services/ProjectService';

test('should create project with valid dates', () => {
  const project = projectService.createProject({
    name: 'Test',
    startDate: '2024-01-01',
    endDate: '2024-12-31',
    // ...
  });
  
  expect(project.status).toBe(ProjectStatus.ACTIVE);
});
```

### Integration Tests

Test service + repository interactions:

```typescript
test('should persist project', () => {
  const project = projectService.createProject({...});
  const repo = new ProjectRepository(projectService);
  repo.create(project);
  
  const retrieved = repo.getById(project.id);
  expect(retrieved).toEqual(project);
});
```

## API Reference

### ProjectService

- `createProject(payload)`: Create new project
- `updateProject(id, updates)`: Update project
- `archiveProject(id)`: Archive project
- `calculateTotalSpend(project)`: Calculate total spend
- `canDeleteProject(project)`: Check if project can be deleted

### TaskService

- `createTask(payload)`: Create new task
- `updateTaskStatus(task, newStatus, allTasks)`: Update task status
- `validateDependencies(task, allTasks)`: Validate dependency graph
- `canTransitionStatus(current, new)`: Check status transition

### TeamService

- `createTeam(payload)`: Create new team
- `deleteTeam(id, projects, tasks)`: Delete team safely
- `canDeleteTeam(id, projects, tasks)`: Check if team can be deleted

### TicketService

- `createTicket(payload)`: Create new ticket
- `updateTicketStatus(ticket, newStatus)`: Update ticket status
- `resolveTicket(id, resolution, resolvedBy)`: Resolve ticket

See source files for complete API documentation.

## Contributing

1. Follow the architecture patterns established in the codebase
2. Add JSDoc comments for all public methods
3. Write tests for business logic
4. Update documentation as needed
5. Ensure TypeScript strict mode compliance

## License

This project is proprietary software.

## Support

For questions or issues:
1. Check [ARCHITECTURE.md](./ARCHITECTURE.md) for architecture details
2. Review inline code comments and JSDoc
3. Consult the API reference above

---

**Built with ❤️ using Clean Architecture principles**
# NexusPM
