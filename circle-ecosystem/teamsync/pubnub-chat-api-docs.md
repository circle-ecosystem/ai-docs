# PubNub Chat SDK API Documentation

## Custom Type Definitions

### ChannelType
```typescript
export type ChannelType = "direct" | "group" | "public" | "unknown"
```

### MessageType
```typescript
export enum MessageType {
  TEXT = "text",
}
```

### MessageActionType
```typescript
export enum MessageActionType {
  REACTIONS = "reactions",
  DELETED = "deleted",
  EDITED = "edited",
}
```

### TextMessageContent
```typescript
export type TextMessageContent = {
  type: MessageType.TEXT
  text: string
  files?: { name: string; id: string; url: string; type?: string }[]
}
```

### EventType
```typescript
export type EventType =
  | "typing"
  | "report"
  | "receipt"
  | "mention"
  | "invite"
  | "custom"
  | "moderation"
```

### MessageActions
```typescript
export type MessageActions = {
  [type: string]: {
    [value: string]: Array<{
      uuid: string
      actionTimetoken: string | number
    }>
  }
}
```

### DeleteParameters
```typescript
export type DeleteParameters = {
  soft?: boolean
}
```

### SendTextOptionParams
```typescript
export type SendTextOptionParams = Omit<PublishParameters, "message" | "channel"> & {
  mentionedUsers?: MessageMentionedUsers
  referencedChannels?: MessageReferencedChannels
  textLinks?: TextLink[]
  quotedMessage?: Message
  files?: FileList | File[] | SendFileParameters["file"][]
}
```

### MessageDraftOptions
```typescript
export type MessageDraftOptions = Omit<PublishParameters, "message" | "channel">
```

### TextLink
```typescript
export type TextLink = {
  startIndex: number
  endIndex: number
  link: string
}
```

### MessageMentionedUsers
```typescript
export type MessageMentionedUsers = {
  [nameOccurrenceIndex: number]: {
    id: string
    name: string
  }
}
```

### MessageReferencedChannels
```typescript
export type MessageReferencedChannels = {
  [nameOccurrenceIndex: number]: {
    id: string
    name: string
  }
}
```

### MixedTextTypedElement
```typescript
export type MixedTextTypedElement =
  | TextTypeElement<"text">
  | TextTypeElement<"mention">
  | TextTypeElement<"plainLink">
  | TextTypeElement<"textLink">
  | TextTypeElement<"channelReference">

type TextTypeElement<T extends TextTypes> = { type: T; content: PayloadForTextTypes[T] }
```

### EventPayloads
```typescript
export type EventPayloads = {
  typing: TypingEventPayload
  report: ReportEventPayload
  receipt: ReceiptEventPayload
  mention: MentionEventPayload
  invite: InviteEventPayload
  moderation: ModerationEventPayload
  custom: CustomEventPayload
}

type TypingEventPayload = {
  value: boolean
}
type ReportEventPayload = {
  text?: string
  reason: string
  reportedMessageTimetoken?: string
  reportedMessageChannelId?: string
  reportedUserId?: string
}
type ReceiptEventPayload = {
  messageTimetoken: string
}
type MentionEventPayload = {
  messageTimetoken: string
  channel: string
}
type InviteEventPayload = {
  channelType: ChannelType | "unknown"
  channelId: string
}
type ModerationEventPayload = {
  channelId: string
  restriction: "muted" | "banned" | "lifted"
  reason?: string
}
type CustomEventPayload = any
```

### EmitEventParams
```typescript
export type EmitEventParams =
  | (TypingEventParams & { payload: TypingEventPayload })
  | (ReportEventParams & { payload: ReportEventPayload })
  | (ReceiptEventParams & { payload: ReceiptEventPayload })
  | (MentionEventParams & { payload: MentionEventPayload })
  | (InviteEventParams & { payload: InviteEventPayload })
  | (CustomEventParams & { payload: CustomEventPayload })
  | (ModerationEventParams & { payload: ModerationEventPayload })
```

### UserMentionData
```typescript
export type UserMentionData = ChannelMentionData | ThreadMentionData

export type ChannelMentionData = {
  event: Event<"mention">
  channelId: string
  message: Message
  userId: string
}

export type ThreadMentionData = {
  event: Event<"mention">
  parentChannelId: string
  threadChannelId: string
  message: Message
  userId: string
}
```

## Chat

### Properties

```typescript
class Chat {
  readonly sdk: PubNub;
  readonly config: ChatConfig;
  readonly currentUser: User;
  readonly errorLogger: ErrorLogger;
  readonly editMessageActionName: string;
  readonly deleteMessageActionName: string;
  readonly reactionsActionName: string;
  readonly accessManager: AccessManager;
}

type ChatConfig = {
  saveDebugLog: boolean;
  typingTimeout: number;
  storeUserActivityInterval: number;
  storeUserActivityTimestamps: boolean;
  pushNotifications: {
    sendPushes: boolean;
    deviceToken?: string;
    deviceGateway: "apns2" | "gcm";
    apnsTopic?: string;
    apnsEnvironment: "development" | "production";
  };
  rateLimitFactor: number;
  rateLimitPerChannel: {
    [key in ChannelType]: number;
  };
  errorLogger?: ErrorLoggerImplementation;
  customPayloads: {
    getMessagePublishBody?: (m: TextMessageContent, channelId: string) => any;
    getMessageResponseBody?: (m: MessageDTOParams) => TextMessageContent;
    editMessageActionName?: string;
    deleteMessageActionName?: string;
    reactionsActionName?: string;
  };
  authKey?: string;
}
```

### Methods

#### init
```typescript
static async init(params: ChatConstructor): Promise<Chat>
```

#### createUser
```typescript
async createUser(
  id: string, 
  data: Omit<UserFields, "id">
): Promise<User>
```

#### createDirectConversation
```typescript
async createDirectConversation({
  user,
  channelId,
  channelData,
  membershipData
}: {
  user: User;
  channelId?: string;
  channelData?: PubNub.ChannelMetadata<PubNub.ObjectCustom>;
  membershipData?: Omit<
    PubNub.SetMembershipsParameters<PubNub.ObjectCustom>,
    "channels" | "include" | "filter"
  > & {
    custom?: PubNub.ObjectCustom;
  };
}): Promise<{
  channel: Channel;
  hostMembership: Membership;
  inviteeMembership: Membership;
}>
```

#### createGroupConversation
```typescript
async createGroupConversation({
  users,
  channelId,
  channelData,
  membershipData
}: {
  users: User[];
  channelId?: string;
  channelData?: PubNub.ChannelMetadata<PubNub.ObjectCustom>;
  membershipData?: Omit<
    PubNub.SetMembershipsParameters<PubNub.ObjectCustom>,
    "channels" | "include" | "filter"
  > & {
    custom?: PubNub.ObjectCustom;
  };
}): Promise<{
  channel: Channel;
  hostMembership: Membership;
  inviteesMemberships: Membership[];
}>
```

#### createPublicConversation
```typescript
async createPublicConversation({
  channelId,
  channelData
}: {
  channelId?: string;
  channelData?: PubNub.ChannelMetadata<PubNub.ObjectCustom>;
} = {}): Promise<Channel>
```

#### currentUser
```typescript
get currentUser(): User
```

#### deleteChannel
```typescript
async deleteChannel(
  id: string, 
  params: DeleteParameters = {}
): Promise<true | Channel>
```

#### deleteUser
```typescript
async deleteUser(
  id: string, 
  params: DeleteParameters = {}
): Promise<true | User>
```

#### downloadDebugLog
```typescript
downloadDebugLog(): Record<string, unknown>
```

#### emitEvent
```typescript
emitEvent(event: EmitEventParams): Promise<any>
```

#### getEventsHistory
```typescript
async getEventsHistory(params: {
  channel: string;
  startTimetoken?: string;
  endTimetoken?: string;
  count?: number;
}): Promise<{
  events: Event<any>[];
  isMore: boolean;
}>
```

#### getChannel
```typescript
async getChannel(id: string): Promise<Channel | null>
```

#### getChannels
```typescript
async getChannels(
  params: Omit<PubNub.GetAllMetadataParameters, "include"> = {}
): Promise<{
  channels: Channel[];
  page: { next: string; prev: string; };
  total: number;
}>
```

#### getChannelSuggestions
```typescript
async getChannelSuggestions(
  text: string,
  options: { limit: number } = { limit: 10 }
): Promise<Channel[]>
```

#### getCurrentUserMentions
```typescript
async getCurrentUserMentions(
  params: { startTimetoken?: string; endTimetoken?: string; count?: number } = {}
): Promise<{
  enhancedMentionsData: UserMentionData[];
  isMore: boolean;
}>
```

#### getUnreadMessagesCounts
```typescript
async getUnreadMessagesCounts(
  params: Omit<GetMembershipsParametersv2, "include"> = {}
): Promise<{
  channel: Channel;
  membership: Membership;
  count: number;
}[]>
```

#### getUser
```typescript
async getUser(id: string): Promise<User | null>
```

#### getUsers
```typescript
async getUsers(
  params: Omit<PubNub.GetAllMetadataParameters, "include"> = {}
): Promise<{
  users: User[];
  page: { next: string; prev: string; };
  total: number;
}>
```

#### getUserSuggestions
```typescript
async getUserSuggestions(
  text: string,
  options: { limit: number } = { limit: 10 }
): Promise<User[]>
```

#### getPushChannels
```typescript
async getPushChannels(): Promise<string[]>
```

#### isPresent
```typescript
async isPresent(userId: string, channelId: string): Promise<boolean>
```

#### listenForEvents
```typescript
listenForEvents<T extends EventType>(
  event: GenericEventParams<T> & { 
    callback: (event: Event<T>) => unknown 
  }
): () => void
```

#### markAllMessagesAsRead
```typescript
async markAllMessagesAsRead(
  params: Omit<GetMembershipsParametersv2, "include"> = {}
): Promise<{
  page: { next: string; prev: string; };
  total: number;
  status: string;
  memberships: Membership[];
} | undefined>
```

#### registerPushChannels
```typescript
async registerPushChannels(channels: string[]): Promise<any>
```

#### setRestrictions
```typescript
async setRestrictions(
  userId: string,
  channelId: string,
  params: { ban?: boolean; mute?: boolean; reason?: string }
): Promise<void>
```

#### unregisterPushChannels
```typescript
async unregisterPushChannels(channels: string[]): Promise<any>
```

#### unregisterAllPushChannels
```typescript
async unregisterAllPushChannels(): Promise<any>
```

#### updateChannel
```typescript
async updateChannel(
  id: string, 
  data: Omit<ChannelFields, "id">
): Promise<Channel>
```

#### updateUser
```typescript
async updateUser(
  id: string, 
  data: Omit<UserFields, "id">
): Promise<User>
```

#### wherePresent
```typescript
async wherePresent(id: string): Promise<string[]>
```

#### whoIsPresent
```typescript
async whoIsPresent(id: string): Promise<string[]>
```

## Channel

### Properties

```typescript
class Channel {
  readonly id: string;
  readonly name?: string;
  readonly custom?: ObjectCustom;
  readonly description?: string;
  readonly updated?: string;
  readonly status?: string;
  readonly type?: ChannelType;
}
```

### Methods

#### connect
```typescript
connect(callback: (message: Message) => void): () => void
```

#### createMessageDraft
```typescript
createMessageDraft(config?: Partial<MessageDraftConfig>): MessageDraft
```

#### delete
```typescript
async delete(options: DeleteParameters = {}): Promise<true | Channel>
```

#### deleteFile
```typescript
async deleteFile(params: { id: string; name: string }): Promise<any>
```

#### forwardMessage
```typescript
async forwardMessage(message: Message): Promise<any>
```

#### getFiles
```typescript
async getFiles(
  params: Omit<PubNub.ListFilesParameters, "channel"> = {}
): Promise<{
  files: { name: string; id: string; url: string }[];
  next: string;
  total: number;
}>
```

#### getHistory
```typescript
async getHistory(
  params: { startTimetoken?: string; endTimetoken?: string; count?: number } = {}
): Promise<{
  messages: Message[];
  isMore: boolean;
}>
```

#### getMessage
```typescript
async getMessage(timetoken: string): Promise<Message>
```

#### getMessageReportsHistory
```typescript
async getMessageReportsHistory(
  params: { startTimetoken?: string; endTimetoken?: string; count?: number } = {}
): Promise<{ 
  events: Event<"report">[];
  isMore: boolean; 
}>
```

#### getMembers
```typescript
async getMembers(
  params: Omit<GetChannelMembersParameters, "channel" | "include"> = {}
): Promise<{
  page: { next: string; prev: string; };
  total: number;
  status: string;
  members: Membership[];
}>
```

#### getPinnedMessage
```typescript
async getPinnedMessage(): Promise<Message | null>
```

#### getTyping
```typescript
getTyping(callback: (typingUserIds: string[]) => unknown): () => void
```

#### getUserRestrictions
```typescript
async getUserRestrictions(user: User): Promise<{
  ban: boolean;
  mute: boolean;
  reason: string | undefined;
}>
```

#### getUsersRestrictions
```typescript
async getUsersRestrictions(
  params?: Pick<PubNub.GetChannelMembersParameters, "limit" | "page" | "sort">
): Promise<{
  page: { next: string; prev: string; };
  total: number;
  status: string;
  restrictions: {
    ban: boolean;
    mute: boolean;
    reason: string | undefined;
    userId: string;
  }[];
}>
```

#### getUserSuggestions
```typescript
async getUserSuggestions(
  text: string,
  options: { limit: number } = { limit: 10 }
): Promise<Membership[]>
```

#### invite
```typescript
async invite(user: User): Promise<Membership>
```

#### inviteMultiple
```typescript
async inviteMultiple(users: User[]): Promise<Membership[]>
```

#### isPresent
```typescript
async isPresent(userId: string): Promise<boolean>
```

#### join
```typescript
async join(
  callback: (message: Message) => void,
  params: Omit<SetMembershipsParameters<ObjectCustom>, "channels" | "include" | "filter"> & {
    custom?: ObjectCustom;
  } = {}
): Promise<{
  membership: Membership;
  disconnect: () => void;
}>
```

#### leave
```typescript
async leave(): Promise<boolean>
```

#### pinMessage
```typescript
async pinMessage(message: Message): Promise<Channel>
```

#### registerForPush
```typescript
registerForPush(): Promise<any>
```

#### sendText
```typescript
async sendText(
  text: string, 
  options: SendTextOptionParams = {}
): Promise<any>
```

#### setRestrictions
```typescript
async setRestrictions(
  user: User, 
  params: { ban?: boolean; mute?: boolean; reason?: string }
): Promise<void>
```

#### startTyping
```typescript
async startTyping(): Promise<any>
```

#### stopTyping
```typescript
async stopTyping(): Promise<any>
```

#### streamPresence
```typescript
async streamPresence(callback: (userIds: string[]) => unknown): Promise<() => void>
```

#### streamReadReceipts
```typescript
async streamReadReceipts(
  callback: (receipts: { [key: string]: string[] }) => unknown
): Promise<() => void>
```

#### streamMessageReports
```typescript
streamMessageReports(callback: (event: Event<"report">) => void): () => void
```

#### streamUpdates
```typescript
streamUpdates(callback: (channel: Channel) => unknown): () => void
```

#### streamUpdatesOn
```typescript
static streamUpdatesOn(
  channels: Channel[], 
  callback: (channels: Channel[]) => unknown
): () => void
```

#### update
```typescript
async update(data: Omit<ChannelFields, "id">): Promise<Channel>
```

#### unpinMessage
```typescript
async unpinMessage(): Promise<Channel>
```

#### unregisterFromPush
```typescript
unregisterFromPush(): Promise<any>
```

#### whoIsPresent
```typescript
async whoIsPresent(): Promise<string[]>
```

## User

### Properties

```typescript
class User {
  readonly id: string;
  readonly name?: string;
  readonly externalId?: string;
  readonly profileUrl?: string;
  readonly email?: string;
  readonly custom?: ObjectCustom;
  readonly status?: string;
  readonly type?: string;
  readonly updated?: string;
  readonly lastActiveTimestamp?: number;
}
```

### Methods

#### active
```typescript
get active(): boolean
```

#### delete
```typescript
async delete(options: DeleteParameters = {}): Promise<true | User>
```

#### getChannelRestrictions
```typescript
async getChannelRestrictions(channel: Channel): Promise<{
  ban: boolean;
  mute: boolean;
  reason: string | undefined;
}>
```

#### getChannelsRestrictions
```typescript
async getChannelsRestrictions(
  params?: Pick<PubNub.GetChannelMembersParameters, "limit" | "page" | "sort">
): Promise<{
  page: { next: string; prev: string; };
  total: number;
  status: string;
  restrictions: {
    ban: boolean;
    mute: boolean;
    reason: string | undefined;
    channelId: string;
  }[];
}>
```

#### getMemberships
```typescript
async getMemberships(
  params: Omit<GetMembershipsParametersv2, "include" | "uuid"> = {}
): Promise<{
  page: { next: string; prev: string; };
  total: number;
  status: string;
  memberships: Membership[];
}>
```

#### isPresentOn
```typescript
async isPresentOn(channelId: string): Promise<boolean>
```

#### setRestrictions
```typescript
async setRestrictions(
  channel: Channel,
  params: { ban?: boolean; mute?: boolean; reason?: string }
): Promise<void>
```

#### streamUpdates
```typescript
streamUpdates(callback: (user: User) => unknown): () => void
```

#### streamUpdatesOn
```typescript
static streamUpdatesOn(
  users: User[], 
  callback: (users: User[]) => unknown
): () => void
```

#### update
```typescript
async update(data: Omit<UserFields, "id">): Promise<User>
```

#### wherePresent
```typescript
async wherePresent(): Promise<string[]>
```

## Message

### Properties

```typescript
class Message {
  readonly timetoken: string;
  readonly content: TextMessageContent;
  readonly channelId: string;
  readonly userId: string;
  readonly actions?: MessageActions;
  readonly meta?: {
    [key: string]: any;
  };
  readonly error?: string;
}
```

### Methods

#### createThread
```typescript
createThread(): Promise<ThreadChannel>
```

#### delete
```typescript
async delete(
  params: DeleteParameters & { preserveFiles?: boolean } = {}
): Promise<boolean | Message>
```

#### deleted
```typescript
get deleted(): boolean
```

#### hasThread
```typescript
get hasThread(): boolean
```

#### editText
```typescript
async editText(newText: string): Promise<Message>
```

#### forward
```typescript
async forward(channelId: string): Promise<any>
```

#### getMessageElements
```typescript
getMessageElements(): MixedTextTypedElement[]
```

#### getThread
```typescript
getThread(): Promise<ThreadChannel>
```

#### hasUserReaction
```typescript
hasUserReaction(reaction: string): boolean
```

#### toggleReaction
```typescript
async toggleReaction(reaction: string): Promise<Message>
```

#### pin
```typescript
async pin(): Promise<void>
```

#### quotedMessage
```typescript
get quotedMessage(): any
```

#### reactions
```typescript
get reactions(): { [key: string]: { uuid: string; actionTimetoken: string | number }[] }
```

#### removeThread
```typescript
removeThread(): Promise<[any, true]>
```

#### report
```typescript
async report(reason: string): Promise<any>
```

#### restore
```typescript
async restore(): Promise<Message | undefined>
```

#### streamUpdates
```typescript
streamUpdates(callback: (message: Message) => unknown): () => void
```

#### streamUpdatesOn
```typescript
static streamUpdatesOn(
  messages: Message[], 
  callback: (messages: Message[]) => unknown
): () => void
```

#### text
```typescript
get text(): string
```

## Membership

### Properties

```typescript
class Membership {
  readonly channel: Channel;
  readonly user: User;
  readonly custom: ObjectCustom | null | undefined;
  readonly updated: string;
  readonly eTag: string;
}
```

### Methods

#### getUnreadMessagesCount
```typescript
async getUnreadMessagesCount(): Promise<number | false>
```

#### lastReadMessageTimetoken
```typescript
get lastReadMessageTimetoken(): string | undefined
```

#### setLastReadMessage
```typescript
async setLastReadMessage(message: Message): Promise<Membership>
```

#### setLastReadMessageTimetoken
```typescript
async setLastReadMessageTimetoken(timetoken: string): Promise<Membership>
```

#### streamUpdates
```typescript
streamUpdates(callback: (membership: Membership) => unknown): () => void
```

#### streamUpdatesOn
```typescript
static streamUpdatesOn(
  memberships: Membership[], 
  callback: (memberships: Membership[]) => unknown
): () => void
```

#### update
```typescript
async update({ custom }: { custom: ObjectCustom }): Promise<Membership>
```

## ThreadChannel

### Properties

```typescript
class ThreadChannel extends Channel {
  readonly parentChannelId: string;
  readonly parentMessage: Message;
}
```

### Methods

#### getHistory
```typescript
async getHistory(
  params: { startTimetoken?: string; endTimetoken?: string; count?: number } = {}
): Promise<{
  messages: ThreadMessage[];
  isMore: boolean;
}>
```

#### pinMessage
```typescript
async pinMessage(message: ThreadMessage): Promise<ThreadChannel>
```

#### pinMessageToParentChannel
```typescript
async pinMessageToParentChannel(message: ThreadMessage): Promise<Channel>
```

#### unpinMessage
```typescript
async unpinMessage(): Promise<ThreadChannel>
```

#### unpinMessageFromParentChannel
```typescript
async unpinMessageFromParentChannel(): Promise<Channel>
```

*Note: ThreadChannel also inherits all methods from Channel.*

## ThreadMessage

### Properties

```typescript
class ThreadMessage extends Message {
  readonly parentChannelId: string;
}
```

### Methods

#### streamUpdatesOn
```typescript
static streamUpdatesOn(
  threadMessages: ThreadMessage[], 
  callback: (threadMessages: ThreadMessage[]) => unknown
): () => void
```

#### pinToParentChannel
```typescript
async pinToParentChannel(): Promise<Channel>
```

#### unpinFromParentChannel
```typescript
async unpinFromParentChannel(): Promise<Channel>
```

*Note: ThreadMessage also inherits all methods from Message.*

## MessageDraft

### Properties

```typescript
class MessageDraft {
  public value: string;
  public quotedMessage: Message | undefined;
  files?: FileList | File[] | SendFileParameters["file"][];
  readonly config: MessageDraftConfig;
}

type MessageDraftConfig = {
  userSuggestionSource: "channel" | "global";
  isTypingIndicatorTriggered: boolean;
  userLimit: number;
  channelLimit: number;
}
```

### Methods

#### onChange
```typescript
async onChange(text: string): Promise<{
  users: {
    nameOccurrenceIndex: number;
    suggestedUsers: User[];
  };
  channels: {
    channelOccurrenceIndex: number;
    suggestedChannels: Channel[];
  };
}>
```

#### addMentionedUser
```typescript
addMentionedUser(user: User, nameOccurrenceIndex: number): void
```

#### addReferencedChannel
```typescript
addReferencedChannel(channel: Channel, channelNameOccurrenceIndex: number): void
```

#### removeReferencedChannel
```typescript
removeReferencedChannel(channelNameOccurrenceIndex: number): void
```

#### removeMentionedUser
```typescript
removeMentionedUser(nameOccurrenceIndex: number): void
```

#### send
```typescript
async send(params: MessageDraftOptions = {}): Promise<any>
```

#### getHighlightedMention
```typescript
getHighlightedMention(selectionStart: number): {
  mentionedUser: User | null;
  nameOccurrenceIndex: number;
}
```

#### addLinkedText
```typescript
addLinkedText(params: {
  text: string;
  link: string;
  positionInInput: number;
}): void
```

#### removeLinkedText
```typescript
removeLinkedText(positionInInput: number): void
```

#### getMessagePreview
```typescript
getMessagePreview(): MixedTextTypedElement[]
```

#### addQuote
```typescript
addQuote(message: Message): void
```

#### removeQuote
```typescript
removeQuote(): void
```

## Event

### Properties

```typescript
class Event<T extends EventType> {
  readonly timetoken: string;
  readonly type: T;
  readonly payload: EventPayloads[T];
  readonly channelId: string;
  readonly userId: string;
}
```

No methods are listed for the Event class in the provided documentation.
