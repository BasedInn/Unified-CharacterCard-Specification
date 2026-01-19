# **Unified Character Card Specification (UCCS)**

**Version:** `0.0.1-consolidated`  
**Date:** 2026-01-18  
**Status:** Informational  
**Abstract:** This document consolidates the specifications for Character Card (CC) versions CCv1,
CCv2, and CCv3 into a single, unambiguous standard for implementing character interchange in
conversational AI applications. It defines the data models, serialization formats (JSON, PNG,
CHARX), and behavioral requirements for parsing and handling character metadata, assets, and
associated lorebooks.

## **1. Introduction**

### **1.1 Purpose**

The Character Card format allows for the portable exchange of AI character definitions, including
personality, scenario, example dialogue, and associated assets. This document unifies the three
existing iterations of the standard (CCv1, CCv2, CCv3) into a single reference for implementers.

### **1.2 Terminology**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC
2119\.

- **MUST / REQUIRED / SHALL**: Absolute requirement.
- **SHOULD / RECOMMENDED**: Strong recommendation; ignore only with full understanding of
  implications.
- **MAY / OPTIONAL**: Truly optional.
- **IN ANY CASE** (CCv3): The application must perform the action even if it violates other rules.

**Additional Terms:**

- **CC**: Character Card.
- **Application**: Software reading or writing the CC.
- **Prompt**: Final text sent to the LLM.
- **Context**: Accumulated history and system instructions.
- **Prefill**: Content pre-populated in the assistant's response slot.
- **Regex Pattern**: Regular Expression pattern used to match text.

### **1.3 Scope and Disclaimer**

This document creates a unified, language-agnostic reference model based strictly on the canonical
CCv1, CCv2, and CCv3 specifications. While the syntax has been normalized (using IDL) for clarity,
all logical behaviors, field semantics, and versioning schemes—including those that deviate from
established industry standards (e.g., Semantic Versioning)—have been preserved verbatim to ensure
strict compatibility.

As such, this document serves only as a consolidated reflection of the upstream sources of truth.
Deficiencies, idiosyncrasies, or non-standard architectural decisions inherent to the original
standards are retained by design. Feedback regarding these foundational design choices should be
directed to the original specification authors.

## **2. Data Models**

This specification uses a generic Interface Definition Language (IDL) to describe the JSON
structure.

- **`String`**: A sequence of Unicode characters.
- **`Integer`**: A number without a fractional component.
- **`Number`**: A general numeric value (integer or floating point).
- **`Boolean`**: `true` or `false`.
- **`Array<T>`**: An ordered list of elements of type `T`.
- **`Map<K, V>`**: An object/dictionary where keys are type `K` and values are type `V`.
- **`?`**: Indicates the field is OPTIONAL.

### **2.1 The CCv1 Standard (Legacy)**

The root object of a CCv1 card is a flat structure.

```rust
struct CharacterCardV1 {
  name: String;         // The character's name
  description: String;  // Detailed description/personality
  personality: String;  // Brief personality summary
  scenario: String;     // Current circumstances/context
  first_mes: String;    // The opening message sent by the character
  mes_example: String;  // Example dialogue patterns
}
```

### **2.2 The CCv2 Standard (Wrapper)**

CCv2 introduces a wrapper to separate metadata from content and allow for extensions.

```rust
struct CharacterCardV2 {
  spec: String;          // MUST be "chara_card_v2"
  spec_version: String;  // MUST be "2.0"
  data: CharacterDataV2;
}

struct CharacterDataV2 {
  /* Inherited from CCv1 */
  name: String;
  description: String;
  personality: String;
  scenario: String;
  first_mes: String;
  mes_example: String;

  /* CCv2 Additions */
  creator_notes: String;               // Notes for the user (not sent to LLM)
  system_prompt: String;               // Overrides global system prompt
  post_history_instructions: String;   // Instructions injected after chat history (Jailbreak/UJB)
  alternate_greetings: Array<String>;  // List of alternative opening messages
  character_book?: Lorebook;           // Embedded Lorebook (World Info)
  tags: Array<String>;                 // Search/Categorization tags
  creator: String;                     // Name of the card author
  character_version: String;           // Version string of the character itself
  extensions: Map<String, Any>;        // Namespace for app-specific data
}
```

### **2.3 The CCv3 Standard (Extended)**

CCv3 extends CCv2 with asset management, macros, and enhanced metadata.

```rust
struct CharacterCardV3 {
  spec: String;          // MUST be "chara_card_v3"
  spec_version: String;  // WARNING: Parsed as Float (NOT SemVer). 3.8 > 3.11.
  data: CharacterDataV3;
}

struct CharacterDataV3 {
  // All fields from CharacterDataV2 are REQUIRED
  // ... (See `CharacterDataV2`) ...

  /* CCv3 Additions */
  nickname?: String;                                 // Alternative name for {{char}} replacement
  creator_notes_multilingual?: Map<String, String>;  // Key is ISO 639-1 code
  source?: Array<String>;                            // URLs or IDs indicating source origin
  group_only_greetings: Array<String>;               // Greetings active only in group chats
  creation_date?: Integer;                           // Unix timestamp (seconds)
  modification_date?: Integer;                       // Unix timestamp (seconds)
  assets?: Array<CharacterAsset>;                    // Embedded or linked assets
}

struct CharacterAsset {
  type: String;  // "icon" | "background" | "user\_icon" | "emotion" | "other"
  uri: String;   // URL, "embeded://path", or "ccdefault:"
  name: String;  // Identifier (e.g., "happy", "main")
  ext: String;   // File extension (png, jpeg, webp)
}
```

## **3. Serialization and Embedding**

Applications MUST support reading from JSON, PNG/APNG, and (for CCv3) CHARX formats.

### **3.1 JSON**

The file content is the raw JSON object corresponding to `CharacterCardV1`, `CharacterCardV2`, or
`CharacterCardV3`.

### **3.2 PNG / APNG (Image Embedding)**

Character data is embedded inside the image metadata using `tEXt` chunks.

- **CCv1/CCv2 Cards**: Data MUST be stored in a `tEXt` chunk with the keyword `chara`. The content
  is a Base64 encoded string of the JSON object.
- **CCv3 Cards**: Data MUST be stored in a `tEXt` chunk with the keyword `ccv3`. The content is a
  Base64 encoded string of the `CharacterCardV3` JSON object.
- **CCv3 Asset Extensions**: CCv3-compliant PNG/APNG files MAY use `tEXt` chunks named
  `chara-ext-asset_:{path}` containing Base64 encoded asset data. This allows asset embedding
  without the CHARX archive format.

**Implementation Rule**: If both `ccv3` and `chara` chunks exist, the application MUST prioritize
`ccv3`.

### **3.3 `CHARX` (CCv3 Archive)**

A `.charx` file is a standard ZIP archive containing resources.

- **Root Structure**: MUST contain a file named `card.json` (the `CharacterCardV3` object).
- **Assets**: MUST be stored in subdirectories following the convention `assets/{type}/{category}/`.
  - `{type}` (Usage):

    | Value        | Purpose                                         |
    | ------------ | ----------------------------------------------- |
    | `icon`       | Primary character portrait or avatar.           |
    | `background` | Background image for the character's interface. |
    | `user_icon`  | Icon representing the user/player.              |
    | `emotion`    | Character expressions or emotional states.      |
    | `other`      | Application-specific or miscellaneous assets.   |

  - `{category}` (Format):

    | Value    | Purpose                         | Example file extension   |
    | -------- | ------------------------------- | ------------------------ |
    | `images` | Static visual assets.           | `.png`, `.jpeg`, `.webp` |
    | `audio`  | Sound effects or voice lines.   | `.mp3`, `.wav`           |
    | `video`  | Animated backgrounds or assets. | `.mp4`, `.webm`          |
    | `l2d`    | Live2D model files.             | `.model3.json`           |
    | `3d`     | 3D meshes and models.           | `.obj`, `.fbx`           |
    | `ai`     | Local model weights or configs. | `.safetensors`, `.ckpt`  |
    | `fonts`  | Custom UI typography.           | `.ttf`, `.otf`           |
    | `code`   | Scripting or extension code.    | `.js`, `.lua`            |
    | `other`  | Uncategorized file formats.     | (Any)                    |

  - Example: `assets/icon/images/my_icon.png`

- **Referencing**: Inside `card.json`, the URI for an asset should be
  `embeded://assets/icon/images/my_icon.png` (Note spelling: embeded with one 'd' in the middle is
  the standard in CCv3 spec).

#### **Additional CHARX Requirements**

- **Encryption**: ZIP files **SHOULD NOT** be encrypted.
- **File Names**: **SHOULD** only use ASCII characters to prevent compatibility issues.
- **Application Data**: Application-specific data **MAY** be stored as a JSON file in the root of
  the zip.
- **Validation**: Applications **MAY** reject CHARX files that are too large, corrupted, invalid, or
  encrypted.

## **4. Field Semantics & Behavior**

### **4.1 Prompt Engineering Fields**

These fields directly influence the LLM generation context.

- **`name`**: The primary identifier.
  - Replaces `{{char}}` (and legacy `<BOT>`) placeholders.
  - If `nickname` is present (CCv3), `nickname` takes precedence for replacements.
- **`description`** / **`personality`**: Usually combined and sent as system instruction or
  top-of-context info.
- **`scenario`**: Describes the current scene.
- **`mes_example`**: MUST be formatted as dialogue examples (e.g.,
  `<START\>\n{{user}}: Hello\n{{char}}: Hi!`). Pruned if context limit is reached.
- **`system_prompt`** (CCv2+): If present and not empty, this MUST replace the user's global system
  prompt settings.
  - **`{{original}}`**: Applications MUST replace this placeholder with the user's global system
    prompt.
    - _Example_: Character prompt `"{{original}}\nAct like a pirate"` + User prompt `"Be helpful"`
      -> Result: `"Be helpful\nAct like a pirate"`.
- **`post_history_instructions`** (CCv2+): "Jailbreak" or "UJB". Instructions injected _after_ the
  chat history for stronger adherence. MUST replace global settings if present. Supports
  `{{original}}`.

**Legacy Placeholders**: Applications MUST support case-insensitive replacement of:

- `{{char}}` or `<BOT>` -> Character Name (or Nickname)
- `{{user}}` or `<USER>` -> User Name

### **4.2 Metadata Fields**

- **`creator_notes`**: Information for the user (e.g., "Use GPT-4"). MUST NOT be sent to the LLM.
- **`alternate_greetings`** (CCv2+): A list of optional starting messages. Implementations should
  allow users to "swipe" or cycle through these.
- **`group_only_greetings`** (CCv3): Greetings that SHOULD only be selectable if the character is
  placed in a group chat context.

### **4.3 Asset Management (CCv3)**

The assets array defines resources.

#### **Asset Object Fields**

- **`type`**: The category of asset.
  - Standard types: `icon`, `background`, `user_icon`, `emotion`.
  - Custom types SHOULD start with `x_` to prevent conflicts.
- **`name`**: Identifier string.
  - For `icon`: `"main"` indicates the primary avatar.
  - For `emotion`: The emotion name (e.g., `"happy"`, `"angry"`).
  - For `background`: Descriptive name (e.g., `"forest"`, `"city"`).
- **`ext`**: File extension in lowercase (e.g., `"png"`, `"webp"`).
  - Applications SHOULD support at least PNG, JPEG, and WebP.
- **`uri`**:
  - `http://` / `https://`: Remote resource.
  - `embeded://`: References a file inside the CHARX zip.
  - `ccdefault:`:
    - For `icon`: Refers to the PNG image the card is embedded in.
    - For `user_icon`: Refers to the application's default user avatar.

#### **Default Behavior**

If the `assets` field is undefined, applications MUST behave as if it contains:

```json
[{ "type": "icon", "uri": "ccdefault:", "name": "main", "ext": "png" }]
```

- **Resolution**: If multiple assets of type icon exist, the one with name: "`main`" is the primary
  avatar.

## **5. Lorebook Specification (Character Book)**

A Lorebook (or World Info) allows dynamic injection of information based on keywords found in the
chat history.

### **5.1 Lorebook Structure**

```rust
struct Lorebook {
  spec: String;                  // Recommended: "lorebook\_v3"
  scan_depth?: Integer;          // How many recent messages to scan for keywords
  token_budget?: Integer;        // Max tokens allowed for lorebook injection
  recursive_scanning?: Boolean;  // If true, injected content is scanned for other keywords
  extensions: Map<String, Any>;
  entries: Array<LorebookEntry>;
}

struct LorebookEntry {
  keys: Array<String>;       // Trigger words
  content: String;           // Text to inject if triggered
  enabled: Boolean;
  insertion_order: Integer;  // Lower number = Higher in context
  case_sensitive?: Boolean;

  /* Optional Metadata */
  name?: String;             // Entry name (AgnAI)
  id?: String | Number;      // Entry ID (ST, Risu)
  comment?: String;          // Comment (ST, Risu)
  extensions: Map<String, Any>;

  /* CCv3 Features (Backwards Compatible as Optional) */
  use_regex: Boolean;              // If true, `keys` are regex patterns
  constant?: Boolean;              // If true, always injected (subject to budget)
  secondary_keys?: Array<String>;  // Must match ONE of these + ONE main key if `selective=true`
  selective?: Boolean;             // Enables secondary_keys logic
  priority?: Integer;              // Discard priority (lower = discarded first)
  position?: String;               // "before_char" | "after_char"
}
```

### **5.2 Injection Logic**

1. **Scan**: Check last `scan_depth` messages for strings in `keys`.
2. **Filter**:
   - If `selective` is `true`, check `secondary_keys`.
   - If `use_regex` is `true`, treat `keys` as RegExp patterns.
3. **Budget**: If total tokens > `token_budget`, remove entries with lowest priority (or
   `insertion_order` if `priority` is unset) until within budget.
4. **Insert**: Place `content` into prompt context based on `position` or `insertion_order`.

### **5.3 Standalone Lorebook Export (CCv3)**

If exporting a lorebook separately from a character card, it **SHOULD** use:

```ts
{
  spec: 'lorebook_v3',
  data: Lorebook
}
```

This allows lorebooks to be shared and imported independently.

## **6. Advanced Features (CCv3)**

### **6.1 Decorators**

CCv3 allows special syntax in Lorebook `content` to control injection behavior. Syntax:
`@@decorator_name value`.

- `@@depth N`: Insert `N` messages back from the latest message.
- `@@instruct_depth N`: Insert `N` tokens back (for instruct models).
- `@@reverse_depth N`: Count `N` messages from the start of context.
- `@@reverse_instruct_depth N`: Count `N` tokens from the start of context.
- `@@scan_depth N`: Override scan depth (messages) for this entry.
- `@@instruct_scan_depth N`: Override scan depth (tokens) for this entry.
- `@@role [system|user|assistant]`: Force the injected content to be treated as a specific role.
- `@@position [loc]`: Insert at specific location (`after_desc`, `before_desc`, `personality`,
  `scenario`).
- `@@activate_only_after N`: Only trigger if message count >= `N`.
- `@@activate_only_every N`: Only trigger if message count is divisible by `N`.
- `@@keep_activate_after_match`: Stays active for the rest of the chat after first trigger.
- `@@dont_activate_after_match`: Deactivates for the rest of the chat after first trigger.
- `@@is_greeting N`: Only trigger if current greeting index is `N` (0 = default).
- `@@is_user_icon [name]`: Only trigger if user icon matches `[name]`.
- `@@additional_keys K1,K2`: Require at least one of these keys to match (AND logic with main keys).
- `@@exclude_keys K1,K2`: Prevent trigger if any of these keys match.
- `@@ignore_on_max_context`: Do not trigger (or trim first) if context limit is reached.
- `@@disable_ui_prompt [type]`: Suppresses the inclusion of the specified prompt field/setting.
  `[type]` is typically `system_prompt` or `post_history_instructions`.
- `@@activate`: Force activation (always triggers).
- `@@dont_activate`: Force deactivation (never triggers).

#### **Fallback Syntax**

Decorators can use `@@@` (three `@` symbols) to specify fallbacks. If an application doesn't
recognize a decorator, it checks the next line.

```
@@risu_only_decorator 4
@@@activate_only_after 4
```

Multiple fallbacks are checked top-to-bottom until one is recognized.

### **6.2 Macros (Curly Braced Syntax)**

Applications MUST support placeholders based on the card version.

#### Universal Macros (CCv1, CCv2, CCv3)

- `{{char}}`: Replaced by nickname (if set in CCv3) or name.
- `{{user}}`: Replaced by the user's persona name.

#### CCv2+ Macros

- `{{original}}`: Used in system_prompt and post_history_instructions to inject the user's global
  default settings.

#### CCv3 Macros (New Standard)

These were unstandardized extensions in previous versions but are strictly part of the CCv3
specification.

- `{{random:A,B,C}}`: Randomly selects `A`, `B`, or `C`.
- `{{pick:A,B,C}}`: Random selection that attempts to remain stable (deterministic) for the same
  context.
- `{{roll:N}}` / `{{roll:dN}}`: Generates a random integer between 1 and N (inclusive). The d prefix
  is optional and case-insensitive. Examples: `{{roll:20}}`, `{{roll:d20}}`, `{{roll:D6}}`.
- `{{// comment}}`: Removed from output (comment). Not used for lorebook matching.
- `{{hidden_key:A}}`: Removed from output, but `A` IS used for recursive lorebook key matching.
- `{{comment:A}}`: Displayed in the UI as an inline comment (not sent to LLM).
- `{{reverse:A}}`: Reverses the string `A` (e.g., "spoilers").

## **7. Implementation Guidelines**

### **7.1 Version Detection**

To determine how to parse an imported file:

1. **Check `spec`**:
   - If `spec == "chara_card_v3"`, parse as **CCv3**.
   - If `spec == "chara_card_v2"`, parse as **CCv2**.
   - If `spec` is missing/undefined, parse as **CCv1**.
2. **Check `spec_version`** (`float` parsing):
   - **CRITICAL**: The CCv3 specification mandates that `spec_version` strings be parsed as
     floating-point numbers. This breaks Semantic Versioning conventions.
   - `"3.8"` (parsed as `3.80f`) > `"3.11"` (parsed as `3.11f`).
   - Implementers MUST NOT use SemVer comparison libraries. Use simple `float` comparison.

### **7.2 Backwards Compatibility**

- **CCv3 reading CCv2**: CCv3 is a superset. Populate missing CCv3 fields (like `assets`) with
  defaults (empty array or standard icon mapping).
- **CCv2 reading CCv1**: Wrap CCv1 fields into the `data` object structure.
- **Handling `character_book`**: Convert legacy CCv1/CCv2 lorebooks to the CCv3 structure by
  assuming `use_regex: false` and `constant: false` unless specified.

### **7.3 Unknown Fields**

Applications MUST NOT delete unknown fields in `extensions` or the root object when
saving/exporting. This ensures forward compatibility and preserves data from other applications.

### **7.4 Backfilling (V3 to V2 Downgrade)**

When saving a CCv3 card to a PNG/APNG file, applications MAY include a legacy CCv2 representation in
the `chara` chunk (in addition to the `ccv3` chunk) to ensure compatibility with older software.

Requirements for Backfilling:

1. **Warning Injection**: If backfilling, the application SHOULD prepend a warning to the
   `creator_notes` field of the V2 object:

   ```
   This character card is Character Card V3, but it is loaded as a Character Card V2. Please use a Character Card V3 compatible application to use this character card properly.
   ```

2. **Decorator Sanitization**: The application SHOULD remove all CCv3-specific decorators (e.g.,
   `@@depth`, `@@roll`) from the Lorebook and prompts in the backfilled V2 object to prevent raw
   syntax from leaking into the chat.

3. **Read Priority**: As specified in Section 3.2, V3-compliant applications MUST prioritize the
   `ccv3` chunk and ignore the backfilled `chara` chunk if both are present.

## **8. Appendix: Design Rationale**

This section explains the design decisions behind the Character Card specification.

### **8.1 Why CCv2 Uses a `data` Wrapper**

CCv1 fields are at the root level. CCv2 nests them inside a `data` object to prevent V1-only editors
from successfully loading a V2 card as V1, which would result in data loss. If a V1 editor loads a
V2 card, it will fail to find the expected root-level fields, alerting the user that the format is
unsupported.

### **8.2 Why `system_prompt` Overrides User Settings**

Prior to CCv2, botmakers had no control over the system prompt users employed. This made it
impossible to ensure consistent character behavior. The specification **requires** that character
`system_prompt` override user settings by default to give botmakers control over the experience.
Users can still override if they choose, but this must be an explicit action.

### **8.3 Why `post_history_instructions` Exists**

It was discovered that system instructions placed **after** conversation history have stronger
influence on model behavior. CCv2 standardizes this mechanism, previously known as "UJB" or
"Jailbreak" in various frontends.

### **8.4 Why CCv3 Uses Float Versioning**

CCv3's decision to use float parsing for `spec_version` (rather than SemVer) is a design choice that
simplifies comparison logic but breaks industry conventions. This is preserved verbatim from the
upstream specification to ensure strict compatibility with existing implementations.

### **8.5 Why Character Books Stack with World Books**

Character-specific lorebooks (**SHOULD**) stack with user world books rather than replace them. This
allows botmakers to include essential character lore while users add their own world-building
context. Character books take precedence in case of conflict.

### **8.6 Why Decorators Were Introduced in CCv3**

Adding new fields for every advanced feature would bloat the specification and confuse newcomers.
Decorators allow power users to access advanced features through a simple text syntax
(`@@decorator value`) without modifying the data structure.
