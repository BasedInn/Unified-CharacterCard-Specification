# **Unified Character Card Specification (UCCS)**

**Version:** `1.0.0-consolidated`  
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

- **CC**: Abbreviation for "Character Card".
- **CCv1 / CCv2 / CCv3**: Abbreviations referring to Version 1, Version 2, and Version 3 of the
  Character Card specification, respectively.
- **Application**: The software reading or writing the CC.
- **Prompt**: The final text sent to the LLM (Large Language Model).
- **Context**: The accumulated history of the chat and system instructions.

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

**Implementation Rule**: If both `ccv3` and `chara` chunks exist, the application MUST prioritize
`ccv3`.

### **3.3 `CHARX` (CCv3 Archive)**

A `.charx` file is a standard ZIP archive containing resources.

- **Root Structure**: MUST contain a file named `card.json` (the `CharacterCardV3` object).
- **Assets**: MUST be stored in subdirectories following the convention `assets/{type}/{category}/`.
    - Example: `assets/icon/images/my_icon.png`
- **Referencing**: Inside `card.json`, the URI for an asset should be
  `embeded://assets/icon/images/my_icon.png` (Note spelling: embeded with one 'd' in the middle is
  the standard in CCv3 spec).

## **4. Field Semantics & Behavior**

### **4.1 Prompt Engineering Fields**

These fields directly influence the LLM generation context.

- **`name`**: The primary identifier. Replaces` {{char}}` placeholders unless nickname is present
  (CCv3).
- **`description`** / **`personality`**: Usually combined and sent as system instruction or
  top-of-context info.
- **`scenario`**: Describes the current scene.
- **`mes_example`**: MUST be formatted as dialogue examples (e.g.,
  `<START\>\n{{user}}: Hello\n{{char}}: Hi!`). Pruned if context limit is reached.
- **`system_prompt`** (CCv2+): If present and not empty, this MUST replace the user's global system
  prompt settings. Supports `{{original}}` placeholder to include the user's global prompt.
- **`post_history_instructions`** (CCv2+): "Jailbreak" or "UJB". Instructions injected _after_ the
  chat history for stronger adherence. MUST replace global settings if present. Supports
  `{{original}}`.

### **4.2 Metadata Fields**

- **`creator_notes`**: Information for the user (e.g., "Use GPT-4"). MUST NOT be sent to the LLM.
- **`alternate_greetings`** (CCv2+): A list of optional starting messages. Implementations should
  allow users to "swipe" or cycle through these.
- **`group_only_greetings`** (CCv3): Greetings that SHOULD only be selectable if the character is
  placed in a group chat context.

### **4.3 Asset Management (CCv3)**

The assets array defines resources.

- **`uri`**:
    - `http://` / `https://`: Remote resource.
    - `embeded://`: References a file inside the CHARX zip.
    - `ccdefault:`:
        - For icon: Refers to the PNG image the card is embedded in.
        - For `user_icon`: Refers to the application's default user avatar.
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

1. **Scan**: Check last scan_depth messages for strings in keys.
2. **Filter**:
    - If `selective` is `true`, check `secondary_keys`.
    - If `use_regex` is `true`, treat `keys` as RegExp patterns.
3. **Budget**: If total tokens > `token_budget`, remove entries with lowest priority (or
   `insertion_order` if priority is unset) until within budget.
4. **Insert**: Place content into prompt context based on position or `insertion_order`.

## **6. Advanced Features (CCv3)**

### **6.1 Decorators**

CCv3 allows special syntax in Lorebook content to control injection behavior. Syntax:
`@@decorator_name value`.

- `@@depth N`: Insert N messages/tokens back from the latest message.
- `@@scan_depth N`: Override scan depth for this specific entry.
- `@@role [system|user|assistant]`: Force the injected content to be treated as a specific role.
- `@@activate_only_after N`: Only trigger if message count > `N`.
- `@@disable_ui_prompt [type]`: Suppresses the inclusion of the specified prompt field/setting.
  `[type]` is typically `system_prompt` or `post_history_instructions`.

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
- `{{// comment}}`: Removed from output (comment).

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
    - `"3.8"` (parsed as `3.80f`) > "3.11" (parsed as `3.11f`).
    - Implementers MUST NOT use SemVer comparison libraries. Use simple `float` comparison.

### **7.2 Backwards Compatibility**

- **CCv3 reading CCv2**: CCv3 is a superset. Populate missing CCv3 fields (like assets) with
  defaults (empty array or standard icon mapping).
- **CCv2 reading CCv1**: Wrap CCv1 fields into the data object structure.
- **Handling character_book**: Convert legacy CCv1/CCv2 lorebooks to the CCv3 structure by assuming
  use_regex: false and constant: false unless specified.

### **7.3 Unknown Fields**

Applications MUST NOT delete unknown fields in extensions or the root object when saving/exporting.
This ensures forward compatibility and preserves data from other applications.

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

3. **Read Priority**: As specified in Section 3.2, V3-compliant applications MUST prioritize the `ccv3`
   chunk and ignore the backfilled `chara` chunk if both are present.
