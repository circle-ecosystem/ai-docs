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
*   **One-to-One Voice and Video Calls**
    * Initiating Calls
    * Incoming Call UI
    * In-Call Interface (1:1)
    * Call Notification and Logs
    * Call Waiting and Switching
    * Performance and Quality

*   **Group Calls (Voice and Video)**
    * Group Call Size: 32 participants
    * Starting a Group Call
      * Call Button
      * Call Link
      * Scheduled Call
    * Group Call Joining
    * In-Call UI (Group Call)

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