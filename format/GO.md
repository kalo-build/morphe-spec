# Go Transpilation Standard (KA:MO1:YAML1->GO1)

## Overview

This document specifies how Morphe (KA:MO1:YAML1) models are transpiled into Go structs and types. The KA:MO1:YAML1->GO1 standard ensures consistent and predictable Go output across projects.

## Supported Features

The `KA:MO1:YAML1->GO1` transpilation standard supports the following Morphe specification features:

✅ **Models** - Transpiled to Go structs  
✅ **Entities** - Transpiled to Go structs with model field references  
✅ **Enums** - Transpiled to Go constants and types  
✅ **Structures** - Transpiled to Go structs  
✅ **EnumFields** - Enum types used as field types  
✅ **ModelRelationPolymorphism** - Polymorphic relationships in models  
✅ **EntityRelationPolymorphism** - Polymorphic relationships in entities  
🚧 **ModelRelationAliasing** - Custom relationship naming (future)  
🚧 **EntityRelationAliasing** - Custom relationship naming (future)

## Models

### Basic Model with Fields

Input (.mod file):

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

Output (.go):

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

Input (.mod file):

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

Output (.go):

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

Input (.mod file):

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

Output (.go):

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

Input (.mod file):

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

Output (.go):

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

Input (.mod file):

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

Output (.go):

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

Input (.mod file):

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

Output (.go):

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

Input (.enum file):

```yaml
name: Nationality
type: String
entries:
  US: 'American'
  DE: 'German'
  FR: 'French'
```

Output (.go):

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

Input (.mod file):

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

Output (.go):

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

Input (.str file):

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

Output (.go):

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

Input (.ent file):

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

Output (.go):

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
