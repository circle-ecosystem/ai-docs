# TeamSync Vision Document

## App Goals 

- Build a communication app that integrates with the Circle Ecosystem to help users communicate via the following means

  - Text Chat 
  - Audio Video Calls 
  - Remote Desktop 

## High Level Features

### Authentication
- integrate with Circle Foundations for authentication

### Contacts 
- sync the contacts list from the Circle Foundations

### Contact Presence
- sync the contacts presence from Circle Foundations

### Chat
*   **Manage channels**
    *   Create, update, and delete direct, public, and group channels
*   **Invite to channels**
    *   Enable users to invite others to join channels by sending invitations
*   **Watch channels**
    *   Enable users to monitor a channel and its messages, allowing them to receive messages without joining as members
*   **Join or leave channels**
    *   Allow users to join channels of interest or relevance and leave channels that are no longer needed
*   **Reference channels**
    *   Let users mention specific channels in a chat conversation by typing `@` followed by at least the first three letters of the channel name
*   **Add typing indicator**
    *   Provide real-time feedback to users when someone is composing a message
*   **Mention users**
    *   Tag specific individuals within a chat by typing `@` followed by the first three letters of a username, triggering a list of suggestions
*   **Manage channel and user access**
    *   Implement detailed permission schemas to control access to specific channels and user metadata within your chat app
*   **Moderate users**
    *   Mute or ban misbehaving users
*   **Send/Receive messages**
    *   Allow users to exchange text-based communications in real-time within channels
*   **Manage messages**
    *   Create, edit, and remove messages
*   **Store historical messages**
    *   Archive past conversations
*   **Restore messages**
    *   Recover deleted messages within a specified timeframe
*   **Forward messages**
    *   Share specific messages with others or across different channels
*   **Add/Remove message reactions**
    *   Add or remove reactions, such as emojis, to messages
*   **Create message drafts**
    *   Enable users to compose messages and review them before sending
*   **Quote messages**
    *   Quote messages within channels
*   **Create message threads**
    *   Start and participate in focused sub-conversations
*   **Send links**
    *   Share URLs in messages
*   **Attach files**
    *   Attach and share files of various formats
*   **Pin messages**
    *   Pin important messages to the top of the chat for easy reference
*   **Track unread messages**
    *   Implement notifications and visual markers for users to easily identify unread messages in channels
*   **Get message read receipts**
    *   Be notified when other channel members have received and viewed a message
*   **Flag/Report offensive messages**
    *   Flag or report messages deemed inappropriate
*   **Receive mobile push notifications**
    *   Support push notifications on mobile devices to alert users of new messages, mentions, or other significant activities

### Calls

#### 1. User Presence Slice
- Real-time user online status
- Custom status messages (Available, Busy, Away)
- User device capabilities detection (camera/microphone)
- Call availability indicators
- Automatic status updates when in calls
- Favorites and recent contacts

#### 2. Basic Calling Slice
- Initiate outgoing calls (direct ringing)
- Receive incoming calls
- Call acceptance/rejection
- Basic audio/video transmission
- Call termination
- Fallback to audio-only mode during poor network conditions
- Enable PIP (Picture-in-Picture) mode for multitasking
- Call event notifications (start, join, leave, end, disconnect)

#### 3. Media Quality Slice
- Camera selection and configuration
- Microphone selection and configuration
- Video resolution and bitrate management
- Bandwidth adaptation
- Network quality indicators
- Background blur/replacement
- Noise suppression
- Lighting adjustment
- Beauty filters
- Change view layouts (speaker, grid, pinned)

#### 4. Call Scheduling Slice
- Schedule future calls with date/time selection
- Create recurring calls (daily, weekly, monthly)
- Generate shareable call links
- Send call invites via email
- Send invites via in-app notifications
- Integration with Google Calendar
- Integration with Outlook Calendar
- Integration with in-app calendar
- Calendar view of upcoming calls
- Call reminders and notifications
- Edit or cancel scheduled calls
- Time zone management

#### 5. Group Call Management Slice
- Create and manage group calls
- View participant list
- Mute/unmute individual participants
- Remove participants from calls
- Raise hand functionality
- Emoji reactions during calls
- Add participants to ongoing calls
- Restrict call access (passcodes, waiting rooms)
- Host approval for participants
- Lock room to prevent new joins
- Transfer host privileges
- Participant capacity management
- In-call polling/voting

#### 6. In-Call Collaboration Slice
- In-call text chat messaging
- File sharing during calls
- Link sharing in chat
- Emoji and reaction support in chat
- Chat history preservation
- Whiteboard collaboration
- Note-taking during calls
- Task assignment
- Chat moderation tools
- Message formatting
- Read receipts

#### 7. Screen Sharing Slice
- Whole screen sharing
- Application window sharing
- Screen selection interface
- Screen share controls (start/stop)
- Screen sharing indicators
- Pause/resume screen sharing
- Adjust screen sharing resolution
- Audio sharing with screen
- Presenter switching
- Screen annotations
- Highlight cursor option
- Mobile screen sharing

#### 8. Remote Control Slice
- Request/grant remote control
- Mouse control transmission
- Keyboard input transmission
- Control session termination
- Switch between multiple screens/monitors
- Transfer files between machines
- Clipboard synchronization
- Annotate remote screen
- Switch control roles between participants
- Enable/disable remote input
- Send keyboard shortcuts (Ctrl+Alt+Del, Alt+Tab)
- Reconnect automatically after interruption
- Monitor session status (connection strength, latency)
- Set session time limits
- Session notifications (start, end, timeouts)
- Log remote session history

#### 9. Call Recording Slice
- Server-side recording
- Recording controls (start/stop)
- Recording status indicators
- Recorded call management
- Recording playback
- Recording permissions management
- Auto-recording option
- Recording encryption
- Recording transcription
- Recording storage management
- Recording sharing options
- Recording retention policies
- Recording notifications for participants

#### 10. Call History Slice
- Call history listing
- Call metadata (duration, participants, timestamps)
- Call categorization (missed, completed, rejected)
- Call history search and filtering
- Call statistics and analytics
- Export call logs
- Delete call history entries
- Call rating and feedback
- Call history synchronization
- Call continuation (redial)

#### 11. Notification Slice
- Incoming call push notifications
- Missed call notifications
- Call reminder notifications
- Notification preferences
- Silent hours configuration
- Custom notification sounds
- Vibration patterns
- LED indicator customization (Android)
- Notification action buttons
- Notification grouping
- Read/unread status
- Do Not Disturb integration

#### 12. Security & Compliance Slice
- End-to-end encryption
- Call privacy controls
- Audit logging
- GDPR compliance tools
- Security settings
- Meeting password protection
- Consent for recording
- Session locking
- Anti-abuse measures
- Data retention controls
- Export user data tools
- IP restriction options
- Two-factor authentication for sensitive calls
- Sensitive information detection

#### 13. Accessibility Slice
- Screen reader compatibility
- Keyboard navigation
- Closed captioning
- Transcription
- High contrast mode
- Font size adjustments
- Color blindness accommodations
- Reduced motion option
- TTY/TDD compatibility
- Assistive technology integrations
- Accessibility preferences
- Keyboard shortcuts
- Focus indicators

#### 14. Core Infrastructure Slice
- Flutter App Structure
- Supabase Integration
- LiveKit Integration
- State Management
- Error Handling
- Event Bus
- Offline capability
- Database schema design
- Performance monitoring
- Crash reporting
- Analytics framework
- Localization infrastructure
- Theme management
- Device capability detection

## Define High Level Architecture Requirements
- Authentication, Contacts, User Profile, User Presence will be imported from Circle Foundations. 
- Platform support:
  - iOS
  - Android
  - Windows
  - Mac
- Achieving parity - Features and UX: 
  - for each Circle customer, the resistance to switching to TeamSync should be minimal. TeamSync should provide them with all the key features they were using in an identical/familiar UX. 
- The Chat and Calls should be treated as indpendent logical apps but brought together via a common UI shell. 
- The app should support a common non-presentation layers and separate presentation layers for different UX requirements. 
- The build system should be able to generate apps for a specified UX. 
- There should be flexibility to have custom UI for desktop vs mobile platforms. 
- offline-first 
- multi-device support 