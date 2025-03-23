# Advanced Session Management Examples

This guide demonstrates the enhanced session management capabilities in Oblix, including metadata, import/export, merging, and copying.

## Prerequisites

Before running these examples, make sure you have:

1. Installed the Oblix SDK: `pip install oblix`
2. Set up your API keys:
   ```python
   import os
   os.environ["OBLIX_API_KEY"] = "your_oblix_api_key"
   os.environ["OPENAI_API_KEY"] = "your_openai_api_key"  # If using OpenAI
   ```

## Setting Up the Client

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor

async def session_management_demo():
    # Initialize client
    client = OblixClient()
    
    # Set up required components
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2",
        endpoint="http://localhost:11434"
    )
    
    # Add a resource monitor (required for orchestration)
    client.hook_agent(ResourceMonitor())
    
    # Rest of the demo will go here...
    
    # Clean up at the end
    await client.shutdown()

# Run the demo
asyncio.run(session_management_demo())
```

## Session Metadata

Metadata enables organizing and filtering sessions:

```python
# Create sessions with different metadata
support_session_id = await client.create_session(
    title="Technical Support - Network Issue",
    metadata={
        "category": "support",
        "priority": "high",
        "customer_id": "cust_12345",
        "product": "Enterprise Server"
    }
)

sales_session_id = await client.create_session(
    title="Sales Call - Acme Inc",
    metadata={
        "category": "sales",
        "deal_size": "$50,000",
        "contact": "jane.doe@acme.com",
        "stage": "proposal"
    }
)

# Find all high priority support sessions
high_priority = client.list_sessions(
    filter_metadata={"category": "support", "priority": "high"}
)
print(f"Found {len(high_priority)} high priority support sessions")

# Update metadata as the session progresses
client.update_session_metadata(
    sales_session_id, 
    {"stage": "negotiation", "next_follow_up": "2023-08-15"}
)

# Get just the metadata without loading the entire session
metadata = client.get_session_metadata(sales_session_id)
print(f"Current sales stage: {metadata['stage']}")
```

## Create and Use Sessions

The convenience methods make session management more intuitive:

```python
# Create and immediately use a new session
session_id = await client.create_and_use_session(
    title="Product Demo",
    metadata={"customer": "Acme Inc", "demo_type": "full_product"}
)

# Now we can immediately start using it without setting current_session_id
await client.execute("Show me how the dashboard analytics work")

# Switch to another session
other_session_exists = client.use_session(other_session_id)
if other_session_exists:
    await client.execute("Let's continue our discussion about pricing")
```

## Session Import/Export

For sharing or backing up sessions:

```python
# Export a session to a file
export_path = "/tmp/important_conversation.json"
await client.export_session(session_id, export_path)
print(f"Session exported to {export_path}")

# Import a session with a new ID
imported_id = await client.import_session(
    export_path,
    new_id=True,  # Generate a new ID to avoid conflicts
    use_immediately=True  # Set as current session
)
print(f"Imported session as {imported_id}")

# Continue the conversation in the imported session
await client.execute("Can we continue from where we left off?")
```

## Session Copying

Create duplicates of sessions for templates or variations:

```python
# Create a template session
template_id = await client.create_session(
    title="Customer Onboarding Template",
    metadata={"type": "template", "version": "1.0"}
)

# Add a welcome message to the template
client.session_manager.save_message(
    template_id,
    "Welcome to our platform! I'll help you get started.",
    role="assistant"
)

# Create a customized copy for a specific customer
customer_id = await client.copy_session(
    template_id,
    new_title="Onboarding - Acme Inc",
    use_immediately=True
)

# Update the metadata for the customer session
client.update_session_metadata(
    customer_id,
    {"customer": "Acme Inc", "type": "onboarding", "start_date": "2023-07-15"}
)

# Continue with the customer-specific onboarding
await client.execute("Hi, I'm the IT manager at Acme Inc. We need to set up 50 user accounts.")
```

## Session Merging

Combine multiple sessions into a unified conversation:

```python
# Create some sessions to merge
session1 = await client.create_session(title="Initial Consultation")
client.current_session_id = session1
await client.execute("I'm having performance issues with my database")

session2 = await client.create_session(title="Technical Follow-up")
client.current_session_id = session2
await client.execute("I checked the query execution plans as suggested")

session3 = await client.create_session(title="Resolution Discussion")
client.current_session_id = session3
await client.execute("After implementing the index changes, performance improved by 80%")

# Merge all three sessions into a unified case history
merged_id = await client.merge_sessions(
    [session1, session2, session3],
    title="Database Performance - Complete Case History",
    use_immediately=True
)

# All messages from all three sessions are now available in chronological order
session_data = client.load_session(merged_id)
print(f"Merged session contains {len(session_data['messages'])} messages")

# Add a summary message to the merged session
await client.execute("Here's a summary of the entire case and resolution steps taken:")
```

## Advanced Use Cases

### Multi-User Application

```python
# Create user-specific sessions with metadata
async def get_or_create_user_session(user_id, user_name):
    # Try to find existing session for this user
    user_sessions = client.list_sessions(
        filter_metadata={"user_id": user_id, "session_type": "active"}
    )
    
    if user_sessions:
        # Use most recent session
        session_id = user_sessions[0]["id"]
        client.use_session(session_id)
        return session_id
    else:
        # Create new session for this user
        return await client.create_and_use_session(
            title=f"Chat with {user_name}",
            metadata={
                "user_id": user_id,
                "user_name": user_name,
                "session_type": "active",
                "created_date": datetime.now().isoformat()
            }
        )

# Example usage
session_id = await get_or_create_user_session("u-12345", "John Doe")
await client.execute("Can you help me with my account settings?")
```

### Session Analytics

```python
# Get all sessions from the last month
all_sessions = client.list_sessions(limit=1000)

# Analyze by category
categories = {}
for session in all_sessions:
    category = session.get("metadata", {}).get("category", "uncategorized")
    if category not in categories:
        categories[category] = []
    categories[category].append(session)

# Print statistics
for category, sessions in categories.items():
    avg_messages = sum(s["message_count"] for s in sessions) / len(sessions)
    print(f"Category: {category}")
    print(f"  Sessions: {len(sessions)}")
    print(f"  Avg messages: {avg_messages:.1f}")
```

### Conversation Templates

```python
# Create a standardized interview template
interview_template_id = await client.create_session(
    title="Job Interview Template - Software Engineer",
    metadata={"type": "template", "role": "software_engineer"}
)

# Add standard questions to the template
standard_questions = [
    "Tell me about your experience with Python.",
    "How do you approach testing your code?",
    "Describe a challenging project you worked on."
]

# Add questions to the template
client.current_session_id = interview_template_id
for question in standard_questions:
    client.session_manager.save_message(
        interview_template_id,
        question,
        role="assistant"
    )

# For each candidate, create a copy of the template
async def start_interview(candidate_name, candidate_id):
    # Create a customized copy
    session_id = await client.copy_session(
        interview_template_id,
        new_title=f"Interview - {candidate_name}",
        use_immediately=True
    )
    
    # Update metadata
    client.update_session_metadata(
        session_id,
        {
            "candidate_name": candidate_name,
            "candidate_id": candidate_id,
            "interview_date": datetime.now().isoformat(),
            "status": "in_progress"
        }
    )
    
    return session_id

# Example usage
interview_id = await start_interview("Jane Smith", "cand-789")
```

## Complete Example

Here's a complete working example that demonstrates key session management features:

```python
import asyncio
import os
from datetime import datetime
from rich.console import Console
from rich.table import Table

from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor

# Initialize rich console for pretty output
console = Console()

async def session_management_demo():
    # Initialize client with API key
    client = OblixClient(oblix_api_key=os.getenv('OBLIX_API_KEY'))
    
    try:
        # Set up required components
        console.print("\n[bold cyan]Setting up Oblix client...[/bold cyan]")
        
        # Hook required models
        await client.hook_model(
            model_type=ModelType.OLLAMA,
            model_name="llama2",
            endpoint="http://localhost:11434"
        )
        
        # Add resource monitor
        client.hook_agent(ResourceMonitor())
        
        # Create a session with metadata
        support_session_id = await client.create_and_use_session(
            title="Customer Support - Product Issue",
            metadata={
                "category": "support",
                "customer_id": "cust_12345",
                "priority": "high"
            }
        )
        console.print(f"Created support session: {support_session_id}")
        
        # Add some conversation
        await client.execute("I'm having trouble with my recent order.")
        
        # Add an assistant message directly
        client.session_manager.save_message(
            client.current_session_id,
            "I'm sorry to hear that. Can you provide your order number?",
            role="assistant"
        )
        
        await client.execute("My order number is #OR-789456.")
        
        # Create a second session
        sales_session_id = await client.create_session(
            title="Sales Call - Acme Inc",
            metadata={
                "category": "sales",
                "stage": "proposal"
            }
        )
        
        # Export the support session
        export_path = os.path.expanduser("~/session_export.json")
        await client.export_session(support_session_id, export_path)
        console.print(f"Exported session to: {export_path}")
        
        # Import with a new ID
        imported_id = await client.import_session(
            export_path, 
            new_id=True,
            use_immediately=True
        )
        console.print(f"Imported session with new ID: {imported_id}")
        
        # Create a copy of the sales session
        copy_id = await client.copy_session(
            sales_session_id,
            new_title="Sales Call - COPY FOR REFERENCE"
        )
        console.print(f"Created copy with ID: {copy_id}")
        
        # Merge the support and sales sessions
        merged_id = await client.merge_sessions(
            [support_session_id, sales_session_id],
            title="Combined Customer Interaction"
        )
        console.print(f"Created merged session with ID: {merged_id}")
        
        # List all sessions
        sessions = client.list_sessions()
        console.print(f"Total sessions: {len(sessions)}")
        
        # Filter for just support sessions
        support_sessions = client.list_sessions(filter_metadata={"category": "support"})
        console.print(f"Found {len(support_sessions)} support sessions")
        
    finally:
        # Clean up resources
        await client.shutdown()
        
        # Clean up export file
        if os.path.exists(export_path):
            os.remove(export_path)

if __name__ == "__main__":
    asyncio.run(session_management_demo())
```

## Best Practices

- **Use metadata consistently**: Define a standard set of metadata fields for your application
- **Create logical categories**: Group sessions by user, type, status, etc.
- **Clean up old sessions**: Implement a policy to archive or delete old sessions
- **Back up important sessions**: Export critical conversations regularly
- **Use templates for consistency**: Create standard templates for common conversation flows
- **Add timestamps to metadata**: Include creation dates and update timestamps in metadata
- **Keep context window in mind**: For very long merged sessions, be aware of model context limits