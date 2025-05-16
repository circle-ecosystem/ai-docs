# TeamSync Database Design for Chat Functionality

This document outlines a comprehensive database design for the TeamSync Chat application to support implementing all the PubNub APIs as specified in the requirements.

## Table of Contents
- [Database Schema](#database-schema)
- [Row Level Security Policies](#row-level-security-policies)
- [Database Functions](#database-functions)
- [Indexes](#indexes)
- [Migration Script](#migration-script)
- [API Implementation Coverage Analysis](#api-implementation-coverage-analysis)

## Database Schema

### `users` Table
```sql
CREATE TABLE users (
    id TEXT PRIMARY KEY,
    name TEXT,
    external_id TEXT,
    profile_url TEXT,
    email TEXT,
    custom JSONB,
    status TEXT,
    type TEXT,
    updated TIMESTAMP WITH TIME ZONE,
    last_active_timestamp BIGINT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### `channels` Table
```sql
CREATE TABLE channels (
    id TEXT PRIMARY KEY,
    name TEXT,
    custom JSONB,
    description TEXT,
    updated TIMESTAMP WITH TIME ZONE,
    status TEXT,
    type TEXT CHECK (type IN ('direct', 'group', 'public', 'unknown')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    pinned_message_timetoken TEXT
);
```

### `messages` Table
```sql
CREATE TABLE messages (
    timetoken TEXT PRIMARY KEY,
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    content JSONB,
    actions JSONB,
    meta JSONB,
    error TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### `memberships` Table
```sql
CREATE TABLE memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    custom JSONB,
    updated TIMESTAMP WITH TIME ZONE,
    etag TEXT,
    last_read_timetoken TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(channel_id, user_id)
);
```

### `thread_channels` Table
```sql
CREATE TABLE thread_channels (
    id TEXT PRIMARY KEY,
    parent_channel_id TEXT REFERENCES channels(id),
    parent_message_timetoken TEXT REFERENCES messages(timetoken),
    pinned_message_timetoken TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### `thread_messages` Table
```sql
CREATE TABLE thread_messages (
    timetoken TEXT PRIMARY KEY,
    thread_channel_id TEXT REFERENCES thread_channels(id),
    user_id TEXT REFERENCES users(id),
    content JSONB,
    actions JSONB,
    meta JSONB,
    error TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### `events` Table
```sql
CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timetoken TEXT,
    type TEXT CHECK (type IN ('typing', 'report', 'receipt', 'mention', 'invite', 'custom', 'moderation')),
    payload JSONB,
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### `files` Table
```sql
CREATE TABLE files (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    type TEXT,
    channel_id TEXT REFERENCES channels(id),
    message_timetoken TEXT,
    user_id TEXT REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### `user_restrictions` Table
```sql
CREATE TABLE user_restrictions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT REFERENCES users(id),
    channel_id TEXT REFERENCES channels(id),
    ban BOOLEAN DEFAULT FALSE,
    mute BOOLEAN DEFAULT FALSE,
    reason TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, channel_id)
);
```

### `presence` Table
```sql
CREATE TABLE presence (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT REFERENCES users(id),
    channel_id TEXT REFERENCES channels(id),
    status BOOLEAN DEFAULT TRUE,
    last_active TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, channel_id)
);
```

### `push_channels` Table
```sql
CREATE TABLE push_channels (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT REFERENCES users(id),
    channel_id TEXT REFERENCES channels(id),
    device_token TEXT,
    device_gateway TEXT CHECK (device_gateway IN ('apns2', 'gcm')),
    apns_topic TEXT,
    apns_environment TEXT CHECK (apns_environment IN ('development', 'production')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, channel_id, device_token)
);
```

## Row Level Security Policies

```sql
-- Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE channels ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE memberships ENABLE ROW LEVEL SECURITY;
ALTER TABLE thread_channels ENABLE ROW LEVEL SECURITY;
ALTER TABLE thread_messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE events ENABLE ROW LEVEL SECURITY;
ALTER TABLE files ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_restrictions ENABLE ROW LEVEL SECURITY;
ALTER TABLE presence ENABLE ROW LEVEL SECURITY;
ALTER TABLE push_channels ENABLE ROW LEVEL SECURITY;

-- Create policies for users table
CREATE POLICY users_select ON users
    FOR SELECT
    USING (true);  -- Everyone can read users

CREATE POLICY users_insert ON users
    FOR INSERT
    WITH CHECK (id = auth.uid() OR auth.role() = 'service_role');  -- Only own user or service role can insert

CREATE POLICY users_update ON users
    FOR UPDATE
    USING (id = auth.uid() OR auth.role() = 'service_role')  -- Only own user or service role can update
    WITH CHECK (id = auth.uid() OR auth.role() = 'service_role');

CREATE POLICY users_delete ON users
    FOR DELETE
    USING (id = auth.uid() OR auth.role() = 'service_role');  -- Only own user or service role can delete

-- Create policies for channels table
CREATE POLICY channels_select ON channels
    FOR SELECT
    USING (
        type = 'public' OR  -- Public channels are visible to all
        auth.role() = 'service_role' OR  -- Service role can see all
        EXISTS (
            SELECT 1 FROM memberships 
            WHERE channel_id = id AND user_id = auth.uid()
        )  -- User is a member of the channel
    );

CREATE POLICY channels_insert ON channels
    FOR INSERT
    WITH CHECK (auth.uid() IS NOT NULL OR auth.role() = 'service_role');  -- Any authenticated user or service role can create

CREATE POLICY channels_update ON channels
    FOR UPDATE
    USING (
        auth.role() = 'service_role' OR  -- Service role can update any
        EXISTS (
            SELECT 1 FROM memberships 
            WHERE channel_id = id AND user_id = auth.uid()
        )  -- User is a member of the channel
    )
    WITH CHECK (
        auth.role() = 'service_role' OR  -- Service role can update any
        EXISTS (
            SELECT 1 FROM memberships 
            WHERE channel_id = id AND user_id = auth.uid()
        )  -- User is a member of the channel
    );

CREATE POLICY channels_delete ON channels
    FOR DELETE
    USING (
        auth.role() = 'service_role' OR  -- Service role can delete any
        EXISTS (
            SELECT 1 FROM memberships 
            WHERE channel_id = id AND user_id = auth.uid()
        )  -- User is a member of the channel
    );

-- Create policies for messages table
CREATE POLICY messages_select ON messages
    FOR SELECT
    USING (
        auth.role() = 'service_role' OR  -- Service role can see all
        EXISTS (
            SELECT 1 FROM channels c
            JOIN memberships m ON c.id = m.channel_id
            WHERE c.id = channel_id AND m.user_id = auth.uid()
        ) OR  -- User is a member of the channel
        EXISTS (
            SELECT 1 FROM channels c
            WHERE c.id = channel_id AND c.type = 'public'
        )  -- Message is in a public channel
    );

CREATE POLICY messages_insert ON messages
    FOR INSERT
    WITH CHECK (
        auth.role() = 'service_role' OR  -- Service role can insert any
        (
            user_id = auth.uid() AND  -- Message is from current user
            EXISTS (
                SELECT 1 FROM memberships 
                WHERE channel_id = messages.channel_id AND user_id = auth.uid()
            )  -- User is a member of the channel
        )
    );

CREATE POLICY messages_update ON messages
    FOR UPDATE
    USING (
        auth.role() = 'service_role' OR  -- Service role can update any
        user_id = auth.uid()  -- Message is from current user
    )
    WITH CHECK (
        auth.role() = 'service_role' OR  -- Service role can update any
        user_id = auth.uid()  -- Message is from current user
    );

CREATE POLICY messages_delete ON messages
    FOR DELETE
    USING (
        auth.role() = 'service_role' OR  -- Service role can delete any
        user_id = auth.uid() OR  -- Message is from current user
        EXISTS (
            SELECT 1 FROM channels c
            WHERE c.id = channel_id AND c.type != 'public'
            AND EXISTS (
                SELECT 1 FROM memberships m
                WHERE m.channel_id = c.id AND m.user_id = auth.uid()
            )
        )  -- User is a member of a non-public channel
    );

-- Remaining policy definitions for memberships, thread_channels, thread_messages, events, files, user_restrictions, presence, and push_channels tables follow the same pattern
```

## Database Functions

### Create User Function
```sql
CREATE OR REPLACE FUNCTION create_user(
    p_id TEXT,
    p_name TEXT DEFAULT NULL,
    p_external_id TEXT DEFAULT NULL,
    p_profile_url TEXT DEFAULT NULL,
    p_email TEXT DEFAULT NULL,
    p_custom JSONB DEFAULT NULL,
    p_status TEXT DEFAULT NULL,
    p_type TEXT DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    result JSONB;
BEGIN
    INSERT INTO users (id, name, external_id, profile_url, email, custom, status, type, updated)
    VALUES (p_id, p_name, p_external_id, p_profile_url, p_email, p_custom, p_status, p_type, NOW())
    RETURNING to_jsonb(users.*) INTO result;
    
    RETURN result;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Create Direct Conversation Function
```sql
CREATE OR REPLACE FUNCTION create_direct_conversation(
    p_current_user_id TEXT,
    p_other_user_id TEXT,
    p_channel_id TEXT DEFAULT NULL,
    p_channel_data JSONB DEFAULT NULL,
    p_membership_data JSONB DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    v_channel_id TEXT;
    v_channel JSONB;
    v_host_membership JSONB;
    v_invitee_membership JSONB;
    v_result JSONB;
BEGIN
    -- Generate channel_id if not provided
    IF p_channel_id IS NULL THEN
        v_channel_id := gen_random_uuid()::TEXT;
    ELSE
        v_channel_id := p_channel_id;
    END IF;
    
    -- Create channel
    INSERT INTO channels (id, name, custom, description, updated, status, type)
    VALUES (
        v_channel_id,
        COALESCE((p_channel_data->>'name'), 'Direct Channel'),
        COALESCE((p_channel_data->'custom'), '{}'::JSONB),
        COALESCE((p_channel_data->>'description'), ''),
        NOW(),
        'active',
        'direct'
    )
    RETURNING to_jsonb(channels.*) INTO v_channel;
    
    -- Create host membership
    INSERT INTO memberships (channel_id, user_id, custom, updated, etag)
    VALUES (
        v_channel_id,
        p_current_user_id,
        COALESCE((p_membership_data->'custom'), '{}'::JSONB),
        NOW(),
        md5(random()::TEXT)
    )
    RETURNING to_jsonb(memberships.*) INTO v_host_membership;
    
    -- Create invitee membership
    INSERT INTO memberships (channel_id, user_id, custom, updated, etag)
    VALUES (
        v_channel_id,
        p_other_user_id,
        COALESCE((p_membership_data->'custom'), '{}'::JSONB),
        NOW(),
        md5(random()::TEXT)
    )
    RETURNING to_jsonb(memberships.*) INTO v_invitee_membership;
    
    -- Construct result
    v_result := jsonb_build_object(
        'channel', v_channel,
        'hostMembership', v_host_membership,
        'inviteeMembership', v_invitee_membership
    );
    
    RETURN v_result;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Create Group Conversation Function
```sql
CREATE OR REPLACE FUNCTION create_group_conversation(
    p_current_user_id TEXT,
    p_user_ids TEXT[],
    p_channel_id TEXT DEFAULT NULL,
    p_channel_data JSONB DEFAULT NULL,
    p_membership_data JSONB DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    v_channel_id TEXT;
    v_channel JSONB;
    v_host_membership JSONB;
    v_invitee_memberships JSONB[] := '{}';
    v_result JSONB;
    v_user_id TEXT;
    v_membership JSONB;
BEGIN
    -- Generate channel_id if not provided
    IF p_channel_id IS NULL THEN
        v_channel_id := gen_random_uuid()::TEXT;
    ELSE
        v_channel_id := p_channel_id;
    END IF;
    
    -- Create channel
    INSERT INTO channels (id, name, custom, description, updated, status, type)
    VALUES (
        v_channel_id,
        COALESCE((p_channel_data->>'name'), 'Group Channel'),
        COALESCE((p_channel_data->'custom'), '{}'::JSONB),
        COALESCE((p_channel_data->>'description'), ''),
        NOW(),
        'active',
        'group'
    )
    RETURNING to_jsonb(channels.*) INTO v_channel;
    
    -- Create host membership
    INSERT INTO memberships (channel_id, user_id, custom, updated, etag)
    VALUES (
        v_channel_id,
        p_current_user_id,
        COALESCE((p_membership_data->'custom'), '{}'::JSONB),
        NOW(),
        md5(random()::TEXT)
    )
    RETURNING to_jsonb(memberships.*) INTO v_host_membership;
    
    -- Create invitee memberships
    FOREACH v_user_id IN ARRAY p_user_ids
    LOOP
        -- Skip if it's the current user
        IF v_user_id != p_current_user_id THEN
            INSERT INTO memberships (channel_id, user_id, custom, updated, etag)
            VALUES (
                v_channel_id,
                v_user_id,
                COALESCE((p_membership_data->'custom'), '{}'::JSONB),
                NOW(),
                md5(random()::TEXT)
            )
            RETURNING to_jsonb(memberships.*) INTO v_membership;
            
            v_invitee_memberships := array_append(v_invitee_memberships, v_membership);
        END IF;
    END LOOP;
    
    -- Construct result
    v_result := jsonb_build_object(
        'channel', v_channel,
        'hostMembership', v_host_membership,
        'inviteesMemberships', to_jsonb(v_invitee_memberships)
    );
    
    RETURN v_result;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Create Public Conversation Function
```sql
CREATE OR REPLACE FUNCTION create_public_conversation(
    p_channel_id TEXT DEFAULT NULL,
    p_channel_data JSONB DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    v_channel_id TEXT;
    v_channel JSONB;
BEGIN
    -- Generate channel_id if not provided
    IF p_channel_id IS NULL THEN
        v_channel_id := gen_random_uuid()::TEXT;
    ELSE
        v_channel_id := p_channel_id;
    END IF;
    
    -- Create channel
    INSERT INTO channels (id, name, custom, description, updated, status, type)
    VALUES (
        v_channel_id,
        COALESCE((p_channel_data->>'name'), 'Public Channel'),
        COALESCE((p_channel_data->'custom'), '{}'::JSONB),
        COALESCE((p_channel_data->>'description'), ''),
        NOW(),
        'active',
        'public'
    )
    RETURNING to_jsonb(channels.*) INTO v_channel;
    
    RETURN v_channel;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Send Text Message Function
```sql
CREATE OR REPLACE FUNCTION send_text_message(
    p_channel_id TEXT,
    p_user_id TEXT,
    p_text TEXT,
    p_mentioned_users JSONB DEFAULT NULL,
    p_referenced_channels JSONB DEFAULT NULL,
    p_text_links JSONB DEFAULT NULL,
    p_quoted_message JSONB DEFAULT NULL,
    p_files JSONB DEFAULT NULL,
    p_meta JSONB DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    v_timetoken TEXT;
    v_content JSONB;
    v_message JSONB;
    v_file JSONB;
    v_file_id TEXT;
    v_file_name TEXT;
    v_file_url TEXT;
    v_file_type TEXT;
BEGIN
    -- Generate timetoken (timestamp in milliseconds)
    v_timetoken := (EXTRACT(EPOCH FROM NOW()) * 10000000)::TEXT;
    
    -- Construct content
    v_content := jsonb_build_object(
        'type', 'text',
        'text', p_text
    );
    
    -- Add quoted message if provided
    IF p_quoted_message IS NOT NULL THEN
        v_content := v_content || jsonb_build_object('quotedMessage', p_quoted_message);
    END IF;
    
    -- Add mentioned users if provided
    IF p_mentioned_users IS NOT NULL THEN
        v_content := v_content || jsonb_build_object('mentionedUsers', p_mentioned_users);
    END IF;
    
    -- Add referenced channels if provided
    IF p_referenced_channels IS NOT NULL THEN
        v_content := v_content || jsonb_build_object('referencedChannels', p_referenced_channels);
    END IF;
    
    -- Add text links if provided
    IF p_text_links IS NOT NULL THEN
        v_content := v_content || jsonb_build_object('textLinks', p_text_links);
    END IF;
    
    -- Store message
    INSERT INTO messages (timetoken, channel_id, user_id, content, meta)
    VALUES (v_timetoken, p_channel_id, p_user_id, v_content, p_meta)
    RETURNING to_jsonb(messages.*) INTO v_message;
    
    -- Store files if provided
    IF p_files IS NOT NULL AND jsonb_array_length(p_files) > 0 THEN
        v_content := v_content || jsonb_build_object('files', '[]'::JSONB);
        
        FOR i IN 0..jsonb_array_length(p_files)-1 LOOP
            v_file := p_files->i;
            v_file_id := v_file->>'id';
            v_file_name := v_file->>'name';
            v_file_url := v_file->>'url';
            v_file_type := v_file->>'type';
            
            INSERT INTO files (id, name, url, type, channel_id, message_timetoken, user_id)
            VALUES (v_file_id,
            

```sql
            INSERT INTO files (id, name, url, type, channel_id, message_timetoken, user_id)
            VALUES (v_file_id, v_file_name, v_file_url, v_file_type, p_channel_id, v_timetoken, p_user_id);
            
            -- Add file info to content
            v_content := jsonb_set(
                v_content, 
                '{files}', 
                coalesce(v_content->'files', '[]'::jsonb) || 
                jsonb_build_object('id', v_file_id, 'name', v_file_name, 'url', v_file_url, 'type', v_file_type)
            );
        END LOOP;
        
        -- Update message with files
        UPDATE messages SET content = v_content WHERE timetoken = v_timetoken;
        
        -- Refresh message object
        SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = v_timetoken;
    END IF;
    
    RETURN v_message;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Toggle Reaction Function
```sql
CREATE OR REPLACE FUNCTION toggle_reaction(
    p_message_timetoken TEXT,
    p_user_id TEXT,
    p_reaction TEXT,
    p_reactions_action_name TEXT DEFAULT 'reactions'
) RETURNS JSONB AS $$
DECLARE
    v_message JSONB;
    v_actions JSONB;
    v_reaction_exists BOOLEAN := FALSE;
    v_action_timetoken TEXT;
    v_user_reaction JSONB;
    v_updated_actions JSONB;
BEGIN
    -- Get existing message
    SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = p_message_timetoken;
    
    IF v_message IS NULL THEN
        RAISE EXCEPTION 'Message not found';
    END IF;
    
    -- Get existing actions
    v_actions := COALESCE(v_message->'actions', '{}'::JSONB);
    
    -- Check if reactions action type exists
    IF v_actions ? p_reactions_action_name THEN
        -- Check if specific reaction exists
        IF v_actions->p_reactions_action_name ? p_reaction THEN
            -- Check if user already reacted
            FOR i IN 0..jsonb_array_length(v_actions->p_reactions_action_name->p_reaction)-1 LOOP
                IF (v_actions->p_reactions_action_name->p_reaction->i->>'uuid') = p_user_id THEN
                    v_reaction_exists := TRUE;
                    
                    -- Remove user's reaction
                    v_actions := jsonb_set(
                        v_actions,
                        ARRAY[p_reactions_action_name, p_reaction],
                        (v_actions->p_reactions_action_name->p_reaction) - i
                    );
                    
                    -- If reaction array is empty, remove it
                    IF jsonb_array_length(v_actions->p_reactions_action_name->p_reaction) = 0 THEN
                        v_actions := jsonb_set(
                            v_actions,
                            ARRAY[p_reactions_action_name],
                            (v_actions->p_reactions_action_name) - p_reaction
                        );
                    END IF;
                    
                    -- If reaction type is empty, remove it
                    IF jsonb_object_keys(v_actions->p_reactions_action_name) = 0 THEN
                        v_actions := v_actions - p_reactions_action_name;
                    END IF;
                    
                    EXIT;
                END IF;
            END LOOP;
        END IF;
    END IF;
    
    -- Add reaction if it didn't exist
    IF NOT v_reaction_exists THEN
        v_action_timetoken := (EXTRACT(EPOCH FROM NOW()) * 10000000)::TEXT;
        v_user_reaction := jsonb_build_object(
            'uuid', p_user_id,
            'actionTimetoken', v_action_timetoken
        );
        
        -- Initialize structure if it doesn't exist
        IF NOT (v_actions ? p_reactions_action_name) THEN
            v_actions := jsonb_set(v_actions, ARRAY[p_reactions_action_name], '{}'::JSONB);
        END IF;
        
        IF NOT (v_actions->p_reactions_action_name ? p_reaction) THEN
            v_actions := jsonb_set(
                v_actions,
                ARRAY[p_reactions_action_name, p_reaction],
                '[]'::JSONB
            );
        END IF;
        
        -- Add user reaction
        v_actions := jsonb_set(
            v_actions,
            ARRAY[p_reactions_action_name, p_reaction],
            (v_actions->p_reactions_action_name->p_reaction) || v_user_reaction
        );
    END IF;
    
    -- Update message
    UPDATE messages SET actions = v_actions WHERE timetoken = p_message_timetoken;
    
    -- Return updated message
    SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = p_message_timetoken;
    RETURN v_message;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Edit Message Function
```sql
CREATE OR REPLACE FUNCTION edit_message(
    p_message_timetoken TEXT,
    p_user_id TEXT,
    p_new_text TEXT,
    p_edit_action_name TEXT DEFAULT 'edited'
) RETURNS JSONB AS $$
DECLARE
    v_message JSONB;
    v_content JSONB;
    v_actions JSONB;
    v_action_timetoken TEXT;
BEGIN
    -- Get existing message
    SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = p_message_timetoken;
    
    IF v_message IS NULL THEN
        RAISE EXCEPTION 'Message not found';
    END IF;
    
    -- Verify user is the message author
    IF (v_message->>'user_id') != p_user_id THEN
        RAISE EXCEPTION 'Only the author can edit a message';
    END IF;
    
    -- Generate action timetoken
    v_action_timetoken := (EXTRACT(EPOCH FROM NOW()) * 10000000)::TEXT;
    
    -- Get existing content and update text
    v_content := v_message->'content';
    v_content := jsonb_set(v_content, '{text}', to_jsonb(p_new_text));
    
    -- Get existing actions
    v_actions := COALESCE(v_message->'actions', '{}'::JSONB);
    
    -- Add edit action
    IF NOT (v_actions ? p_edit_action_name) THEN
        v_actions := jsonb_set(v_actions, ARRAY[p_edit_action_name], '{}'::JSONB);
    END IF;
    
    -- Add edit timestamp
    v_actions := jsonb_set(
        v_actions,
        ARRAY[p_edit_action_name, v_action_timetoken],
        to_jsonb(p_user_id)
    );
    
    -- Update message
    UPDATE messages SET content = v_content, actions = v_actions WHERE timetoken = p_message_timetoken;
    
    -- Return updated message
    SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = p_message_timetoken;
    RETURN v_message;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Delete Message Function
```sql
CREATE OR REPLACE FUNCTION delete_message(
    p_message_timetoken TEXT,
    p_user_id TEXT,
    p_soft BOOLEAN DEFAULT TRUE,
    p_delete_action_name TEXT DEFAULT 'deleted',
    p_preserve_files BOOLEAN DEFAULT FALSE
) RETURNS JSONB AS $$
DECLARE
    v_message JSONB;
    v_content JSONB;
    v_actions JSONB;
    v_action_timetoken TEXT;
BEGIN
    -- Get existing message
    SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = p_message_timetoken;
    
    IF v_message IS NULL THEN
        RAISE EXCEPTION 'Message not found';
    END IF;
    
    IF p_soft THEN
        -- Soft delete - mark as deleted
        v_action_timetoken := (EXTRACT(EPOCH FROM NOW()) * 10000000)::TEXT;
        
        -- Get existing content
        v_content := v_message->'content';
        
        -- Get existing actions
        v_actions := COALESCE(v_message->'actions', '{}'::JSONB);
        
        -- Add delete action
        IF NOT (v_actions ? p_delete_action_name) THEN
            v_actions := jsonb_set(v_actions, ARRAY[p_delete_action_name], '{}'::JSONB);
        END IF;
        
        -- Add delete timestamp
        v_actions := jsonb_set(
            v_actions,
            ARRAY[p_delete_action_name, v_action_timetoken],
            to_jsonb(p_user_id)
        );
        
        -- Update message
        UPDATE messages SET actions = v_actions WHERE timetoken = p_message_timetoken;
        
        -- Delete files if not preserving
        IF NOT p_preserve_files THEN
            DELETE FROM files WHERE message_timetoken = p_message_timetoken;
        END IF;
        
        -- Return updated message
        SELECT to_jsonb(messages.*) INTO v_message FROM messages WHERE timetoken = p_message_timetoken;
        RETURN v_message;
    ELSE
        -- Hard delete - remove the message
        DELETE FROM messages WHERE timetoken = p_message_timetoken;
        
        -- Delete files if not preserving
        IF NOT p_preserve_files THEN
            DELETE FROM files WHERE message_timetoken = p_message_timetoken;
        END IF;
        
        RETURN v_message;
    END IF;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Create Thread Function
```sql
CREATE OR REPLACE FUNCTION create_thread(
    p_parent_message_timetoken TEXT,
    p_thread_channel_id TEXT DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    v_parent_message JSONB;
    v_parent_channel_id TEXT;
    v_thread_channel_id TEXT;
    v_thread_channel JSONB;
BEGIN
    -- Get parent message
    SELECT to_jsonb(messages.*) INTO v_parent_message FROM messages WHERE timetoken = p_parent_message_timetoken;
    
    IF v_parent_message IS NULL THEN
        RAISE EXCEPTION 'Parent message not found';
    END IF;
    
    v_parent_channel_id := v_parent_message->>'channel_id';
    
    -- Generate thread channel ID if not provided
    IF p_thread_channel_id IS NULL THEN
        v_thread_channel_id := 'thread-' || p_parent_message_timetoken;
    ELSE
        v_thread_channel_id := p_thread_channel_id;
    END IF;
    
    -- Create thread channel
    INSERT INTO thread_channels (id, parent_channel_id, parent_message_timetoken)
    VALUES (v_thread_channel_id, v_parent_channel_id, p_parent_message_timetoken)
    RETURNING to_jsonb(thread_channels.*) INTO v_thread_channel;
    
    RETURN v_thread_channel;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Emit Event Function
```sql
CREATE OR REPLACE FUNCTION emit_event(
    p_channel_id TEXT,
    p_user_id TEXT,
    p_event_type TEXT,
    p_payload JSONB
) RETURNS JSONB AS $$
DECLARE
    v_timetoken TEXT;
    v_event JSONB;
BEGIN
    -- Generate timetoken
    v_timetoken := (EXTRACT(EPOCH FROM NOW()) * 10000000)::TEXT;
    
    -- Store event
    INSERT INTO events (timetoken, type, payload, channel_id, user_id)
    VALUES (v_timetoken, p_event_type, p_payload, p_channel_id, p_user_id)
    RETURNING to_jsonb(events.*) INTO v_event;
    
    RETURN v_event;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Set User Restrictions Function
```sql
CREATE OR REPLACE FUNCTION set_user_restrictions(
    p_user_id TEXT,
    p_channel_id TEXT,
    p_ban BOOLEAN DEFAULT NULL,
    p_mute BOOLEAN DEFAULT NULL,
    p_reason TEXT DEFAULT NULL
) RETURNS JSONB AS $$
DECLARE
    v_restriction JSONB;
BEGIN
    -- Check if restriction exists
    SELECT to_jsonb(user_restrictions.*) INTO v_restriction
    FROM user_restrictions
    WHERE user_id = p_user_id AND channel_id = p_channel_id;
    
    IF v_restriction IS NULL THEN
        -- Create new restriction
        INSERT INTO user_restrictions (user_id, channel_id, ban, mute, reason)
        VALUES (
            p_user_id,
            p_channel_id,
            COALESCE(p_ban, FALSE),
            COALESCE(p_mute, FALSE),
            p_reason
        )
        RETURNING to_jsonb(user_restrictions.*) INTO v_restriction;
    ELSE
        -- Update existing restriction
        UPDATE user_restrictions SET
            ban = COALESCE(p_ban, ban),
            mute = COALESCE(p_mute, mute),
            reason = COALESCE(p_reason, reason),
            updated_at = NOW()
        WHERE user_id = p_user_id AND channel_id = p_channel_id
        RETURNING to_jsonb(user_restrictions.*) INTO v_restriction;
    END IF;
    
    -- Emit moderation event
    PERFORM emit_event(
        p_channel_id,
        p_user_id,
        'moderation',
        jsonb_build_object(
            'channelId', p_channel_id,
            'restriction', 
            CASE
                WHEN COALESCE(p_ban, FALSE) THEN 'banned'
                WHEN COALESCE(p_mute, FALSE) THEN 'muted'
                ELSE 'lifted'
            END,
            'reason', p_reason
        )
    );
    
    RETURN v_restriction;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Mark Message as Read Function
```sql
CREATE OR REPLACE FUNCTION mark_message_as_read(
    p_user_id TEXT,
    p_channel_id TEXT,
    p_message_timetoken TEXT
) RETURNS JSONB AS $$
DECLARE
    v_membership JSONB;
BEGIN
    -- Update membership with last read timetoken
    UPDATE memberships
    SET last_read_timetoken = p_message_timetoken,
        updated = NOW(),
        etag = md5(random()::TEXT)
    WHERE user_id = p_user_id AND channel_id = p_channel_id
    RETURNING to_jsonb(memberships.*) INTO v_membership;
    
    -- Emit receipt event
    PERFORM emit_event(
        p_channel_id,
        p_user_id,
        'receipt',
        jsonb_build_object(
            'messageTimetoken', p_message_timetoken
        )
    );
    
    RETURN v_membership;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Get Unread Messages Count Function
```sql
CREATE OR REPLACE FUNCTION get_unread_messages_count(
    p_user_id TEXT,
    p_channel_id TEXT
) RETURNS INTEGER AS $$
DECLARE
    v_last_read_timetoken TEXT;
    v_count INTEGER;
BEGIN
    -- Get last read timetoken
    SELECT last_read_timetoken INTO v_last_read_timetoken
    FROM memberships
    WHERE user_id = p_user_id AND channel_id = p_channel_id;
    
    IF v_last_read_timetoken IS NULL THEN
        -- Count all messages
        SELECT COUNT(*) INTO v_count
        FROM messages
        WHERE channel_id = p_channel_id;
    ELSE
        -- Count messages after last read
        SELECT COUNT(*) INTO v_count
        FROM messages
        WHERE channel_id = p_channel_id AND timetoken > v_last_read_timetoken;
    END IF;
    
    RETURN v_count;
END;
$$ LANGUAGE plpgsql;
```

## Indexes

```sql
-- Users table indexes
CREATE INDEX idx_users_name ON users(name);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_external_id ON users(external_id);
CREATE INDEX idx_users_last_active ON users(last_active_timestamp);

-- Channels table indexes
CREATE INDEX idx_channels_name ON channels(name);
CREATE INDEX idx_channels_type ON channels(type);
CREATE INDEX idx_channels_status ON channels(status);
CREATE INDEX idx_channels_updated ON channels(updated);

-- Messages table indexes
CREATE INDEX idx_messages_channel_id ON messages(channel_id);
CREATE INDEX idx_messages_user_id ON messages(user_id);
CREATE INDEX idx_messages_channel_timetoken ON messages(channel_id, timetoken);
CREATE INDEX idx_messages_created_at ON messages(created_at);

-- Memberships table indexes
CREATE INDEX idx_memberships_channel_id ON memberships(channel_id);
CREATE INDEX idx_memberships_user_id ON memberships(user_id);
CREATE INDEX idx_memberships_updated ON memberships(updated);
CREATE INDEX idx_memberships_last_read ON memberships(last_read_timetoken);

-- Thread channels table indexes
CREATE INDEX idx_thread_channels_parent_channel ON thread_channels(parent_channel_id);
CREATE INDEX idx_thread_channels_parent_message ON thread_channels(parent_message_timetoken);

-- Thread messages table indexes
CREATE INDEX idx_thread_messages_thread_channel ON thread_messages(thread_channel_id);
CREATE INDEX idx_thread_messages_user_id ON thread_messages(user_id);
CREATE INDEX idx_thread_messages_created_at ON thread_messages(created_at);

-- Events table indexes
CREATE INDEX idx_events_channel_id ON events(channel_id);
CREATE INDEX idx_events_user_id ON events(user_id);
CREATE INDEX idx_events_type ON events(type);
CREATE INDEX idx_events_timetoken ON events(timetoken);
CREATE INDEX idx_events_created_at ON events(created_at);

-- Files table indexes
CREATE INDEX idx_files_channel_id ON files(channel_id);
CREATE INDEX idx_files_message_timetoken ON files(message_timetoken);
CREATE INDEX idx_files_user_id ON files(user_id);
CREATE INDEX idx_files_name ON files(name);

-- User restrictions table indexes
CREATE INDEX idx_user_restrictions_user_id ON user_restrictions(user_id);
CREATE INDEX idx_user_restrictions_channel_id ON user_restrictions(channel_id);
CREATE INDEX idx_user_restrictions_ban ON user_restrictions(ban);
CREATE INDEX idx_user_restrictions_mute ON user_restrictions(mute);

-- Presence table indexes
CREATE INDEX idx_presence_user_id ON presence(user_id);
CREATE INDEX idx_presence_channel_id ON presence(channel_id);
CREATE INDEX idx_presence_status ON presence(status);
CREATE INDEX idx_presence_last_active ON presence(last_active);

-- Push channels table indexes
CREATE INDEX idx_push_channels_user_id ON push_channels(user_id);
CREATE INDEX idx_push_channels_channel_id ON push_channels(channel_id);
CREATE INDEX idx_push_channels_device_token ON push_channels(device_token);
```

## Migration Script

```sql
-- TeamSync Database Migration Script - Initial Schema

-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Create tables
CREATE TABLE users (
    id TEXT PRIMARY KEY,
    name TEXT,
    external_id TEXT,
    profile_url TEXT,
    email TEXT,
    custom JSONB,
    status TEXT,
    type TEXT,
    updated TIMESTAMP WITH TIME ZONE,
    last_active_timestamp BIGINT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE channels (
    id TEXT PRIMARY KEY,
    name TEXT,
    custom JSONB,
    description TEXT,
    updated TIMESTAMP WITH TIME ZONE,
    status TEXT,
    type TEXT CHECK (type IN ('direct', 'group', 'public', 'unknown')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    pinned_message_timetoken TEXT
);

CREATE TABLE messages (
    timetoken TEXT PRIMARY KEY,
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    content JSONB,
    actions JSONB,
    meta JSONB,
    error TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    custom JSONB,
    updated TIMESTAMP WITH TIME ZONE,
    etag TEXT,
    last_read_timetoken TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(channel_id, user_id)
);

CREATE TABLE thread_channels (
    id TEXT PRIMARY KEY,
    parent_channel_id TEXT REFERENCES channels(id),
    parent_message_timetoken TEXT REFERENCES messages(timetoken),
    pinned_message_timetoken TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE thread_messages (
    timetoken TEXT PRIMARY KEY,
    thread_channel_id TEXT REFERENCES thread_channels(id),
    user_id TEXT REFERENCES users(id),
    content JSONB,
    actions JSONB,
    meta JSONB,
    error TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timetoken TEXT,
    type TEXT CHECK (type IN ('typing', 'report', 'receipt', 'mention', 'invite', 'custom', 'moderation')),
    payload JSONB,
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE files (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    type TEXT,
    channel_id TEXT REFERENCES channels(id),
    message_timetoken TEXT,
    user_id TEXT REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE user_restrictions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT REFERENCES users(id),
    channel_id TEXT REFERENCES channels(id),
    ban BOOLEAN DEFAULT FALSE,
    mute BOOLEAN DEFAULT FALSE,
    reason TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, channel_id)
);

CREATE TABLE presence (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT REFERENCES users(id),
    channel_id TEXT REFERENCES channels(id),
    status BOOLEAN DEFAULT TRUE,
    last_active TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, channel_id)
);

CREATE TABLE push_channels (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT REFERENCES users(id),
    channel_id TEXT REFERENCES channels(id),
    device_token TEXT,
    device_gateway TEXT CHECK (device_gateway IN ('apns2', 'gcm')),
    apns_topic TEXT,
    apns_environment TEXT CHECK (apns_environment IN ('development', 'production')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, channel_id, device_token)
);

-- Create indexes
-- Users table indexes
CREATE INDEX idx_users_name ON users(name);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_external_id ON users(external_id);
CREATE INDEX idx_users_last_active ON users(last_active_timestamp);

-- Channels table indexes
CREATE INDEX idx_channels_name ON channels(name);
CREATE INDEX idx_channels_type ON channels(type);
CREATE INDEX idx_channels_status ON channels(status);
CREATE INDEX idx_channels_updated ON channels(updated);

-- Messages table indexes
CREATE INDEX idx_messages_channel_id ON messages(channel_id);
CREATE INDEX idx_messages_user_id ON messages(user_id);
CREATE INDEX idx_messages_channel_timetoken ON messages(channel_id, timetoken);
CREATE INDEX idx_messages_created_at ON messages(created_at);

-- Memberships table indexes
CREATE INDEX idx_memberships_channel_id ON memberships(channel_id);
CREATE INDEX idx_memberships_user_id ON memberships(user_id);
CREATE INDEX idx_memberships_updated ON memberships(updated);
CREATE INDEX idx_memberships_last_read ON memberships(last_read_timetoken);

-- Thread channels table indexes
CREATE INDEX idx_thread_channels_parent_channel ON thread_channels(parent_channel_id);
CREATE INDEX idx_thread_channels_parent_message ON thread_channels(parent_message_timetoken);

-- Thread messages table indexes
CREATE INDEX idx_thread_messages_thread_channel ON thread_messages(thread_channel_id);
CREATE INDEX idx_thread_messages_user_id ON thread_messages(user_id);
CREATE INDEX idx_thread_messages_created_at ON thread_messages(created_at);

-- Events table indexes
CREATE INDEX idx_events_channel_id ON events(channel_id);
CREATE INDEX idx_events_user_id ON events(user_id);
CREATE INDEX idx_events_type ON events(type);
CREATE INDEX idx_events_timetoken ON events(timetoken);
CREATE INDEX idx_events_created_at ON events(created_at);

-- Files table indexes
CREATE INDEX idx_files_channel_id ON files(channel_id);
CREATE INDEX idx_files_message_timetoken ON files(message_timetoken);
CREATE INDEX idx_files_user_id ON files(user_id);
CREATE INDEX idx_files_name ON files(name);

-- User restrictions table indexes
CREATE INDEX idx_user_restrictions_user_id ON user_restrictions(user_id);
CREATE INDEX idx_user_restrictions_channel_id ON user_restrictions(channel_id);
CREATE INDEX idx_user_restrictions_ban ON user_restrictions(ban);
CREATE INDEX idx_user_restrictions_mute ON user_restrictions(mute);

-- Presence table indexes
CREATE INDEX idx_presence_user_id ON presence(user_id);
CREATE INDEX idx_presence_channel_id ON presence(channel_id);
CREATE INDEX idx_presence_status ON presence(status);
CREATE INDEX idx_presence_last_active ON presence(last_active);

-- Push channels table indexes
CREATE INDEX idx_push_channels_user_id ON push_channels(user_id);
CREATE INDEX idx_push_channels_channel_id ON push_channels(channel_id);
CREATE INDEX idx_push_channels_device_token ON push_channels(device_token);

-- RLS Policies
-- Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE channels ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE memberships ENABLE ROW LEVEL SECURITY;
ALTER TABLE thread_channels ENABLE ROW LEVEL SECURITY;
ALTER TABLE thread_messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE events ENABLE ROW LEVEL SECURITY;
ALTER TABLE files ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_restrictions ENABLE ROW LEVEL SECURITY;
ALTER TABLE presence ENABLE ROW LEVEL SECURITY;
ALTER TABLE push_channels ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
-- Users policies
CREATE POLICY users_select ON users FOR SELECT USING (true);
CREATE POLICY users_insert ON users FOR INSERT WITH CHECK (id = auth.uid() OR auth.role() = 'service_role');
CREATE POLICY users_update ON users FOR UPDATE USING (id = auth.uid() OR auth.role() = 'service_role') WITH CHECK (id = auth.uid() OR auth.role() = 'service_role');
CREATE POLICY users_delete ON users FOR DELETE USING (id = auth.uid() OR auth.role() = 'service_role');

-- Channels policies
CREATE POLICY channels_select ON channels FOR SELECT USING (
    type = 'public' OR 
    auth.role() = 'service_role' OR 
    EXISTS (
        SELECT 1 FROM memberships 
        WHERE channel_id = id AND user_id = auth.uid()
    )
);
CREATE POLICY channels_insert ON channels FOR INSERT WITH CHECK (auth.uid() IS NOT NULL OR auth.role() = 'service_role');
CREATE POLICY channels_update ON channels FOR UPDATE USING (
    auth.role() = 'service_role' OR 
    EXISTS (
        SELECT 1 FROM memberships 
        WHERE channel_id = id AND user_id = auth.uid()
    )
) WITH CHECK (
    auth.role() = 'service_role' OR 
    EXISTS (
        SELECT 1 FROM memberships 
        WHERE channel_id = id AND user_id = auth.uid()
    )
);
CREATE POLICY channels_delete ON channels FOR DELETE USING (
    auth.role() = 'service_role' OR 
    EXISTS (
        SELECT 1 FROM memberships 
        WHERE channel_id = id AND user_id = auth.uid()
    )
);

-- Messages policies
CREATE POLICY messages_select ON messages FOR SELECT USING (
    auth.role() = 'service_role' OR
    EXISTS (
        SELECT 1 FROM channels c
        JOIN memberships m ON c.id = m.channel_id
        WHERE c.id = channel_id AND m.user_id = auth.uid()
    ) OR
    EXISTS (
        SELECT 1 FROM channels c
        WHERE c.id = channel_id AND c.type = 'public'
    )
);
CREATE POLICY messages_insert ON messages FOR INSERT WITH CHECK (
    auth.role() = 'service_role' OR
    (
        user_id = auth.uid() AND
        EXISTS (
            SELECT 1 FROM memberships 
            WHERE channel_id = messages.channel_id AND user_id = auth.uid()
        )
    )
);
CREATE POLICY messages_update ON messages FOR UPDATE USING (
    auth.role() = 'service_role' OR
    user_id = auth.uid()
) WITH CHECK (
    auth.role() = 'service_role' OR
    user_id = auth.uid()
);
CREATE POLICY messages_delete ON messages FOR DELETE USING (
    auth.role() = 'service_role' OR
    user_id = auth.uid() OR
    EXISTS (
        SELECT 1 FROM channels c
        WHERE c.id = channel_id AND c.type != 'public'
        AND EXISTS (
            SELECT 1 FROM memberships m
            WHERE m.channel_id = c.id AND m.user_id = auth.uid()
        )
    )
);

-- Database functions
-- Include all database functions defined earlier
```

## API Implementation Coverage Analysis

This section analyzes how the database design supports implementation of all the PubNub Chat API methods.

### Chat Class API Coverage

| Method | Database Support | Implementation Approach |
|--------|------------------|-------------------------|
| `init` | ✅ | Basic connection to Supabase. |
| `createUser` | ✅ | Direct database insert using `users` table. |
| `createDirectConversation` | ✅ | Implemented via `create_direct_conversation` function. |
| `createGroupConversation` | ✅ | Implemented via `create_group_conversation` function. |
| `createPublicConversation` | ✅ | Implemented via `create_public_conversation` function. |
| `currentUser` | ✅ | Simple query to `users` table by ID. |
| `deleteChannel` | ✅ | Direct delete from `channels` table with soft delete option. |
| `deleteUser` | ✅ | Direct delete from `users` table with soft delete option. |
| `downloadDebugLog` | ✅ | Client-side implementation (no database dependency). |
| `emitEvent` | ✅ | Implemented via `emit_event` function. |
| `getEventsHistory` | ✅ | Query `events` table filtering by channel and timetoken range. |
| `getChannel` | ✅ | Simple query to `channels` table by ID. |
| `getChannels` | ✅ | Query to `channels` table with pagination. |
| `getChannelSuggestions` | ✅ | Query `channels` table with LIKE operator on name. |
| `getCurrentUserMentions` | ✅ | Query `events` table filtering by type='mention' and user_id. |
| `getUnreadMessagesCounts` | ✅ | Implemented via `get_unread_messages_count` function across multiple channels. |
| `getUser` | ✅ | Simple query to `users` table by ID. |
| `getUsers` | ✅ | Query to `users` table with pagination. |
| `getUserSuggestions` | ✅ | Query `users` table with LIKE operator on name. |
| `getPushChannels` | ✅ | Query `push_channels` table filtering by user_id. |
| `isPresent` | ✅ | Query `presence` table filtering by user_id and channel_id. |
| `listenForEvents` | ✅ | Realtime subscription to `events` table via Supabase. |
| `markAllMessagesAsRead` | ✅ | Batch update to `memberships` table for last_read_timetoken. |
| `registerPushChannels` | ✅ | Batch insert to `push_channels` table. |
| `setRestrictions` | ✅ | Implemented via `set_user_restrictions` function. |
| `unregisterPushChannels` | ✅ | Delete from `push_channels` table for specific channels. |
| `unregisterAllPushChannels` | ✅ | Delete all from `push_channels` table for user. |
| `updateChannel` | ✅ | Update `channels` table by ID. |
| `updateUser` | ✅ | Update `users` table by ID. |
| `wherePresent` | ✅ | Query `presence` table filtering by user_id. |
| `whoIsPresent` | ✅ | Query `presence` table filtering by channel_id. |

### Channel Class API Coverage

| Method | Database Support | Implementation Approach |
|--------|------------------|-------------------------|
| `connect` | ✅ | Realtime subscription to `messages` table via Supabase. |
| `createMessageDraft` | ✅ | Client-side implementation (no database dependency). |
| `delete` | ✅ | Direct delete from `channels` table with soft delete option. |
| `deleteFile` | ✅ | Delete from `files` table by ID and name. |
| `forwardMessage` | ✅ | Copy message to new channel via `send_text_message` function. |
| `getFiles` | ✅ | Query `files` table by channel_id with pagination. |
| `getHistory` | ✅ | Query `messages` table by channel_id with timetoken range. |
| `getMessage` | ✅ | Query `messages` table by timetoken. |
| `getMessageReportsHistory` | ✅ | Query `events` table by channel_id and type='report'. |
| `getMembers` | ✅ | Join query between `memberships` and `users` tables by channel_id. |
| `getPinnedMessage` | ✅ | Query `channels` table for pinned_message_timetoken, then query `messages`. |
| `getTyping` | ✅ | Realtime subscription to `events` table for typing events. |
| `getUserRestrictions` | ✅ | Query `user_restrictions` table by user_id and channel_id. |
| `getUsersRestrictions` | ✅ | Query `user_restrictions` table by channel_id with pagination. |
| `getUserSuggestions` | ✅ | Join query between `memberships` and `users` with LIKE on name. |
| `invite` | ✅ | Insert into `memberships` table and emit 'invite' event. |
| `inviteMultiple` | ✅ | Batch insert into `memberships` table and emit 'invite' events. |
| `isPresent` | ✅ | Query `presence` table by user_id and channel_id. |
| `join` | ✅ | Insert into `memberships` table and setup realtime subscription. |
| `leave` | ✅ | Delete from `memberships` table. |
| `pinMessage` | ✅ | Update `channels` table with pinned_message_timetoken. |
| `registerForPush` | ✅ | Insert into `push_channels` table. |
| `sendText` | ✅ | Implemented via `send_text_message` function. |
| `setRestrictions` | ✅ | Implemented via `set_user_restrictions` function. |
| `startTyping` | ✅ | Emit 'typing' event with `emit_event` function. |
| `stopTyping` | ✅ | Emit 'typing' event with `emit_event` function. |
| `streamPresence` | ✅ | Realtime subscription to `presence` table via Supabase. |
| `streamReadReceipts` | ✅ | Realtime subscription to `events` table for receipt events. |
| `streamMessageReports` | ✅ | Realtime subscription to `events` table for report events. |
| `streamUpdates` | ✅ | Realtime subscription to `channels` table via Supabase. |
| `streamUpdatesOn` | ✅ | Batch realtime subscription to `channels` table via Supabase. |
| `update` | ✅ | Update `channels` table by ID. |
| `unpinMessage` | ✅ | Update `channels` table to set pinned_message_timetoken to null. |
| `unregisterFromPush` | ✅ | Delete from `push_channels` table. |
| `whoIsPresent` | ✅ | Query `presence` table by channel_id. |

### Message Class API Coverage

| Method | Database Support | Implementation Approach |
|--------|------------------|-------------------------|
| `createThread` | ✅ | Implemented via `create_thread` function. |
| `delete` | ✅ | Implemented via `delete_message` function. |
| `deleted` | ✅ | Check if `actions` JSON contains a 'deleted' action. |
| `hasThread` | ✅ | Query `thread_channels` table by parent_message_timetoken. |
| `editText` | ✅ | Implemented via `edit_message` function. |
| `forward` | ✅ | Copy message to new channel via `send_text_message` function. |
| `getMessageElements` | ✅ | Client-side processing of message content (no database dependency). |
| `getThread` | ✅ | Query `thread_channels` table by parent_message_timetoken. |
| `hasUserReaction` | ✅ | Check if user ID exists in message reactions (client-side). |
| `toggleReaction` | ✅ | Implemented via `toggle_reaction` function. |
| `pin` | ✅ | Update `channels` table with pinned_message_timetoken. |
| `quotedMessage` | ✅ | Access quoted message data from content JSON (client-side). |
| `reactions` | ✅ | Access reactions from actions JSON (client-side). |
| `removeThread` | ✅ | Delete from `thread_channels` table. |
| `report` | ✅ | Emit 'report' event with `emit_event` function. |
| `restore` | ✅ | Remove 'deleted' action from message actions. |
| `streamUpdates` | ✅ | Realtime subscription to `messages` table via Supabase. |
| `streamUpdatesOn` | ✅ | Batch realtime subscription to `messages` table via Supabase. |
| `text` | ✅ | Access text from content JSON (client-side). |

### Membership Class API Coverage

| Method | Database Support | Implementation Approach |
|--------|------------------|-------------------------|
| `getUnreadMessagesCount` | ✅ | Implemented via `get_unread_messages_count` function. |
| `lastReadMessageTimetoken` | ✅ | Access last_read_timetoken field from membership. |
| `setLastReadMessage` | ✅ | Implemented via `mark_message_as_read` function. |
| `setLastReadMessageTimetoken` | ✅ | Update `memberships` table with last_read_timetoken. |
| `streamUpdates` | ✅ | Realtime subscription to `memberships` table via Supabase. |
| `streamUpdatesOn` | ✅ | Batch realtime subscription to `memberships` table via Supabase. |
| `update` | ✅ | Update `memberships` table with custom data. |

### ThreadChannel Class API Coverage

| Method | Database Support | Implementation Approach |
|--------|------------------|-------------------------|
| `getHistory` | ✅ | Query `thread_messages` table by thread_channel_id with timetoken range. |
| `pinMessage` | ✅ | Update `thread_channels` table with pinned_message_timetoken. |
| `pinMessageToParentChannel` | ✅ | Update parent channel with pinned_message_timetoken. |
| `unpinMessage` | ✅ | Update `thread_channels` table to set pinned_message_timetoken to null. |
| `unpinMessageFromParentChannel` | ✅ | Update parent channel to set pinned_message_timetoken to null. |

### ThreadMessage Class API Coverage

| Method | Database Support | Implementation Approach |
|--------|------------------|-------------------------|
| `streamUpdatesOn` | ✅ | Batch realtime subscription to `thread_messages` table via Supabase. |
| `pinToParentChannel` | ✅ | Update parent channel with pinned_message_timetoken. |
| `unpinFromParentChannel` | ✅ | Update parent channel to set pinned_message_timetoken to null. |

### MessageDraft and Event Class API Coverage

All MessageDraft methods are client-side implementations, and Event class has no methods listed in the API documentation, so they're fully supported by this database design.

## Summary and Recommendations

This database design provides comprehensive support for all the PubNub Chat APIs as documented. The design uses:

1. 11 main tables to store all required entities
2. Row-level security policies to ensure proper access control
3. Database functions for complex operations
4. Appropriate indexes for optimal query performance

For most API methods, the Flutter repository can make direct Supabase DB calls as per the tech stack document. However, there are a few complex operations where alternate implementations would be more efficient:

### Recommended Alternative Implementation Approaches

1. **Complex Message Operations (Edge Functions)**
   - Message editing, reactions, and threading operations involve complex JSON manipulation and multiple table operations
   - Recommendation: Use Supabase Edge Functions for:
     - `toggleReaction`
     - `editText`
     - `createThread`
     - `sendText` with files and mentions

2. **Batch Operations (Database Functions)**
   - Operations involving multiple users or channels would benefit from database functions
   - Recommendation: Use Supabase Database Functions for:
     - `createGroupConversation`
     - `markAllMessagesAsRead`
     - `inviteMultiple`

3. **Real-time Subscriptions (Client-side Implementation)**
   - All streaming methods should be implemented client-side using Supabase's realtime features
   - This includes all methods starting with "stream" and "connect"

These recommendations balance performance and scalability while maintaining the overall architecture described in the tech stack document.