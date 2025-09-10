# Fix for Modified Files Display Issue

## Problem
Your new mynah-ui component for displaying modified files works in the UI but doesn't work with the language server integration.

## Root Cause
The language server already tracks modified files through the `fileList` property in `ChatMessage`, but your component might be expecting the data in a different format or location.

## Solution

### 1. Check Your Component's Data Source
Make sure your mynah-ui component is reading from the correct property. The language server populates modified files in:

```typescript
// In ChatMessage interface
interface ChatMessage {
  fileList?: FileList;
  // ... other properties
}

// FileList structure
interface FileList {
  rootFolderTitle?: string;
  filePaths?: string[];
  deletedFiles?: string[];
  details?: Record<string, FileDetails>;
}
```

### 2. Verify the Language Server is Sending Data
The language server already populates `fileList` in these scenarios:
- File write operations (`fsWrite`, `fsReplace`)
- File read operations (`fsRead`)
- Directory listings (`listDirectory`)
- Search operations (`grepSearch`, `fileSearch`)

### 3. Debug Steps
1. Check if your component is looking for `fileList` in the message object
2. Verify the message structure being passed to your component
3. Ensure the component handles the `FileList` interface correctly

### 4. Example Integration
If your component expects a different format, you might need to transform the data:

```typescript
// In your mynah-ui component
const getModifiedFiles = (message: ChatMessage) => {
  if (message.fileList?.filePaths) {
    return message.fileList.filePaths.map(filePath => ({
      path: filePath,
      details: message.fileList?.details?.[filePath],
      // Transform to your expected format
    }));
  }
  return [];
};
```

## Files to Check
- `agenticChatResultStream.ts` - Handles merging fileList data
- `agenticChatController.ts` - Populates fileList for tool operations
- Your mynah-ui component - Ensure it reads from the correct property

## Next Steps
1. Verify your component is accessing `message.fileList`
2. Check the console for any data structure mismatches
3. Test with a simple file write operation to see if data flows correctly