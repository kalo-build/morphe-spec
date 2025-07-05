# Go Transpilation Standard (KA:MO1:YAML1->GO1)

## Overview

This document specifies how Morphe (KA:MO1:YAML1) models are transpiled into Go structs and types. The KA:MO1:YAML1->GO1 standard ensures consistent and predictable Go output across projects.

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

## Polymorphism

**Note**: Polymorphism support for Go transpilation is planned for future implementation. The following documentation represents the intended design and will be updated as implementation progresses.

### Polymorphic Base Model

Input (.mod file):

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

Output (.go):

```go
package models

import "errors"

type ContentItemType string

const (
    ContentItemTypeContent ContentItemType = "content_item"
    ContentItemTypeArticle ContentItemType = "article"  
    ContentItemTypeVideo   ContentItemType = "video"
)

type ContentItem interface {
    GetID() uint
    GetTitle() string
    GetType() ContentItemType
}

type BaseContentItem struct {
    ID    uint
    Title string
    Type  ContentItemType
}

func (b BaseContentItem) GetID() uint { return b.ID }
func (b BaseContentItem) GetTitle() string { return b.Title }
func (b BaseContentItem) GetType() ContentItemType { return b.Type }
```

### Polymorphic Subclass Models

Input (.mod file):

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

Output (.go):

```go
package models

type Article struct {
    BaseContentItem
    Content   string
    WordCount int
}

func NewArticle(id uint, title, content string, wordCount int) *Article {
    return &Article{
        BaseContentItem: BaseContentItem{
            ID:    id,
            Title: title,
            Type:  ContentItemTypeArticle,
        },
        Content:   content,
        WordCount: wordCount,
    }
}
```

### Polymorphic Factory Functions

```go
package models

func NewContentItemFromType(itemType ContentItemType, data map[string]interface{}) (ContentItem, error) {
    switch itemType {
    case ContentItemTypeArticle:
        return &Article{
            BaseContentItem: BaseContentItem{
                ID:    data["id"].(uint),
                Title: data["title"].(string),
                Type:  itemType,
            },
            Content:   data["content"].(string),
            WordCount: data["word_count"].(int),
        }, nil
    case ContentItemTypeVideo:
        return &Video{
            BaseContentItem: BaseContentItem{
                ID:    data["id"].(uint),
                Title: data["title"].(string),
                Type:  itemType,
            },
            Duration:   data["duration"].(int),
            Resolution: data["resolution"].(string),
        }, nil
    default:
        return nil, errors.New("unknown content item type")
    }
}
```

## Aliased Relationships

### Aliased Model Relationships

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
  Owner:
    type: HasOne
    aliased: Person
  Employees:
    type: HasMany
    aliased: Person
```

Output (.go):

```go
package models

type Company struct {
    ID          uint
    Name        string
    OwnerID     *uint
    Owner       *Person  // Aliased to Person
    EmployeeIDs []uint
    Employees   []Person // Aliased to Person
}

type CompanyIDPrimary struct {
    ID uint
}

// Type-safe access methods for aliased relationships
func (c *Company) GetOwner() *Person {
    return c.Owner
}

func (c *Company) SetOwner(person *Person) {
    c.Owner = person
    if person != nil {
        c.OwnerID = &person.ID
    } else {
        c.OwnerID = nil
    }
}

func (c *Company) GetEmployees() []Person {
    return c.Employees
}

func (c *Company) AddEmployee(person Person) {
    c.Employees = append(c.Employees, person)
    c.EmployeeIDs = append(c.EmployeeIDs, person.ID)
}
```

### Aliased Entity Relationships

Input (.ent file):

```yaml
name: CompanyProfile
fields:
  ID:
    type: Company.ID
  Name:
    type: Company.Name
  OwnerName:
    type: Company.Owner.Name
  EmployeeCount:
    type: Company.Employees.Count
identifiers:
  primary: ID
```

Output (.go):

```go
package entities

type CompanyProfile struct {
    ID            uint
    Name          string
    OwnerName     string
    EmployeeCount int
}

type CompanyProfileIDPrimary struct {
    ID uint
}

// Constructor with aliased relationship field resolution
func NewCompanyProfile(company models.Company) CompanyProfile {
    profile := CompanyProfile{
        ID:            company.ID,
        Name:          company.Name,
        EmployeeCount: len(company.Employees),
    }
    
    if company.Owner != nil {
        profile.OwnerName = company.Owner.Name
    }
    
    return profile
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
