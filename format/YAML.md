# YAML Format Specification (`KA:MO1:YAML1`)

## Overview

This document specifies the YAML format for defining Morphe (`KA:MO1`) data models. The `KA:MO1:YAML1` standard ensures consistent and predictable YAML structure across projects.

## Supported Features

The `KA:MO1:YAML1` format supports all core Morphe specification features:

✅ **Models** - Primary persisted data structures  
✅ **Entities** - Business data aggregation structures  
✅ **Enums** - Predefined constant value sets  
✅ **Structures** - Standalone, non-persisted field groupings  
✅ **EnumFields** - Using enums as field types  
✅ **ModelRelationPolymorphism** - Polymorphic relationships in models  
✅ **EntityRelationPolymorphism** - Polymorphic relationships in entities  
🚧 **ModelRelationAliasing** - Custom relationship naming (future)  
🚧 **EntityRelationAliasing** - Custom relationship naming (future)

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

### Polymorphic Relationships

#### HasOnePoly and HasManyPoly

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

#### ForOnePoly and ForManyPoly

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

### EnumFields - Using Enums as Field Types

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
  FavoriteConstant:
    type: MathematicalConstant  # References the MathematicalConstant enum
identifiers:
  primary: ID
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

### Polymorphic Relationships

#### HasOnePoly and HasManyPoly

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

#### ForOnePoly and ForManyPoly

```yaml
name: Comment
fields:
  ID:
    type: Comment.ID
    attributes:
      - immutable
      - mandatory
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

```yaml
name: Tag
fields:
  ID:
    type: Tag.ID
    attributes:
      - immutable
      - mandatory
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

## Contributing

See the main [Morphe `KA:MO1` specification](https://github.com/kalo-build/morphe-spec) for contribution guidelines.

## License

This project is licensed under the MIT License.
