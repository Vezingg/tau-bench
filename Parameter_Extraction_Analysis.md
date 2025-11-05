# Parameter Extraction Issue Analysis

## Overview

During testing of airline and retail workflows, Observed issues with smaller language models in the parameter extraction component.

## Test Environment

- **Workflows Tested**: Airline Workflow, Retail Workflow
- **Models Evaluated**: 
  - 24B parameters (Mistral Small)
  - 20B parameters (GPT OSS 20B)
  - 14B parameters (Qwen3 14B)
  - 7B parameters (Mistral 7B)

## Results Summary

| Model Size | Parameter Extraction Performance |
|------------|----------------------------------|
| 24B - 20B  | Minimal errors observed |
| 14B        | Some parameter extraction errors |
| 7B         | Significant parameter extraction failures |

## Issue Details

**Note**: Parameter extraction errors occurred across ALL workflow commands. The examples below are representative samples of the error patterns observed system-wide.

### Error Pattern 1: Missing Parameters

**Example Command**: `cancel_pending_order`

```
PARAMETER EXTRACTION ERROR FOR COMMAND 'cancel_pending_order'
Missing parameter values: order_id, reason
use the get_user_details command(s) to get order_id information. OR...

Provide corrected parameter values in the exact order specified below, separated by commas:
order_id, reason
Check your command name if the wrong command was executed.
```

### Error Pattern 2: Invalid Parameter Format

**Example Command**: `get_order_details`

```
PARAMETER EXTRACTION ERROR FOR COMMAND 'get_order_details'
Invalid parameter values: order_id '<order_id>', order_id '<order_id>'

order_id: Please use the format matching pattern ^(#[\w\d]+|NOT_FOUND)$ (e.g., #W0000000)
Provide corrected parameter values in the exact order specified below, separated by commas:
order_id, order_id
Check your command name if the wrong command was executed.
```

### Error Pattern 3: Persistent Retry Failures

The agent exhibits repetitive error behavior, repeatedly attempting the same malformed command across multiple command types but couldn't recover (abort) from the issue:

```json
{
  "name": "get_order_details",
  "kwargs": {},
  "response_text": "PARAMETER EXTRACTION ERROR FOR COMMAND 'get_order_details'\nInvalid parameter values: order_id ':  <param_order_id>', order_id ':  <param_order_id>'\n\norder_id: Please use the format matching pattern ^(#[\\w\\d]+|NOT_FOUND)$ (e.g., #W0000000)\nProvide corrected parameter values in the exact order specified below, separated by commas:\norder_id, order_id\nCheck your command name if the wrong command was executed.",
  "success": false
},
{
  "name": "ErrorCorrection/abort",
  "kwargs": {},
  "response_text": "command aborted\n",
  "success": true
},
{
  "name": "get_order_details",
  "kwargs": {},
  "response_text": "PARAMETER EXTRACTION ERROR FOR COMMAND 'get_order_details'\nInvalid parameter values: order_id ':  <param_order_id>', order_id ':  <param_order_id>'\n\norder_id: Please use the format matching pattern ^(#[\\w\\d]+|NOT_FOUND)$ (e.g., #W0000000)\nProvide corrected parameter values in the exact order specified below, separated by commas:\norder_id, order_id\nCheck your command name if the wrong command was executed.",
  "success": false
},
{
  "name": "ErrorCorrection/abort",
  "kwargs": {},
  "response_text": "command aborted\n",
  "success": true
},
```

## Key Observations

1. **Model Size Correlation**: Parameter extraction errors increase inversely with model parameter count
2. **Error Types**: 
   - Missing required parameters
   - Invalid parameter formatting
   - Inability to parse parameter patterns correctly
3. **Recovery Behavior**: Smaller models struggle to recover from parameter extraction errors, often repeating the same mistakes