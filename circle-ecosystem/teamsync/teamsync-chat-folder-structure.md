# TeamSync Flutter App: Repository Implementation Folder Structure Report

## Executive Summary

This report outlines the recommended folder structure for the TeamSync Flutter application, with a focus on repository implementation using vertical slice architecture. The structure is optimized for Claude Code development, ensuring "context containment" for AI-assisted coding while maintaining software engineering best practices.

## Full Folder Structure

```
/assets                              # Global shared assets
  /images                            # App-wide images
    /logo                            # App logo variations
    /icons                           # Common UI icons
    /backgrounds                     # Background images
  /fonts                             # Typography assets
  /animations                        # Lottie animations
  /sounds                            # Audio files

/lib
  /chat                              # Chat application
    /shared                          # Shared chat components
      /models                        # Shared domain models
      /utils                         # Shared utilities
      /assets                        # Chat-specific shared assets
    
    /integration                     # Cross-feature integration tests
    
    /events_system                   # Events & System slice (APIs 1-5)
      /models                        # Domain models for this slice
        event_model.dart             # Event domain model
        event_model_test.dart        # Tests collocated with model
      /repository
        /interfaces                  # Repository interfaces (contracts)
          i_event_repository.dart    # PubNub-inspired API interface
          i_event_repository_test.dart # Interface tests
        /implementations             # Concrete implementations
          event_repository_impl.dart # Interface implementation
          event_repository_impl_test.dart # Implementation tests
      /drift                         # Local database components
        event_table.dart            # Drift table definition
        event_dao.dart              # Data Access Object
        event_dao_test.dart         # DAO tests
      /assets                        # Feature-specific assets
      /integration_tests             # Feature-specific integration tests
    
    /user_management                 # User Management slice (APIs 6-16)
      # Same structure as events_system
    
    /channel_management              # Channel Management slice (APIs 17-36)
      # Same structure as events_system
    
    /messaging                       # Messaging slice (APIs 37-45)
      # Same structure as events_system
    
    /files_media                     # Files & Media slice (APIs 46-47)
      # Same structure as events_system
    
    /message_interactions            # Message Interactions slice (APIs 48-54)
      # Same structure as events_system
    
    /message_drafts                  # Message Drafts slice (APIs 55-67)
      # Same structure as events_system
    
    /threading                       # Threading slice (APIs 68-78)
      # Same structure as events_system
    
    /moderation                      # Moderation slice (APIs 79-88)
      # Same structure as events_system
    
    /presence_notifications          # Presence & Notifications slice (APIs 89-114)
      # Same structure as events_system
    
    /core_infrastructure             # Shared infrastructure for chat
      /supabase                      # Supabase client initialization
        supabase_client.dart         # Client configuration
        supabase_client_test.dart    # Client tests
        /realtime                    # Realtime functionality
          supabase_realtime_setup.dart # Base realtime setup
          realtime_provider.dart     # Main provider for handling events
          realtime_provider_test.dart # Provider tests
      /drift                         # Drift database configuration
        database.dart                # Main database definition
        migrations.dart              # Migration management
        database_test.dart           # Database tests
      /sync                          # Data synchronization
      /connectivity                  # Network management
  
  /calls                             # Calls application (placeholder)
    # Similar structure to chat - to be developed later
  
  /app_shared                        # Shared across chat and calls
    /models                          # Truly global models
    /utils                           # Truly global utilities
    /constants                       # Global constants

/integration_test                    # Standard Flutter integration test entry points
  chat_app_test.dart                 # Entry point that imports all chat tests
  calls_app_test.dart                # Entry point for calls tests
  complete_app_test.dart             # Entry point for full app tests

/supabase                            # Supabase database scripts
  /migrations                        # Migration files for CI/CD
    20240510120000_init.sql          # Timestamp-prefixed migration files
    # Additional migrations
  /schema                            # Reference schema definitions
    # Folders mirroring vertical slices
    /events_system
      events_table.sql
      events_rls_policies.sql
      events_functions.sql
    # Other domain schema definitions
  /seed                              # Seed data for development
  /edge_functions                    # Edge functions

/test_config                         # Test configuration
  test_setup.dart                    # Test setup script
  test_utils.dart                    # Common test utilities

/claude_code_helpers                 # Resources for Claude Code
  /templates                         # Code templates for consistency
  /prompts                           # Example prompts for code generation
  /examples                          # Complete examples
```

## Structure Explanation and Justification

### 1. Vertical Slice Architecture

The structure implements vertical slice architecture by organizing code around features rather than technical layers:

- **Benefits**: Each feature/slice can be developed independently with minimal cross-slice dependencies
- **Containment**: All related code for a feature is in one place, optimizing for Claude Code development
- **Organization**: Features map directly to the API groups in `teamsync-chat-api-list.md`

### 2. Collocated Tests

Tests are placed alongside their implementation code:

- **Justification**: Maximizes context containment for Claude Code
- **Benefit**: Reduces context switching when writing or modifying code and tests
- **Clarity**: Makes the relationship between implementation and tests explicit

### 3. Supabase Integration

Supabase is handled through a hybrid approach:

- **Core Initialization**: Lives in `/core_infrastructure/supabase`
- **SQL Definition**: Maintained in `/supabase` folder with standard migration structure
- **Reasoning**: Balances the need for proper migration management with vertical slice containment

### 4. Drift Database Structure

Drift database components are distributed across features:

- **Table Definitions**: Contained within feature slices
- **Central Registry**: Core infrastructure registers all tables
- **Benefit**: Maintains vertical slice cohesion while supporting Drift's structure requirements

### 5. Integration Tests

Integration tests use a hybrid approach:

- **Feature Integration Tests**: Live within feature slices for context containment
- **Cross-Feature Tests**: In `/chat/integration` for tests that span features
- **Standard Entry Points**: In `/integration_test` to follow Flutter conventions
- **Reason**: Balances Flutter conventions with context containment goals

### 6. Asset Management

Assets are managed at three levels:

- **Global Assets**: In root `/assets` folder for app-wide resources
- **Chat-Specific Assets**: In `/chat/shared/assets` for chat-wide resources
- **Feature-Specific Assets**: In each feature's `/assets` folder
- **Benefit**: Keeps assets close to where they're used

### 7. Core Infrastructure

Core shared services are centralized:

- **Supabase Client**: Single initialization point
- **Drift Database**: Central configuration with distributed tables
- **Realtime Handling**: Core setup with feature-specific extensions
- **Reason**: Avoids duplication while maintaining clean organization

## Sample Code Examples

### 1. Domain Model (Messaging Feature)

```dart
// /lib/chat/messaging/models/message_model.dart

import 'package:teamSync/app_shared/models/base_model.dart';

/// Status of a message
enum MessageStatus {
  sending,
  sent,
  delivered,
  read,
  failed
}

/// Message domain model
class Message extends BaseModel {
  final String id;
  final String channelId;
  final String senderId;
  final String content;
  final DateTime timestamp;
  final MessageStatus status;
  
  Message({
    required this.id,
    required this.channelId,
    required this.senderId,
    required this.content,
    required this.timestamp,
    required this.status,
  });
  
  /// Converts Message domain model to database entity
  MessageEntity toEntity() {
    return MessageEntity(
      id: id,
      channelId: channelId,
      senderId: senderId,
      content: content,
      timestamp: timestamp,
      status: status.index,
    );
  }
  
  /// Creates Message domain model from database entity
  factory Message.fromEntity(MessageEntity entity) {
    return Message(
      id: entity.id,
      channelId: entity.channelId,
      senderId: entity.senderId,
      content: entity.content,
      timestamp: entity.timestamp,
      status: MessageStatus.values[entity.status],
    );
  }
  
  /// Creates a copy with updated fields
  Message copyWith({
    String? id,
    String? channelId,
    String? senderId,
    String? content,
    DateTime? timestamp,
    MessageStatus? status,
  }) {
    return Message(
      id: id ?? this.id,
      channelId: channelId ?? this.channelId,
      senderId: senderId ?? this.senderId,
      content: content ?? this.content,
      timestamp: timestamp ?? this.timestamp,
      status: status ?? this.status,
    );
  }
}
```

### 2. Repository Interface (Messaging Feature)

```dart
// /lib/chat/messaging/repository/interfaces/i_message_repository.dart

import 'package:teamSync/chat/messaging/models/message_model.dart';

/// PubNub-inspired repository interface for message operations
abstract class IMessageRepository {
  /// Sends a text message to a channel
  /// 
  /// Returns the message ID on success
  Future<String> sendText({
    required String channelId,
    required String content,
  });
  
  /// Retrieves a specific message by ID
  Future<Message> getMessage({
    required String channelId,
    required String messageId,
  });
  
  /// Gets message history for a channel
  Future<List<Message>> getHistory({
    required String channelId,
    int limit = 50,
    String? startTimeToken,
    String? endTimeToken,
  });
  
  /// Updates message content
  Future<void> editText({
    required String messageId,
    required String newContent,
  });
  
  /// Deletes a message
  Future<void> delete({
    required String messageId,
  });
  
  /// Restores a deleted message
  Future<void> restore({
    required String messageId,
  });
  
  /// Listens for message updates
  Stream<Message> streamUpdates({
    required String messageId,
  });
  
  /// Batch listen for message updates
  Stream<List<Message>> streamUpdatesOn({
    required List<String> messageIds,
  });
}
```

### 3. Repository Implementation (Messaging Feature)

```dart
// /lib/chat/messaging/repository/implementations/message_repository_impl.dart

import 'package:supabase_flutter/supabase_flutter.dart';
import 'package:uuid/uuid.dart';
import 'package:teamSync/chat/messaging/models/message_model.dart';
import 'package:teamSync/chat/messaging/repository/interfaces/i_message_repository.dart';
import 'package:teamSync/chat/messaging/drift/message_dao.dart';

class MessageRepositoryImpl implements IMessageRepository {
  final SupabaseClient _supabaseClient;
  final MessageDao _messageDao;
  
  MessageRepositoryImpl(this._supabaseClient, this._messageDao);
  
  @override
  Future<String> sendText({
    required String channelId,
    required String content,
  }) async {
    // Generate a UUID for the new message
    final messageId = const Uuid().v4();
    final userId = _supabaseClient.auth.currentUser!.id;
    final timestamp = DateTime.now().toUtc();
    
    // Create message object
    final message = Message(
      id: messageId,
      channelId: channelId,
      senderId: userId,
      content: content,
      timestamp: timestamp,
      status: MessageStatus.sending,
    );
    
    // Save to local DB first (optimistic update)
    await _messageDao.insertMessage(message.toEntity());
    
    try {
      // Send to Supabase
      await _supabaseClient
          .from('messages')
          .insert({
            'id': message.id,
            'channel_id': message.channelId,
            'sender_id': message.senderId,
            'content': message.content,
            'timestamp': message.timestamp.toIso8601String(),
          });
      
      // Update local status to sent
      await _messageDao.updateMessageStatus(
        message.id, 
        MessageStatus.sent.index,
      );
      
      return messageId;
    } catch (e) {
      // Update local status to failed
      await _messageDao.updateMessageStatus(
        message.id, 
        MessageStatus.failed.index,
      );
      rethrow;
    }
  }
  
  @override
  Future<Message> getMessage({
    required String channelId,
    required String messageId,
  }) async {
    // Try to get from local DB first
    final localMessage = await _messageDao.getMessageById(messageId);
    
    if (localMessage != null) {
      return Message.fromEntity(localMessage);
    }
    
    // If not in local DB, fetch from Supabase
    final response = await _supabaseClient
        .from('messages')
        .select()
        .eq('id', messageId)
        .eq('channel_id', channelId)
        .single();
    
    final message = Message(
      id: response['id'],
      channelId: response['channel_id'],
      senderId: response['sender_id'],
      content: response['content'],
      timestamp: DateTime.parse(response['timestamp']),
      status: MessageStatus.sent,
    );
    
    // Save to local DB
    await _messageDao.insertMessage(message.toEntity());
    
    return message;
  }
  
  // Other method implementations...
  
  @override
  Stream<Message> streamUpdates({
    required String messageId,
  }) {
    return _messageDao.watchMessageById(messageId).map(
      (entity) => entity != null 
          ? Message.fromEntity(entity) 
          : throw Exception('Message not found'),
    );
  }
}
```

### 4. Drift Table and DAO (Messaging Feature)

```dart
// /lib/chat/messaging/drift/message_table.dart

import 'package:drift/drift.dart';

/// Drift table definition for messages
class Messages extends Table {
  TextColumn get id => text()();
  TextColumn get channelId => text().named('channel_id')();
  TextColumn get senderId => text().named('sender_id')();
  TextColumn get content => text()();
  DateTimeColumn get timestamp => dateTime()();
  IntColumn get status => integer()();
  
  @override
  Set<Column> get primaryKey => {id};
}
```

```dart
// /lib/chat/messaging/drift/message_dao.dart

import 'package:drift/drift.dart';
import 'package:teamSync/chat/core_infrastructure/drift/database.dart';
import 'package:teamSync/chat/messaging/drift/message_table.dart';

/// Data Access Object for messages
class MessageDao {
  final AppDatabase _db;
  
  MessageDao(this._db);
  
  /// Insert a new message
  Future<void> insertMessage(MessageEntity message) {
    return _db.into(_db.messages).insert(message);
  }
  
  /// Update message status
  Future<void> updateMessageStatus(String messageId, int status) {
    return (_db.update(_db.messages)
      ..where((m) => m.id.equals(messageId)))
      .write(MessagesCompanion(status: Value(status)));
  }
  
  /// Get message by ID
  Future<MessageEntity?> getMessageById(String messageId) {
    return (_db.select(_db.messages)
      ..where((m) => m.id.equals(messageId)))
      .getSingleOrNull();
  }
  
  /// Watch specific message for changes
  Stream<MessageEntity?> watchMessageById(String messageId) {
    return (_db.select(_db.messages)
      ..where((m) => m.id.equals(messageId)))
      .watchSingleOrNull();
  }
  
  /// Watch messages for a specific channel
  Stream<List<MessageEntity>> watchMessages(String channelId) {
    return (_db.select(_db.messages)
      ..where((m) => m.channelId.equals(channelId))
      ..orderBy([(m) => OrderingTerm.asc(m.timestamp)]))
      .watch();
  }
  
  /// Upsert multiple messages
  Future<void> upsertMessages(List<MessageEntity> messages) async {
    await _db.batch((batch) {
      for (final message in messages) {
        batch.insert(
          _db.messages,
          message,
          mode: InsertMode.insertOrReplace,
        );
      }
    });
  }
}
```

### 5. Core Database (Drift)

```dart
// /lib/chat/core_infrastructure/drift/database.dart

import 'package:drift/drift.dart';
import 'package:teamSync/chat/messaging/drift/message_table.dart';
import 'package:teamSync/chat/channel_management/drift/channel_table.dart';
import 'package:teamSync/chat/user_management/drift/user_table.dart';
// Other imports...

part 'database.g.dart';

/// Main database configuration
@DriftDatabase(
  tables: [
    Messages,
    Channels,
    Users,
    // Other tables...
  ],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);
  
  @override
  int get schemaVersion => 1;
  
  @override
  MigrationStrategy get migration {
    return MigrationStrategy(
      onCreate: (Migrator m) async {
        await m.createAll();
      },
      onUpgrade: (Migrator m, int from, int to) async {
        if (from < 2) {
          // Migration logic
          await m.addColumn(messages, messages.status);
        }
      },
    );
  }
}
```

### 6. Supabase Realtime

```dart
// /lib/chat/core_infrastructure/supabase/realtime/supabase_realtime_setup.dart

import 'package:supabase_flutter/supabase_flutter.dart';
import 'package:teamSync/chat/core_infrastructure/supabase/realtime/realtime_provider.dart';

/// Setup Supabase realtime subscriptions
void setupSupabaseRealtime(SupabaseClient supabaseClient, RealtimeNotifier realtimeNotifier) {
  // Subscribe to 'messages' table
  supabaseClient
      .from('messages')
      .on(SupabaseEventTypes.insert, (payload) {
        // Dispatch to realtime provider
        realtimeNotifier.handleRealtimeEvent('messages', SupabaseEventTypes.insert, payload);
      })
      .subscribe();
  
  // Subscribe to other tables...
  supabaseClient
      .from('channels')
      .on(SupabaseEventTypes.insert, (payload) {
        realtimeNotifier.handleRealtimeEvent('channels', SupabaseEventTypes.insert, payload);
      })
      .subscribe();
}
```

```dart
// /lib/chat/core_infrastructure/supabase/realtime/realtime_provider.dart

import 'package:riverpod/riverpod.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

typedef RealtimeEventHandler = void Function(String table, String eventType, Map<String, dynamic> payload);

/// Provider to handle Supabase realtime events
@riverpod
class RealtimeNotifier extends _$RealtimeNotifier {
  final Map<String, List<RealtimeEventHandler>> _handlers = {};
  
  /// Register a handler for a specific table
  void registerHandler(String table, RealtimeEventHandler handler) {
    if (!_handlers.containsKey(table)) {
      _handlers[table] = [];
    }
    _handlers[table]!.add(handler);
  }
  
  /// Handle realtime event and dispatch to registered handlers
  void handleRealtimeEvent(String table, String eventType, Map<String, dynamic> payload) {
    if (_handlers.containsKey(table)) {
      for (final handler in _handlers[table]!) {
        handler(table, eventType, payload);
      }
    }
  }
  
  @override
  void build() {
    // Initialize
    ref.onDispose(() {
      // Cleanup
    });
  }
}
```

### 7. Feature-specific Realtime Handlers

```dart
// /lib/chat/messaging/supabase/messaging_realtime_handlers.dart

import 'package:teamSync/chat/core_infrastructure/supabase/realtime/realtime_provider.dart';
import 'package:teamSync/chat/messaging/drift/message_dao.dart';
import 'package:teamSync/chat/messaging/models/message_model.dart';

/// Messaging-specific realtime event handlers
class MessagingRealtimeHandlers {
  final MessageDao _messageDao;
  
  MessagingRealtimeHandlers(this._messageDao);
  
  /// Initialize and register handlers
  void initialize(RealtimeNotifier realtimeNotifier) {
    realtimeNotifier.registerHandler('messages', _handleMessageEvent);
  }
  
  /// Handle message-related events
  void _handleMessageEvent(String table, String eventType, Map<String, dynamic> payload) async {
    if (table != 'messages') return;
    
    final data = payload['new'];
    
    if (eventType == SupabaseEventTypes.insert) {
      // Convert to entity and insert to local DB
      final message = MessageEntity(
        id: data['id'],
        channelId: data['channel_id'],
        senderId: data['sender_id'],
        content: data['content'],
        timestamp: DateTime.parse(data['timestamp']),
        status: MessageStatus.sent.index,
      );
      
      await _messageDao.insertMessage(message);
    }
    
    // Handle other event types...
  }
}
```

### 8. Unit Test Example

```dart
// /lib/chat/messaging/repository/implementations/message_repository_impl_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:supabase_flutter/supabase_flutter.dart';
import 'package:teamSync/chat/messaging/models/message_model.dart';
import 'package:teamSync/chat/messaging/repository/implementations/message_repository_impl.dart';
import 'package:teamSync/chat/messaging/drift/message_dao.dart';

class MockSupabaseClient extends Mock implements SupabaseClient {}
class MockGotrueClient extends Mock implements GoTrueClient {}
class MockMessageDao extends Mock implements MessageDao {}
class MockSupabaseQueryBuilder extends Mock implements SupabaseQueryBuilder {}

void main() {
  late MockSupabaseClient mockSupabaseClient;
  late MockGoTrueClient mockGoTrueClient;
  late MockMessageDao mockMessageDao;
  late MessageRepositoryImpl messageRepository;

  setUp(() {
    mockSupabaseClient = MockSupabaseClient();
    mockGoTrueClient = MockGoTrueClient();
    mockMessageDao = MockMessageDao();
    
    when(() => mockSupabaseClient.auth).thenReturn(mockGoTrueClient);
    
    messageRepository = MessageRepositoryImpl(
      mockSupabaseClient,
      mockMessageDao,
    );
  });

  group('sendText', () {
    test('should save message locally and remotely when successful', () async {
      // Arrange
      const channelId = 'test-channel';
      const content = 'Hello, world!';
      const userId = 'user-123';
      
      final mockUser = User(
        id: userId,
        appMetadata: {},
        userMetadata: {},
        aud: '',
        createdAt: '',
      );
      
      final mockQueryBuilder = MockSupabaseQueryBuilder();
      
      // Mock auth current user
      when(() => mockGoTrueClient.currentUser).thenReturn(mockUser);
      
      // Mock Supabase query chain
      when(() => mockSupabaseClient.from('messages')).thenReturn(mockQueryBuilder);
      when(() => mockQueryBuilder.insert(any())).thenAnswer((_) async => null);
      
      // Mock message DAO methods
      when(() => mockMessageDao.insertMessage(any())).thenAnswer((_) async {});
      when(() => mockMessageDao.updateMessageStatus(any(), any()))
          .thenAnswer((_) async {});
      
      // Act
      final result = await messageRepository.sendText(
        channelId: channelId,
        content: content,
      );
      
      // Assert
      verify(() => mockMessageDao.insertMessage(any())).called(1);
      verify(() => mockQueryBuilder.insert(any())).called(1);
      verify(() => mockMessageDao.updateMessageStatus(
        any(),
        MessageStatus.sent.index,
      )).called(1);
      
      expect(result, isA<String>());
    });
    
    // Additional tests...
  });
}
```

### 9. Integration Test Example

```dart
// /lib/chat/messaging/integration_tests/messaging_flow_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:teamSync/chat/messaging/models/message_model.dart';
import 'package:teamSync/chat/messaging/repository/implementations/message_repository_impl.dart';
import 'package:teamSync/chat/channel_management/repository/implementations/channel_repository_impl.dart';

class MockMessageRepository extends Mock implements MessageRepositoryImpl {}
class MockChannelRepository extends Mock implements ChannelRepositoryImpl {}

void main() {
  late MockMessageRepository messageRepository;
  late MockChannelRepository channelRepository;

  setUp(() {
    messageRepository = MockMessageRepository();
    channelRepository = MockChannelRepository();
  });

  group('Messaging flow', () {
    test('Send and retrieve message flow', () async {
      // Arrange
      const channelId = 'test-channel';
      const content = 'Test message';
      const messageId = 'message-1';
      
      // Mock the sendText call
      when(() => messageRepository.sendText(
        channelId: channelId,
        content: content,
      )).thenAnswer((_) async => messageId);
      
      // Mock the getMessage call
      when(() => messageRepository.getMessage(
        channelId: channelId,
        messageId: messageId,
      )).thenAnswer((_) async => Message(
        id: messageId,
        channelId: channelId,
        senderId: 'user-1',
        content: content,
        timestamp: DateTime.now(),
        status: MessageStatus.sent,
      ));
      
      // Act
      final sentMessageId = await messageRepository.sendText(
        channelId: channelId,
        content: content,
      );
      
      final retrievedMessage = await messageRepository.getMessage(
        channelId: channelId,
        messageId: sentMessageId,
      );
      
      // Assert
      expect(sentMessageId, equals(messageId));
      expect(retrievedMessage.id, equals(messageId));
      expect(retrievedMessage.content, equals(content));
      expect(retrievedMessage.status, equals(MessageStatus.sent));
    });
    
    // More integration tests...
  });
}
```

### 10. Supabase Schema Example

```sql
-- /supabase/schema/messaging/messages_table.sql

-- Table Definition
CREATE TABLE IF NOT EXISTS "public"."messages" (
  "id" uuid NOT NULL,
  "channel_id" uuid NOT NULL,
  "sender_id" uuid NOT NULL,
  "content" text NOT NULL,
  "timestamp" timestamptz NOT NULL DEFAULT now(),
  "is_deleted" boolean NOT NULL DEFAULT false,
  PRIMARY KEY ("id"),
  FOREIGN KEY ("channel_id") REFERENCES "public"."channels"("id") ON DELETE CASCADE,
  FOREIGN KEY ("sender_id") REFERENCES "public"."users"("id") ON DELETE CASCADE
);

-- Indices
CREATE INDEX IF NOT EXISTS "messages_channel_id_idx" ON "public"."messages" ("channel_id");
CREATE INDEX IF NOT EXISTS "messages_sender_id_idx" ON "public"."messages" ("sender_id");
CREATE INDEX IF NOT EXISTS "messages_timestamp_idx" ON "public"."messages" ("timestamp");
```

```sql
-- /supabase/schema/messaging/messages_rls_policies.sql

-- RLS Policies for Messages
ALTER TABLE "public"."messages" ENABLE ROW LEVEL SECURITY;

-- Allow members of a channel to read messages in that channel
CREATE POLICY "Channel members can read messages" ON "public"."messages"
  FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM "public"."channel_members"
      WHERE 
        "channel_members"."user_id" = auth.uid() AND
        "channel_members"."channel_id" = "messages"."channel_id"
    )
  );

-- Allow users to insert their own messages in channels they are members of
CREATE POLICY "Users can insert their own messages" ON "public"."messages"
  FOR INSERT
  WITH CHECK (
    "sender_id" = auth.uid() AND
    EXISTS (
      SELECT 1 FROM "public"."channel_members"
      WHERE 
        "channel_members"."user_id" = auth.uid() AND
        "channel_members"."channel_id" = "messages"."channel_id"
    )
  );
```

## Conclusion

The proposed folder structure for TeamSync Flutter application supports:

1. **Vertical Slice Architecture**: Organizing code around features for independent development
2. **Claude Code Optimization**: Maximizing context containment to enhance AI-assisted development
3. **Test Integration**: Collocating tests with implementation for better context
4. **Database Management**: Balancing standard migration practices with feature organization
5. **Feature Isolation**: Minimizing cross-feature dependencies
6. **Adherence to Patterns**: Following PubNub-inspired API design

By implementing this structure, the TeamSync app will benefit from:
- Clearer code organization
- More efficient development using Claude Code
- Better testability
- Proper separation of concerns
- Scalable architecture for future growth

The structure provides a solid foundation for implementing the extensive API list while maintaining good software engineering practices and optimizing for AI-assisted development.
