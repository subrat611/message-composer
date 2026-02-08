# Message Composer - Design Doc

## Functional Requirements

- Rich text (markdown support)
  - `*bold*` it should instantly convert to bold text.
  - `>` it should become blockquote.
  - `"``` ```"` it should instandly convert to code block.
- Emoji Picker
  - user can add emojis to the rich text block/area.
  - it have a emoji picker.
- Mentions: when user write `@` a popover will shown that consist list of users within the channel.
- Draft persistence: the data should be persistence if user refreshes the tab or close the browser. the unset message should be restored.
- user can upload/attach files and images.
- Composer support max message size of X character.

**Important**

- Paste content to composer (this is our critical part because based on this will look how our data model look like)
  - as part of current development it will only support pasting of the bold, italic, blockquote, code block, mention and html string. (no converage for pasting excel or google doc complex things).

## Non-functional Requirements

- Accessibility (A11Y): the composer support keyboard navigation and screen readers (ARIA labels).
- Security (XSS): content should be sanitized to prevent cross-site scripting attack.
- Optimistic UI: when user click on send button, the content should appear in the chat immediatly.
- On writing within the composer, it should not block the main thread, it should be performant.

<br />

<details>
<summary><b>Optimistic UI</b></summary>

## What is Optimistic UI?

Optimistic UI is a frontend design pattern where the interface updates immediately after a user action, acting as if the server has already responded successfully, instead of waiting for the actual server response.

### How It Works (The "Act First, Verify Later" Model)

- Trigger: The user performs an action (e.g., clicks "Send" on a message).
- Optimistic Update: The UI immediately adds that message to the chat list. usually with a "pending" visual state (like slightly greyed out opacity).
- Background Request: The browser sends the API request to the server.
- Reconciliation:
  - Success (Happy Path): The server returns 200 OK. The UI removes the "pending" style. The user barely notices this step.
  - Failure (Sad Path): The server returns an error (or times out). The UI must roll back the change (remove the message) and alert the user (e.g., "Message failed to send. Retry?").

### Example: The "Like" Button (Instagram/Twitter)

Without Optimistic UI, clicking "Like" would feel sluggish:

`Click -> Wait 200ms -> Server says OK -> Heart turns red.`

With Optimistic UI:

`Click -> Heart turns red instantly -> Request sent in background.`

### Key Challenges

- State Management (Rollback):
  How do you undo the change if it fails? You need to keep a snapshot of the previous state before the mutation occurred. Libraries like TanStack Query (React Query) or Apollo Client have built-in support for onMutate (optimistic update) and onError (rollback).

- Race Conditions:
  If a user clicks "Like" then immediately "Unlike" before the first request finishes, you must ensure the final UI state matches the final server state.

- ID Generation:
  If you add a message to a list optimistically, it doesn't have a database ID yet. You typically assign it a temporary UUID (e.g., temp-id-123) so React can render it with a key. When the server responds with the real ID, you swap them.

</details>
