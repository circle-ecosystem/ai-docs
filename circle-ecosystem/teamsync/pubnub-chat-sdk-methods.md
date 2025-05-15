## PubNub Chat SDK API Methods

### Chat Methods
* Chat.createUser()
* Chat.createDirectConversation()
* Chat.createGroupConversation()
* Chat.createPublicConversation()
* Chat.currentUser
* Chat.deleteChannel()
* Chat.deleteUser()
* Chat.downloadDebugLog()
* Chat.emitEvent()
* Chat.getEventsHistory()
* Chat.getChannel()
* Chat.getChannels()
* Chat.getChannelSuggestions()
* Chat.getCurrentUserMentions()
* Chat.fetchUnreadMessagesCounts()
* Chat.getUser()
* Chat.getUsers()
* Chat.getUserSuggestions()
* Chat.getPushChannels()
* Chat.init()
* Chat.isPresent()
* Chat.listenForEvents()
* Chat.markAllMessagesAsRead()
* Chat.registerPushChannels()
* Chat.setRestrictions()
* Chat.unregisterPushChannels()
* Chat.unregisterAllPushChannels()
* Chat.updateChannel()
* Chat.updateUser()
* Chat.wherePresent()
* Chat.whoIsPresent()

### Channel Methods
* Channel.connect()
* Channel.createMessageDraft()
* Channel.createMessageDraftV2()
* Channel.delete()
* Channel.deleteFile()
* Channel.forwardMessage()
* Channel.getFiles()
* Channel.getHistory()
* Channel.getMessage()
* Channel.getMessageReportsHistory()
* Channel.getMembers()
* Channel.getPinnedMessage()
* Channel.getTyping()
* Channel.getUserRestrictions()
* Channel.getUsersRestrictions()
* Channel.getUserSuggestions()
* Channel.invite()
* Channel.inviteMultiple()
* Channel.isPresent()
* Channel.join()
* Channel.leave()
* Channel.pinMessage()
* Channel.registerForPush()
* Channel.sendText()
* Channel.setRestrictions()
* Channel.startTyping()
* Channel.stopTyping()
* Channel.streamPresence()
* Channel.streamReadReceipts()
* Channel.streamMessageReports()
* Channel.streamUpdates()
* Channel.streamUpdatesOn()
* Channel.update()
* Channel.unpinMessage()
* Channel.unregisterFromPush()
* Channel.whoIsPresent()

### User Methods
* User.active
* User.delete()
* User.getChannelRestrictions()
* User.getChannelsRestrictions()
* User.getMemberships()
* User.isPresentOn()
* User.setRestrictions()
* User.streamUpdates()
* User.streamUpdatesOn()
* User.update()
* User.wherePresent()

### Message Methods
* Message.createThread()
* Message.delete()
* Message.deleted
* Message.hasThread
* Message.editText()
* Message.forward()
* Message.getMessageElements()
* Message.getThread()
* Message.hasUserReaction()
* Message.toggleReaction()
* Message.pin()
* Message.quotedMessage
* Message.reactions
* Message.removeThread()
* Message.report()
* Message.restore()
* Message.streamUpdates()
* Message.streamUpdatesOn()
* Message.text

### Membership Methods
* Membership.getUnreadMessagesCount()
* Membership.lastReadMessageTimetoken
* Membership.setLastReadMessage()
* Membership.setLastReadMessageTimetoken()
* Membership.streamUpdates()
* Membership.streamUpdatesOn()
* Membership.update()

### ThreadChannel Methods
* ThreadChannel.getHistory()
* ThreadChannel.pinMessage()
* ThreadChannel.pinMessageToParentChannel()
* ThreadChannel.unpinMessage()
* ThreadChannel.unpinMessageFromParentChannel()
* ThreadChannel.[All Channel Methods]

### ThreadMessage Methods
* ThreadMessage.streamUpdatesOn()
* ThreadMessage.pinToParentChannel()
* ThreadMessage.unpinFromParentChannel()
* ThreadMessage.[All Message Methods]

### MessageDraftV2 Methods
* MessageDraftV2.[No specific methods listed in documentation]

### MessageDraft Methods
* MessageDraft.[No specific methods listed in documentation]
