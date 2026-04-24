# API Files

## Structure

Use a single exported object per domain with methods for each endpoint. Type all parameters and return values:

```tsx
import { axiosInstance } from '@/shared/api/axiosInstance';
import { LocateListItem, LocateDetail, SyncResult, SyncStatus } from '../types';

export const locatesApi = {
  list: async (jobId: number): Promise<LocateListItem[]> => {
    const response = await axiosInstance.get('/locates/list/', {
      params: { job_id: jobId },
    });
    return response.data;
  },

  getDetails: async (locateId: number): Promise<LocateDetail> => {
    const response = await axiosInstance.get(`/locates/details/${locateId}/`);
    return response.data;
  },

  create: async (formData: FormData): Promise<{ success: boolean; locateId?: number }> => {
    const response = await axiosInstance.post('/locates/submit/', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    });
    return response.data;
  },

  terminate: async (locateId: number): Promise<{ success: boolean; message: string }> => {
    const response = await axiosInstance.post('/locates/terminate/', { id: locateId });
    return response.data;
  },
};
```

## Naming Conventions

Use consistent method names that describe the operation:

| Operation               | Name                          | Example                              |
|-------------------------|-------------------------------|--------------------------------------|
| Fetch a list            | `list`                        | `locatesApi.list(jobId)`             |
| Fetch a single record   | `getDetails` or `getById`     | `locatesApi.getDetails(locateId)`    |
| Create a record         | `create` or `submit`          | `locatesApi.create(formData)`        |
| Update a record         | `update`                      | `locatesApi.update(id, data)`        |
| Delete / terminate      | `delete` or domain-specific   | `locatesApi.terminate(locateId)`     |
| Domain-specific action  | Descriptive verb              | `locatesApi.sync(jobId)`             |

## Rules

!!! warning "No business logic in API files"
    API files only handle request/response. Validation, formatting, and error interpretation belong in hooks or utils.

- **Type everything.** Every parameter and return type should be explicit. Avoid `any`.
- **Keep it thin.** Each method should be a single API call. If you need to chain calls, do that in a hook or service function.
- **Use the shared axios instance.** Don't create new axios instances per file — use the one from `shared/api/`.
- **Comments are optional.** If the method name and types are clear, a comment just restates the obvious. Add comments only for non-obvious behavior (e.g., why `Content-Type` is set manually for file uploads).
