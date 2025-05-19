# Supabase Realtime Configuration for TeamSync

You're asking an excellent question. I didn't explicitly cover the Supabase Realtime configuration which is critical for implementing all the streaming and real-time functionality in the PubNub-inspired API.

## Realtime Configuration Overview

Supabase Realtime is built on Phoenix Channels and allows the application to listen to changes in the PostgreSQL database via websockets. By default, Supabase provides realtime capabilities, but they need to be explicitly configured for the tables you want to track.

Here's what we need to set up:

1. Create a publication for tables we want to track
2. Configure realtime options for specific tables and operations
3. Set up appropriate security for realtime events

## Database Migration Script for Realtime

Add the following to your migration script:

```sql
-- Enable realtime for TeamSync tables

-- Create a publication for all changes
CREATE PUBLICATION teamsync_realtime FOR ALL TABLES;

-- Alternatively, you can specify exactly which tables to include
-- (Recommended for production to limit unnecessary events)
DROP PUBLICATION IF EXISTS teamsync_realtime;
CREATE PUBLICATION teamsync_realtime FOR TABLE
    users,
    channels,
    messages,
    memberships,
    thread_channels,
    thread_messages,
    events,
    presence;

-- Enable specific operations per table
ALTER PUBLICATION teamsync_realtime ADD TABLE users WITH (publish = 'insert, update, delete');
ALTER PUBLICATION teamsync_realtime ADD TABLE channels WITH (publish = 'insert, update, delete');
ALTER PUBLICATION teamsync_realtime ADD TABLE messages WITH (publish = 'insert, update, delete');
ALTER PUBLICATION teamsync_realtime ADD TABLE memberships WITH (publish = 'insert, update');
ALTER PUBLICATION teamsync_realtime ADD TABLE thread_channels WITH (publish = 'insert, update, delete');
ALTER PUBLICATION teamsync_realtime ADD TABLE thread_messages WITH (publish = 'insert, update, delete');
ALTER PUBLICATION teamsync_realtime ADD TABLE events WITH (publish = 'insert');
ALTER PUBLICATION teamsync_realtime ADD TABLE presence WITH (publish = 'insert, update, delete');
```

## Supabase Project Configuration

Additionally, you'll need to configure the Supabase project through the Supabase dashboard:

1. Go to the Supabase dashboard for your project
2. Navigate to "Database" → "Replication" → "Realtime"
3. Ensure the publication is created and the tables you want to track are included
4. Configure the broadcast modes for each table:
   - For most tables, use "Send all changes"
   - For sensitive tables, consider "Send changes when authenticated"

You can also configure these settings programmatically through Supabase's management API if you're using CI/CD.

## Client-Side Implementation

In your Flutter application, you'll need to subscribe to realtime events. Here's how to implement this for different scenarios:

```dart
// Initialize Supabase client with realtime enabled
final supabase = SupabaseClient(
  'your-supabase-url',
  'your-supabase-anon-key',
  realtimeClientOptions: const RealtimeClientOptions(
    eventsPerSecond: 40,
  ),
);

// Subscribe to channel messages
Stream<Message> connectToChannel(String channelId) {
  return supabase
    .from('messages')
    .stream(primaryKey: ['timetoken'])
    .eq('channel_id', channelId)
    .order('timetoken', ascending: true)
    .map((records) => records.map((record) => Message.fromJson(record)).toList());
}

// Get typing events for a channel
Stream<List<String>> getTypingUsers(String channelId) {
  return supabase
    .from('events')
    .stream(primaryKey: ['id'])
    .eq('channel_id', channelId)
    .eq('type', 'typing')
    .map((records) => records
        .where((record) => record['payload']['value'] == true)
        .map((record) => record['user_id'] as String)
        .toList());
}

// Subscribe to presence changes
Stream<List<String>> streamPresence(String channelId) {
  return supabase
    .from('presence')
    .stream(primaryKey: ['id'])
    .eq('channel_id', channelId)
    .eq('status', true)
    .map((records) => records
        .map((record) => record['user_id'] as String)
        .toList());
}

// Stream updates for a specific message
Stream<Message> streamMessageUpdates(String timetoken) {
  return supabase
    .from('messages')
    .stream(primaryKey: ['timetoken'])
    .eq('timetoken', timetoken)
    .map((records) => records.isNotEmpty ? Message.fromJson(records.first) : null)
    .where((message) => message != null)
    .cast<Message>();
}

// Stream updates for a specific channel
Stream<Channel> streamChannelUpdates(String channelId) {
  return supabase
    .from('channels')
    .stream(primaryKey: ['id'])
    .eq('id', channelId)
    .map((records) => records.isNotEmpty ? Channel.fromJson(records.first) : null)
    .where((channel) => channel != null)
    .cast<Channel>();
}
```

## Performance Considerations

Realtime subscriptions can impact performance, so keep these best practices in mind:

1. Only subscribe to tables and records that are currently relevant to the user
2. Unsubscribe when components are disposed or data is no longer needed
3. Use filters (`eq`, `in`, etc.) to limit the data being streamed
4. For high-volume tables like messages, consider pagination and only subscribing to recent messages

## Realtime Guards for Security

The Row Level Security (RLS) policies we defined earlier apply to realtime subscriptions as well, ensuring users only receive events they have permission to see.

## Example Implementation for Chat.connect Method

```dart
// Implementation of Channel.connect method
class Channel {
  // ...
  
  Function connect(void Function(Message) callback) {
    final subscription = supabase
      .from('messages')
      .stream(primaryKey: ['timetoken'])
      .eq('channel_id', this.id)
      .order('timetoken', ascending: false)
      .limit(1)
      .map((records) => records.isNotEmpty ? Message.fromJson(records.first) : null)
      .where((message) => message != null)
      .cast<Message>()
      .listen(callback);
    
    // Return a function that can be called to disconnect
    return () {
      subscription.cancel();
    };
  }
  
  // ...
}
```

## Supabase Edge Functions for Complex Event Broadcasting

For some complex scenarios, you might want to use Edge Functions to broadcast events to specific clients. Here's an example of a notification Edge Function:

```typescript
// supabase/functions/notify-mention/index.ts
import { serve } from 'https://deno.land/std@0.131.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2.0.0'

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL') ?? '',
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? ''
  )
  
  const { message, mentionedUsers } = await req.json()
  
  // Broadcast to a specific set of users (those mentioned)
  for (const userId of mentionedUsers) {
    await supabase
      .from('events')
      .insert({
        type: 'mention',
        channel_id: message.channelId,
        user_id: userId,
        timetoken: (Date.now() * 10000).toString(),
        payload: {
          messageTimetoken: message.timetoken,
          channel: message.channelId
        }
      })
  }

  return new Response(
    JSON.stringify({ success: true }),
    { headers: { 'Content-Type': 'application/json' } },
  )
})
```

## Additional Supabase Realtime Configuration For GitHub Actions

If you're using GitHub Actions for CI/CD, you can automate the Supabase Realtime configuration using the Supabase CLI. Add this to your workflow:

```yaml
name: Configure Supabase Realtime

on:
  push:
    branches:
      - main
    paths:
      - 'supabase/migrations/**'

jobs:
  configure-realtime:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Supabase CLI
        uses: supabase/setup-cli@v1
        
      - name: Configure Realtime
        run: |
          supabase login -k ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          supabase db push -p ${{ secrets.SUPABASE_DB_PASSWORD }}
        env:
          SUPABASE_PROJECT_ID: ${{ secrets.SUPABASE_PROJECT_ID }}
```

This realtime configuration will ensure that all the necessary real-time functionality required by the PubNub-inspired API is properly supported in your Supabase database.