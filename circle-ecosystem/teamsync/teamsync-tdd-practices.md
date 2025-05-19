# Test Driven Development for Flutter + Supabase Chat Repository APIs

## 1. Understanding TDD in Our Context

### What is TDD for Flutter + Supabase Repository APIs?

Test Driven Development (TDD) in the context of Flutter API development with Supabase backend is a software development approach where tests are written before implementing the actual API functionality. Based on the TeamSync architecture, we're focused on testing repository layer implementations that bridge between the Presentation Layer and both Supabase backend and Drift local database. Our repository APIs follow a "PubNub-inspired" interface.

### Is TDD Meaningful in This Context?

TDD is particularly valuable for repository API development because:

1. **Defined API Contract**: The 114 APIs in the TeamSync API list have clear expectations, making them ideal candidates for test-first development
2. **Complex Data Flow**: Repository methods manage bidirectional flow between Supabase and Drift, which requires careful testing
3. **Asynchronous Operations**: Many methods return Futures or Streams that benefit from systematic testing
4. **Error Handling**: Repository methods must gracefully handle network failures and validate data

## 2. TDD Process for Flutter + Supabase Repository APIs

### The Basic TDD Cycle for Repository Methods

1. **Write a Failing Test**:
   - Create a test that defines the expected behavior of a repository method
   - For example, test that `MessageRepository.sendMessage()` creates a message in both local DB and Supabase
   - The test should fail since the implementation doesn't exist yet

2. **Implement Repository Method**:
   - Write the minimum code required to make the test pass
   - For example, implement the MessageRepository.sendMessage() method that:
     - Creates a message object with a UUID
     - Stores it in the local Drift database (optimistic update)
     - Sends it to Supabase
     - Updates local status based on success/failure

3. **Refactor**:
   - Clean up the implementation while keeping tests passing
   - Extract common patterns, optimize code, improve naming

4. **Repeat for Next API**:
   - Move to the next API method following the recommended sequence in the API list

### Special TDD Considerations for Different API Types

1. **Request APIs (e.g., sendMessage, getChannel)**:
   - Test input validation, error handling, expected outputs
   - Verify both local DB updates and Supabase API calls

2. **Stream/Subscription APIs (e.g., listenForEvents, streamUpdates)**:
   - Test stream initialization, event emission, and stream termination
   - Verify that local DB changes are properly reflected in streams
   - Test how streams react to Supabase realtime events

3. **Event Emission APIs (e.g., emitEvent)**:
   - Test that events are properly dispatched to listeners
   - Verify event data structure and content

## 3. Recommended Tools and Frameworks

### Core Testing Frameworks

1. **Flutter Test**: Base framework for all Flutter testing
2. **Mocktail**: Recommended for mocking Supabase, Drift, and other dependencies
   - Better than mockito for null safety support
   - Simpler syntax without code generation requirements

### Database and API Testing

1. **Drift Testing Utilities**: 
   - `createDriftExecutor()` for in-memory database during tests
   - Test DB migrations and queries

2. **Supabase Mock Client**:
   - Create a mock Supabase client using Mocktail
   - Simulate API responses, realtime events, and errors

### Specialized Testing Tools

1. **Riverpod_test**: 
   - Useful for integration tests that include providers
   - Not essential for pure repository tests, but valuable when testing repository-provider interactions

2. **Fake_async**: 
   - Test time-dependent behavior without actual delays
   - Useful for testing timeouts, retries, and cache expiration

3. **Stream_matcher**:
   - Assert against stream events in a readable way
   - Particularly useful for testing Subscription APIs

## 4. Thumbrules and Guidelines for Effective TDD

### Scope of Tests

1. **Single API Focus**: 
   - Initially, each test should focus on a single API method
   - Follow the API implementation sequence in the document (e.g., start with Chat.emitEvent() before more complex APIs)

2. **Integration Tests Between Related APIs**:
   - After individual method tests pass, create integration tests for related APIs within a slice
   - Example: Test that `Channel.sendText()` and `Channel.getHistory()` work together correctly

3. **Cross-Slice Testing**:
   - Add select tests for critical cross-slice interactions
   - Example: Test that user presence updates (Presence slice) affect message display (Messaging slice)

### Test Organization and Quality

1. **Arrange-Act-Assert Pattern**:
   - Clearly separate test setup, method execution, and assertions
   - Use descriptive variable names and comments

2. **Test Both Happy and Error Paths**:
   - Test successful execution with valid inputs
   - Test failure modes with invalid inputs, network errors, etc.
   - Test edge cases (empty lists, large payloads, etc.)

3. **Mock External Dependencies**:
   - Mock Supabase client to avoid actual network calls
   - Mock Drift database to use in-memory storage
   - Define repeatable mock behaviors for consistent tests

4. **Realistic Test Data**:
   - Use factory methods to generate realistic test models
   - Avoid hardcoded test data spread across test files

## 5. Folder Structure for Tests in Vertical Slice Architecture

### Recommended Structure

```
lib/
├── features/
│   ├── messaging/
│   │   ├── data/
│   │   │   ├── repositories/
│   │   │   │   └── message_repository.dart
│   │   ├── domain/
│   │   │   ├── models/
│   │   │   │   └── message.dart
│   ├── channel_management/
│   │   ├── data/
│   │   │   ├── repositories/
│   │   │   │   └── channel_repository.dart
│   │   ├── domain/
│   │   │   ├── models/
│   │   │   │   └── channel.dart
test/
├── features/
│   ├── messaging/
│   │   ├── data/
│   │   │   ├── repositories/
│   │   │   │   ├── message_repository_test.dart
│   │   │   │   └── message_repository_integration_test.dart
│   │   ├── mocks/
│   │   │   └── message_mocks.dart
│   ├── channel_management/
│   │   ├── data/
│   │   │   ├── repositories/
│   │   │   │   ├── channel_repository_test.dart
│   │   │   │   └── channel_repository_integration_test.dart
│   │   ├── mocks/
│   │   │   └── channel_mocks.dart
├── common/
│   ├── mocks/
│   │   ├── supabase_mock.dart
│   │   └── drift_mock.dart
│   ├── test_utils/
│   │   └── test_helpers.dart
```

### Key Principles

1. **Mirror Production Code Structure**:
   - Test folders should mirror the production code structure
   - Each repository gets its own test file

2. **Shared Test Resources**:
   - Common mocks and test utilities in a shared location
   - Slice-specific mocks in feature-specific mock folders

3. **Separate Integration Tests**:
   - Unit tests focus on individual methods
   - Integration tests verify feature workflows across multiple methods

## 6. High-Quality Prompt for Claude Code to Implement TDD

```
## Task: Implement a TeamSync Repository Method using Test-Driven Development

I need you to help me implement the following API from the TeamSync Chat SDK using a TDD approach:

API: [SPECIFIC_API_NAME] (e.g., "Channel.sendText()")
Category: [REQUEST_API or STREAM_API or EVENT_API]
Slice: [SLICE_NAME] (e.g., "Messaging")
Sequence #: [NUMBER] (from the API list)

## Development Steps:

1. First, write a comprehensive test file for the repository method that:
   - Tests happy path scenarios
   - Tests error handling scenarios
   - Tests any edge cases
   - Properly mocks Supabase and Drift dependencies

2. Then, implement the repository method to make the tests pass:
   - Follow the PubNub-inspired API pattern
   - Handle optimistic updates for write operations
   - Implement proper error handling
   - Follow the bidirectional sync pattern between Supabase and Drift

3. Run the tests and fix any issues until all tests pass.

## Technical Context:

The repository methods connect Flutter UI to both Supabase backend and Drift local database. 
They should handle:
- Optimistic updates (local first, then server)
- Error handling and rollback
- Proper data transformation between DTOs and domain models
- Reactive streams for subscription APIs

## Code Structure Guidelines:

- Use Mocktail for mocking
- Follow AAA pattern (Arrange-Act-Assert) in tests
- Use descriptive test names following "methodName_scenario_expectedResult" format
- Implement clean, readable code with proper documentation

I'll provide feedback after each step. Let's start with writing the tests.
```

## 7. Detailed Implementation Example: TDD for Channel.sendText()

To further illustrate how TDD would be applied in practice, let's look at a concrete example for one of the most fundamental APIs: Channel.sendText() (API #37 in the list).

### Step 1: Write the Test File

```dart
// test/features/messaging/data/repositories/message_repository_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:teamsync/features/messaging/data/repositories/message_repository.dart';
import 'package:teamsync/features/messaging/domain/models/message.dart';
import 'package:uuid/uuid.dart';

// Mocks
class MockSupabaseClient extends Mock implements SupabaseClient {}
class MockMessageDao extends Mock implements MessageDao {}
class MockUuid extends Mock implements Uuid {}

void main() {
  late MessageRepository repository;
  late MockSupabaseClient supabaseClient;
  late MockMessageDao messageDao;
  late MockUuid uuid;
  
  // Test constants
  const channelId = 'channel-123';
  const userId = 'user-456';
  const messageId = 'msg-789';
  const messageContent = 'Hello, world!';
  
  setUp(() {
    supabaseClient = MockSupabaseClient();
    messageDao = MockMessageDao();
    uuid = MockUuid();
    
    // Mock authentication
    final authResponse = MockAuthResponse();
    when(() => supabaseClient.auth).thenReturn(authResponse);
    when(() => authResponse.currentUser).thenReturn(MockUser(id: userId));
    
    // Mock UUID generation
    when(() => uuid.v4()).thenReturn(messageId);
    
    repository = MessageRepository(
      supabaseClient, 
      messageDao,
      uuidGenerator: uuid,
    );
  });
  
  group('sendText', () {
    test('should save message to local DB first, then to Supabase, and update status', () async {
      // Arrange
      final now = DateTime.now();
      // Mock successful DB operations
      when(() => messageDao.insertMessage(any())).thenAnswer((_) => Future.value());
      when(() => messageDao.updateMessageStatus(any(), any())).thenAnswer((_) => Future.value());
      
      // Mock successful Supabase operation
      when(() => supabaseClient.from('messages').insert(any())).thenAnswer(
        (_) => Future.value({'id': messageId}),
      );
      
      // Act
      final result = await repository.sendText(
        channelId: channelId,
        content: messageContent,
      );
      
      // Assert
      // Verify message was saved to local DB first
      verify(() => messageDao.insertMessage(any(
        that: predicate<MessageEntity>((entity) => 
          entity.id == messageId &&
          entity.channelId == channelId &&
          entity.senderId == userId &&
          entity.content == messageContent &&
          entity.status == MessageStatus.sending.index
        ),
      )));
      
      // Verify message was sent to Supabase
      verify(() => supabaseClient.from('messages').insert({
        'id': messageId,
        'channel_id': channelId,
        'sender_id': userId,
        'content': messageContent,
        'timestamp': any(that: isA<String>()),
      }));
      
      // Verify status was updated to sent
      verify(() => messageDao.updateMessageStatus(
        messageId, 
        MessageStatus.sent.index,
      ));
      
      // Verify correct return value
      expect(result, messageId);
    });
    
    test('should update message status to failed when Supabase operation fails', () async {
      // Arrange
      when(() => messageDao.insertMessage(any())).thenAnswer((_) => Future.value());
      when(() => messageDao.updateMessageStatus(any(), any())).thenAnswer((_) => Future.value());
      
      // Mock failed Supabase operation
      when(() => supabaseClient.from('messages').insert(any())).thenThrow(
        Exception('Network error'),
      );
      
      // Act & Assert
      await expectLater(
        () => repository.sendText(
          channelId: channelId,
          content: messageContent,
        ),
        throwsException,
      );
      
      // Verify message status was updated to failed
      verify(() => messageDao.updateMessageStatus(
        messageId, 
        MessageStatus.failed.index,
      ));
    });
    
    // More tests for edge cases, validation, etc.
  });
}
```

### Step 2: Implement the Repository Method

```dart
// lib/features/messaging/data/repositories/message_repository.dart
class MessageRepository {
  final SupabaseClient _supabaseClient;
  final MessageDao _messageDao;
  final Uuid _uuidGenerator;
  
  MessageRepository(
    this._supabaseClient, 
    this._messageDao, 
    {Uuid? uuidGenerator}
  ) : _uuidGenerator = uuidGenerator ?? const Uuid();
  
  /// Sends a text message to the specified channel
  /// 
  /// Returns the ID of the sent message on success
  /// Throws an exception if the message cannot be sent
  Future<String> sendText({
    required String channelId,
    required String content,
  }) async {
    // Input validation
    if (content.trim().isEmpty) {
      throw ArgumentError('Message content cannot be empty');
    }
    
    // Generate a UUID for the new message
    final messageId = _uuidGenerator.v4();
    final userId = _supabaseClient.auth.currentUser!.id;
    final timestamp = DateTime.now().toUtc();
    
    // Create message entity
    final messageEntity = MessageEntity(
      id: messageId,
      channelId: channelId,
      senderId: userId,
      content: content,
      timestamp: timestamp,
      status: MessageStatus.sending.index,
    );
    
    // Save to local DB first (optimistic update)
    await _messageDao.insertMessage(messageEntity);
    
    try {
      // Send to Supabase
      await _supabaseClient
          .from('messages')
          .insert({
            'id': messageId,
            'channel_id': channelId,
            'sender_id': userId,
            'content': content,
            'timestamp': timestamp.toIso8601String(),
          });
      
      // Update local status to sent
      await _messageDao.updateMessageStatus(
        messageId, 
        MessageStatus.sent.index,
      );
      
      return messageId;
    } catch (e) {
      // Update local status to failed
      await _messageDao.updateMessageStatus(
        messageId, 
        MessageStatus.failed.index,
      );
      rethrow;
    }
  }
}
```

## 8. Conclusion and Best Practices

Implementing Test Driven Development for the TeamSync Chat API will provide several benefits:

1. **Higher Quality Code**: Tests ensure the repository methods handle edge cases and errors properly
2. **Better Maintenance**: Tests document the expected behavior of each API method
3. **Faster Development**: Initial investment in tests accelerates later implementation
4. **Confidence in Refactoring**: Tests allow safe refactoring and optimization

The key to successful TDD in this context is to:

1. **Start Simple**: Begin with foundational APIs and build up
2. **Mock Carefully**: Create realistic mocks of Supabase and Drift
3. **Test Incrementally**: Follow the API implementation sequence
4. **Realistic Data Flow**: Tests should model the real data flow between UI, local DB, and Supabase

By following these guidelines and using the recommended tools, you can effectively implement Test Driven Development for the TeamSync Chat API with Claude Code.