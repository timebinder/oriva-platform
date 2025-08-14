# @oriva/plugin-sdk

TypeScript SDK for developing Oriva plugins with comprehensive APIs for entries, templates, user management, UI interactions, and plugin storage.

## Installation

```bash
npm install @oriva/plugin-sdk
```

## Quick Start

```typescript
import { OrivaPluginSDK, PluginContext } from '@oriva/plugin-sdk';

// Initialize the SDK with your plugin context
const context: PluginContext = {
  pluginId: 'your-plugin-id',
  version: '1.0.0',
  userId: 'user-id',
  permissions: ['entries:read', 'entries:write'],
  apiKey: 'your-api-key',
  baseUrl: 'https://api.oriva.com',
};

const sdk = new OrivaPluginSDK(context);

// Use the SDK APIs
async function createEntry() {
  try {
    const entry = await sdk.entries.createEntry({
      title: 'My First Entry',
      content: 'Hello, Oriva!',
      audience: 'public',
    });
    console.log('Entry created:', entry);
  } catch (error) {
    console.error('Failed to create entry:', error);
  }
}
```

## React Hooks

The SDK includes React hooks for easy integration:

```typescript
import { useEntries, useUser, useStorage } from '@oriva/plugin-sdk/hooks';

function MyPluginComponent({ sdk }) {
  const { entries, loading, createEntry } = useEntries(sdk);
  const { user } = useUser(sdk);
  const { value: settings, setValue: setSettings } = useStorage(sdk, 'settings', {});

  const handleCreateEntry = async () => {
    await createEntry({
      title: 'New Entry',
      content: 'Content here...',
    });
  };

  return (
    <div>
      <h1>Welcome, {user?.displayName}</h1>
      <button onClick={handleCreateEntry}>Create Entry</button>
      {loading ? <p>Loading...</p> : <EntryList entries={entries} />}
    </div>
  );
}
```

## API Reference

### Core SDK

#### `OrivaPluginSDK`

Main SDK class providing access to all APIs.

```typescript
const sdk = new OrivaPluginSDK(context, config?);

// Available APIs
sdk.entries   // Entry management
sdk.templates // Template management
sdk.user      // User information
sdk.ui        // UI interactions
sdk.storage   // Plugin storage
```

### Entry API

Manage Oriva entries with full CRUD operations.

```typescript
// Get entries with filtering
const entries = await sdk.entries.getEntries({
  status: 'published',
  limit: 10,
  search: 'keyword',
});

// Create a new entry
const entry = await sdk.entries.createEntry({
  title: 'My Entry',
  content: 'Entry content',
  sections: [
    { type: 'heading', content: 'Introduction', order: 0 },
    { type: 'body', content: 'Main content...', order: 1 },
  ],
  audience: 'public',
});

// Update an entry
const updated = await sdk.entries.updateEntry({
  id: 'entry-id',
  title: 'Updated Title',
});

// Delete an entry
await sdk.entries.deleteEntry('entry-id');
```

### Template API

Work with Oriva templates for structured content creation.

```typescript
// Get templates
const templates = await sdk.templates.getTemplates({
  category: 'blog',
  sortBy: 'rating',
});

// Create a template
const template = await sdk.templates.createTemplate({
  name: 'Blog Post',
  description: 'Standard blog post template',
  category: 'blog',
  sections: [
    { type: 'heading', placeholder: 'Enter title...', order: 0 },
    { type: 'body', placeholder: 'Write your post...', order: 1 },
  ],
});

// Rate a template
await sdk.templates.rateTemplate('template-id', 5);
```

### User API

Access user information and manage preferences.

```typescript
// Get current user
const user = await sdk.user.getCurrentUser();

// Update user profile
const updated = await sdk.user.updateProfile({
  displayName: 'New Name',
  bio: 'Updated bio',
});

// Manage plugin-specific preferences
await sdk.user.setPreference('theme', 'dark');
const theme = await sdk.user.getPreference('theme');
```

### UI API

Interact with the Oriva user interface.

```typescript
// Show notifications
await sdk.ui.showNotification({
  title: 'Success',
  message: 'Entry saved successfully',
  type: 'success',
});

// Show modals
const result = await sdk.ui.showModal({
  title: 'Confirm Action',
  content: 'Are you sure you want to delete this entry?',
  actions: [
    { label: 'Cancel', variant: 'secondary' },
    { label: 'Delete', variant: 'danger' },
  ],
});

// Navigate to screens
await sdk.ui.navigate({
  screen: 'CreateEntry',
  params: { templateId: 'template-id' },
});

// Show toast messages
await sdk.ui.showToast('Changes saved!', 'success');
```

### Storage API

Persist plugin data with a key-value store.

```typescript
// Store data
await sdk.storage.set('user-preferences', {
  theme: 'dark',
  notifications: true,
});

// Retrieve data
const preferences = await sdk.storage.get('user-preferences');

// Check if key exists
const exists = await sdk.storage.has('user-preferences');

// Delete data
await sdk.storage.delete('user-preferences');

// Atomic operations
await sdk.storage.increment('view-count', 1);
await sdk.storage.push('recent-items', newItem);

// TTL support
await sdk.storage.setWithTTL('session-data', data, 3600); // 1 hour
```

## Permissions

Your plugin must declare required permissions in its manifest:

```json
{
  "permissions": [
    "entries:read",
    "entries:write",
    "templates:read",
    "user:read",
    "ui:notifications",
    "storage:read",
    "storage:write"
  ]
}
```

Available permissions:
- `entries:read` - Read entries
- `entries:write` - Create and update entries
- `entries:delete` - Delete entries
- `templates:read` - Read templates
- `templates:write` - Create and update templates
- `user:read` - Read user information
- `user:write` - Update user profiles
- `ui:notifications` - Show notifications
- `ui:modals` - Display modals
- `ui:navigation` - Navigate between screens
- `storage:read` - Read plugin storage
- `storage:write` - Write to plugin storage

## Error Handling

The SDK provides comprehensive error handling:

```typescript
import { ApiError } from '@oriva/plugin-sdk';

try {
  await sdk.entries.createEntry(entryData);
} catch (error) {
  if (error instanceof ApiError) {
    console.error('API Error:', error.message);
    console.error('Error Code:', error.code);
    console.error('Status:', error.status);
  }
}
```

## Rate Limiting

API calls are rate-limited to ensure fair usage:
- General API: 1000 requests/hour per plugin
- Write operations: 200 requests/hour per plugin
- Storage operations: 500 requests/hour per plugin
- UI operations: 100 requests/hour per plugin

## TypeScript Support

The SDK is built with TypeScript and provides full type safety:

```typescript
import type { Entry, Template, User, PluginManifest } from '@oriva/plugin-sdk';

// All API responses are properly typed
const entry: Entry = await sdk.entries.getEntry('entry-id');
const templates: Template[] = await sdk.templates.getTemplates();
```

## Examples

### Blog Plugin

```typescript
import { OrivaPluginSDK } from '@oriva/plugin-sdk';

class BlogPlugin {
  constructor(private sdk: OrivaPluginSDK) {}

  async createBlogPost(title: string, content: string, tags: string[]) {
    // Create entry with blog template
    const entry = await this.sdk.entries.createEntry({
      title,
      content,
      templateId: 'blog-template-id',
      audience: 'public',
    });

    // Store tags in plugin storage
    await this.sdk.storage.set(`entry-tags-${entry.id}`, tags);

    // Show success notification
    await this.sdk.ui.showToast('Blog post created!', 'success');

    return entry;
  }

  async getMyPosts() {
    return this.sdk.entries.getEntries({
      templateId: 'blog-template-id',
      status: 'published',
    });
  }
}
```

### Analytics Plugin

```typescript
import { OrivaPluginSDK } from '@oriva/plugin-sdk';

class AnalyticsPlugin {
  constructor(private sdk: OrivaPluginSDK) {}

  async trackEvent(event: string, data: any) {
    // Store event in plugin storage
    const events = await this.sdk.storage.get('events') || [];
    events.push({
      event,
      data,
      timestamp: new Date().toISOString(),
    });

    await this.sdk.storage.set('events', events);
  }

  async getAnalytics() {
    const events = await this.sdk.storage.get('events') || [];
    const stats = await this.sdk.entries.getEntryStats();

    return {
      totalEvents: events.length,
      entryStats: stats,
      recentEvents: events.slice(-10),
    };
  }

  async showAnalyticsDashboard() {
    const analytics = await this.getAnalytics();

    await this.sdk.ui.showModal({
      title: 'Analytics Dashboard',
      content: analytics,
      size: 'large',
    });
  }
}
```

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

- Documentation: https://developers.oriva.com
- Email: developers@oriva.com
- Discord: https://discord.gg/oriva