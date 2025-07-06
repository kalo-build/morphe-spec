# TypeScript Transpilation Standard (KA:MO1:YAML1->TS1)

## Overview

This document specifies how Morphe (KA:MO1:YAML1) models are transpiled into TypeScript type definitions. The KA:MO1:YAML1->TS1 standard ensures consistent and predictable TypeScript output across projects.

## Supported Features

The `KA:MO1:YAML1->TS1` transpilation standard supports the following Morphe specification features:

✅ **Models** - Transpiled to TypeScript type definitions  
✅ **Entities** - Transpiled to TypeScript type definitions with model field references  
✅ **Enums** - Transpiled to TypeScript enums  
✅ **Structures** - Transpiled to TypeScript type definitions  
✅ **EnumFields** - Enum types used as field types  
✅ **ModelRelationPolymorphism** - Polymorphic relationships in models  
✅ **EntityRelationPolymorphism** - Polymorphic relationships in entities  
🚧 **ModelRelationAliasing** - Custom relationship naming (future)  
🚧 **EntityRelationAliasing** - Custom relationship naming (future)

## Models

### Basic Model with Fields

Input (.mod file):

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

Output (.d.ts):

```ts
export type User = {
  id: number
  email: string
  createdAt: Date
}

export type UserIDPrimary = {
  id: number
}

export type UserIDEmail = {
  email: string  
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

Output (.d.ts):

```ts
import { ContactInfo } from "./contact-info"

export type Person = {
  id: number
  name: string
  contactInfoID?: number
  contactInfo?: ContactInfo
}

export type PersonIDPrimary = {
  id: number
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
  Employee:
    type: HasMany
```

Output (.d.ts):

```ts
import { Employee } from "./employee"

export type Company = {
  id: number
  name: string
  employeeIDs?: number[]
  employees?: Employee[]
}

export type CompanyIDPrimary = {
  id: number
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

Output (.d.ts):

```ts
import { Person } from "./person"

export type ContactInfo = {
  id: number
  email: string
  personID?: number
  person?: Person
}

export type ContactInfoIDPrimary = {
  id: number
}
```

### ForMany Relationship

Input (.mod file):

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

Output (.d.ts):

```ts
import { Article } from "./article"

export type Tag = {
  id: number
  name: string
  articleIDs?: number[]
  articles?: Article[]
}

export type TagIDPrimary = {
  id: number
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

Output (.d.ts):

```ts
import { Comment } from "./comment"
import { Tag } from "./tag"

export type Person = {
  id: number
  firstName: string
  lastName: string
  commentID?: number
  comment?: Comment
  tagIDs?: string[]
  tags?: Tag[]
}

export type PersonIDPrimary = {
  id: number
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

Output (.d.ts):

```ts
import { Person } from "./person"
import { Company } from "./company"

export type Comment = {
  id: number
  content: string
  createdAt: string
  commentableID?: string
  commentableType?: string
  commentable?: Person | Company
}

export type CommentIDPrimary = {
  id: number
}
```

## Enumerations

Input (.enum file):

```yaml
name: UserRole
type: String
entries:
  Admin: ADMIN
  Editor: EDITOR
  Viewer: VIEWER
```

Output (.d.ts):

```ts
export enum UserRole {
  Admin = "ADMIN",
  Editor = "EDITOR",
  Viewer = "VIEWER"
}
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
  Role:
    type: UserRole
identifiers:
  primary: ID
```

Output (.d.ts):

```ts
import { Nationality } from "../enums/nationality"
import { UserRole } from "../enums/user-role"

export type Person = {
  id: number
  firstName: string
  lastName: string
  nationality: Nationality
  role: UserRole
}

export type PersonIDPrimary = {
  id: number
}
```

## Structures

Input (.str file):

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

Output (.d.ts):

```ts
export type Address = {
  street: string
  city: string
  postalCode: string
  country: string
}
```
[Previous content remains unchanged until after the Structures section]

## Entities

Entities are high-level business objects that can combine and transform fields from multiple models. They support the same relationship types as models but can reference specific model fields.

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

Output (.d.ts):

```ts
import { Nationality } from "../enums/nationality"
import { Company } from "./company"

export type Person = {
  id: number
  lastName: string
  nationality: Nationality
  email: string
  companyID?: number
  company?: Company
}

export type PersonIDPrimary = {
  id: number
}
```

### Entity with HasMany Relationship

Input (.ent file):

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

Output (.d.ts):

```ts
import { Person } from "./person"

export type Company = {
  id: number
  name: string
  taxID: string
  personIDs?: number[]
  persons?: Person[]
}

export type CompanyIDPrimary = {
  id: number
}
```

### Key Differences from Models

1. Field Types
   - Models use primitive types (String, AutoIncrement, etc.)
   - Entities reference model fields using dot notation (Model.Field)
   - Entities can access nested fields (Model.Related.Field)

2. Type Resolution
   - Entity field types are resolved from their referenced model fields
   - Relationships are preserved and transformed the same way as models
   - Enums are imported from their respective locations

3. Generated Output
   - Entities generate similar TypeScript types to models
   - Import paths are adjusted to maintain correct references
   - All relationships are optional by default in entities

## Type Mappings

Morphe Type | TypeScript Type
------------|---------------
UUID | string
AutoIncrement | number
String | string
Integer | number
Float | number
Boolean | boolean
Time | Date
Date | Date
Protected | string
Sealed | string

## Contributing

See the main KA:MO1 specification for contribution guidelines.

## License

This project is licensed under the MIT License.
