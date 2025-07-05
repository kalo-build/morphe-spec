# TypeScript Transpilation Standard (KA:MO1:YAML1->TS1)

## Overview

This document specifies how Morphe (KA:MO1:YAML1) models are transpiled into TypeScript type definitions. The KA:MO1:YAML1->TS1 standard ensures consistent and predictable TypeScript output across projects.

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

## Polymorphism

**Note**: Polymorphism support for TypeScript transpilation is planned for future implementation. The following documentation represents the intended design and will be updated as implementation progresses.

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

Output (.d.ts):

```ts
export enum ContentItemType {
  ContentItem = "content_item",
  Article = "article",
  Video = "video"
}

export interface BaseContentItem {
  id: number
  title: string
  type: ContentItemType
}

export interface ContentItem extends BaseContentItem {
  type: ContentItemType
}

export type ContentItemUnion = 
  | (BaseContentItem & { type: ContentItemType.ContentItem })
  | Article 
  | Video

export type ContentItemIDPrimary = {
  id: number
}
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

Output (.d.ts):

```ts
import { BaseContentItem, ContentItemType } from "./content-item"

export interface Article extends BaseContentItem {
  type: ContentItemType.Article
  content: string
  wordCount: number
}

export type ArticleIDPrimary = {
  id: number
}

// Type guards for polymorphic instances
export function isArticle(item: ContentItemUnion): item is Article {
  return item.type === ContentItemType.Article
}

// Factory function
export function createArticle(
  id: number, 
  title: string, 
  content: string, 
  wordCount: number
): Article {
  return {
    id,
    title,
    type: ContentItemType.Article,
    content,
    wordCount
  }
}
```

### Polymorphic Factory Functions

```ts
import { ContentItemUnion, ContentItemType } from "./content-item"
import { Article, createArticle } from "./article"
import { Video, createVideo } from "./video"

export function createContentItem(
  type: ContentItemType,
  data: Record<string, any>
): ContentItemUnion {
  switch (type) {
    case ContentItemType.Article:
      return createArticle(
        data.id,
        data.title,
        data.content,
        data.wordCount
      )
    case ContentItemType.Video:
      return createVideo(
        data.id,
        data.title,
        data.duration,
        data.resolution
      )
    default:
      throw new Error(`Unknown content item type: ${type}`)
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

Output (.d.ts):

```ts
import { Person } from "./person"

export type Company = {
  id: number
  name: string
  ownerID?: number
  owner?: Person    // Aliased to Person
  employeeIDs?: number[]
  employees?: Person[]  // Aliased to Person
}

export type CompanyIDPrimary = {
  id: number
}

// Type-safe utility functions for aliased relationships
export interface CompanyWithOwner extends Company {
  owner: Person
  ownerID: number
}

export interface CompanyWithEmployees extends Company {
  employees: Person[]
  employeeIDs: number[]
}

export function hasOwner(company: Company): company is CompanyWithOwner {
  return company.owner !== undefined && company.ownerID !== undefined
}

export function hasEmployees(company: Company): company is CompanyWithEmployees {
  return company.employees !== undefined && company.employees.length > 0
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

Output (.d.ts):

```ts
import { Company } from "../models/company"

export type CompanyProfile = {
  id: number
  name: string
  ownerName?: string
  employeeCount: number
}

export type CompanyProfileIDPrimary = {
  id: number
}

// Factory function with aliased relationship field resolution
export function createCompanyProfile(company: Company): CompanyProfile {
  return {
    id: company.id,
    name: company.name,
    ownerName: company.owner?.name,
    employeeCount: company.employees?.length ?? 0
  }
}

// Type guards for profile validation
export function isValidCompanyProfile(profile: Partial<CompanyProfile>): profile is CompanyProfile {
  return (
    typeof profile.id === 'number' &&
    typeof profile.name === 'string' &&
    typeof profile.employeeCount === 'number'
  )
}
```

### Discriminated Union Types

For complex polymorphic relationships with aliased models:

```ts
export type CompanyRelationshipType = 
  | { type: 'owner', person: Person }
  | { type: 'employee', person: Person }

export function createCompanyRelationship(
  type: 'owner' | 'employee',
  person: Person
): CompanyRelationshipType {
  return { type, person }
}

export function isOwnerRelationship(
  rel: CompanyRelationshipType
): rel is { type: 'owner', person: Person } {
  return rel.type === 'owner'
}
```

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
