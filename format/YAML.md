# YAML Format Specification (`KA:MO1:YAML1`)

## Overview

This document specifies the YAML format for defining Morphe (`KA:MO1`) data models. The `KA:MO1:YAML1` standard ensures consistent and predictable YAML structure across projects.

## File Extensions

- `.mod` - Model definitions
- `.ent` - Entity definitions
- `.str` - Structure definitions
- `.enum` - Enumeration definitions

## Models

### Basic Model with Fields

```yaml
name: User
fields:
  ID:
    type: AutoIncrement
    attributes:
      - mandatory
  Email:
    type: String
  CreatedAt:
    type: Time
identifiers:
  primary: ID
  email: Email
```

### HasOne Relationship

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
  ContactInfo:
    type: HasOne
```

### HasMany Relationship

```yaml
name: Company
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
identifiers:
  primary: ID
related:
  Employee:
    type: HasMany
```

### ForOne Relationship

```yaml
name: ContactInfo
fields:
  ID:
    type: AutoIncrement
  Email:
    type: String
identifiers:
  primary: ID
related:
  Person:
    type: ForOne
```

### ForMany Relationship

```yaml
name: Tag
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
identifiers:
  primary: ID
related:
  Article:
    type: ForMany
```

### Aliased Relationships

```yaml
name: Company
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
identifiers:
  primary: ID
related:
  Owner:
    type: HasOne
    aliased: Person
  Employees:
    type: HasMany
    aliased: Person
```

## Polymorphism

### Polymorphic Base Model

```yaml
name: ContentItem
fields:
  ID:
    type: AutoIncrement
  Title:
    type: String
  Type:
    type: String
identifiers:
  primary: ID
polymorphic:
  discriminator: Type
  identity: content_item
```

### Polymorphic Subclass Models

```yaml
name: Article
extends: ContentItem
fields:
  Content:
    type: String
  WordCount:
    type: Integer
polymorphic:
  identity: article
```

```yaml
name: Video
extends: ContentItem
fields:
  Duration:
    type: Integer
  Resolution:
    type: String
polymorphic:
  identity: video
```

### Polymorphic Entities

```yaml
name: ContentSummary
extends: ContentItem
fields:
  ID:
    type: ContentItem.ID
  Title:
    type: ContentItem.Title
  Type:
    type: ContentItem.Type
  Summary:
    type: ContentItem.Content  # For articles
    fallback: ContentItem.Duration  # For videos
polymorphic:
  identity: content_summary
```

## Enumerations

*Example (string):*

```yaml
name: UserRole
type: String
entries:
  Admin: ADMIN
  Editor: EDITOR
  Viewer: VIEWER
```
*Example (float):*

```yaml
name: MathematicalConstant
type: Float
entries:
  Pi: 3.14159265359
  Euler: 2.71828182846
  GoldenRatio: 1.61803398875
  SilverRatio: 2.41421356237
```

## Structures

```yaml
name: Address
fields:
  Street:
    type: String
  City:
    type: String
  PostalCode:
    type: String
  Country:
    type: String
```

## Entities

### Basic Entity

```yaml
name: Person
fields:
  ID:
    type: Person.ID
    attributes:
      - immutable
      - mandatory
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

### Entity with HasMany Relationship

```yaml
name: Company
fields:
  ID:
    type: Company.ID
    attributes:
      - immutable
      - mandatory
  Name:
    type: Company.Name
  TaxID:
    type: Company.TaxID
identifiers:
  primary: ID
related:
  Person:
    type: HasMany
```

## Field Types

### Atomic Types

- `UUID`: RFC-4122 compatible UUID string
- `AutoIncrement`: Numeric, auto-incrementable record ID
- `String`: Variable-length string
- `Integer`: Whole number value
- `Float`: Floating-point decimal value
- `Boolean`: true/false value
- `Time`: Timestamp with UTC offset
- `Date`: Date with UTC offset and zero time values
- `Protected`: Encryptable value for sensitive information
- `Sealed`: Hashable value that cannot be decrypted

### Enumeration Types

Enums are defined with:
- `name`: The enum type name
- `type`: The primitive type (`String`, `Integer`, `Float`)
- `entries`: Key-value pairs of enum values

## Field Attributes

Fields can have zero or more attributes specified as a list:

```yaml
fields:
  ID:
    type: AutoIncrement
    attributes:
      - mandatory
      - immutable
      - unique
```

Concrete attributes are not determined by this specification, but are offered as a utility for additional processing options in plugin implementations.

Examples of potentially useful attributes might include:
- `mandatory`: Field must have a value
- `immutable`: Field cannot be modified after creation
- `unique`: Field value must be unique across records
- `indexed`: Field should be indexed for performance
- `protected`: Field should be encrypted at rest
- `sealed`: Field should be hashed and cannot be decrypted

## Identifiers

Models and entities must have at least one `primary` identifier specified:

```yaml
identifiers:
  primary: ID
  email: Email
  name:
    - FirstName
    - LastName
```

Identifier types:
- `primary`: The primary identifier (required)
- Named identifiers: Additional unique identifiers
- Composite identifiers: Lists of fields that together form a unique identifier

## Relationships

Relationships are defined in the `related` section:

```yaml
related:
  ContactInfo:
    type: HasOne
  Employees:
    type: HasMany
  Tags:
    type: ForMany
```

Relationship types:
- `HasOne`: One-to-one relationship where the current model owns the related model
- `HasMany`: One-to-many relationship where the current model owns multiple related models
- `ForOne`: One-to-one relationship where the current model belongs to one related model
- `ForMany`: Many-to-many relationship between models

### Aliased Relationships

When multiple relationships reference the same target model, use the `aliased` attribute to specify the actual target while maintaining semantic relationship names:

```yaml
related:
  Owner:
    type: HasOne
    aliased: Person
  Employees:
    type: HasMany
    aliased: Person
```

This allows the same model (`Person`) to serve different roles within the relationship context.

## Contributing

See the main [Morphe `KA:MO1` specification](https://github.com/kalo-build/morphe-spec) for contribution guidelines.

## License

This project is licensed under the MIT License.
