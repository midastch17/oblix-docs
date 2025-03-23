# Session Management

Oblix provides powerful session management capabilities to handle conversational interactions with AI models. This page explains how sessions work and how to use them effectively.

## What are Sessions?

Sessions in Oblix are persistent containers for conversation history that:

1. **Store** messages between users and AI models
2. **Maintain** context across multiple interactions
3. **Persist** across application restarts
4. **Enable** stateful conversations
5. **Support** metadata for organization and filtering
6. **Allow** importing/exporting for portability

Sessions are particularly useful for building chat applications, virtual assistants, or any interactive AI experience where context matters.

## Session Lifecycle

A typical session lifecycle includes:

1. **Creation** - A new session is created with a unique ID and optional metadata
2. **Message Exchange** - User and AI messages are added to the session
3. **Context Maintenance** - Context is preserved between interactions
4. **Metadata Management** - Sessions can be tagged with metadata for organization
5. **Persistence** - Session data is saved to disk
6. **Retrieval** - Sessions can be loaded by ID for continued interaction
7. **Merging/Copying** - Sessions can be copied or merged for advanced workflows
8. **Export/Import** - Sessions can be exported for sharing or backup
9. **Deletion** - Sessions can be deleted when no longer needed

## Working with Sessions

### Creating a Session

```python
from oblix import OblixClient

client = OblixClient(oblix_api_key="your_api_key")

# Create a new session with metadata
session_id = await client.create_session(
    title="Customer Support Chat",
    initial_context={"customer_id": "cust_123", "subscription_tier": "premium"},
    metadata={
        "category": "support",
        "priority": "high",
        "agent_id": "agent_456"
    }
)

print(f"Created session: {session_id}")

# Create and immediately use a session
session_id = await client.create_and_use_session(
    title="Product Demo",
    metadata={"demo_type": "product_tour", "customer": "Acme Inc"}
)
```

### Executing Prompts in a Session

```python
# Set the current session
client.current_session_id = session_id
# Or use the convenience method
client.use_session(session_id)

# Execute prompts in the session context
response = await client.execute("Hello, I'm having trouble with my account")

# The context is automatically maintained
follow_up_response = await client.execute("Can you help me reset my password?")
```

### Using Chat Methods

For chat applications, Oblix provides simplified methods:

```python
# Send a single message in a session
response = await client.chat_once("Hello, how can I help you today?")
session_id = response["session_id"]

# Continue the conversation in the same session
response = await client.chat_once("I need help with my subscription", session_id)
```

### Interactive Chat Sessions

For command-line or interactive applications:

```python
# Start an interactive chat session
await client.start_chat()

# Or resume an existing session
await client.start_chat(session_id="existing_session_id")
```

### Managing Sessions

Oblix provides methods to list, load, and delete sessions:

```python
# List recent sessions
sessions = client.list_sessions(limit=10)
for session in sessions:
    print(f"ID: {session['id']} | Title: {session['title']} | Messages: {session['message_count']}")

# Filter sessions by metadata
support_sessions = client.list_sessions(filter_metadata={"category": "support"})
high_priority = client.list_sessions(filter_metadata={"priority": "high"})

# Load a specific session
session_data = client.load_session("session_id_here")

# Delete a session
success = client.delete_session("session_id_here")
```

## Session Storage

By default, Oblix stores sessions as JSON files in a local directory:

- **Default location**: `~/.oblix/sessions/`
- **Format**: Each session is stored as a separate JSON file
- **Naming**: Files are named using the session ID

You can customize the storage location:

```python
from oblix.sessions import SessionManager

# Custom session directory
session_manager = SessionManager(base_dir="/path/to/custom/directory")

# Use with client
client = OblixClient(oblix_api_key="your_api_key")
client.session_manager = session_manager
```

## Session Structure

Each session contains:

- **ID**: Unique identifier for the session
- **Title**: Human-readable title for the session
- **Created/Updated**: Timestamps for creation and last update
- **Messages**: Array of message objects with:
  - **Role**: 'user' or 'assistant'
  - **Content**: Message text
  - **Timestamp**: When the message was added
- **Context**: Optional dictionary for additional context
- **Metadata**: Optional dictionary for categorization and filtering

Example session structure:

```json
{
  "id": "5f3e9a7b-6c1d-4d8e-9e7a-9b8c7d6e5f4a",
  "title": "Technical Support Chat",
  "created_at": "2023-07-15T14:30:22.123Z",
  "updated_at": "2023-07-15T14:35:46.789Z",
  "messages": [
    {
      "id": "msg_1",
      "role": "user",
      "content": "I'm having trouble installing the software.",
      "timestamp": "2023-07-15T14:30:22.123Z"
    },
    {
      "id": "msg_2",
      "role": "assistant",
      "content": "I'm sorry to hear that. What operating system are you using?",
      "timestamp": "2023-07-15T14:30:25.456Z"
    },
    {
      "id": "msg_3",
      "role": "user",
      "content": "I'm using Windows 11.",
      "timestamp": "2023-07-15T14:35:46.789Z"
    }
  ],
  "context": {
    "user_id": "user_789",
    "product": "Oblix SDK"
  },
  "metadata": {
    "category": "technical_support",
    "priority": "medium",
    "status": "active"
  }
}
```

## Enhanced Session Management

Oblix provides advanced session management features for building sophisticated applications.

### Metadata Management

Sessions can be tagged with metadata for organization, filtering, and tracking:

```python
# Add metadata when creating a session
session_id = await client.create_session(
    title="Sales Call",
    metadata={
        "customer": "Acme Inc",
        "industry": "manufacturing",
        "deal_size": "enterprise",
        "sales_rep": "jane_doe"
    }
)

# Update metadata for an existing session
client.update_session_metadata(session_id, {
    "status": "follow_up_required",
    "next_contact_date": "2023-08-15"
})

# Get just the metadata without loading the entire session
metadata = client.get_session_metadata(session_id)
if metadata["status"] == "follow_up_required":
    print(f"Follow-up needed by: {metadata['next_contact_date']}")

# Filter sessions by metadata
active_sales = client.list_sessions(filter_metadata={
    "industry": "manufacturing",
    "status": "follow_up_required"
})
```

### Session Import/Export

For sharing conversations or creating backups:

```python
# Export a session to a file
await client.export_session(session_id, "/path/to/export/session.json")

# Import a session from a file
imported_id = await client.import_session(
    "/path/to/import/session.json", 
    new_id=True,               # Assign a new ID to avoid conflicts
    use_immediately=True       # Set as the current session
)
```

### Session Copying

Create duplicates of sessions for templates or variations:

```python
# Create a copy of a session
copy_id = await client.copy_session(
    session_id,
    new_title="Copy - Customer Onboarding Template",
    use_immediately=True
)

# Update metadata on the copy
client.update_session_metadata(copy_id, {
    "template_version": "2.0",
    "last_updated": "2023-07-20"
})
```

### Session Merging

Combine multiple sessions to create a unified conversation history:

```python
# Merge multiple sessions
merged_id = await client.merge_sessions(
    [session_id_1, session_id_2, session_id_3],
    title="Combined Support Case",
    use_immediately=True
)

# The merged session contains all messages from the source sessions,
# properly ordered by timestamp
```

## Context Management

Sessions maintain conversation history, which is used as context for future interactions. By default, Oblix includes recent messages when generating responses, allowing models to understand the conversation flow.

You can also provide additional context when creating a session:

```python
# Create session with initial context
session_id = await client.create_session(
    initial_context={
        "user_name": "Alex",
        "preferences": {
            "language": "English",
            "format": "concise"
        }
    }
)
```

## Model Consistency in Sessions

When using sessions with multiple available models, Oblix attempts to maintain consistency by:

1. Using the same model for all messages in a session when possible
2. Falling back to alternative models only when necessary
3. Including sufficient context to maintain conversation flow even when switching models

## Advanced Usage

### Building Multi-User Applications

For applications with multiple users, use metadata to organize sessions:

```python
# Create a session for a specific user
user_session_id = await client.create_session(
    title=f"Chat with {user_name}",
    metadata={
        "user_id": user_id,
        "app_id": app_id,
        "tenant_id": tenant_id
    }
)

# Find all sessions for a specific user
user_sessions = client.list_sessions(filter_metadata={"user_id": user_id})
```

### Creating Session Templates

Use session copying to create reusable templates:

```python
# Create a template session with initial messages and context
template_id = await client.create_session(
    title="Customer Onboarding Template",
    metadata={"type": "template", "version": "1.0"}
)

# Add predefined messages
client.session_manager.save_message(
    template_id,
    "Welcome to Acme Inc! I'm your virtual assistant and I'll help you get started with our platform.",
    role="assistant"
)

# When a new customer signs up, create a customized copy
customer_session_id = await client.copy_session(
    template_id,
    new_title=f"Onboarding - {customer_name}",
    use_immediately=True
)

# Update metadata for the customer's session
client.update_session_metadata(customer_session_id, {
    "type": "customer_session",
    "customer_id": customer_id,
    "signup_date": signup_date
})
```

### Session Analytics

Use session metadata and structure for analytics:

```python
# Get all support sessions
support_sessions = client.list_sessions(filter_metadata={"category": "support"})

# Calculate average message count
total_messages = sum(session["message_count"] for session in support_sessions)
avg_messages = total_messages / len(support_sessions) if support_sessions else 0

# Analyze resolution status
resolved = len([s for s in support_sessions if s["metadata"].get("status") == "resolved"])
resolution_rate = resolved / len(support_sessions) if support_sessions else 0

print(f"Support sessions: {len(support_sessions)}")
print(f"Average messages per session: {avg_messages:.1f}")
print(f"Resolution rate: {resolution_rate:.1%}")
```

## Best Practices

When working with sessions:

- **Use metadata** for organization and searchability
- **Create dedicated sessions** for different conversation types
- **Include relevant initial context** to guide the conversation
- **Export important sessions** for backup or sharing
- **Clean up old sessions** to manage storage
- **Monitor session length** for very long conversations
- **Consider model consistency** when choosing models for sessions
- **Use session templates** for common conversation patterns

By effectively using Oblix's enhanced session management capabilities, you can build sophisticated conversational AI applications that maintain context, organize interactions, and provide a seamless user experience.

