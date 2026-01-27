# Unified Character Card Specification

**Status:** Consolidated Reference Document  
**Version:** `2.0.1-consolidated`  
**Date:** 2026-01-26  
**Based On:** Character Card V2 (malfoyslastname/character-card-spec-v2), Character Card V3
(kwaroran/character-card-spec-v3)

---

## Disclaimer

This document is a **downstream consolidation** of the upstream specifications:

- **Character Card V2 Specification** — https://github.com/malfoyslastname/character-card-spec-v2
- **Character Card V3 Specification** — https://github.com/kwaroran/character-card-spec-v3

This consolidated document is **supplementary** and does not supersede the upstream specifications.
Where discrepancies exist between this document and the upstream sources, the upstream sources are
authoritative. This document represents the idiosyncrasies of the upstream specifications as-is,
including:

1. Deviations from common best practices and de facto standards
2. Inconsistencies in field naming conventions (snake_case vs camelCase)
3. Differences in keyword definitions between V2 and V3
4. Optional fields that are implementation-dependent
5. Behavioral ambiguities documented in Section 12

Implementers **SHOULD** consult the upstream repositories for the latest updates and errata.

---

## Table of Contents

1. [Scope](#1-scope)
2. [Normative References](#2-normative-references)
3. [Terms and Definitions](#3-terms-and-definitions)
4. [Conformance Keywords](#4-conformance-keywords)
5. [Embedding Methods](#5-embedding-methods)
6. [Data Structures](#6-data-structures)
7. [Character Card Fields](#7-character-card-fields)
8. [Lorebook (Character Book)](#8-lorebook-character-book)
9. [Decorators](#9-decorators)
10. [Curly Braced Syntax (Macros)](#10-curly-braced-syntax-macros)
11. [Placeholder Substitution](#11-placeholder-substitution)
12. [Unresolved Ambiguities](#12-unresolved-ambiguities)
13. [Version Compatibility](#13-version-compatibility)

---

## 1. Scope

This specification defines the data format for Character Cards, a structured representation of AI
chatbot characters used across various frontend applications. Character Cards encapsulate character
identity, personality, behavioral instructions, example dialogues, and supplementary metadata.

This specification covers:

- Three specification versions: V1, V2, and V3
- Embedding methods for storing Character Cards within image and archive files
- Required and optional fields for character definition
- Lorebook (world book) entries for contextual information injection
- Macro substitution and decorator systems for advanced prompt engineering

This specification does NOT cover:

- Transport protocols for Character Card distribution
- Authentication or authorization mechanisms
- Specific AI model prompt formatting requirements
- Rendering or display requirements for frontends

---

## 2. Normative References

This specification is self-contained. The following references are informational:

- **PNG Specification (ISO/IEC 15948):** Defines the PNG image format and its chunk structure,
  including tEXt chunks used for embedding Character Card data.
- **APNG Specification:** Extension to PNG for animated images; follows the same chunk structure.
- **ZIP File Format (PKWARE APPNOTE):** Defines the archive format used for CHARX files.
- **JSON (ECMA-404):** JavaScript Object Notation, the data interchange format used for Character
  Card serialization.
- **ECMAScript Regular Expressions:** The regular expression syntax used in lorebook key matching
  when regex mode is enabled.
- **Base64 (RFC 4648):** The encoding scheme used for embedding JSON data in image metadata.
- **ISO 639-1:** Language tags used in multilingual creator notes.
- **Unicode UTF-8 (RFC 3629):** Character encoding for all string values.

---

## 3. Terms and Definitions

For the purposes of this specification, the following terms and definitions apply:

### 3.1 Application

Software that reads, writes, or processes Character Card objects. This includes frontends, editors,
and conversion tools.

### 3.2 Frontend

An application that uses Character Cards to conduct conversations with AI models. Examples include
SillyTavern, RisuAI, and Agnai.

### 3.3 Character Editor

An application or component used to create, modify, and export Character Card objects.

### 3.4 Character Card

A structured data object containing the definition of an AI chatbot character, including its name,
description, personality, and behavioral instructions.

### 3.5 Lorebook

A collection of entries containing contextual information that is conditionally inserted into
prompts based on trigger conditions. Also known as "World Book," "World Info," or "Memory Book" in
various implementations.

### 3.6 Prompt

The text sent to an AI language model, constructed from Character Card fields, conversation history,
and lorebook entries.

### 3.7 Chat Log

The conversation history between the user and the AI character, consisting of alternating user and
assistant messages.

### 3.8 Greeting

The initial message displayed at the start of a conversation, originating from the character rather
than the user.

### 3.9 Swipe

An alternative message variant at the same position in the conversation. For greetings, this refers
to alternate initial messages.

### 3.10 Creator Note

Metadata intended for human readers (users and other creators), not for inclusion in AI prompts.

### 3.11 Decorator

A structured annotation within lorebook content that modifies the behavior of that entry. Decorators
begin with `@@` and appear at the start of the content field.

### 3.12 Curly Braced Syntax (CBS)

A macro substitution syntax using double curly braces (e.g., `{{char}}`) that is replaced with
dynamic values at runtime. Also known as "macros."

### 3.13 Prefill

A technique where partial assistant responses are provided to the AI model to guide its output.
Supported by some API providers.

### 3.14 Asset

A binary file (image, audio, video, etc.) associated with a Character Card, such as character
portraits or background images.

### 3.15 Group Chat

A conversation involving multiple AI characters simultaneously, as opposed to a one-on-one
conversation.

---

## 4. Conformance Keywords

The key words in this specification are to be interpreted as follows:

| Keyword         | Synonyms        | Definition                                                                                                                  |
| --------------- | --------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **MUST**        | REQUIRED, SHALL | An absolute requirement of the specification. Non-conformance constitutes a specification violation.                        |
| **MUST NOT**    | SHALL NOT       | An absolute prohibition. Non-conformance constitutes a specification violation.                                             |
| **SHOULD**      | RECOMMENDED     | A strong recommendation. Deviations are permitted only when the implications are fully understood and carefully considered. |
| **SHOULD NOT**  | NOT RECOMMENDED | A strong discouragement. The behavior may be acceptable in specific circumstances when implications are understood.         |
| **MAY**         | OPTIONAL        | Truly optional. Implementations may include or omit this item freely.                                                       |
| **MAY NOT**     |                 | Truly optional to exclude. The item is not forbidden.                                                                       |
| **IN ANY CASE** |                 | The described action takes precedence over other rules in the specification, even if it creates a conflict.                 |

---

## 5. Embedding Methods

Character Card data may be embedded using multiple methods. This section defines each method and its
requirements.

### 5.1 JSON File

Character Card objects **MAY** be stored as standalone JSON files.

**Requirements:**

- The file extension **SHOULD** be `.json`.
- The file **MUST** contain a valid JSON object conforming to the Character Card structure.
- UTF-8 encoding **MUST** be used.

**Considerations:**

- Standalone JSON files lack associated imagery and are discouraged for distribution due to reduced
  user-friendliness.

### 5.2 PNG/APNG Embedding

#### 5.2.1 V1 and V2 Embedding

Character Card V1 and V2 objects **MAY** be embedded in PNG or APNG files.

**Requirements:**

- The Character Card JSON **MUST** be serialized as a UTF-8 string.
- The UTF-8 string **MUST** be encoded using Base64.
- The Base64 string **MUST** be stored in a PNG `tEXt` chunk with the keyword `chara`.

**Note:** V1 specification references "Chara EXIF metadata field." This is a misnomer; PNG files do
not contain EXIF metadata. Implementations **MUST** use `tEXt` chunks.

#### 5.2.2 V3 Embedding

Character Card V3 objects **MAY** be embedded in PNG or APNG files.

**Requirements:**

- The Character Card JSON **MUST** be serialized as a UTF-8 string.
- The UTF-8 string **MUST** be encoded using Base64.
- The Base64 string **MUST** be stored in a PNG `tEXt` chunk with the keyword `ccv3`.

**Backward Compatibility:**

- Applications **MAY** include a backfilled V2-compatible representation in the `chara` chunk.
- When backfilling V2, applications **SHOULD** add a warning to the `creator_notes` field indicating
  that the card is V3 and should be used with a V3-compatible application.
- When importing, if both `chara` and `ccv3` chunks are present, applications **SHOULD** use the
  `ccv3` chunk.
- The backfilled `chara` chunk **SHOULD** be discarded on import when `ccv3` is present.

#### 5.2.3 Embedded Assets in PNG (V3)

PNG/APNG files **MAY** contain embedded assets as additional tEXt chunks.

**Requirements:**

- The `tEXt` chunk keyword **MUST** be `chara-ext-asset_:{path}` where `{path}` is the asset path.
- The chunk value **MUST** be Base64-encoded binary data.
- Assets are accessed via the URI `__asset:{path}`.

**Recommendation:** This method is provided for legacy compatibility. New implementations **SHOULD**
use CHARX format for assets.

### 5.3 WEBP Embedding

WEBP embedding is NOT covered by this specification due to technical ambiguities in the V1
specification. Implementations **MAY** support WEBP but behavior is undefined.

### 5.4 CHARX Format (V3 Only)

CHARX is an archive format for Character Card V3 objects with embedded assets.

**Structure:**

- A CHARX file is a ZIP archive.
- The ZIP archive **MUST** contain a `card.json` file at the root containing the CharacterCardV3
  object.
- The ZIP archive **MUST NOT** be encrypted.
- File names and paths within the archive **SHOULD** use only ASCII characters.

**Asset Organization:**

Assets **SHOULD** be organized as follows:

| Asset Type                             | Directory Path          |
| -------------------------------------- | ----------------------- |
| Images (.png, .avif, .jpg, .webp)      | `assets/{type}/images/` |
| Audio (.mp3, .wav, .ogg)               | `assets/{type}/audio/`  |
| Video (.mp4, .webm)                    | `assets/{type}/video/`  |
| Live2D models                          | `assets/{type}/l2d/`    |
| 3D models (.mmd, .obj)                 | `assets/{type}/3d/`     |
| AI models (.safetensors, .ckpt, .onnx) | `assets/{type}/ai/`     |
| Fonts (.otf, .ttf)                     | `assets/{type}/fonts/`  |
| Code (.lua, .js)                       | `assets/{type}/code/`   |
| Other                                  | `assets/{type}/other/`  |

The `{type}` value corresponds to the asset's usage context (e.g., `icon`, `background`, `emotion`).

**Application-Specific Data:**

- Applications **MAY** store additional JSON files at the archive root for application-specific
  data.

**Rejection:**

- Applications **MAY** reject CHARX files that are too large, corrupted, invalid ZIP archives, or
  encrypted.

---

## 6. Data Structures

This section defines the data structures using an Interface Definition Language (IDL). The IDL is
C-like with the following conventions:

### 6.1 IDL Conventions

| Syntax             | Meaning                                                                             |
| ------------------ | ----------------------------------------------------------------------------------- |
| `String`           | A UTF-8 encoded string of variable length                                           |
| `Integer`          | A signed integer (precision implementation-defined, minimum 32-bit)                 |
| `Float`            | A floating-point number (precision implementation-defined, minimum 64-bit IEEE 754) |
| `Boolean`          | A boolean value (`true` or `false`)                                                 |
| `Array<T>`         | A dynamically-sized ordered collection of elements of type `T`                      |
| `Record<K, V>`     | An associative map with keys of type `K` and values of type `V`                     |
| `Optional<T>`      | A value that may be present (of type `T`) or absent                                 |
| `Literal<V>`       | A string field that **MUST** have the exact value `V`                               |
| `Union<A, B>`      | A value that may be either type `A` or type `B`                                     |
| `Enum { A, B, C }` | A string value restricted to one of the listed options                              |

### 6.2 Character Card V1

```idl
struct CharacterCardV1 {
    String name;
    String description;
    String personality;
    String scenario;
    String first_mes;
    String mes_example;
}
```

**Field Default:** All fields are mandatory and **MUST** default to empty string (`""`), not `null`
or absent.

### 6.3 Character Card V2

```idl
struct CharacterCardV2 {
    Literal<"chara_card_v2"> spec;
    Literal<"2.0"> spec_version;
    CharacterCardV2Data data;
}

struct CharacterCardV2Data {
    // V1 fields (inherited)
    String name;
    String description;
    String personality;
    String scenario;
    String first_mes;
    String mes_example;

    // V2 additions
    String creator_notes;
    String system_prompt;
    String post_history_instructions;
    Array<String> alternate_greetings;
    Optional<CharacterBook> character_book;
    Array<String> tags;
    String creator;
    String character_version;
    Record<String, Any> extensions;
}

struct CharacterBook {
    Optional<String> name;
    Optional<String> description;
    Optional<Integer> scan_depth;
    Optional<Integer> token_budget;
    Optional<Boolean> recursive_scanning;
    Record<String, Any> extensions;
    Array<CharacterBookEntry> entries;
}

struct CharacterBookEntry {
    Array<String> keys;
    String content;
    Record<String, Any> extensions;
    Boolean enabled;
    Integer insertion_order;
    Optional<Boolean> case_sensitive;
    Optional<String> name;
    Optional<Integer> priority;
    Optional<Integer> id;
    Optional<String> comment;
    Optional<Boolean> selective;
    Optional<Array<String>> secondary_keys;
    Optional<Boolean> constant;
    Optional<Enum { "before_char", "after_char" }> position;
}
```

### 6.4 Character Card V3

```idl
struct CharacterCardV3 {
    Literal<"chara_card_v3"> spec;
    Literal<"3.0"> spec_version;
    CharacterCardV3Data data;
}

struct CharacterCardV3Data {
    // Inherited from V2
    String name;
    String description;
    String personality;
    String scenario;
    String first_mes;
    String mes_example;
    String creator_notes;
    String system_prompt;
    String post_history_instructions;
    Array<String> alternate_greetings;
    Array<String> tags;
    String creator;
    String character_version;
    Record<String, Any> extensions;

    // Changed from V2
    Optional<Lorebook> character_book;

    // V3 additions
    Optional<Array<Asset>> assets;
    Optional<String> nickname;
    Optional<Record<String, String>> creator_notes_multilingual;
    Optional<Array<String>> source;
    Array<String> group_only_greetings;
    Optional<Integer> creation_date;
    Optional<Integer> modification_date;
}

struct Asset {
    String type;
    String uri;
    String name;
    String ext;
}

struct Lorebook {
    Optional<String> name;
    Optional<String> description;
    Optional<Integer> scan_depth;
    Optional<Integer> token_budget;
    Optional<Boolean> recursive_scanning;
    Record<String, Any> extensions;
    Array<LorebookEntry> entries;
}

struct LorebookEntry {
    Array<String> keys;
    String content;
    Record<String, Any> extensions;
    Boolean enabled;
    Integer insertion_order;
    Optional<Boolean> case_sensitive;
    Boolean use_regex;
    Optional<Boolean> constant;
    Optional<String> name;
    Optional<Integer> priority;
    Optional<Union<Integer, String>> id;
    Optional<String> comment;
    Optional<Boolean> selective;
    Optional<Array<String>> secondary_keys;
    Optional<Enum { "before_char", "after_char" }> position;
}
```

### 6.5 Standalone Lorebook Export (V3)

```idl
struct LorebookExport {
    Literal<"lorebook_v3"> spec;
    Lorebook data;
}
```

### 6.6 Union Type for Import

Applications supporting multiple versions **SHOULD** use:

```idl
typedef CharacterCard = Union<CharacterCardV1, CharacterCardV2, CharacterCardV3>;
```

---

## 7. Character Card Fields

This section specifies the semantics and requirements for each field.

### 7.1 Core Identity Fields

#### 7.1.1 `name`

| Property       | Value                        |
| -------------- | ---------------------------- |
| Type           | `String`                     |
| Required       | Yes                          |
| Default        | `""`                         |
| Used in Prompt | Yes (as substitution target) |

The character's name. Used to identify the character and as the replacement value for `{{char}}`
placeholders (unless overridden by `nickname` in V3).

#### 7.1.2 `nickname` (V3 Only)

| Property       | Value                        |
| -------------- | ---------------------------- |
| Type           | `Optional<String>`           |
| Required       | No                           |
| Default        | Absent                       |
| Used in Prompt | Yes (as substitution target) |

An alternative display name. When present and non-empty, the placeholders `{{char}}`, `<char>`, and
`<bot>` **MUST** be replaced with the `nickname` value instead of `name`.

### 7.2 Character Definition Fields

#### 7.2.1 `description`

| Property       | Value                         |
| -------------- | ----------------------------- |
| Type           | `String`                      |
| Required       | Yes                           |
| Default        | `""`                          |
| Used in Prompt | SHOULD be included by default |

A description of the character. **SHOULD** be included in prompts by default.

**Alternative UI Labels:**

- ZoltanAI: "Personality"
- Agnai: "Persona Attributes"
- SillyTavern: "Description"

#### 7.2.2 `personality`

| Property       | Value                         |
| -------------- | ----------------------------- |
| Type           | `String`                      |
| Required       | Yes                           |
| Default        | `""`                          |
| Used in Prompt | SHOULD be included by default |

A concise summary of the character's personality traits. **SHOULD** be included in prompts by
default.

**Alternative UI Labels:**

- ZoltanAI: "Summary"
- SillyTavern: "Personality summary"

#### 7.2.3 `scenario`

| Property       | Value                         |
| -------------- | ----------------------------- |
| Type           | `String`                      |
| Required       | Yes                           |
| Default        | `""`                          |
| Used in Prompt | SHOULD be included by default |

The context and circumstances of the conversation. **SHOULD** be included in prompts by default.

### 7.3 Conversation Fields

#### 7.3.1 `first_mes`

| Property       | Value                    |
| -------------- | ------------------------ |
| Type           | `String`                 |
| Required       | Yes                      |
| Default        | `""`                     |
| Used in Prompt | Yes (as initial message) |

The character's first message (greeting) in a conversation.

**Requirements:**

- The character **MUST** send the first message in a conversation.
- The first message **MUST** be the value of `first_mes`.

#### 7.3.2 `alternate_greetings` (V2+)

| Property       | Value                                 |
| -------------- | ------------------------------------- |
| Type           | `Array<String>`                       |
| Required       | Yes (V2+)                             |
| Default        | `[]`                                  |
| Used in Prompt | Yes (as alternative initial messages) |

Alternative first messages. Applications **MUST** provide a mechanism (swipes) to select among
`first_mes` and each element of `alternate_greetings`.

#### 7.3.3 `group_only_greetings` (V3 Only)

| Property       | Value                        |
| -------------- | ---------------------------- |
| Type           | `Array<String>`              |
| Required       | Yes                          |
| Default        | `[]`                         |
| Used in Prompt | Yes (in group chat contexts) |

Additional greetings available only in group chat contexts. These greetings **SHOULD** NOT appear in
one-on-one conversations.

#### 7.3.4 `mes_example`

| Property       | Value         |
| -------------- | ------------- |
| Type           | `String`      |
| Required       | Yes           |
| Default        | `""`          |
| Used in Prompt | Conditionally |

Example conversations demonstrating the character's speech patterns and behavior.

**Format:**

```
<START>
{{user}}: [user message]
{{char}}: [character response]
<START>
{{user}}: [another user message]
{{char}}: [another response]
```

**Requirements:**

- The `<START>` marker indicates the beginning of a new example conversation segment.
- Applications **MAY** transform `<START>` (e.g., into a system message reading "Start a new
  conversation").
- Example conversations **SHOULD** be included in prompts by default until actual conversation
  history fills the context window.
- When context space is limited, examples **SHOULD** be pruned to make room for actual conversation.
- This pruning behavior **MAY** be user-configurable.

### 7.4 Prompt Override Fields

#### 7.4.1 `system_prompt` (V2+)

| Property       | Value     |
| -------------- | --------- |
| Type           | `String`  |
| Required       | Yes (V2+) |
| Default        | `""`      |
| Used in Prompt | Yes       |

Character-specific system prompt override.

**Requirements:**

- If non-empty, applications **MUST** replace the default/global system prompt with this value by
  default.
- If empty, applications **MUST** use the default/global system prompt.
- Applications **MUST** support the `{{original}}` placeholder, which is replaced with the system
  prompt that would have been used without this override.
- Applications **MAY** provide mechanisms to override or supplement this field, but such overrides
  **MUST** NOT be the default behavior.

#### 7.4.2 `post_history_instructions` (V2+)

| Property       | Value     |
| -------------- | --------- |
| Type           | `String`  |
| Required       | Yes (V2+) |
| Default        | `""`      |
| Used in Prompt | Yes       |

Instructions inserted after the conversation history, also known as "jailbreak" or "UJB" (User
Jailbreak) in some applications.

**Requirements:**

- If non-empty, applications **MUST** replace the default post-history instructions with this value
  by default.
- If empty, applications **MUST** use the default post-history instructions.
- Applications **MUST** support the `{{original}}` placeholder.
- Applications **MAY** provide mechanisms to override or supplement this field, but such overrides
  **MUST** NOT be the default behavior.

### 7.5 Metadata Fields

#### 7.5.1 `creator_notes` (V2+)

| Property       | Value        |
| -------------- | ------------ |
| Type           | `String`     |
| Required       | Yes (V2+)    |
| Default        | `""`         |
| Used in Prompt | **MUST NOT** |

Notes from the character creator intended for users and other creators.

**Requirements:**

- **MUST NOT** be included in prompts sent to AI models.
- **SHOULD** be prominently displayed to users (at least one paragraph **SHOULD** be visible).

**V3 Behavior:**

- If `creator_notes_multilingual` is present and contains a matching language, that localized
  version takes precedence.
- If no matching language is found, `creator_notes` is used as the fallback.

#### 7.5.2 `creator_notes_multilingual` (V3 Only)

| Property       | Value                              |
| -------------- | ---------------------------------- |
| Type           | `Optional<Record<String, String>>` |
| Required       | No                                 |
| Default        | Absent                             |
| Used in Prompt | **MUST NOT**                       |

Localized versions of creator notes. Keys **MUST** be valid ISO 639-1 language tags (e.g., `"en"`,
`"ja"`).

**Selection Algorithm:**

1. Check for exact match with application's language setting.
2. Check for language-only match (ignoring region/script).
3. Fall back to `creator_notes`.

#### 7.5.3 `creator` (V2+)

| Property       | Value        |
| -------------- | ------------ |
| Type           | `String`     |
| Required       | Yes (V2+)    |
| Default        | `""`         |
| Used in Prompt | **MUST NOT** |

The name or identifier of the character's creator.

#### 7.5.4 `character_version` (V2+)

| Property       | Value        |
| -------------- | ------------ |
| Type           | `String`     |
| Required       | Yes (V2+)    |
| Default        | `""`         |
| Used in Prompt | **MUST NOT** |

Version identifier for the character card. **MAY** be used for display and sorting.

#### 7.5.5 `tags` (V2+)

| Property       | Value           |
| -------------- | --------------- |
| Type           | `Array<String>` |
| Required       | Yes (V2+)       |
| Default        | `[]`            |
| Used in Prompt | **SHOULD NOT**  |

Categorization tags for the character.

**Requirements:**

- No restrictions on tag string content.
- **SHOULD NOT** be used in prompt engineering.
- **MAY** be used for frontend sorting and filtering.
- Filtering **SHOULD** be case-insensitive.

#### 7.5.6 `source` (V3 Only)

| Property       | Value                     |
| -------------- | ------------------------- |
| Type           | `Optional<Array<String>>` |
| Required       | No                        |
| Default        | Absent                    |
| Used in Prompt | **MUST NOT**              |

URLs or identifiers indicating the origin of the character card.

**Requirements:**

- Elements **SHOULD** be HTTP/HTTPS URLs or platform-specific identifiers.
- This field **SHOULD NOT** be user-editable.
- Applications **MAY** provide UI to open source URLs.
- Applications **SHOULD** only append to this array and **SHOULD NOT** modify or remove existing
  elements unless the element was added by the same application.
- Applications **MAY** remove elements if they significantly impair performance or are harmful.

#### 7.5.7 `creation_date` (V3 Only)

| Property       | Value               |
| -------------- | ------------------- |
| Type           | `Optional<Integer>` |
| Required       | No                  |
| Default        | Absent              |
| Used in Prompt | **MUST NOT**        |

Unix timestamp (seconds since 1970-01-01 00:00:00 UTC) of character card creation. Applications
**SHOULD** set this when creating a new character and **SHOULD NOT** modify it thereafter.

#### 7.5.8 `modification_date` (V3 Only)

| Property       | Value               |
| -------------- | ------------------- |
| Type           | `Optional<Integer>` |
| Required       | No                  |
| Default        | Absent              |
| Used in Prompt | **MUST NOT**        |

Unix timestamp (seconds since 1970-01-01 00:00:00 UTC) of last modification. Applications **SHOULD**
update this when saving changes.

### 7.6 Extension Fields

#### 7.6.1 `extensions` (V2+)

| Property       | Value                 |
| -------------- | --------------------- |
| Type           | `Record<String, Any>` |
| Required       | Yes (V2+)             |
| Default        | `{}`                  |
| Used in Prompt | Application-dependent |

A container for application-specific or experimental data.

**Requirements:**

- **MUST** default to an empty object.
- **MAY** contain any arbitrary JSON-serializable key-value pairs.
- Applications **MUST NOT** destroy unknown key-value pairs when importing and exporting.
- Applications **SHOULD** namespace keys to prevent conflicts:
  - Preferred: `"appname/key"` (e.g., `"agnai/voice"`)
  - Acceptable: `"appname_key"` (e.g., `"agnai_voice"`)
  - Acceptable: `"appname": { "key": value }` (e.g., `"agnai": { "voice": ... }`)

### 7.7 Asset Fields (V3 Only)

#### 7.7.1 `assets`

| Property       | Value                       |
| -------------- | --------------------------- |
| Type           | `Optional<Array<Asset>>`    |
| Required       | No                          |
| Default        | See below                   |
| Used in Prompt | No (visual/audio resources) |

Binary assets associated with the character.

**Default Value (when absent or undefined):**

```json
[
  {
    "type": "icon",
    "uri": "ccdefault:",
    "name": "main",
    "ext": "png"
  }
]
```

The special URI `ccdefault:` indicates the default behavior:

- For PNG/APNG embedding: the host image itself
- For CHARX: implementation-defined

**Asset Object Structure:**

| Field  | Type     | Description                               |
| ------ | -------- | ----------------------------------------- |
| `type` | `String` | The asset category (see below)            |
| `uri`  | `String` | Location of the asset                     |
| `name` | `String` | Identifier for the asset                  |
| `ext`  | `String` | File extension (lowercase, e.g., `"png"`) |

**URI Schemes:**

| Scheme                | Description                                     |
| --------------------- | ----------------------------------------------- |
| `ccdefault:`          | Default/embedded asset                          |
| `embeded://`          | Asset embedded in CHARX or PNG extension chunks |
| `__asset:{path}`      | Asset in PNG extension chunk at `{path}`        |
| `http://`, `https://` | Remote asset                                    |

**Asset Types:**

| Type         | Description                             |
| ------------ | --------------------------------------- |
| `icon`       | Character portrait/avatar               |
| `background` | Conversation background image           |
| `user_icon`  | User's avatar when using this character |
| `emotion`    | Character expression/emotion sprite     |

**Behavior:**

- For `icon`: Applications **SHOULD** use the first `icon` asset with `name` equal to `"main"` as
  the default character avatar.
- For `background`: Applications **SHOULD** use the first `background` asset with `name` equal to
  `"main"` as the default background.
- For `user_icon`: Applications **SHOULD** use the asset as the user's avatar.
- For `emotion`: The `name` field identifies the emotion (e.g., `"happy"`, `"sad"`).

Applications **MAY** define custom asset types. Custom types **SHOULD** be prefixed with `x_` to
avoid conflicts with future specification additions.

Applications **MAY** ignore assets they do not support but **MUST** preserve unrecognized assets on
export.

**Format Support:** Applications **SHOULD** support at least PNG, JPEG, and WebP formats. If `uri`
is `ccdefault:`, the `ext` field **SHOULD** be ignored.

### 7.8 Specification Version Fields

#### 7.8.1 `spec`

| Version | Value             |
| ------- | ----------------- |
| V2      | `"chara_card_v2"` |
| V3      | `"chara_card_v3"` |

Applications **SHOULD NOT** consider a card as conforming to a specification version if this field
does not match exactly.

#### 7.8.2 `spec_version`

| Version | Value   |
| ------- | ------- |
| V2      | `"2.0"` |
| V3      | `"3.0"` |

**V3 Versioning Behavior:**

- Future minor versions (e.g., `"3.1"`) may be parsed as floats for comparison.
- Applications **SHOULD NOT** reject cards with higher `spec_version` values.
- Applications **SHOULD** alert users when loading cards from newer specification versions.
- Applications **SHOULD** fill missing fields with default values when importing newer versions.

---

## 8. Lorebook (Character Book)

The lorebook (called `character_book` in V2 and `character_book` in V3) provides contextual
information entries that are conditionally inserted into prompts.

### 8.1 Lorebook Container Fields

#### 8.1.1 `name`

| Property       | Value              |
| -------------- | ------------------ |
| Type           | `Optional<String>` |
| Used in Prompt | **SHOULD NOT**     |

Identifier for the lorebook. Not used in prompt engineering.

#### 8.1.2 `description`

| Property       | Value              |
| -------------- | ------------------ |
| Type           | `Optional<String>` |
| Used in Prompt | **SHOULD NOT**     |

Description or notes about the lorebook.

#### 8.1.3 `scan_depth`

| Property | Value               |
| -------- | ------------------- |
| Type     | `Optional<Integer>` |

The number of recent messages to scan when checking for key matches. If not specified,
implementation-defined default applies.

#### 8.1.4 `token_budget`

| Property | Value               |
| -------- | ------------------- |
| Type     | `Optional<Integer>` |

Maximum token count for all inserted lorebook entries combined. When exceeded, lower-priority
entries are removed.

#### 8.1.5 `recursive_scanning`

| Property | Value               |
| -------- | ------------------- |
| Type     | `Optional<Boolean>` |

When `true`, lorebook entry content may trigger other lorebook entries (i.e., keys are matched
against inserted content, not just conversation history).

#### 8.1.6 `extensions`

| Property | Value                 |
| -------- | --------------------- |
| Type     | `Record<String, Any>` |
| Default  | `{}`                  |

Application-specific lorebook data. Same preservation requirements as character-level `extensions`.

#### 8.1.7 `entries`

| Property | Value                  |
| -------- | ---------------------- |
| Type     | `Array<LorebookEntry>` |
| Required | Yes                    |

The lorebook entries. **MAY** be an empty array.

### 8.2 Lorebook Entry Fields

#### 8.2.1 `keys`

| Property | Value           |
| -------- | --------------- |
| Type     | `Array<String>` |
| Required | Yes             |

Trigger words or patterns. An entry matches if the chat log contains any of these keys (subject to
other conditions).

**Standard Mode (`use_regex` = `false` or absent):**

- Keys are matched as literal substrings.
- Matching is case-insensitive by default (see `case_sensitive`).

**Regex Mode (`use_regex` = `true`, V3 only):**

- Keys are interpreted as regular expression patterns.
- Applications **MAY** use only the first key for performance reasons.

#### 8.2.2 `secondary_keys`

| Property | Value                     |
| -------- | ------------------------- |
| Type     | `Optional<Array<String>>` |

Secondary trigger words used with `selective` mode. When `selective` is `true`, the entry matches
only if at least one primary key AND at least one secondary key are found.

**V3 Note:** Ignored when `use_regex` is `true`.

#### 8.2.3 `selective`

| Property | Value               |
| -------- | ------------------- |
| Type     | `Optional<Boolean>` |
| Default  | `false`             |

When `true`, enables secondary key requirement (see `secondary_keys`).

#### 8.2.4 `content`

| Property | Value    |
| -------- | -------- |
| Type     | `String` |
| Required | Yes      |

The text to insert into the prompt when the entry matches.

**Requirements:**

- If the entry matches, the content **MUST** be added to the prompt exactly once (regardless of how
  many times keys match).
- If content is empty, nothing is added to the prompt.
- Content **MAY** contain decorators (V3), which are processed and removed before insertion.

#### 8.2.5 `enabled`

| Property | Value     |
| -------- | --------- |
| Type     | `Boolean` |
| Required | Yes       |

When `false`, the entry **MUST NOT** match **IN ANY CASE**.

#### 8.2.6 `insertion_order`

| Property | Value     |
| -------- | --------- |
| Type     | `Integer` |
| Required | Yes       |

Determines the relative order of matched entries in the prompt. Lower values are inserted earlier
(higher in the prompt).

When entries have equal `insertion_order`, the result is implementation-defined.

If `priority` is not specified, entries with lower `insertion_order` **MAY** be removed first when
reaching `token_budget`.

#### 8.2.7 `case_sensitive`

| Property | Value               |
| -------- | ------------------- |
| Type     | `Optional<Boolean>` |
| Default  | `false`             |

When `true`, key matching is case-sensitive. When `false` or absent, matching is case-insensitive.

#### 8.2.8 `constant`

| Property | Value                                                  |
| -------- | ------------------------------------------------------ |
| Type     | `Optional<Boolean>` (V2), Implementation-required (V3) |
| Default  | `false`                                                |

When `true`, the entry **MUST** match regardless of key matches, subject to `enabled` and budget
constraints.

**V3 Note:** Implementation of this field is **REQUIRED** in V3 (was optional in V2).

#### 8.2.9 `use_regex` (V3 Only)

| Property | Value     |
| -------- | --------- |
| Type     | `Boolean` |
| Required | Yes (V3)  |

When `true`, `keys` are interpreted as regular expression patterns instead of literal strings.

**Implementation Notes:**

- Applications **MAY** limit regex complexity for performance.
- Applications **MAY** ignore entries with malicious regex patterns (e.g., ReDoS-vulnerable).
- The `re2` regex engine is **RECOMMENDED** for server-side implementations.

#### 8.2.10 `name`

| Property       | Value              |
| -------------- | ------------------ |
| Type           | `Optional<String>` |
| Used in Prompt | **SHOULD NOT**     |

Human-readable identifier for the entry. Not used in prompt engineering.

#### 8.2.11 `id`

| Property       | Value                              |
| -------------- | ---------------------------------- |
| Type           | `Optional<Union<Integer, String>>` |
| Used in Prompt | **SHOULD NOT**                     |

Machine identifier for the entry. V2 specifies `Integer`; V3 allows `Integer` or `String`.

#### 8.2.12 `comment`

| Property       | Value              |
| -------------- | ------------------ |
| Type           | `Optional<String>` |
| Used in Prompt | **SHOULD NOT**     |

Notes about the entry. Not used in prompt engineering.

#### 8.2.13 `priority`

| Property | Value               |
| -------- | ------------------- |
| Type     | `Optional<Integer>` |

Priority for budget-constrained removal. Lower values are removed first when `token_budget` is
reached.

#### 8.2.14 `position`

| Property | Value                                            |
| -------- | ------------------------------------------------ |
| Type     | `Optional<Enum { "before_char", "after_char" }>` |

Position relative to character definitions.

- `"before_char"`: Insert before character definition fields.
- `"after_char"`: Insert after character definition fields.

---

## 9. Decorators

Decorators are a V3 feature providing inline modifiers for lorebook entry behavior. They appear at
the beginning of the `content` field.

### 9.1 Syntax

```
@@decorator_name [value]
```

**Rules:**

- Decorators start with `@@` and end with a newline.
- Values are space-separated from the decorator name.
- Multiple values are comma-separated (no spaces).
- Boolean decorators (flags) have no value.
- All decorators **MUST** appear before any non-decorator content.
- Decorators **MUST** be removed from content before prompt insertion.
- Unrecognized decorators **MUST** be ignored.
- If multiple decorators of the same name appear, only the first is considered (unless specified
  otherwise).

**Fallback Syntax:**

Fallback decorators use `@@@` and follow their primary decorator:

```
@@primary_decorator value
@@@fallback_decorator fallback_value
```

If the application does not recognize `@@primary_decorator`, it checks `@@@fallback_decorator`.
Multiple fallbacks are checked in order.

### 9.2 Activation Decorators

#### 9.2.1 `@@activate_only_after`

**Value:** Integer  
**Behavior:** The entry **SHOULD NOT** match until the chat log's assistant message count exceeds
the specified value.

If message counting is not possible, the entry **SHOULD** NOT match until the Nth user input is
received.

#### 9.2.2 `@@activate_only_every`

**Value:** Integer  
**Behavior:** The entry **SHOULD NOT** match unless the chat log's assistant message count is
divisible by the specified value.

If message counting is not possible, the fallback behavior is implementation-defined.

#### 9.2.3 `@@keep_activate_after_match`

**Value:** None (flag)  
**Behavior:** Once the entry matches, it **SHOULD** remain active **IN ANY CASE** for subsequent
messages, even if keys no longer match.

#### 9.2.4 `@@dont_activate_after_match`

**Value:** None (flag)  
**Behavior:** Once the entry matches, it **SHOULD NOT** match again **IN ANY CASE** for subsequent
messages.

#### 9.2.5 `@@activate`

**Value:** None (flag)  
**Behavior:** The entry **SHOULD** match **IN ANY CASE**, regardless of key matches or other
conditions.

#### 9.2.6 `@@dont_activate`

**Value:** None (flag)  
**Behavior:** The entry **SHOULD NOT** match **IN ANY CASE**, unless `@@activate` is also present.
Useful for disabling entries or as fallback targets.

### 9.3 Positioning Decorators

#### 9.3.1 `@@depth`

**Value:** Integer  
**Behavior:** Insert the content at the specified depth in the chat log (0 = most recent, higher =
earlier).

**Special Case (depth = 0):**

- If `@@role` is `assistant` and the environment supports prefill, insert as a prefill message.
- Multiple prefill entries are concatenated in `insertion_order`.

If chat-based insertion is not possible:

- If depth > total message count: insert in a low-priority position.
- Otherwise: insert in a high-priority position.

Ignored if `@@position` is present.

#### 9.3.2 `@@instruct_depth`

**Value:** Integer  
**Behavior:** Same as `@@depth` but measures depth in tokens rather than messages. Suitable for
non-chat contexts.

Ignored if `@@position` is present.

#### 9.3.3 `@@reverse_depth`

**Value:** Integer  
**Behavior:** Same as `@@depth` but counting from the oldest message. Equivalent to
`@@depth <total_messages - value>`.

Ignored if `@@position` or `@@depth` is present.

#### 9.3.4 `@@reverse_instruct_depth`

**Value:** Integer  
**Behavior:** Same as `@@instruct_depth` but counting from the beginning. Equivalent to
`@@instruct_depth <total_tokens - value>`.

Ignored if `@@position` or `@@instruct_depth` is present.

#### 9.3.5 `@@position`

**Value:** String  
**Behavior:** Insert content at a named position in the prompt structure.

| Value         | Position                       |
| ------------- | ------------------------------ |
| `after_desc`  | After the `description` field  |
| `before_desc` | Before the `description` field |
| `personality` | In the personality section     |
| `scenario`    | In the scenario section        |

Applications **MAY** support additional positions. Unrecognized positions **SHOULD** be ignored.

#### 9.3.6 `@@role`

**Value:** `"assistant"`, `"system"`, or `"user"`  
**Behavior:** Treat the content as having the specified role. Relevant for chat-format APIs.

### 9.4 Scanning Decorators

#### 9.4.1 `@@scan_depth`

**Value:** Integer  
**Behavior:** Override the lorebook's `scan_depth` for this entry only.

#### 9.4.2 `@@instruct_scan_depth`

**Value:** Integer  
**Behavior:** Same as `@@scan_depth` but measured in tokens rather than messages.

### 9.5 Conditional Decorators

#### 9.5.1 `@@is_greeting`

**Value:** Integer  
**Behavior:** The entry **MUST NOT** match unless the active greeting index equals the specified
value.

- Index 0 = `first_mes`
- Index 1+ = corresponding element of `alternate_greetings`

Ignored if greeting detection is not possible.

#### 9.5.2 `@@is_user_icon`

**Value:** String  
**Behavior:** The entry **SHOULD NOT** match unless the active user icon's `name` equals the
specified value.

Ignored if user icon detection is not possible.

#### 9.5.3 `@@ignore_on_max_context`

**Value:** None (flag)  
**Behavior:** The entry **SHOULD NOT** match when context is at maximum capacity, or **SHOULD** be
trimmed first when maximum context is reached.

Ignored if context limit detection is not possible.

### 9.6 Key Modifier Decorators

#### 9.6.1 `@@additional_keys`

**Value:** Comma-separated strings  
**Behavior:** The entry **SHOULD NOT** match unless the chat log contains at least one of these
additional keys (in addition to the primary `keys` requirement).

This decorator **MAY** appear multiple times (values accumulate).

When `use_regex` is `true`, values are treated as regex patterns.

#### 9.6.2 `@@exclude_keys`

**Value:** Comma-separated strings  
**Behavior:** The entry **SHOULD NOT** match if the chat log contains any of these keys.

Ignored when `use_regex` is `true`.

### 9.7 UI Control Decorators

#### 9.7.1 `@@disable_ui_prompt`

**Value:** String (`"post_history_instructions"` or `"system_prompt"`)  
**Behavior:** Applications **MAY** disable the specified UI-level prompt when this entry is active.

Applications **MAY** define additional UI prompt types.

### 9.8 V2 Backfill

When converting V3 to V2 format, all decorators **SHOULD** be removed from content.

---

## 10. Curly Braced Syntax (Macros)

Curly Braced Syntax (CBS), also called macros, provides dynamic value substitution in text fields.

### 10.1 Character and User Macros

#### 10.1.1 `{{char}}`

**Aliases:** `<char>`, `<bot>`, `<BOT>` (V1 only)  
**Replacement:** The character's `nickname` (if present and non-empty) or `name`.

#### 10.1.2 `{{user}}`

**Aliases:** `<USER>` (V1 only)  
**Replacement:** The user's display name or current persona name.

### 10.2 Randomization Macros

#### 10.2.1 `{{random:A,B,C...}}`

**Replacement:** One randomly selected value from the comma-separated list.

**Escape:** Use `::` for literal commas (e.g., `{{random:a::b,c}}` may produce `"a,b"` or `"c"`).

#### 10.2.2 `{{pick:A,B,C...}}`

**Replacement:** One value from the comma-separated list, selected to be consistent across identical
conditions within the same context.

Applications **SHOULD** make effort to return the same value for identical `{{pick:...}}`
expressions in the same generation context.

#### 10.2.3 `{{roll:N}}`

**Replacement:** A random integer from 1 to N (inclusive).

**Alternate Syntax:** `{{roll:dN}}` or `{{roll:DN}}` (e.g., `{{roll:d6}}` = `{{roll:6}}`).

### 10.3 Comment Macros

#### 10.3.1 `{{// A}}`

**Replacement:** Empty string.  
**Behavior:** The content `A` is discarded entirely. **SHOULD NOT** be used for lorebook key
matching.

#### 10.3.2 `{{hidden_key:A}}`

**Replacement:** Empty string.  
**Behavior:** Same as `{{// A}}` but the value **SHOULD** be considered for recursive lorebook
scanning.

#### 10.3.3 `{{comment: A}}`

**Replacement:** Empty string (in prompts).  
**Behavior:** The content `A` **MAY** be displayed to the user as an inline comment in the UI but
**MUST** NOT be sent to the AI model. **SHOULD NOT** be used for lorebook key matching.

### 10.4 Text Transformation Macros

#### 10.4.1 `{{reverse:A}}`

**Replacement:** The string `A` with characters in reverse order.

**Example:** `{{reverse:Hello}}` → `"olleH"`

### 10.5 Prompt Override Macros

#### 10.5.1 `{{original}}`

**Context:** Only valid within `system_prompt` and `post_history_instructions` fields.  
**Replacement:** The default/global value that would have been used if the character-specific field
were empty.

---

## 11. Placeholder Substitution

This section defines the processing requirements for placeholder substitution.

### 11.1 Fields Subject to Substitution

The following fields **MUST** have placeholders replaced:

| Field                       | V1  | V2  | V3  |
| --------------------------- | --- | --- | --- |
| `description`               | ✓   | ✓   | ✓   |
| `personality`               | ✓   | ✓   | ✓   |
| `scenario`                  | ✓   | ✓   | ✓   |
| `first_mes`                 | ✓   | ✓   | ✓   |
| `mes_example`               | ✓   | ✓   | ✓   |
| `alternate_greetings`       | —   | ✓   | ✓   |
| `group_only_greetings`      | —   | —   | ✓   |
| `system_prompt`             | —   | ✓   | ✓   |
| `post_history_instructions` | —   | ✓   | ✓   |
| Lorebook `content`          | —   | ✓   | ✓   |

### 11.2 Processing Order

1. Decorators are parsed and removed from lorebook content (V3).
2. All CBS macros are evaluated and replaced.
3. Character/user placeholders (`{{char}}`, `{{user}}`, etc.) are replaced.

### 11.3 Case Sensitivity

Placeholder matching **MUST** be case-insensitive (e.g., `{{CHAR}}`, `{{Char}}`, and `{{char}}` are
equivalent).

### 11.4 Unresolved Placeholders

Behavior for unrecognized placeholders is undefined. Applications **MAY**:

- Leave them unchanged
- Remove them
- Display a warning

---

## 12. Unresolved Ambiguities

This section documents ambiguities in the upstream specifications that cannot be resolved within
this consolidated document.

### 12.1 `{{user}}` in `name` Field

**Source:** V1 Specification  
**Issue:** Whether `{{user}}` and `<USER>` should be replaced inside the `name` field is explicitly
marked as "UNSPECIFIED."

**Recommendation:** Applications **SHOULD NOT** perform substitution in the `name` field to avoid
circular references.

### 12.2 WEBP Embedding

**Source:** V1 Specification  
**Issue:** WEBP embedding is explicitly not covered "due to technical ambiguities."

**Recommendation:** Implementations requiring WEBP support should define their own embedding
mechanism or convert to PNG.

### 12.3 Default User Name

**Source:** V1 Specification  
**Issue:** A default value for the user's name "**MUST** exist" but no specific default is mandated.

**Recommendation:** Common defaults include `"User"`, `"You"`, or `"Anon"`.

### 12.4 Lorebook Entry Ordering with Equal Values

**Issue:** Behavior when multiple entries have the same `insertion_order` is not specified.

**Recommendation:** Implementations **SHOULD** use a stable secondary sort (e.g., by array index).

### 12.5 `**MAY** NOT` vs `**MAY**` in V3

**Source:** V3 Specification  
**Issue:** The V3 spec defines `**MAY** NOT` with the adjective "forbidden" but states it means
"truly optional." This is contradictory.

**Clarification:** Based on context, `**MAY** NOT` should be interpreted as equivalent to `**MAY**`
(the item is optional to omit).

### 12.6 `{{original}}` Scope

**Source:** V2 Specification  
**Issue:** The `{{original}}` placeholder description for `post_history_instructions` incorrectly
references "system_prompt" in the replacement text.

**Clarification:** `{{original}}` in `post_history_instructions` should be replaced with the default
post-history instructions, not the system prompt.

### 12.7 V3 `group_only_greetings` Requirement

**Source:** V3 Specification  
**Issue:** The TypeScript interface shows `group_only_greetings: Array<string>` (required), but
behavioral text does not mandate a default.

**Recommendation:** Treat as required with default value `[]`.

### 12.8 PNG Chunk Naming ("Chara" vs "chara")

**Source:** V1 Specification  
**Issue:** V1 refers to "Chara" EXIF metadata, but PNG uses tEXt chunks. The chunk keyword is
`chara` (lowercase in practice).

**Clarification:** Use lowercase `chara` for the tEXt chunk keyword.

### 12.9 Prefill Concatenation Order

**Source:** V3 Specification  
**Issue:** When multiple entries with `@@depth 0` and `@@role assistant` exist, they are
concatenated "in insertion_order," but it's unclear if this means ascending or descending.

**Recommendation:** Concatenate in ascending `insertion_order` (lower values first).

### 12.10 Regex in `secondary_keys`

**Source:** V3 Specification  
**Issue:** States `secondary_keys` "**SHOULD** be ignored" when `use_regex` is true, but
`@@additional_keys` works with regex. Inconsistent behavior.

**Recommendation:** Follow specification literally; use `@@additional_keys` for regex scenarios
requiring secondary conditions.

---

## 13. Version Compatibility

### 13.1 Detection Algorithm

```
function detectVersion(data):
    if data.spec == "chara_card_v3":
        return "V3"
    if data.spec == "chara_card_v2":
        return "V2"
    if data.name exists AND data.data does not exist:
        return "V1"
    return "Unknown"
```

### 13.2 V1 to V2 Upgrade

To upgrade a V1 card to V2:

```json
{
  "spec": "chara_card_v2",
  "spec_version": "2.0",
  "data": {
    "name": "<V1.name>",
    "description": "<V1.description>",
    "personality": "<V1.personality>",
    "scenario": "<V1.scenario>",
    "first_mes": "<V1.first_mes>",
    "mes_example": "<V1.mes_example>",
    "creator_notes": "",
    "system_prompt": "",
    "post_history_instructions": "",
    "alternate_greetings": [],
    "tags": [],
    "creator": "",
    "character_version": "",
    "extensions": {}
  }
}
```

### 13.3 V2 to V3 Upgrade

To upgrade a V2 card to V3:

1. Copy all V2 `data` fields to V3 `data`.
2. Set `spec` to `"chara_card_v3"`.
3. Set `spec_version` to `"3.0"`.
4. Rename `character_book` entries to conform to V3 `Lorebook` structure.
5. Add `use_regex: false` to all lorebook entries.
6. Add `group_only_greetings: []`.

### 13.4 V3 to V2 Downgrade

To downgrade a V3 card to V2:

1. Copy compatible fields from V3 `data` to V2 `data`.
2. Set `spec` to `"chara_card_v2"`.
3. Set `spec_version` to `"2.0"`.
4. Remove all decorators from lorebook entry content.
5. Remove V3-only fields: `assets`, `nickname`, `creator_notes_multilingual`, `source`,
   `group_only_greetings`, `creation_date`, `modification_date`.
6. Remove `use_regex` from lorebook entries.
7. Convert lorebook entry `id` from string to integer if necessary.

**Warning:** V3-to-V2 downgrade is lossy. Advanced features will be lost.

### 13.5 Forward Compatibility (V3+)

For future versions beyond V3:

1. Applications **SHOULD NOT** reject cards with `spec_version` greater than `"3.0"`.
2. Applications **SHOULD** alert users that the card was created with a newer specification.
3. Applications **SHOULD** fill missing fields with defaults.
4. Applications **SHOULD** ignore unrecognized fields rather than failing.
5. Applications **MUST** preserve unrecognized fields on export.

---

## Appendix A: Complete IDL Summary

```idl
// ============================================================
// Primitive Types
// ============================================================

typedef String;          // UTF-8 encoded string, variable length
typedef Integer;         // Signed integer, minimum 32-bit
typedef Float;           // IEEE 754 double precision (64-bit)
typedef Boolean;         // true or false
typedef Any;             // Any JSON-serializable value

// ============================================================
// Generic Types
// ============================================================

typedef Array<T>;                // Dynamic array of type T
typedef Optional<T>;             // T or absent
typedef Record<K, V>;            // Map with key type K, value type V
typedef Literal<V>;              // Exact string value V
typedef Union<A, B>;             // Either A or B
typedef Enum { values... };      // One of enumerated string values

// ============================================================
// Character Card V1
// ============================================================

struct CharacterCardV1 {
    String name;
    String description;
    String personality;
    String scenario;
    String first_mes;
    String mes_example;
}

// ============================================================
// Character Card V2
// ============================================================

struct CharacterCardV2 {
    Literal<"chara_card_v2"> spec;
    Literal<"2.0"> spec_version;
    CharacterCardV2Data data;
}

struct CharacterCardV2Data {
    String name;
    String description;
    String personality;
    String scenario;
    String first_mes;
    String mes_example;
    String creator_notes;
    String system_prompt;
    String post_history_instructions;
    Array<String> alternate_greetings;
    Optional<CharacterBook> character_book;
    Array<String> tags;
    String creator;
    String character_version;
    Record<String, Any> extensions;
}

struct CharacterBook {
    Optional<String> name;
    Optional<String> description;
    Optional<Integer> scan_depth;
    Optional<Integer> token_budget;
    Optional<Boolean> recursive_scanning;
    Record<String, Any> extensions;
    Array<CharacterBookEntry> entries;
}

struct CharacterBookEntry {
    Array<String> keys;
    String content;
    Record<String, Any> extensions;
    Boolean enabled;
    Integer insertion_order;
    Optional<Boolean> case_sensitive;
    Optional<String> name;
    Optional<Integer> priority;
    Optional<Integer> id;
    Optional<String> comment;
    Optional<Boolean> selective;
    Optional<Array<String>> secondary_keys;
    Optional<Boolean> constant;
    Optional<Enum { "before_char", "after_char" }> position;
}

// ============================================================
// Character Card V3
// ============================================================

struct CharacterCardV3 {
    Literal<"chara_card_v3"> spec;
    Literal<"3.0"> spec_version;
    CharacterCardV3Data data;
}

struct CharacterCardV3Data {
    String name;
    String description;
    String personality;
    String scenario;
    String first_mes;
    String mes_example;
    String creator_notes;
    String system_prompt;
    String post_history_instructions;
    Array<String> alternate_greetings;
    Array<String> tags;
    String creator;
    String character_version;
    Record<String, Any> extensions;
    Optional<Lorebook> character_book;
    Optional<Array<Asset>> assets;
    Optional<String> nickname;
    Optional<Record<String, String>> creator_notes_multilingual;
    Optional<Array<String>> source;
    Array<String> group_only_greetings;
    Optional<Integer> creation_date;
    Optional<Integer> modification_date;
}

struct Asset {
    String type;
    String uri;
    String name;
    String ext;
}

struct Lorebook {
    Optional<String> name;
    Optional<String> description;
    Optional<Integer> scan_depth;
    Optional<Integer> token_budget;
    Optional<Boolean> recursive_scanning;
    Record<String, Any> extensions;
    Array<LorebookEntry> entries;
}

struct LorebookEntry {
    Array<String> keys;
    String content;
    Record<String, Any> extensions;
    Boolean enabled;
    Integer insertion_order;
    Optional<Boolean> case_sensitive;
    Boolean use_regex;
    Optional<Boolean> constant;
    Optional<String> name;
    Optional<Integer> priority;
    Optional<Union<Integer, String>> id;
    Optional<String> comment;
    Optional<Boolean> selective;
    Optional<Array<String>> secondary_keys;
    Optional<Enum { "before_char", "after_char" }> position;
}

struct LorebookExport {
    Literal<"lorebook_v3"> spec;
    Lorebook data;
}

// ============================================================
// Union Type for Multi-Version Support
// ============================================================

typedef CharacterCard = Union<CharacterCardV1, CharacterCardV2, CharacterCardV3>;
```

---

## Appendix B: Decorator Quick Reference

| Decorator                     | Value Type | Description                               |
| ----------------------------- | ---------- | ----------------------------------------- |
| `@@activate_only_after`       | Integer    | Match only after N messages               |
| `@@activate_only_every`       | Integer    | Match only every N messages               |
| `@@keep_activate_after_match` | —          | Stay active after first match             |
| `@@dont_activate_after_match` | —          | Deactivate after first match              |
| `@@activate`                  | —          | Always match                              |
| `@@dont_activate`             | —          | Never match (unless `@@activate` present) |
| `@@depth`                     | Integer    | Insert at message depth                   |
| `@@instruct_depth`            | Integer    | Insert at token depth                     |
| `@@reverse_depth`             | Integer    | Insert at reverse message depth           |
| `@@reverse_instruct_depth`    | Integer    | Insert at reverse token depth             |
| `@@position`                  | String     | Insert at named position                  |
| `@@role`                      | String     | Set message role                          |
| `@@scan_depth`                | Integer    | Override scan depth for this entry        |
| `@@instruct_scan_depth`       | Integer    | Override scan depth (tokens)              |
| `@@is_greeting`               | Integer    | Match only for specific greeting          |
| `@@is_user_icon`              | String     | Match only for specific user icon         |
| `@@ignore_on_max_context`     | —          | Skip when context is full                 |
| `@@additional_keys`           | String,... | Require additional keys                   |
| `@@exclude_keys`              | String     |

## Appendix C: Data Structure's Conceptual Diagram

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="CC_Specs_Dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="CC_Specs_Light.svg">
  <img 
    alt="Diagram showing relationship and structures of an implementation of the specifications"
    src="default-image.png"
  >
</picture>
