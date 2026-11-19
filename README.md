 Message Manager — PROG5121 Part 3

Part 3 of the PROG5121 Java chat app capstone. It picks up the JSON store written in Part 2, loads it into the message arrays the brief names, and adds the **Stored Messages** menu with search, longest-message, delete-by-hash and a sent-message report. JUnit 5 tests cover each requirement against the brief's fixed test data.

## Features

**Menu option 4 — Stored Messages**

- **a.** Show the sender and recipient of every message held
- **b.** Show the longest message, measured across *all* messages and flags
- **c.** Search by message ID and return that message
- **d.** Show every message sent to a given recipient
- **e.** Delete a message by its hash and confirm the removal

**Menu option 5 — Display report**

Lists every sent message with its hash, recipient, text and a length verdict.

## How it works

`MessageManager` declares the five arrays from the brief — `sentMessages`, `disregardedMessages`, `stMessages`, `msgHashes`, `msgIds` — and fills them only from the JSON file. Nothing is hard-coded; `loadTestData()` exists purely as the test hook that seeds the brief's five messages.

`readStoredData(Path)` walks the file one line at a time, reads the `id`, `number`, `recipient`, `message`, `hash` and `msgFlag` fields (`SENTREAD` / `STORE` / `DISREGARD`) and files each message into the array its flag selects. Stored hashes from the file are reused; only a missing one is computed fresh.

Delete works off `msgHashes`, finds the matching message, removes it from the arrays and the object list, and returns the full confirmation block. The longest-message scan reads every message, which is why the disregarded `"Hi dinner time."` beats the stored text.

## Requirements

- JDK 11 or newer (the code uses `Path`, `List.remove` and `switch` on strings — no preview features)
- JUnit 5 on the test classpath (`org.junit.jupiter` via Maven/Gradle, or the standalone jar in `lib/`)

## Running

```bash
javac -d build -cp "lib/*" src/com/mycompany/chatapp/*.java
java -cp "build" com.mycompany.chatapp.Main
```

The JSON file must exist before the Stored Messages option can do anything — it is the artifact Part 2 writes. A sample at `data/storedMessages.json`:

```json
[
  { "id": "1234567890", "number": 0, "recipient": "+27834557896", "message": "Did you get the cake?", "hash": ":0:THECAK", "msgFlag": "SENTREAD" },
  { "id": "1234567891", "number": 1, "recipient": "+27838884567", "message": "Where are you? You are late! I have asked you to be on time.", "hash": ":1:WHERED", "msgFlag": "STORE" }
]
```

## Tests

```bash
mvn test
```

`MessageManagerTest` seeds messages with `loadTestData()` — which assigns the fixed hashes `:0:HASH0` to `:4:HASH4` so the delete assertion is deterministic — then checks:

| Test | Asserts |
| --- | --- |
| `sentMessagesArrayCorrectlyPopulated` | Message 1 is `"Did you get the cake?"` |
| `displayLongestMessage` | Returns `"Hi dinner time."` |
| `searchMessageID` | ID `1234567892` returns `"It is dinner time !"` |
| `searchAll` | Recipient `+27838884567` returns all three of their messages |
| `deleteMessageByHash` | Hash `:1:HASH1` returns the full deleted-message block |
| `displayReport` | Report contains the sent messages' hash, recipient and text |

`SelfTest.java` is a no-JUnit harness for a quick manual run; it writes its own fixture under `data/` and prints PASS/FAIL per check.

## Message flags

| `msgFlag` | Array | Meaning |
| --- | --- | --- |
| `SENTREAD` | `sentMessages` | Sent, appears in the report |
| `STORE` | `stMessages` | Stored for later |
| `DISREGARD` | `disregardedMessages` | Kept but not acted on |

Any flag other than the first two is treated as stored.

## Notes and limits

- Written against the module's Part 1/Part 2 names: `Message` fields `messageID`, `messageNumber`, `recipient`, `messageText`, `messageHash`, and methods `getMessageText()`, `getRecipient()`, `getMessageHash()`, `hashMessage()`, `checkRecipientCell()`, `checkMessageLength()`, `checkMessageID()`. A different Part 2 means a find-and-replace across `MessageManager`.
- The hashes in `loadTestData()` are fixed tokens, not real hash-function output. If you assert on real hashes, swap in computed values from `hashMessage()`.
- The JSON reader is line-based rather than a full parser: it handles the flat array Part 2 produces and would need rework for nested objects.
- `hashMessage` in `Message.java` here is a placeholder shape (first two ID digits + number + ID digits 2–4 + word count, capped at 10). Drop in your Part 1 implementation if it differs.
- No persistence write-back yet — deletes apply to memory only.
