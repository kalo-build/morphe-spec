# Morphe Specification (KA:MO1) - Unified Application Data Modeling

## Table of Contents

- [Morphe - Application Data Modeling Specification](#morphe---application-data-modeling-specification)
  - [Overview](#overview)
  - [Specification Versions](#specification-versions)
  - [Key Features](#key-features)
  - [Specification Features](#specification-features)
  - [Motivation](#motivation)
  - [Field Types](#field-types)
    - [Atomic Field Types](#atomic-field-types)
    - [Enumeration Field Types](#enumeration-field-types)
  - [Structures](#structures)
  - [Enums](#enums)
  - [Models](#models)
    - [Model Fields](#model-fields)
    - [Identifiers](#identifiers)
    - [Related](#related)
      - [Supported Ownership Values](#supported-ownership-values)
      - [Supported Cardinality Values](#supported-cardinality-values)
      - [Relation Attributes](#relation-attributes)
  - [Entities](#entities)
    - [Entity Fields](#entity-fields)
      - [Indirected Types](#indirected-types)
    - [Entity Identifiers](#entity-identifiers)
    - [Related](#related-1)
      - [Supported Ownership Values](#supported-ownership-values-1)
      - [Supported Cardinality Values](#supported-cardinality-values-1)
      - [Relation Attributes](#relation-attributes-1)
  - [Contributing](#contributing)
  - [License](#license)

## Overview

Morphe (KA:MO1) is a simple, human-readable base data modeling specification designed to facilitate consistent transpilation of models into multiple programming languages.

The name represents the ancient Greek "form" or "shape", implying an ideal form or prototype from which other forms are derived. This symbolizes how declaratively modelled data is generatively transformed into code across different formats and languages.

## Specification Versions

Version | Status | Description | Docs
--------|---------|------------|------
KA:MO1 | 🚧 In Progress | First stable release of the Morphe core specification | This document
KA:MO1:YAML1 | 🚧 In Progress | YAML format standard for KA:MO1 | [format/YAML.md](format/YAML.md)
KA:MO1:TS1 | 🚧 In Progress | TypeScript format standard for KA:MO1 | [format/TS.md](format/TS.md)
KA:MO1:GO1 | 🚧 In Progress | Go format standard for KA:MO1 | [format/GO.md](format/GO.md)
KA:MO1:PSQL1 | 🚧 In Progress | PostgreSQL format standard for KA:MO1 | [format/PSQL.md](format/PSQL.md)

## Specification Hierarchy

The Morphe specification system uses a hierarchical naming scheme to clearly identify different types of specifications:

### Base Specification (`KA:MO1`)
- `KA:` - Organization/context prefix
- `MO1` - Base specification identifier (Morphe version 1)
- *Example:* `KA:MO1` - The core Morphe data modeling specification

### Format Specification (`KA:MO1:YAML1`)
- Extends base specification with format identifier
- `YAML1` - Format representation identifier (YAML version 1)
- *Example:* `KA:MO1:YAML1` - The YAML format specification for Morphe

### Base Format
To rally ecosystem support around a canonical standard, specifications can define a "base format" that serves as the primary reference implementation. For Morphe, **YAML** (`KA:MO1:YAML1`) serves as the base format, providing:

- **Canonical Reference**: All examples and documentation use YAML as the primary format
- **Ecosystem Alignment**: Plugin developers can focus on YAML-first implementations
- **Interoperability**: A common format that all tools and implementations can understand
- **Developer Experience**: Consistent, human-readable syntax across the ecosystem

Other formats (Go, TypeScript, PostgreSQL) are considered transpilation targets from the base YAML format.

### Examples in Base Specification
This specification uses **YAML examples** rather than pseudo-code for the following reasons:

**Advantages of YAML Examples:**
- **Concrete Implementation**: Provides actionable, copy-pasteable examples
- **Canonical Reference**: Serves as the authoritative syntax reference
- **Developer Experience**: Immediately usable examples reduce learning curve
- **Consistency**: Aligns with the base format used across the ecosystem
- **Validation**: Real syntax can be tested and validated

**Trade-offs Considered:**
- **Format Coupling**: Examples are tied to YAML syntax, but this is mitigated by YAML being the canonical base format
- **Abstraction Level**: Less abstract than pseudo-code, but the concrete examples provide more value for implementation

Since YAML serves as Morphe's base format, using YAML examples creates a cohesive developer experience while maintaining the specification's practical utility.

### Motivation

This hierarchical naming scheme provides several benefits:
1. Clear separation between base concepts and format representations
2. Support for multiple format representations of the same base specification
3. Ability to transpile between different formats (e.g., YAML to Go, YAML to TypeScript)
4. Version tracking at each level of the specification
5. Extensibility for future formats

### Example Specification Types

1. Base Specifications:
   - `KA:MO1` - Core Morphe data modeling specification

2. Format Specifications:
   - `KA:MO1:YAML1` - YAML format representation
   - `KA:MO1:GO1` - Go format representation
   - `KA:MO1:TS1` - TypeScript format representation
   - `KA:MO1:PSQL1` - PostgreSQL format representation

## Key Features

1. 📖 **Single Source of Truth**
   * Define your data models once in a structured format
   * Automatic code generation across your stack
   * Consistent type definitions across languages
   * Reduced manual synchronization effort

2. 🔄 **Rich Type System**
   * Built-in primitive types (UUID, String, Integer, etc.)
   * Custom enumeration support
   * Relationship modeling (HasOne, HasMany, etc.)
   * Support for protected and sealed fields

3. 🔧 **Flexible Architecture**
   * `Models` for persisted data structures
   * `Structures` for reusable field groups
   * `Entities` for business data aggregation
   * `Enums` for constant value sets
   * Clear separation of concerns

4. 📏 **Standardized Versioning**
   * Core spec versioning (KAMO<x>)
   * Language-specific standards (KAMO<x>-LANG<y>)
   * Backward compatibility guidelines
   * Clear upgrade paths

## Specification Features

The Morphe specification defines a comprehensive set of features that compile plugins can implement. Each plugin can document which features it supports, ensuring clarity about capabilities and limitations.

### Core Features

| Feature | Description | Spec Status |
|---------|-------------|--------|
| **Models** | Primary persisted data structures with fields, identifiers, and relationships | ✅ Complete |
| **Entities** | Business data aggregation structures that reference model fields | ✅ Complete |
| **Enums** | Predefined constant value sets with primitive types | ✅ Complete |
| **Structures** | Standalone, non-persisted field groupings for DTOs | ✅ Complete |

### Advanced Features

| Feature | Description | Spec Status |
|---------|-------------|--------|
| **EnumFields** | Using enums as field types in models, entities, and structures | ✅ Complete |
| **StructureComposition** | Using structures or enums as field types within structures (nested DTOs) | ✅ Complete |
| **ModelRelationPolymorphism** | Polymorphic relationships (HasOnePoly, HasManyPoly, ForOnePoly, ForManyPoly) in models | ✅ Complete |
| **EntityRelationPolymorphism** | Polymorphic relationships in entities | ✅ Complete |
| **ModelRelationAttributes** | Attributes on model relationships (e.g., `optional`) | ✅ Complete |
| **EntityRelationAttributes** | Attributes on entity relationships (e.g., `optional`) | ✅ Complete |
| **ModelRelationAliasing** | Custom relationship naming and aliasing in models | ✅ Complete |
| **EntityRelationAliasing** | Custom relationship naming and aliasing in entities | ✅ Complete |

### Plugin Implementation

Compile plugins should document their feature support using this standardized list. For example:

```yaml
# Plugin Feature Matrix
supported_features:
  - Models
  - Entities
  - Enums
  - ModelRelationPolymorphism
  - EntityRelationPolymorphism
unsupported_features:
  - ModelRelationAttributes
  - EntityRelationAttributes
  - ModelRelationAliasing
  - EntityRelationAliasing
  - EnumFields
  - Structures
```

This ensures users understand exactly what functionality is available when using specific plugins.

## Motivation

As your application's codebase grows, structural changes become increasingly expensive. 

Let's take something as simple as adding a `country` field to addresses managed by your application. To achieve this across the stack, you typically need to manually update the following things:

* `address` SQL table creation scripts
* `address` SQL queries
* Backend type definitions, such as an `Address` class
* Associated automated test arrangement and assertion blocks
* API endpoint documentation
* Front-end type definitions, such as an `Address` typescript type
* Front-end form components
* ...

Tooling exists to reduce the manual work needed for some of these steps. Common examples might be an ORM that generates SQL scripts automatically from backend types or GraphQL typescript type generators from GraphQL query files. But these tools are fragmented and tend to leverage multiple disparate "sources of truth" across your application.

The Morphe specification aims to provide a simple schema for automatically updating your code artifacts across technologies, eliminating human error, and accelerating development velocity. 

By consolidating your application's data structures into a single source of truth for your entire stack, you only have to touch one place instead of many and let your selected plugins handle the rest.

## Field Types

### Atomic Field Types

Atomic field types are type primitives that represent a single, indivisible unit of data required for defining fields on higher-order data structures.

* `UUID`: A RFC-4122 compatible UUID string
* `AutoIncrement`: A classic numeric, auto-incrementable record ID
* `String`: A variable-length string
* `Integer`: A numeric value for zero, whole numbers, and their negative counterparts
* `Float`: A numeric floating-point decimal value
* `Boolean`: A boolean (true / false) value
* `Time`: A timestamp value with a UTC offset
* `Date`: A timestamp value as a date with a UTC offset and zero time values
* `Protected`: An encryptable and decryptable value for sensitive information such as API keys
* `Sealed`: A hashable value that can not be decrypted (typically passwords)

### Enumeration Field Types

Enums can be used as field types within models, entities, and structures by referencing their name. When used as a field type, the enum enforces that only predefined values from the enum's entries can be assigned to that field.

*Example:* Using `Nationality` enum as a field type

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
  FirstName:
    type: String
  LastName:
    type: String
  Nationality:
    type: Nationality  # References the Nationality enum
```

See the [Enums](#enums) section for complete enum definition syntax and examples.

## Structures

Structures represent standalone, reusable, non-persisted groupings of fields, for more flexible use inside of application projects, for example as subtypes or DTOs (data-transfer-objects).

Each structure consists of a `name`, and a set of `fields`.

**Note:** Unlike models, structures do not have relationships or identifiers, as they are not persisted. Structure fields may use [atomic types](#atomic-field-types), [enum types](#enumeration-field-types), or other structures (composition).

*Example:* `address.str` (primitive fields only)

```yaml
name: Address
fields:
  Street:
    type: String
  HouseNr:
    type: String
  ZipCode:
    type: String
  City:
    type: String
  Unit:
    type: String
    attributes:
      - optional
```

*Example:* Structure with an enum field (`invoice.str`)

```yaml
name: Invoice
fields:
  ID:
    type: String
  Status:
    type: InvoiceStatus   # References the InvoiceStatus enum
  Amount:
    type: Integer
```

*Example:* Structure composition — one structure embedding another (`invoice_with_line_item.str`)

```yaml
name: InvoiceWithLineItem
fields:
  ID:
    type: String
  LineItem:
    type: InvoiceLineItem   # References another structure (nested DTO)
    attributes:
      - optional
```

Where `InvoiceLineItem` is a separate structure (e.g., in `invoice_line_item.str`). This is composition, not a relational reference: the embedded structure is part of the same DTO.

Structure fields support the same [first-class attributes](#first-class-attributes) as model fields. In particular, the `optional` attribute can be used to mark individual fields as not required.

## Enums

Enums are predefined sets of constant values that can be used as types for fields within models, entities, and structures. They enforce consistency by limiting possible values to a fixed set.

Each enum consists of a `name`, a primitive `type` (`String`, `Integer`, `Float`), and a set of `entries`. Entries are defined as symbol names with an associated primitive representation.

*Example:* `nationality.enum`

```yaml
name: Nationality
type: String
entries:
  US: 'American'
  DE: 'German'
  FR: 'French'
```

*Example:* `universal-number.enum`

```yaml
name: UniversalNumber
type: Float
entries:
  Pi: 3.1415926535
  Euler: 2.7182818285
```

### Enum Usage

Enums can be used in three ways:

1. **As Field Types**: Reference the enum name as a field type (see [Enumeration Field Types](#enumeration-field-types))
2. **Data Validation**: Ensure only valid values are assigned to fields
3. **Code Generation**: Generate constants, types, or lookup tables in target languages

## Models

Models are the primary persisted data structures. They are somewhat analogous to relational SQL tables.

Each model consists of a `name`, a set of `fields`, a set of `identifiers`, and `related` models.

*Example:* `person.mod`

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
  FirstName:
    type: String
  LastName:
    type: String
  Nationality:
    type: Nationality
  OrganizationID:
    type: UUID
    attributes:
      - optional
identifiers:
  primary: ID
  name:
    - FirstName
    - LastName
related:
  ContactInfo:
    type: HasOne
  Company:
    type: ForOne
```

### Model Fields

Denoted by the `fields:` key, the model fields are specified as a uniquely named key and the finer-grained field configuration.

Each field has exactly one type.

Each field may have a list of lower, snake-case attributes. Attributes are used to convey additional field-level semantics to transpiling plugins.

#### First-Class Attributes

The following attributes are recognized as first-class by many Morphe plugins:

| Attribute  | Applies to | Description |
|------------|------------|-------------|
| `optional` | Models, Structures, Entities | Marks a field as optional. All fields are **required by default**. When `optional` is present, the generated output uses the language-appropriate nullable/optional representation (e.g., `*T` in Go, `T?` in TypeScript, `.optional()` in Zod). |
| `immutable` | Entities | Marks a field as immutable (read-only after creation). Preserved as metadata in generated struct tags. |

Additional, unconstrained attributes may also be specified and will be passed through to transpiling implementations.

### Identifiers

Denoted by the `identifiers:` key, this section always requires a `primary:` identifier with a corresponding field name.

Composite or secondary identifier groups can be added with a unique key (i.e. `addr:`), and a list of relevant fields. Identifier fields prefixed with `rel:` reference a relation declared in the `related:` section. Transpiling plugins resolve `rel:` references to the appropriate storage-level key(s) (e.g., a `ForOne` relation becomes the foreign key column). Only `ForOne` and `ForOnePoly` relations may be referenced in identifiers, as they represent local foreign keys.

*Example:* Composite identifier referencing relations

```yaml
name: TaskTag
fields:
  ID:
    type: UUID
identifiers:
  primary: ID
  taskTag:
    - rel:Task
    - rel:Tag
related:
  Task:
    type: ForOne
  Tag:
    type: ForOne
```

In this example, the `taskTag` identifier ensures uniqueness across the `Task` and `Tag` relations. The `rel:` prefix explicitly marks these as relation references rather than field names. Plugins resolve each `rel:` reference to the appropriate storage-level key(s) in generated output.

### Related

Denoted by the `related:` key, this section lists all related models. This means both dependencies and dependents (prerequisites).

Each relationship is characterized by an ownership and a cardinality that differentiates between 1 and n related entities.

#### Supported Ownership Values

* `For`: The current model is "for" related models, when the current model is a dependent on the existence of related models.
  * *Example:* "Address ForOne User": Address is reliant on the existence of a specified User, because Address explicitly references User identifiers.
* `Has`: The current model "has" related models, when the current model is required by the related models.
  * *Example:* "Address HasMany Document": Address supports the reliance of Documents on the existence of specified Addresses, because Documents explicitly reference Address identifiers.

#### Supported Cardinality Values

* `One`: Relationships have a cardinality of one, when the current model's relationship supports one related model instance.
  * *Example:* "Address ForOne User": Address is related to one user.
* `Many`: Relationships have a cardinality of many, when the current model's relationship supports a variable number of related model instances.
  * *Example:* "Owner HasMany Address": Owner is related to many addresses.

#### Relation Attributes

Both model and entity relations may have a list of `attributes` that convey additional semantics to transpiling plugins. This mirrors the `attributes` mechanism on fields but applies to the relationship itself.

| Attribute  | Applies to | Description |
|------------|------------|-------------|
| `optional` | Model and entity relations | Marks the relationship as optional. All relations are **required by default**. When `optional` is present, transpiling plugins use the appropriate representation (e.g., nullable foreign keys in SQL, pointer types in Go, optional properties in TypeScript). |

Additional, unconstrained attributes may also be specified and will be passed through to transpiling implementations.

#### Relation Field Generation Semantics

Each `ForOne` / `ForOnePoly` relation produces two generated fields: a **foreign key (FK) field** and a **relation object field**. These have distinct semantics:

| Concern | FK Field (e.g., `OrganizationID`) | Relation Object (e.g., `Organization`) |
|---------|----------------------------------|----------------------------------------|
| Represents | Data ownership constraint | Optionally loaded related data |
| Required relation | Non-nullable (e.g., Go `string`, SQL `NOT NULL`) | Always optional/nullable (e.g., Go `*Organization`) |
| Optional relation | Nullable (e.g., Go `*string`, SQL nullable) | Always optional/nullable (e.g., Go `*Customer`) |
| Rationale | FK is always present in the row | Relation object is only populated after a JOIN/eager load |

**FK fields** follow the `optional` attribute directly — required relations produce non-nullable FKs, optional relations produce nullable FKs. This maps cleanly to SQL `NOT NULL` vs nullable column semantics.

**Relation object fields** are always nullable/optional regardless of whether the relation itself is required. The FK guarantees the relationship exists, but the full related record may or may not have been loaded. A nil/undefined relation object means "not fetched," not "no relationship."

`HasMany` / `HasManyPoly` relations produce slice/array fields (e.g., Go `[]Task`, TS `Task[]`), where nil/empty represents "not loaded" or "none."

*Example:* Expected generated output for a model with both required and optional relations

```yaml
name: Task
related:
  Project:
    type: ForOne                    # required
  Assignee:
    type: ForOne
    aliased: Membership
    attributes:
      - optional                    # optional
```

Go output:
```go
type Task struct {
    ProjectID   string       // required FK — non-nullable
    Project     *Project     // relation object — always pointer
    AssigneeID  *string      // optional FK — nullable
    Assignee    *Membership  // relation object — always pointer
}
```

TypeScript output:
```typescript
type Task = {
    projectID: string;        // required FK
    project?: Project;        // relation object — always optional
    assigneeID?: string;      // optional FK
    assignee?: Membership;    // relation object — always optional
}
```

*Example:* Optional model relationship

```yaml
name: Task
fields:
  ID:
    type: UUID
  Title:
    type: String
identifiers:
  primary: ID
related:
  Project:
    type: ForOne
    attributes:
      - optional
  Author:
    type: ForOne
```

In this example, `Project` is an optional dependency (a task may exist without a project), while `Author` is required.

### Polymorphic Relationships

Polymorphic relationships allow models to be related to multiple different model types through a single association. Morphe supports four types of polymorphic relationships:

#### HasOnePoly and HasManyPoly (with `through`)

These relationships indicate that the current model "has" related models through a polymorphic interface.

- `HasOnePoly`: One-to-one polymorphic relationship
- `HasManyPoly`: One-to-many polymorphic relationship  
- `through`: Specifies the polymorphic interface name
- `aliased`: (Optional) Specifies the actual target model name when different from the relationship name

*Example:* `person.mod` with polymorphic relationships

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
  FirstName:
    type: String
  LastName:
    type: String
identifiers:
  primary: ID
related:
  Comment:
    type: HasOnePoly
    through: Commentable
  Tag:
    type: HasManyPoly
    through: Taggable
```

#### ForOnePoly and ForManyPoly (with `for`)

These relationships indicate that the current model belongs to one or many of the specified model types.

- `ForOnePoly`: Many-to-one polymorphic relationship
- `ForManyPoly`: Many-to-many polymorphic relationship
- `for`: Specifies the list of possible target model types

*Example:* `comment.mod` with polymorphic relationship

```yaml
name: Comment
fields:
  ID:
    type: AutoIncrement
  Content:
    type: String
  CreatedAt:
    type: String
identifiers:
  primary: ID
related:
  Commentable:
    type: ForOnePoly
    for:
      - Person
      - Company
```

*Example:* `tag.mod` with polymorphic relationship

```yaml
name: Tag
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
  Color:
    type: String
identifiers:
  primary: ID
related:
  Taggable:
    type: ForManyPoly
    for:
      - Person
      - Company
```

### Relationship Aliasing

Morphe supports relationship aliasing, which allows you to use custom names for relationships while referencing existing model types. This is particularly useful when a model needs multiple relationships to the same target model type.

#### Model Relationship Aliasing

Use the `aliased` property to specify the actual target model:

*Example:* `person.mod` with aliased relationships

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
identifiers:
  primary: ID
related:
  WorkContact:
    type: ForOne
    aliased: Contact  # References the Contact model
  PersonalContact:
    type: ForOne
    aliased: Contact  # Same target model with different relationship name
  WorkProject:
    type: ForMany
    aliased: Project
  PersonalProject:
    type: ForMany
    aliased: Project
```

#### Polymorphic Relationship Aliasing

Like regular relationships, polymorphic relationships support the `aliased` property to decouple the relationship name from the target model name. When `aliased` is not specified, the relationship name itself is used as the target model name.

*Example:* `post.mod` with polymorphic aliasing for semantic field naming

```yaml
name: Post
fields:
  ID:
    type: AutoIncrement
  Title:
    type: String
identifiers:
  primary: ID
related:
  Note:  # Semantic field name (used in generated code)
    type: HasOnePoly
    through: Commentable
    aliased: Comment  # Actual model type (Comment model must exist)
  Comment:  # Without aliased, this references a Comment model directly
    type: HasManyPoly
    through: Commentable
```

This pattern is particularly useful for creating meaningful field names in generated code while maintaining the polymorphic relationship structure.

## Entities

Entities are indirect data structures that route internally to model field subsets for business data flattening and aggregation. They are analogous to SQL views, decoupling domain data from underlying technical data structures (Models).

Entities consist of a `name`, a set of `fields`, `identifiers`, and related `entities`. Identifiers and field types are inherited from models.

*Example:* `person.ent`

```yaml
name: Person
fields:
  ID:
    type: Person.ID
    attributes:
      - immutable
  LastName:
    type: Person.LastName
  Nationality:
    type: Person.Nationality
  Email:
    type: Person.ContactInfo.Email
identifiers:
  primary: ID
related:
  Company:
    type: ForOne
```

### Entity Fields

Denoted by the `fields:` key, the entity fields are specified as a uniquely named key (example: `Street:`) and the finer-grained field configuration.

Each field may have a list of lower, snake-case attributes. The same [first-class attributes](#first-class-attributes) defined for model fields apply to entity fields. The `immutable` attribute is particularly common on entity ID fields to indicate they should not be modified after creation.

### Entity Identifiers

Denoted by the `identifiers:` key, this section specifies how the entity can be uniquely identified. Entity identification strategies will be expanded in future versions to support various approaches like virtual/composite keys, hierarchical IDs, and surrogate bridging.

Currently, entity identification supports globally unique IDs from a single referenced model field.

#### Indirected Types

Entity field types are indirected model field paths. The path begins with a root model (*Example:* `type: User.Address.Street` -> `User` as the "root model"), and includes related models, terminating in a field of the last related model name. The field inherits the type from the terminal model.

### Entity Layer vs. Model Layer

Entities operate across two distinct layers:

1. **Field resolution (model layer):** Entity field types are indirected paths that traverse **model-level** relations. For example, `Task.Assignee.User.FirstName` walks the `Task` model's `Assignee` relation (which aliases `Membership`), then the `Membership` model's `User` relation, and finally resolves the `FirstName` field from the `User` model. No entity-level relation declaration is required for field path traversal — the model layer handles resolution independently.

2. **Entity relations (entity layer):** The `related:` section declares relationships **between entities**. These are a separate concern from field paths and represent the entity-level graph used for code generation (e.g., repository interfaces, API routes, navigation).

A given entity may traverse model relations in its field paths that it does not declare in its own `related:` section, and vice versa. The two are independent.

### Related

Denoted by the `related:` key, this section lists all related entities. This means both dependencies and dependents (prerequisites).

Relations must be **explicitly bi-directional**: if entity A declares a relation to entity B, entity B must declare the corresponding inverse relation to entity A. For example, if `Invoice` declares `Customer: ForOne`, then `Customer` must declare `Invoice: HasMany`.

Each relationship is characterized by an `ownership` and a `cardinality` that differentiates between `1` and `n` related entities.

#### Supported Ownership Values

* `For`: The current entity is dependent on related entities.
  * *Example:* "Address ForOne User": Address is reliant on the existence of a specified User, because Address explicitly references User identifiers.
* `Has`: The current entity is required by related entities.
  * *Example:* "Address HasMany Document": Address supports the reliance of Documents on the existence of specified Addresses, because Documents explicitly reference Address identifiers.

#### Supported Cardinality Values

* `One`: The relationship supports one related entity instance.
  * *Example:* "Address ForOne User": Address is related to one user.
* `Many`: The relationship supports multiple related entity instances.
  * *Example:* "Owner HasMany Address": Owner is related to many addresses.

#### Relation Attributes

Entity relations support the same `attributes` mechanism as model relations. See [Relation Attributes](#relation-attributes) for the full list of first-class attributes.

*Example:* Optional entity relationship

```yaml
name: Task
fields:
  ID:
    type: Task.ID
    attributes:
      - immutable
  Title:
    type: Task.Title
identifiers:
  primary: ID
related:
  Project:
    type: ForOne
    attributes:
      - optional
  Author:
    type: ForOne
```

### Polymorphic Relationships

Entities support the same polymorphic relationship types as models, allowing them to represent complex business relationships across multiple entity types. Like models, the `aliased` property is optional - when not specified, the relationship name is used as the target entity name.

#### HasOnePoly and HasManyPoly (with `through`)

*Example:* `person.ent` with polymorphic relationships

```yaml
name: Person
fields:
  ID:
    type: Person.ID
    attributes:
      - immutable
  LastName:
    type: Person.LastName
  Nationality:
    type: Person.Nationality
identifiers:
  primary: ID
related:
  Comment:
    type: HasOnePoly
    through: Commentable
  Tag:
    type: HasManyPoly
    through: Taggable
```

#### ForOnePoly and ForManyPoly (with `for`)

*Example:* `comment.ent` with polymorphic relationship

```yaml
name: Comment
fields:
  ID:
    type: Comment.ID
    attributes:
      - immutable
  Content:
    type: Comment.Content
  CreatedAt:
    type: Comment.CreatedAt
identifiers:
  primary: ID
related:
  Commentable:
    type: ForOnePoly
    for:
      - Person
      - Company
```

*Example:* `tag.ent` with polymorphic relationship

```yaml
name: Tag
fields:
  ID:
    type: Tag.ID
    attributes:
      - immutable
  Name:
    type: Tag.Name
  Color:
    type: Tag.Color
identifiers:
  primary: ID
related:
  Taggable:
    type: ForManyPoly
    for:
      - Person
      - Company
```

### Entity Relationship Aliasing

Entities support the same aliasing patterns as models:

*Example:* `person-profile.ent` with aliased relationships

```yaml
name: PersonProfile
fields:
  ID:
    type: Person.ID
    attributes:
      - immutable
  Name:
    type: Person.Name
  WorkEmail:
    type: Person.WorkContact.Email  # Traversing through aliased relationship
  PersonalPhone:
    type: Person.PersonalContact.Phone
identifiers:
  primary: ID
related:
  PrimaryCompany:
    type: ForOne
    aliased: Company
  SecondaryCompany:
    type: ForOne
    aliased: Company
```

Entity fields can traverse through aliased model relationships using the relationship name (not the aliased target). The aliasing resolution happens transparently during compilation.

## Contributing

We welcome contributions to the Morphe specification and its format-specific implementations! Here's how you can help:

### Types of Contributions

1. **Specification Improvements**
   * Clarifying existing documentation
   * Suggesting new features or field types
   * Identifying gaps or inconsistencies
   * Improving examples and use cases

2. **Implementation Feedback**
   * Testing format-specific implementations
   * Reporting bugs or unexpected behavior
   * Suggesting optimizations for generated code
   * Proposing new format implementations

### How to Contribute

1. **Issues First**
   * Start by creating an issue to discuss your proposed changes
   * For bugs, include the specification version (e.g., KA:MO1, KA:MO1:YAML1)
   * For features, explain the use case and expected benefits

2. **Pull Requests**
   * Fork the relevant repository
   * Create a branch with a descriptive name
   * Make focused, atomic commits with clear messages
   * Reference the related issue in your PR description

### Style Guidelines

1. **Specification Changes**
   * Use clear, concise language
   * Provide examples for new features
   * Maintain consistent formatting with existing docs
   * Follow format best practices in examples

2. **Code Contributions**
   * Follow the format-specific style guide
   * Include tests for new features
   * Update documentation as needed
   * Ensure backwards compatibility

### Questions or Need Help?

Open a discussion in the relevant repository's issues section. We aim to respond within a few business days.

## License

This project is licensed under the MIT License.