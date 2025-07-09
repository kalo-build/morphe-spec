# Go Format Specification (KA:MO1:GO1)

## Overview

This document specifies how Morphe specification concepts are represented in Go. The `KA:MO1:GO1` format provides Go structs and types that correspond to Morphe data models, entities, enums, and structures.

*Note: Examples reference the base YAML format for clarity, as YAML serves as the canonical representation in the Morphe ecosystem.*

## Supported Features

The `KA:MO1:GO1` format supports the following Morphe specification features:

✅ **Models** - Represented as Go structs  
✅ **Entities** - Represented as Go structs with model field references  
✅ **Enums** - Represented as Go constants and types  
✅ **Structures** - Represented as Go structs  
✅ **EnumFields** - Enum types used as field types  
✅ **ModelRelationPolymorphism** - Polymorphic relationships in models  
✅ **EntityRelationPolymorphism** - Polymorphic relationships in entities  
🚧 **ModelRelationAliasing** - Custom relationship naming (future)  
🚧 **EntityRelationAliasing** - Custom relationship naming (future)

## Models

### Basic Model with Fields

A basic model with fields and identifiers:

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
    attributes:
      - mandatory
  FirstName:
    type: String
  LastName:
    type: String
identifiers:
  primary: ID
  name:
    - FirstName
    - LastName
```

Go representation:

```go
package models

type Person struct {
    ID        uint   
    FirstName string 
    LastName  string 
}

type PersonIDPrimary struct {
    ID uint
}

type PersonIDName struct {
    FirstName string
    LastName  string
}
```

### HasOne Relationship

A model with a HasOne relationship:

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

Go representation:

```go
package models

type Person struct {
    ID            uint
    Name          string
    ContactInfoID *uint
    ContactInfo   *ContactInfo
}

type PersonIDPrimary struct {
    ID uint
}
```

### HasMany Relationship

A model with a HasMany relationship:

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
  Person:
    type: HasMany
```

Go representation:

```go
package models

type Company struct {
    ID        uint
    Name      string
    PersonIDs []uint
    Persons   []Person
}

type CompanyIDPrimary struct {
    ID uint
}
```

### ForOne Relationship

A model with a ForOne relationship:

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

Go representation:

```go
package models

type ContactInfo struct {
    ID       uint
    Email    string
    PersonID *uint
    Person   *Person
}

type ContactInfoIDPrimary struct {
    ID uint
}
```

### Polymorphic Relationships

#### HasOnePoly and HasManyPoly

A model with polymorphic HasOne and HasMany relationships:

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

Go representation:

```go
package models

type Person struct {
    ID             uint
    FirstName      string
    LastName       string
    CommentID      *uint
    Comment        *Comment
    TagIDs         []string
    Tags           []Tag
}

type PersonIDPrimary struct {
    ID uint
}
```

#### ForOnePoly and ForManyPoly

A model with a polymorphic ForOne relationship:

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

Go representation:

```go
package models

type Comment struct {
    ID              uint
    Content         string
    CreatedAt       string
    CommentableID   *uint
    CommentableType *string
    Commentable     any // Person or Company
}

type CommentIDPrimary struct {
    ID uint
}
```

## Enumerations

An enumeration definition:

```yaml
name: Nationality
type: String
entries:
  US: 'American'
  DE: 'German'
  FR: 'French'
```

Go representation:

```go
package enums

type Nationality string

const (
    NationalityUS Nationality = "American"
    NationalityDE Nationality = "German"
    NationalityFR Nationality = "French"
)
```

### EnumFields - Using Enums as Field Types

A model using enums as field types:

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
identifiers:
  primary: ID
```

Go representation:

```go
package models

import (
    "github.com/org/pkg/enums"
)

type Person struct {
    ID          uint
    FirstName   string
    LastName    string
    Nationality enums.Nationality
}

type PersonIDPrimary struct {
    ID uint
}
```

## Structures

A structure definition:

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
```

Go representation:

```go
package structures

type Address struct {
    Street  string
    HouseNr string
    ZipCode string
    City    string
}
```

## Entities

Entities are high-level business objects that can combine and transform fields from multiple models.

### Basic Entity

A basic entity with referenced model fields:

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

Go representation:

```go
package entities

import (
    "github.com/org/pkg/enums"
)

type Person struct {
    ID          uint
    LastName    string
    Nationality enums.Nationality
    Email       string
    CompanyID   *uint
    Company     *Company
}

type PersonIDPrimary struct {
    ID uint
}
```

## Type Mappings

Morphe Type | Go Type
------------|--------
UUID | string
AutoIncrement | uint
String | string
Integer | int
Float | float64
Boolean | bool
Time | time.Time
Date | time.Time
Protected | string
Sealed | string

## Contributing

See the main [Morphe KA:MO1 specification](https://github.com/kalo-build/morphe-spec) for contribution guidelines.

## License

This project is licensed under the MIT License.
