# TeamSync Vertical Slice Architecture for Chat App

Based on the provided PubNub SDK methods and TeamSync vision documents, below are the identified logical vertical slices that organize related functionality while minimizing coupling between different parts of the system. Each slice encompasses both the API methods and corresponding UI screens.

## Vertical Slices Table

| Vertical Slice Name | API List | Screens Listing |
|---------------------|----------|----------------|
| User Management | • Chat.createUser()<br>• Chat.currentUser<br>• Chat.deleteUser()<br>• Chat.getUser()<br>• Chat.getUsers()<br>• Chat.getUserSuggestions()<br>• Chat.updateUser()<br>• User.active<br>• User.delete()<br>• User.update()<br>• User.getMemberships()<br>• User.streamUpdates()<br>• User.streamUpdatesOn() | • User Profile Screen<br>• User Settings Screen<br>• User Directory/List Screen<br>• User Search Screen<br>• User Edit Screen |
| Channel Management | • Chat.createDirectConversation()<br>• Chat.createGroupConversation()<br>• Chat.createPublicConversation()<br>• Chat.deleteChannel()<br>• Chat.getChannel()<br>• Chat.getChannels()<br>• Chat.getChannelSuggestions()<br>• Chat.updateChannel()<br>• Channel.connect()<br>• Channel.delete()<br>• Channel.update()<br>• Channel.invite()<br>• Channel.inviteMultiple()<br>• Channel.join()<br>• Channel.leave()<br>• Channel.getMembers()<br>• Channel.getUserSuggestions()<br>• Membership.update() | • Channel List Screen<br>• Channel Creation Screen<br>• Channel Settings/Details Screen<br>• Channel Member Management Screen<br>• Channel Join/Leave Screen<br>• Channel Search Screen |
| Messaging | • Channel.sendText()<br>• Channel.getMessage()<br>• Channel.getHistory()<br>• Message.editText()<br>• Message.delete()<br>• Message.restore()<br>• Message.text<br>• Message.deleted<br>• Message.getMessageElements()<br>• Message.streamUpdates()<br>• Message.streamUpdatesOn() | • Message List/Chat View Screen<br>• Message Composer Screen<br>• Message Edit Interface<br>• Message History/Search Screen |
| Message Interactions | • Message.hasUserReaction()<br>• Message.toggleReaction()<br>• Message.reactions<br>• Message.pin()<br>• Message.quotedMessage<br>• Channel.pinMessage()<br>• Channel.unpinMessage()<br>• Channel.getPinnedMessage()<br>• Message.forward()<br>• Channel.forwardMessage() | • Reaction UI Component<br>• Pinned Messages Screen<br>• Quote Message Interface<br>• Forward Message Screen |
| Threading | • Message.createThread()<br>• Message.hasThread<br>• Message.getThread()<br>• Message.removeThread()<br>• ThreadChannel.getHistory()<br>• ThreadChannel.pinMessage()<br>• ThreadChannel.unpinMessage()<br>• ThreadChannel.pinMessageToParentChannel()<br>• ThreadChannel.unpinMessageFromParentChannel()<br>• ThreadMessage.streamUpdatesOn()<br>• ThreadMessage.pinToParentChannel()<br>• ThreadMessage.unpinFromParentChannel() | • Thread View Screen<br>• Thread List Component<br>• Thread Creation Interface |
| Presence & Notifications | • Chat.isPresent()<br>• Chat.wherePresent()<br>• Chat.whoIsPresent()<br>• Channel.isPresent()<br>• Channel.whoIsPresent()<br>• Channel.startTyping()<br>• Channel.stopTyping()<br>• Channel.getTyping()<br>• Channel.streamPresence()<br>• User.isPresentOn()<br>• User.wherePresent()<br>• Chat.registerPushChannels()<br>• Chat.unregisterPushChannels()<br>• Chat.unregisterAllPushChannels()<br>• Chat.getPushChannels()<br>• Channel.registerForPush()<br>• Channel.unregisterFromPush()<br>• Channel.streamReadReceipts()<br>• Chat.fetchUnreadMessagesCounts()<br>• Chat.markAllMessagesAsRead()<br>• Membership.getUnreadMessagesCount()<br>• Membership.lastReadMessageTimetoken<br>• Membership.setLastReadMessage()<br>• Membership.setLastReadMessageTimetoken()<br>• Membership.streamUpdates()<br>• Membership.streamUpdatesOn()<br>• Chat.getCurrentUserMentions() | • Presence Indicators Component<br>• Typing Indicators Component<br>• Online Status Display Component<br>• Notification Settings Screen<br>• Unread Counts Display Component<br>• Push Notification Management Screen<br>• Mention List Screen<br>• Read Receipts View |
| Files & Media | • Channel.deleteFile()<br>• Channel.getFiles() | • File Attachment Interface<br>• File Gallery/List Screen<br>• File Preview Screen |
| Moderation | • Chat.setRestrictions()<br>• Channel.setRestrictions()<br>• Channel.getUserRestrictions()<br>• Channel.getUsersRestrictions()<br>• User.getChannelRestrictions()<br>• User.getChannelsRestrictions()<br>• User.setRestrictions()<br>• Message.report()<br>• Channel.getMessageReportsHistory()<br>• Channel.streamMessageReports() | • Message Report Interface<br>• User Restriction Management Screen<br>• Channel Moderation Dashboard<br>• Report History Screen |
| Message Drafts | • Channel.createMessageDraft()<br>• Channel.createMessageDraftV2()<br>• MessageDraft methods<br>• MessageDraftV2 methods | • Draft Message Interface<br>• Saved Drafts List Screen |
| Events & System | • Chat.emitEvent()<br>• Chat.getEventsHistory()<br>• Chat.listenForEvents()<br>• Chat.init()<br>• Chat.downloadDebugLog()<br>• Channel.streamUpdates()<br>• Channel.streamUpdatesOn() | • Event Log/History Screen<br>• System Settings Screen<br>• Debug Log Screen |

## Rationale for Slice Organization

Each vertical slice is organized around a cohesive business capability with minimal coupling between slices:

1. **User Management**: Centralizes all user-related operations including profile management, user search, and membership tracking.

2. **Channel Management**: Handles all conversation creation, management, and membership operations for different channel types (direct, group, public).

3. **Messaging**: Core message functionality for sending, receiving, and managing messages within channels.

4. **Message Interactions**: Extensions to basic messaging like reactions, pinning, and forwarding that enhance the core messaging experience.

5. **Threading**: Dedicated functionality for threaded conversations, which have their own special behaviors and UI requirements.

6. **Presence & Notifications**: Groups together real-time presence features (typing indicators, online status) with notification management and read receipts.

7. **Files & Media**: Focused on file attachment and management within conversations.

8. **Moderation**: Dedicated to content moderation, user restrictions, and reporting features.

9. **Message Drafts**: Handles saving and managing draft messages.

10. **Events & System**: System-level functionality including initialization, event handling, and debugging.

This organization aligns with the TeamSync architecture guidelines, providing clear boundaries between slices while ensuring that each slice represents a complete feature set that includes both API methods and corresponding UI screens.