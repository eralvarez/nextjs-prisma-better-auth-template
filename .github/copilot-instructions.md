# Copilot Instructions

## Tech Stack

This is a **Next.js 16** project with the following technologies:

- **Framework**: Next.js 16.0.10 (App Router)
- **React**: React 19.2.1 (latest)
- **TypeScript**: TypeScript 5
- **UI Library**: Material-UI v7.3.6 (@mui/material)
- **Styling**: Emotion (@emotion/react, @emotion/styled)
- **Linting**: ESLint 9 with Next.js and Prettier configs
- **Code Formatting**: Prettier 3.7.4
- **Fonts**: Geist Sans & Geist Mono (Google Fonts)

## Project Structure

```
/src
  /app              # Next.js App Router pages and layouts
  /components       # React components (each in its own folder)
    /myComponent
      MyComponent.tsx
      index.ts
  theme.ts          # MUI theme configuration
```

## Coding Guidelines

### General

- Use **TypeScript** for all files
- Follow **strict TypeScript** settings (strict mode enabled)
- Use **ES2017** as the compilation target
- Use **functional components** only - no class-based components
- All functional components must be **named exports**
- **Default exports** are only allowed for App Router files (page.tsx, layout.tsx, etc.)
- Event handlers and functions inside components must be written as **arrow functions**

### Next.js Specific

- Use **App Router** conventions (app directory)
- Mark client components with `'use client'` directive when needed
- Use server components by default (no directive needed)
- Follow Next.js 16 conventions for:
  - Metadata API for SEO
  - File-based routing
  - Loading states, error boundaries
  - Server and Client Components

### Material-UI Integration

- Use `AppRouterCacheProvider` from `@mui/material-nextjs/v15-appRouter` in root layout
- Wrap app with `ThemeProvider` and include `CssBaseline`
- Define custom theme in `theme.ts` using `createTheme()`
- Use MUI components with TypeScript for type safety
- Apply Geist fonts through MUI theme's typography settings

### Import Paths

- Use **absolute imports** from `src/` (configured in tsconfig.json)
- Example: `import { theme } from 'theme'` instead of `import { theme } from '../theme'`
- No need for `@/` prefix - import directly from the module name

### Styling

- Use **Emotion** for styled components and CSS-in-JS
- Use MUI's `sx` prop for component-level styling
- Define theme customizations in `theme.ts`
- Leverage MUI's theming system for consistency

### Code Quality

- **ESLint**: Run `npm run lint` to check for issues
- **Prettier**: Run `npm run format` to format code
- Follow eslint-config-next rules
- Maintain Prettier compatibility (eslint-config-prettier)
- All code should pass both ESLint and Prettier checks

### Component Development

- Create reusable components in `/src/components`
- **Component folder structure**:
  - Each component lives in its own folder: `someComponent/SomeComponent.tsx`
  - Folder name: **camelCase** (e.g., `myButton/`)
  - Component file: **PascalCase** (e.g., `MyButton.tsx`)
  - Include an `index.ts` file that re-exports the component:
    ```typescript
    export { MyButton } from './MyButton';
    ```
  - This enables clean imports: `import { MyButton } from 'components/MyButton'`
- **Always use functional components** - class components are not allowed
- **Export components using named exports**: `export const MyComponent = () => { ... }`
- Use TypeScript interfaces/types for props
- Write all handlers and internal functions as **arrow functions**:
  ```tsx
  const handleClick = () => { ... };
  const handleSubmit = (e: FormEvent) => { ... };
  ```
- Prefer composition over inheritance
- Keep components focused and single-responsibility
- Use React 19 features when appropriate (hooks, concurrent features)

### File Naming

- Use **PascalCase** for component files: `MyComponent.tsx`
- Use **camelCase** for utility/helper files: `useMyHook.ts`
- Use **kebab-case** for route segments in app directory

### Server Actions (Backend Interactions)

Server Actions are located in `/src/actions` folder and handle all database and backend interactions.

**Structure and conventions:**

- Create separate files for each domain/model (e.g., `users.ts`, `companies.ts`, `posts.ts`, `members.ts`)
- Use **camelCase** for action file names
- Each file exports multiple arrow function actions prefixed with `'use server'` directive
- Action names follow pattern: `create[Entity]`, `update[Entity]`, `delete[Entity]`, `get[Entity]`, `get[Entities]`
- All actions must have TypeScript return types and input validation
- Use Prisma client from `lib/prisma` for database operations
- Handle errors gracefully and return meaningful error messages
- Include proper type definitions for input parameters and responses

**Example Server Action file structure:**

```typescript
'use server';

import { prisma } from 'lib/prisma';
import type { User } from 'generated/prisma/models';

export const createUser = async (data: {
  name: string;
  email: string;
}): Promise<{ success: boolean; user?: User; error?: string }> => {
  try {
    const user = await prisma.user.create({
      data: {
        id: crypto.randomUUID(),
        name: data.name,
        email: data.email,
        emailVerified: false,
      },
    });
    return { success: true, user };
  } catch (error) {
    return { success: false, error: 'Failed to create user' };
  }
};

export const updateUser = async (
  id: string,
  data: Partial<{ name: string; email: string }>
): Promise<{ success: boolean; user?: User; error?: string }> => {
  try {
    const user = await prisma.user.update({
      where: { id },
      data,
    });
    return { success: true, user };
  } catch (error) {
    return { success: false, error: 'Failed to update user' };
  }
};

export const deleteUser = async (id: string): Promise<{ success: boolean; error?: string }> => {
  try {
    await prisma.user.delete({
      where: { id },
    });
    return { success: true };
  } catch (error) {
    return { success: false, error: 'Failed to delete user' };
  }
};

export const getUser = async (id: string): Promise<{ user?: User; error?: string }> => {
  try {
    const user = await prisma.user.findUnique({
      where: { id },
    });
    return { user };
  } catch (error) {
    return { error: 'Failed to fetch user' };
  }
};
```

**Client-side usage in components:**

```typescript
'use client';

import { useTransition } from 'react';
import { createUser } from 'actions/users';

export const UserForm = () => {
  const [isPending, startTransition] = useTransition();

  const handleSubmit = async (formData: FormData) => {
    startTransition(async () => {
      const result = await createUser({
        name: formData.get('name') as string,
        email: formData.get('email') as string,
      });
      if (result.success) {
        // Handle success
      } else {
        // Handle error
      }
    });
  };

  return (
    // JSX
  );
};
```

### Validation with Zod

Use **Zod** for runtime schema validation in Server Actions and client-side validation. Define schemas for data validation.

**Structure and conventions:**

- Create schema files in `/src/lib/schemas` or co-locate with actions
- Use **camelCase** for schema file names: `userSchemas.ts`
- Export schemas that match your models and forms
- Use Zod for both input validation in Server Actions and client-side form validation

**Example Zod schema file:**

```typescript
import { z } from 'zod';

export const createUserSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100, 'Name must be less than 100 characters'),
  email: z.string().email('Invalid email address'),
});

export const updateUserSchema = z
  .object({
    name: z.string().min(1).max(100).optional(),
    email: z.string().email().optional(),
  })
  .refine((data) => Object.keys(data).length > 0, 'At least one field must be provided');

export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
```

**Using Zod in Server Actions:**

```typescript
'use server';

import { prisma } from 'lib/prisma';
import { createUserSchema } from 'lib/schemas/userSchemas';

export const createUser = async (data: unknown): Promise<{ success: boolean; user?: any; error?: string }> => {
  try {
    const validatedData = createUserSchema.parse(data);
    const user = await prisma.user.create({
      data: {
        id: crypto.randomUUID(),
        ...validatedData,
        emailVerified: false,
      },
    });
    return { success: true, user };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: error.errors[0].message };
    }
    return { success: false, error: 'Failed to create user' };
  }
};
```

### Form Handling with React Hook Form

Use **React Hook Form** for efficient form state management and validation in client components.

**Structure and conventions:**

- Use `useForm` hook from react-hook-form for form state management
- Integrate with Zod using `@hookform/resolvers/zod`
- Display field-level errors from validation
- Use MUI components with `Controller` for form inputs when needed

**Example form component with React Hook Form:**

```typescript
'use client';

import { useTransition } from 'react';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { TextField, Button, Box, Alert } from '@mui/material';
import { createUserSchema, type CreateUserInput } from 'lib/schemas/userSchemas';
import { createUser } from 'actions/users';

export const UserForm = () => {
  const [isPending, startTransition] = useTransition();
  const [serverError, setServerError] = useTransition();

  const {
    control,
    handleSubmit,
    formState: { errors },
    reset,
  } = useForm<CreateUserInput>({
    resolver: zodResolver(createUserSchema),
    defaultValues: {
      name: '',
      email: '',
    },
  });

  const onSubmit = (data: CreateUserInput) => {
    setServerError(async () => {
      const result = await createUser(data);
      if (result.success) {
        reset();
        // Show success message
      } else {
        setServerError(result.error || 'Failed to create user');
      }
    });
  };

  return (
    <Box component="form" onSubmit={handleSubmit(onSubmit)} noValidate>
      {serverError && <Alert severity="error">{serverError}</Alert>}

      <Controller
        name="name"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Name"
            error={!!errors.name}
            helperText={errors.name?.message}
            fullWidth
            margin="normal"
            disabled={isPending}
          />
        )}
      />

      <Controller
        name="email"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Email"
            type="email"
            error={!!errors.email}
            helperText={errors.email?.message}
            fullWidth
            margin="normal"
            disabled={isPending}
          />
        )}
      />

      <Button
        type="submit"
        variant="contained"
        fullWidth
        sx={{ mt: 2 }}
        disabled={isPending}
      >
        {isPending ? 'Creating...' : 'Create User'}
      </Button>
    </Box>
  );
};
```

**Error handling best practices:**

- Always display validation errors returned from Server Actions
- Show field-specific errors from form validation
- Use MUI's `helperText` prop to display error messages under fields
- Handle network errors gracefully with try-catch in useTransition callbacks
- Provide clear, user-friendly error messages

### UI Loading States

**Always implement loading states for all user interactions:**

- Use `useTransition` hook to track async operations
- Disable buttons and form inputs during pending operations (`disabled={isPending}`)
- Show loading indicators (spinners, progress bars, skeleton screens) when appropriate
- Update button text to reflect loading state (e.g., "Creating..." instead of "Create")
- Disable form submission while a request is in progress to prevent duplicate submissions
- Use MUI's `CircularProgress` or `Skeleton` components for visual feedback
- Show loading states for data fetching operations (initial load, pagination, filtering)

**Example with loading button state:**

```typescript
<Button
  type="submit"
  variant="contained"
  fullWidth
  sx={{ mt: 2 }}
  disabled={isPending}
>
  {isPending ? <CircularProgress size={24} sx={{ mr: 1 }} /> : null}
  {isPending ? 'Loading...' : 'Submit'}
</Button>
```

**Example with input field loading state:**

```typescript
<TextField
  {...field}
  label="Name"
  error={!!errors.name}
  helperText={errors.name?.message}
  fullWidth
  margin="normal"
  disabled={isPending}
/>
```

**Example with skeleton loading for data:**

```typescript
import { Skeleton, Box } from '@mui/material';

if (isLoading) {
  return (
    <Box>
      <Skeleton variant="text" sx={{ mb: 2 }} />
      <Skeleton variant="rectangular" height={40} />
    </Box>
  );
}
```

## Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm start` - Start production server
- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check formatting without changing files

## Best Practices

1. Always use TypeScript types/interfaces - avoid `any`
2. Leverage Next.js 16 and React 19 latest features
3. Use MUI components for consistency
4. Keep server components as the default, use client components only when needed
5. Follow the established project structure
6. Write clean, maintainable, and well-documented code
7. Test code changes with `npm run lint` and `npm run format:check`
