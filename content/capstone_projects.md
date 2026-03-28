# Capstone Projects: Modern Java Mastery

## Overview

These 5 capstone projects are designed to test your understanding of modern Java features. Each project builds on the concepts you've learned throughout the course, from basic syntax to advanced concurrency.

**Important Rules:**
- ✅ Use **only** Java Standard Library (no frameworks like Spring, Hibernate, etc.)
- ✅ Apply modern Java features (Records, Sealed Classes, Pattern Matching, Virtual Threads, etc.)
- ✅ Write clean, production-quality code
- ✅ Include proper error handling
- ❌ No AI-generated code - write it yourself to learn!

---

## Project 1: Time-Zone Meeting Scheduler (CLI)

### Difficulty: ⭐⭐☆☆☆ (Beginner-Intermediate)

### Concepts Applied
- Module Imports (`import module java.base`)
- Records for data modeling
- `java.time` API (`ZonedDateTime`, `ZoneId`, `Duration`)
- Implicit I/O (`println()`, `readln()`)
- Text Blocks
- Optional for null safety
- Collections (`ArrayList`, `HashMap`)

### Project Description

Build a command-line tool that helps teams schedule meetings across different time zones. The tool should handle:
- Multiple participants in different time zones
- Finding common available time slots
- Automatic Daylight Saving Time (DST) adjustments
- Meeting duration calculation
- Conflict detection

### Requirements

#### 1. Data Models

Create Records for:
- **Participant**: `record Participant(String name, ZoneId timezone)`
- **TimeSlot**: `record TimeSlot(ZonedDateTime start, ZonedDateTime end)`
- **Meeting**: `record Meeting(String title, List<Participant> participants, TimeSlot slot)`

#### 2. Core Features

**Feature A: Add Participants**
- Prompt user to add participants with their time zones
- Validate time zone input using `ZoneId.of()`
- Store participants in a collection
- Handle invalid time zone gracefully

**Feature B: Propose Meeting Time**
- Ask for meeting time in one participant's time zone
- Convert to all other participants' time zones
- Display the time for each participant clearly

Example output:
```
Meeting: Project Review (2 hours)
- Alice (America/New_York): Monday, Jan 15, 2026 at 2:00 PM EST
- Bob (Europe/London): Monday, Jan 15, 2026 at 7:00 PM GMT
- Chen (Asia/Tokyo): Tuesday, Jan 16, 2026 at 4:00 AM JST
```

**Feature C: Find Common Time Slots**
- Accept working hours for each participant (e.g., 9 AM - 5 PM)
- Find time slots where ALL participants are available
- Consider time zone differences
- Account for DST transitions (if crossing DST boundary)

**Feature D: Schedule Meeting**
- Store scheduled meetings
- Check for conflicts with existing meetings
- Display confirmation with times in all time zones

**Feature E: List Scheduled Meetings**
- Show all upcoming meetings
- Sort by date/time
- Display times in each participant's local zone

#### 3. CLI Menu Structure

```
=== Time-Zone Meeting Scheduler ===
1. Add Participant
2. View Participants
3. Propose Meeting Time
4. Find Common Availability
5. Schedule Meeting
6. View Scheduled Meetings
7. Exit

Enter choice:
```

#### 4. Edge Cases to Handle

- **DST Transitions**: A meeting scheduled during DST switch
  - Example: 2:00 AM on DST change day (doesn't exist in some zones)
  
- **Date Line Crossing**: Participant in Auckland (ahead) vs participant in Hawaii (behind)
  - Meeting on Monday for some, Tuesday for others

- **Invalid Time Zones**: User enters "America/NewYork" instead of "America/New_York"

- **Impossible Meetings**: No common working hours exist
  - Example: 9 AM-5 PM in Tokyo vs 9 AM-5 PM in New York (12-hour difference)

#### 5. Advanced Features (Optional Challenges)

- **Recurring Meetings**: Handle weekly/monthly recurrence
- **Best Time Suggestion**: Recommend times that work best for majority
- **Calendar Export**: Generate `.ics` file format
- **Time Zone Abbreviations**: Display PST/EST instead of full names

### Implementation Steps

**Step 1: Setup (10 minutes)**
- Create `module-info.java` with `import module java.base`
- Define all Record types
- Create main class with CLI loop

**Step 2: Participant Management (30 minutes)**
- Implement add/view participants
- Use `HashMap<String, Participant>` for storage
- Validate time zones with try-catch

**Step 3: Time Conversion (45 minutes)**
- Implement single meeting time conversion
- Create helper method: `convertTime(ZonedDateTime, ZoneId)`
- Test with multiple time zones

**Step 4: Availability Finder (90 minutes)**
- Define working hours per participant
- Calculate overlapping time slots
- Handle day boundaries (midnight crossing)

**Step 5: Meeting Storage (45 minutes)**
- Implement meeting scheduling
- Conflict detection algorithm
- List all meetings sorted by time

**Step 6: Testing & Polish (60 minutes)**
- Test DST edge cases
- Test date line crossing
- Add input validation
- Improve user experience

### Expected Outputs

**Example Session:**
```
=== Add Participant ===
Name: Alice
Time Zone: America/New_York
✓ Added Alice (America/New_York)

Name: Bob
Time Zone: Europe/London
✓ Added Bob (Europe/London)

=== Propose Meeting ===
Title: Sprint Planning
Reference Participant: Alice
Date (YYYY-MM-DD): 2026-03-15
Time (HH:MM): 14:00
Duration (minutes): 90

Meeting Preview:
Sprint Planning (1.5 hours)
- Alice (America/New_York): Sunday, Mar 15 at 2:00 PM EDT
- Bob (Europe/London): Sunday, Mar 15 at 7:00 PM BST

Schedule this meeting? (yes/no): yes
✓ Meeting scheduled!
```

### Learning Outcomes

By completing this project, you'll master:
- `ZonedDateTime` for time zone-aware dates
- `ZoneId` and `ZoneOffset` handling
- Duration calculations
- DST automatic adjustments
- Records for immutable data
- Optional for safety
- CLI user interaction patterns

### Evaluation Criteria

- ✅ Correctly handles time zone conversions
- ✅ Accounts for DST automatically
- ✅ Detects scheduling conflicts
- ✅ Clean separation of concerns (data models, business logic, UI)
- ✅ Proper error handling for invalid inputs
- ✅ User-friendly output formatting

---

## Project 2: "Messy" Log Analyzer

### Difficulty: ⭐⭐⭐☆☆ (Intermediate)

### Concepts Applied
- Sequenced Collections (`getFirst()`, `getLast()`, `reversed()`)
- Stream API (`map`, `filter`, `reduce`)
- Gatherers (JEP 461) for custom intermediate operations
- Optional for missing data
- Pattern Matching for `instanceof`
- Records for structured data
- Text Blocks for sample data

### Project Description

Build a log file analyzer that processes corrupted or incomplete log files. The analyzer should:
- Parse messy log lines with inconsistent formats
- Extract structured data from unstructured text
- Use Gatherers to create sliding windows of events
- Detect patterns like "3 errors in 10 seconds"
- Generate analytics reports

### Requirements

#### 1. Data Models

Create Records for:
- **LogEntry**: `record LogEntry(Instant timestamp, LogLevel level, String service, String message, Optional<String> userId)`
- **LogLevel**: `enum LogLevel { DEBUG, INFO, WARN, ERROR, FATAL }`
- **EventWindow**: `record EventWindow(Instant startTime, Instant endTime, List<LogEntry> events)`
- **Alert**: `record Alert(String type, int count, Duration timespan, List<LogEntry> entries)`

#### 2. Log Format Examples

Your parser should handle these messy formats:

```
# Standard format
2026-01-30T14:32:45Z [INFO] UserService: User logged in userId=123

# Missing timestamp (use previous entry's time + 1 second)
[ERROR] PaymentService: Transaction failed

# Wrong order (timestamp at end)
Database connection timeout [WARN] DBService 2026-01-30T14:33:00Z

# Missing log level (default to INFO)
2026-01-30T14:33:15Z AuthService: Token validated userId=456

# Completely broken (skip it)
This is garbage data }{][

# Multi-line stack trace
2026-01-30T14:34:00Z [ERROR] APIService: NullPointerException
    at com.example.Controller.handle(Controller.java:45)
    at com.example.Router.route(Router.java:23)
```

#### 3. Core Features

**Feature A: Robust Parsing**
- Parse various log formats into LogEntry records
- Handle missing fields with sensible defaults
- Skip completely invalid lines
- Preserve original line for debugging
- Use Pattern Matching to detect format types

**Feature B: Stream Processing**
- Load log file as Stream
- Filter by log level
- Filter by time range
- Filter by service name
- Support method chaining

**Feature C: Gatherers for Windows**

Implement custom gatherer that creates sliding windows:
```
Input: Stream of log entries
Output: Stream of 10-second windows

Example:
Window 1: Events from 14:32:40 to 14:32:50
Window 2: Events from 14:32:45 to 14:32:55  (5-second slide)
Window 3: Events from 14:32:50 to 14:33:00
```

Use `Stream.gather()` with custom gatherer implementation.

**Feature D: Pattern Detection**

Detect these patterns:
- **Error Spike**: 3+ errors in 10 seconds
- **Service Down**: No logs from a service for 60 seconds
- **User Activity Burst**: Same user ID appears 10+ times in 5 seconds
- **Repeating Errors**: Same error message 5+ times
- **Slow Requests**: Response time > 5 seconds (if timing data present)

**Feature E: Analytics Report**

Generate report with:
- Total logs processed
- Logs by level (DEBUG: 50, INFO: 200, ERROR: 5)
- Top 5 most active services
- Timeline of error spikes
- Alerts triggered

#### 4. Implementation Tasks

**Task 1: Parser (60 minutes)**
- Create regex patterns for each log format
- Implement `Optional<LogEntry> parseLine(String line)`
- Handle all edge cases
- Test with sample data

**Task 2: Streaming (45 minutes)**
- Read file as Stream using `Files.lines()`
- Chain filter operations
- Implement lazy evaluation
- Test with large files (100K+ lines)

**Task 3: Custom Gatherer (90 minutes)**
- Implement `Gatherer<LogEntry, ?, EventWindow>`
- Create sliding time windows
- Handle window boundaries
- Test window accuracy

**Task 4: Pattern Detection (120 minutes)**
- Implement each detection algorithm
- Use windows from gatherer
- Generate Alert records
- Prioritize alerts by severity

**Task 5: Report Generation (45 minutes)**
- Aggregate statistics using `Collectors`
- Format output with Text Blocks
- Create visual timeline (ASCII chart)
- Export to JSON/CSV option

#### 5. Sample Log File

Provide this as test data:

```
2026-01-30T14:30:00Z [INFO] UserService: Server started
2026-01-30T14:30:05Z [DEBUG] DBService: Connection pool initialized
2026-01-30T14:30:10Z [INFO] UserService: User login userId=101
[ERROR] PaymentService: Transaction timeout
2026-01-30T14:30:15Z [WARN] CacheService: Cache miss for key=user:101
2026-01-30T14:30:16Z [ERROR] PaymentService: Transaction timeout
2026-01-30T14:30:17Z [ERROR] PaymentService: Transaction timeout
2026-01-30T14:30:20Z [INFO] UserService: User logout userId=101
Database query slow (5200ms) [WARN] DBService 2026-01-30T14:30:25Z
2026-01-30T14:30:30Z [FATAL] APIService: Out of memory
# Logs stop here for 65 seconds
2026-01-30T14:31:35Z [INFO] APIService: Server restarted
```

**Expected Alerts:**
- Error Spike: 3 PaymentService errors in 7 seconds (14:30:10 to 14:30:17)
- Service Down: APIService silent for 65 seconds
- Fatal Error: Out of memory at 14:30:30

#### 6. Advanced Features (Optional)

- **Log Correlation**: Link related events (request ID tracking)
- **Anomaly Detection**: Statistical outliers in timing
- **Interactive Mode**: Live log tailing with real-time alerts
- **Distributed Logs**: Merge logs from multiple servers

### Expected Output

```
=== Log Analysis Report ===
File: application.log
Duration: 14:30:00 to 14:31:35 (95 seconds)

Summary:
- Total Lines: 11
- Parsed Successfully: 10
- Skipped (invalid): 1

Logs by Level:
- DEBUG: 1 (10%)
- INFO: 4 (40%)
- WARN: 2 (20%)
- ERROR: 3 (30%)
- FATAL: 1 (10%)

Top Services:
1. UserService: 3 logs
2. PaymentService: 3 logs
3. APIService: 2 logs
4. DBService: 2 logs

🚨 ALERTS (3):
[HIGH] Error Spike: 3 errors in 7 seconds
  - PaymentService: Transaction timeout (3 occurrences)
  - Window: 14:30:10 to 14:30:17

[HIGH] Service Down: APIService
  - Silent period: 65 seconds
  - Last log: 14:30:30 (FATAL: Out of memory)
  - Resumed: 14:31:35

[CRITICAL] Fatal Error Detected
  - Service: APIService
  - Message: Out of memory
  - Time: 14:30:30
```

### Learning Outcomes

- Robust parsing with error recovery
- Stream API mastery
- Custom Gatherer implementation
- Pattern detection algorithms
- Sequenced Collections usage
- Optional for defensive programming

### Evaluation Criteria

- ✅ Handles all messy log formats correctly
- ✅ Gatherer creates accurate sliding windows
- ✅ Detects all required patterns
- ✅ Efficient stream processing (no unnecessary iterations)
- ✅ Clear, actionable alert messages
- ✅ Comprehensive error handling

---

## Project 3: Fast JSON Parser

### Difficulty: ⭐⭐⭐⭐☆ (Advanced)

### Concepts Applied
- Sealed Interfaces for type safety
- Pattern Matching for Switch
- Records for AST nodes
- Recursion for nested structures
- Text Blocks for test cases
- String manipulation
- Exception handling

### Project Description

Build a JSON parser from scratch (no libraries!) that:
- Parses JSON text into a structured Abstract Syntax Tree (AST)
- Uses sealed interfaces for the type hierarchy
- Supports all JSON types (objects, arrays, strings, numbers, booleans, null)
- Handles nested structures with recursion
- Provides clean error messages for invalid JSON

### Requirements

#### 1. Type Hierarchy (Sealed Interfaces)

Define sealed interface hierarchy:

```
sealed interface JsonValue permits JsonObject, JsonArray, JsonString, 
                                     JsonNumber, JsonBoolean, JsonNull {}

record JsonObject(Map<String, JsonValue> fields) implements JsonValue {}
record JsonArray(List<JsonValue> elements) implements JsonValue {}
record JsonString(String value) implements JsonValue {}
record JsonNumber(double value) implements JsonValue {}
record JsonBoolean(boolean value) implements JsonValue {}
record JsonNull() implements JsonValue {}
```

#### 2. Core Features

**Feature A: Tokenization**
- Break JSON text into tokens
- Token types: `{`, `}`, `[`, `]`, `,`, `:`, string, number, true, false, null
- Handle whitespace
- Detect syntax errors early

**Feature B: Recursive Parser**
- Parse object: `{ "key": value, ... }`
- Parse array: `[ value1, value2, ... ]`
- Parse primitives: strings, numbers, booleans, null
- Handle nested structures recursively

**Feature C: String Parsing**
- Handle escape sequences: `\"`, `\\`, `\n`, `\t`, `\r`
- Handle Unicode escapes: `\u0041` (letter A)
- Validate string format

**Feature D: Number Parsing**
- Support integers: `42`
- Support decimals: `3.14`
- Support exponents: `1.5e10`, `2E-5`
- Support negative: `-42`

**Feature E: Pretty Printing**
- Convert AST back to formatted JSON
- Indentation support
- Configurable indent size

#### 3. Implementation Tasks

**Task 1: Tokenizer (90 minutes)**
- Create `Token` record: `record Token(TokenType type, String value, int position)`
- Implement `List<Token> tokenize(String json)`
- Handle all token types
- Track position for error messages

**Task 2: Parser Core (120 minutes)**
- Implement `JsonValue parse(String json)`
- Create `Parser` class with token stream
- Implement recursive descent parsing
- Use Pattern Matching for Switch on tokens

**Task 3: Error Handling (60 minutes)**
- Define custom exceptions: `JsonParseException`
- Include position in error messages
- Handle common errors:
  - Unexpected token
  - Unclosed string
  - Missing comma
  - Trailing comma
  - Invalid number format

**Task 4: Pretty Printer (45 minutes)**
- Implement `String toJson(JsonValue value, int indent)`
- Use Pattern Matching for Switch on JsonValue types
- Recursive formatting for nested structures

**Task 5: Testing (90 minutes)**
- Test valid JSON samples
- Test invalid JSON (expect errors)
- Test edge cases
- Performance test with large JSON

#### 4. Test Cases

**Valid JSON:**
```json
{
  "name": "Alice",
  "age": 30,
  "isActive": true,
  "balance": 1250.50,
  "tags": ["user", "premium"],
  "address": {
    "city": "New York",
    "zip": "10001"
  },
  "metadata": null
}
```

**Invalid JSON (should error):**
```json
# Missing closing brace
{"name": "Alice"

# Trailing comma
{"name": "Alice",}

# Invalid string (unescaped quote)
{"message": "He said "hello""}

# Invalid number
{"value": 12.34.56}

# Missing colon
{"name" "Alice"}
```

#### 5. Advanced Features (Optional)

- **Streaming Parser**: Parse large JSON without loading into memory
- **JSON Path**: Query JSON with paths like `$.address.city`
- **Schema Validation**: Validate against a schema
- **JSON Diff**: Compare two JSON structures
- **Comments Support**: Allow `//` and `/* */` comments (non-standard)

#### 6. Usage Example

```java
void main() {
    String json = """
        {
            "user": {
                "name": "Alice",
                "scores": [95, 87, 92]
            }
        }
        """;
    
    try {
        JsonValue parsed = JsonParser.parse(json);
        
        // Pattern matching to navigate
        switch (parsed) {
            case JsonObject(var fields) -> {
                switch (fields.get("user")) {
                    case JsonObject(var userFields) -> {
                        switch (userFields.get("name")) {
                            case JsonString(var name) -> 
                                println("User: " + name);
                            default -> {}
                        }
                    }
                    default -> {}
                }
            }
            default -> {}
        }
        
        // Pretty print
        String formatted = JsonPrinter.toJson(parsed, 2);
        println(formatted);
        
    } catch (JsonParseException e) {
        println("Parse error at position " + e.getPosition() + ": " + e.getMessage());
    }
}
```

### Expected Output

```
User: Alice
{
  "user": {
    "name": "Alice",
    "scores": [
      95.0,
      87.0,
      92.0
    ]
  }
}
```

### Learning Outcomes

- Sealed interfaces for closed type hierarchies
- Exhaustive pattern matching
- Recursive parsing algorithms
- String processing and escaping
- Error recovery strategies
- Abstract Syntax Tree design

### Evaluation Criteria

- ✅ Parses all valid JSON correctly
- ✅ Rejects invalid JSON with clear errors
- ✅ Handles deeply nested structures
- ✅ Exhaustive pattern matching (no default needed)
- ✅ Clean separation: tokenizer, parser, printer
- ✅ Efficient (no unnecessary allocations)

---

## Project 4: Concurrent Web Scraper

### Difficulty: ⭐⭐⭐⭐☆ (Advanced)

### Concepts Applied
- Virtual Threads for high concurrency
- Structured Concurrency (`StructuredTaskScope`)
- HttpClient for async HTTP requests
- Shutdown policies (ShutdownOnFailure)
- Collections (ConcurrentHashMap)
- Pattern Matching
- Records for results

### Project Description

Build a web scraper that:
- Downloads 100+ web pages concurrently using virtual threads
- Uses structured concurrency to manage tasks
- Cancels all pending downloads if one fails
- Extracts links from HTML
- Follows links up to N levels deep
- Respects rate limits
- Generates a sitemap

### Requirements

#### 1. Data Models

Create Records for:
- **UrlTask**: `record UrlTask(String url, int depth)`
- **PageResult**: `record PageResult(String url, int statusCode, String content, List<String> links, Duration loadTime)`
- **ScrapeError**: `record ScrapeError(String url, String errorMessage, Instant timestamp)`
- **Sitemap**: `record Sitemap(Map<String, PageResult> pages, List<ScrapeError> errors, Duration totalTime)`

#### 2. Core Features

**Feature A: Concurrent Page Fetching**
- Use Virtual Threads (one per URL)
- Use HttpClient for HTTP requests
- Set timeouts (5 seconds per request)
- Handle HTTP errors gracefully

**Feature B: Structured Concurrency**
- Wrap all downloads in `StructuredTaskScope.ShutdownOnFailure`
- If any critical page fails (e.g., homepage), cancel all others
- If optional page fails, continue with others
- Implement both policies

**Feature C: Link Extraction**
- Parse HTML to find `<a href="...">` tags
- Convert relative URLs to absolute
- Filter out external domains (stay on same domain)
- Avoid duplicate URLs

**Feature D: Depth-Limited Crawling**
- Start at seed URL (depth 0)
- Follow links to depth N (e.g., 3)
- Track visited URLs to avoid loops
- Use BFS or DFS strategy

**Feature E: Rate Limiting**
- Limit concurrent requests (e.g., 10 at a time)
- Delay between requests to same domain (e.g., 100ms)
- Use Semaphore or similar mechanism

#### 3. Implementation Tasks

**Task 1: HTTP Client Setup (30 minutes)**
- Create HttpClient with timeout
- Implement `PageResult fetch(String url)`
- Handle redirects
- Measure load time

**Task 2: Link Extraction (60 minutes)**
- Simple regex or string parsing for `<a>` tags
- Extract `href` attribute
- Resolve relative URLs using `URI.resolve()`
- Filter same-domain links only

**Task 3: Concurrent Crawler (90 minutes)**
- Implement BFS with virtual threads
- Use `ConcurrentHashMap<String, PageResult>` for visited
- Queue of URLs to process
- Launch virtual thread per URL

**Task 4: Structured Concurrency (60 minutes)**
- Wrap depth level in StructuredTaskScope
- Implement ShutdownOnFailure for critical pages
- Collect results from all subtasks

**Task 5: Rate Limiting (45 minutes)**
- Use Semaphore to limit concurrent requests
- Track last request time per domain
- Implement delay mechanism

**Task 6: Sitemap Generation (30 minutes)**
- Collect all PageResults
- Generate hierarchical sitemap
- Display as tree or JSON

#### 4. Example Usage

```
=== Web Scraper ===
Seed URL: https://example.com
Max Depth: 2
Max Concurrent: 10

Starting crawl...

[Depth 0] Fetching https://example.com (1 page)
✓ https://example.com (200 OK, 1.2s, 15 links found)

[Depth 1] Fetching 15 pages...
✓ https://example.com/about (200 OK, 0.8s, 5 links)
✓ https://example.com/products (200 OK, 1.1s, 20 links)
...
✗ https://example.com/broken (404 Not Found)

[Depth 2] Fetching 45 pages...
✓ https://example.com/products/item1 (200 OK, 0.9s)
...

Crawl completed in 12.5 seconds
- Pages fetched: 58
- Errors: 2
- Total links discovered: 127
```

#### 5. Edge Cases

- **Circular Links**: A → B → A
  - Solution: Track visited URLs

- **Infinite Redirects**: A → B → C → A
  - Solution: Limit redirect count in HttpClient

- **Malformed URLs**: `href="javascript:void(0)"`
  - Solution: Skip non-http(s) URLs

- **Slow Pages**: Page takes 30 seconds to load
  - Solution: Request timeout

- **One Page Fails**: Critical error cancels all
  - Solution: StructuredTaskScope.ShutdownOnFailure

#### 6. Advanced Features (Optional)

- **Respect robots.txt**: Parse and obey crawling rules
- **Image Downloads**: Fetch and save images
- **Content Extraction**: Parse article text
- **Distributed Crawling**: Multiple machines
- **Persistent Storage**: Save to database

### Expected Output Format

**Sitemap (Tree):**
```
example.com/
├── about/
│   ├── team/
│   └── contact/
├── products/
│   ├── item1/
│   ├── item2/
│   └── item3/
└── blog/
    ├── post1/
    └── post2/

Total: 11 pages
Errors: 0
Time: 8.2 seconds
```

**Sitemap (JSON):**
```json
{
  "root": "https://example.com",
  "pages": [
    {
      "url": "https://example.com",
      "status": 200,
      "depth": 0,
      "loadTime": "1.2s",
      "links": 15
    },
    ...
  ],
  "errors": [],
  "stats": {
    "totalPages": 58,
    "totalTime": "12.5s"
  }
}
```

### Learning Outcomes

- Virtual threads for I/O-bound concurrency
- Structured concurrency patterns
- Modern HTTP client usage
- URL manipulation and resolution
- Concurrent data structures
- Graceful error handling at scale

### Evaluation Criteria

- ✅ Successfully downloads 100+ pages concurrently
- ✅ Uses virtual threads efficiently
- ✅ StructuredTaskScope properly cancels on failure
- ✅ No duplicate page downloads
- ✅ Rate limiting works correctly
- ✅ Clean sitemap output
- ✅ Handles network errors gracefully

---

## Project 5: P2P Chat Node

### Difficulty: ⭐⭐⭐⭐⭐ (Expert)

### Concepts Applied
- Module System (JPMS) with Services
- ServiceLoader for plugin discovery
- Scoped Values for context propagation
- Virtual Threads for connections
- Structured Concurrency for message handling
- Sealed interfaces for message types
- Records for data
- Networking (ServerSocket, Socket)

### Project Description

Build a peer-to-peer chat application where:
- Each node is both client and server
- Nodes discover each other
- Messages are broadcast to all connected peers
- Plugins extend functionality (emoji reactions, encryption, logging)
- User identity propagates through the stack using Scoped Values
- All connections use virtual threads

### Requirements

#### 1. Module System Setup

**Main Module (`module-info.java`):**
```
module chat.core {
    exports chat.api;  // Plugin API
    uses chat.api.MessagePlugin;  // Declares plugin service
    
    requires java.base;
}
```

**Plugin Module Example (`module-info.java`):**
```
module chat.plugin.emoji {
    requires chat.core;
    provides chat.api.MessagePlugin 
        with chat.plugin.emoji.EmojiPlugin;
}
```

#### 2. Data Models

**Sealed Message Hierarchy:**
```
sealed interface Message permits TextMessage, JoinMessage, 
                                  LeaveMessage, HeartbeatMessage {}

record TextMessage(String sender, String content, Instant timestamp) implements Message {}
record JoinMessage(String nodeId, String address) implements Message {}
record LeaveMessage(String nodeId) implements Message {}
record HeartbeatMessage(String nodeId, Instant timestamp) implements Message {}
```

**Other Records:**
```
record Peer(String nodeId, String address, int port, Instant lastSeen) {}
record UserContext(String username, String nodeId, Instant loginTime) {}
```

#### 3. Core Features

**Feature A: Service Discovery**
- Nodes announce presence via UDP broadcast
- Listen for announcements from other nodes
- Maintain list of active peers
- Remove inactive peers after timeout

**Feature B: P2P Connections**
- Each node runs server socket (listens for connections)
- Connect to discovered peers
- Use virtual threads for each connection
- Handle reconnection on failure

**Feature C: Message Broadcasting**
- Send message to all connected peers
- Use structured concurrency for parallel sends
- Cancel all sends if critical peer fails
- Handle send failures gracefully

**Feature D: Plugin System**
- Define `MessagePlugin` interface
- Load plugins via `ServiceLoader`
- Plugins can:
  - Transform messages before send
  - React to received messages
  - Add commands (e.g., `/emoji :smile:`)
  
**Feature E: Scoped Values for User Context**
- User identity available everywhere via ScopedValue
- No need to pass username through method chains
- Thread-safe context propagation

#### 4. Plugin API

Define plugin interface:
```
package chat.api;

public interface MessagePlugin {
    String getName();
    
    // Transform outgoing message
    String transformOutgoing(String message);
    
    // React to incoming message
    void onMessageReceived(String sender, String message);
    
    // Handle custom commands
    boolean handleCommand(String command, String args);
}
```

**Example Plugins to Implement:**

**A. Emoji Plugin**
- Converts `:smile:` to 😊
- Converts `:heart:` to ❤️
- Provides `/emoji` command to list available emojis

**B. Encryption Plugin**
- Encrypts messages with simple Caesar cipher
- Provides `/encrypt on|off` command

**C. Logger Plugin**
- Logs all messages to file
- Includes timestamps
- Provides `/export` command

#### 5. Implementation Tasks

**Task 1: Module Setup (45 minutes)**
- Create `chat.core` module
- Define `chat.api` package with interfaces
- Create plugin modules
- Configure `module-info.java` files

**Task 2: Network Server (90 minutes)**
- ServerSocket listening on port
- Accept connections in virtual threads
- Read messages from socket
- Broadcast to all peers

**Task 3: Peer Discovery (60 minutes)**
- UDP broadcast sender (announce presence every 5s)
- UDP listener (discover peers)
- Maintain `ConcurrentHashMap<String, Peer>`
- Remove stale peers

**Task 4: Message Handling (90 minutes)**
- Serialize/deserialize messages
- Use Pattern Matching to handle message types
- Broadcast with StructuredTaskScope
- Queue messages if peer unavailable

**Task 5: Plugin System (60 minutes)**
- Implement ServiceLoader discovery
- Load all plugins at startup
- Chain plugin transforms
- Invoke plugin callbacks

**Task 6: Scoped Values Integration (45 minutes)**
- Define `ScopedValue<UserContext>`
- Bind context when handling connection
- Access in nested methods
- Propagate to child threads

#### 6. Example Usage

**Starting a Node:**
```
=== P2P Chat Node ===
Enter your username: Alice
Starting node on port 8000...

Discovered plugins:
- EmojiPlugin
- EncryptionPlugin
- LoggerPlugin

Listening for peers...
✓ Discovered peer: Bob (192.168.1.100:8001)
✓ Discovered peer: Charlie (192.168.1.101:8002)

Connected to 2 peers.

[Alice] Hello everyone! :wave:
[Bob] Hey Alice! 👋
[Charlie] Welcome!

/emoji list
Available emojis:
:smile: → 😊
:heart: → ❤️
:wave: → 👋
...

[Alice] I :heart: this chat!
[Sent] I ❤️ this chat!
```

**Plugin Processing Flow:**
```
User Input: "I :heart: this chat!"
    ↓
EmojiPlugin.transformOutgoing()
    → "I ❤️ this chat!"
    ↓
EncryptionPlugin.transformOutgoing()
    → "L#❤️#wklv#fkdw$" (if enabled)
    ↓
Send to all peers
    ↓
Peers receive & decrypt
    ↓
LoggerPlugin.onMessageReceived()
    → Writes to chat.log
```

#### 7. Edge Cases

- **Peer Disconnects**: Remove from active list
- **Network Partition**: Reconnect when available
- **Message Loop**: Don't rebroadcast own messages
- **Plugin Failure**: Continue without failing plugin
- **No Peers**: Allow local testing

#### 8. Advanced Features (Optional)

- **Private Messages**: `/msg Bob Hello`
- **File Transfer**: Send files to peers
- **Message History**: Store last 100 messages
- **Rooms/Channels**: Multiple chat rooms
- **E2E Encryption**: Real encryption (not Caesar)
- **NAT Traversal**: Connect across networks

### Architecture Diagram

```
┌─────────────────────────────────────────┐
│           Chat Node (Alice)             │
├─────────────────────────────────────────┤
│  Main Thread (Virtual)                  │
│  ├─ ServerSocket Listener               │
│  ├─ UDP Discovery Broadcaster           │
│  └─ User Input Handler                  │
│                                          │
│  Connection Handlers (Virtual Threads)  │
│  ├─ Connection to Bob                   │
│  ├─ Connection to Charlie               │
│  └─ Incoming Connection from David      │
│                                          │
│  Plugin System (ServiceLoader)          │
│  ├─ EmojiPlugin                         │
│  ├─ EncryptionPlugin                    │
│  └─ LoggerPlugin                        │
│                                          │
│  Scoped Values                          │
│  └─ USER_CONTEXT (propagates to all)    │
└─────────────────────────────────────────┘
```

### Learning Outcomes

- JPMS module system with services
- ServiceLoader for plugin architecture
- Scoped Values for context propagation
- Virtual threads for network I/O
- Structured concurrency for fan-out operations
- Network programming (TCP + UDP)
- Plugin architecture design

### Evaluation Criteria

- ✅ Nodes discover each other automatically
- ✅ Messages broadcast to all peers
- ✅ Plugins load via ServiceLoader
- ✅ Plugins can transform messages
- ✅ Scoped Values propagate user context
- ✅ Virtual threads handle connections
- ✅ Graceful handling of peer failures
- ✅ Clean module separation (core vs plugins)

---

## General Guidelines for All Projects

### Code Quality Standards

**1. Modern Java Features**
- Use `var` where type is obvious
- Use Text Blocks for multi-line strings
- Use Records instead of classes for data
- Use Pattern Matching instead of `instanceof` + cast
- Use `Optional` instead of null checks

**2. Error Handling**
- Catch specific exceptions
- Provide meaningful error messages
- Don't swallow exceptions silently
- Use try-with-resources for closeable resources

**3. Code Organization**
- One class per file (except nested classes)
- Group related methods together
- Keep methods short (<30 lines)
- Use descriptive variable names

**4. Documentation**
- Document public APIs
- Explain complex algorithms
- Add examples for non-obvious usage

### Testing Strategy

For each project, test:
- **Happy path**: Normal operation
- **Edge cases**: Empty inputs, max values, min values
- **Error cases**: Invalid inputs, network failures
- **Performance**: Large datasets, many concurrent operations

### Submission Checklist

For each project, ensure:
- ✅ Code compiles without warnings
- ✅ All features implemented
- ✅ Edge cases handled
- ✅ No hardcoded values (use configuration)
- ✅ Clean code (no commented-out code)
- ✅ README with usage instructions

### Time Estimates

| Project | Estimated Time |
|---------|----------------|
| 1. Time-Zone Meeting Scheduler | 8-12 hours |
| 2. Messy Log Analyzer | 10-15 hours |
| 3. Fast JSON Parser | 12-18 hours |
| 4. Concurrent Web Scraper | 15-20 hours |
| 5. P2P Chat Node | 20-30 hours |

**Total: 65-95 hours** (approximately 2-3 months at 10 hours/week)

---

## Final Tips

1. **Start Simple**: Implement core features first, then add advanced features

2. **Test Incrementally**: Test each feature as you build it

3. **Read Documentation**: Refer to Java API docs frequently

4. **Debug Systematically**: Use debugger, print statements, and logging

5. **Refactor Regularly**: Improve code structure as you learn

6. **Ask for Help**: When stuck for >30 minutes, seek guidance (but don't copy code!)

7. **Have Fun**: These projects should be challenging but enjoyable!

---

## What's Next After Capstones?

After completing all 5 projects, you'll be ready for:
- Real-world Java development roles
- Contributing to open-source Java projects
- Advanced topics (Spring Framework, Microservices, etc.)
- System design interviews
- Building your own applications

**Congratulations on reaching this milestone! Happy coding! 🚀**
