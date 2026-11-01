# Live ACP task details verification

The adapter build on `datons-next` was run through a minimal ACP client against Claude Code with `asyncTasks` and all three `asyncTaskDetails` capabilities enabled. This trace contains only the relevant JSON-RPC fields; session IDs, request IDs, and output paths are normalized. No secrets or full prompt text are included.

- Monitor: `bq3qdaxzw` was announced with `taskType: "monitor"` and `canStop: true`; its output returned `monitor started`, and the stop request returned `stopped: true` followed by a stopped state.
- Background shell: `bacbejpwx` was announced with `taskType: "shell"`; the output request returned `acp-shell-output`.
- Schedule: `CronCreate` produced schedule `839a9958`, `*/5 * * * *`, recurring, with prompt `Say acp-cron-fired`. The first empty schedule list and the later populated list were both sent.

## Relevant JSON-RPC trace

```jsonl
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"async_task_spawned","asyncTaskId":"bq3qdaxzw","name":"harmless echo then sleep","taskType":"monitor","description":"harmless echo then sleep","showInTranscript":false,"canStop":true,"toolCallId":"toolu_01FgRiKWGTXVAYLovDSE27r6"}}}
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"session_schedules_update","schedules":[]}}}
{"jsonrpc":"2.0","id":"<request-1>","method":"_session/async_task/output","params":{"sessionId":"<session>","asyncTaskId":"bq3qdaxzw"}}
{"jsonrpc":"2.0","id":"<request-1>","result":{"output":"monitor started\n"}}
{"jsonrpc":"2.0","id":"<request-2>","method":"_session/async_task/stop","params":{"sessionId":"<session>","asyncTaskId":"bq3qdaxzw"}}
{"jsonrpc":"2.0","id":"<request-2>","result":{"stopped":true}}
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"async_task_state_update","asyncTaskId":"bq3qdaxzw","state":"stopped"}}}
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"async_task_state_update","asyncTaskId":"bq3qdaxzw","state":"stopped","outputFilePath":"<output-file>"}}}
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"async_task_spawned","asyncTaskId":"bacbejpwx","name":"Print a line then wait 30 seconds","taskType":"shell","description":"Print a line then wait 30 seconds","showInTranscript":false,"canStop":true,"toolCallId":"toolu_01UaaXoSM9j5Tksb3ytUaWwR"}}}
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"async_task_progress","asyncTaskId":"bacbejpwx","outputFilePath":"<output-file>"}}}
{"jsonrpc":"2.0","id":"<request-3>","method":"_session/async_task/output","params":{"sessionId":"<session>","asyncTaskId":"bacbejpwx"}}
{"jsonrpc":"2.0","id":"<request-3>","result":{"output":"acp-shell-output\n"}}
{"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<session>","update":{"sessionUpdate":"session_schedules_update","schedules":[{"id":"839a9958","schedule":"*/5 * * * *","recurring":true,"prompt":"Say acp-cron-fired"}]}}}
```

## Known SDK boundary

SDK 0.3.312 exposes `session_crons` on the `Stop` and `SubagentStop` hook inputs, but not on `SessionStart` or a public query method. A newly loaded session with no adapter snapshot therefore reports schedules at its next `Stop` hook; a known live snapshot is resent after load or resume. The same SDK declares `get_task_output` on the control wire but has no public `Query` method for it, so this adapter calls the existing control through the query instance.
