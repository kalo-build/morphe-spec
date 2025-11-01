# PostgreSQL Format Specification (`KA:MO1:PSQL1`)

This document specifies how Morphe specification concepts are represented in Go. The `KA:MO1:PSQL1` format provides Go structs and types that correspond to Morphe data models, entities, enums, and structures.

*Note: Examples reference the base YAML format for clarity, as YAML serves as the canonical representation in the Morphe ecosystem.*

- Entity relationships are not explicitly represented in SQL views, as views are read-only and cannot enforce referential integrity constraints that already exist at the table level. Instead, the relationships should be used in query contexts and for reflection / documentation purposes.

## Overview

This document specifies how Morphe specification concepts are represented in PostgreSQL. The `KA:MO1:PSQL1` format provides PostgreSQL database schemas that correspond to Morphe data models, entities, enums, and structures.

*Note: Examples reference the base YAML format for clarity, as YAML serves as the canonical representation in the Morphe ecosystem.*

## Supported Features

The `KA:MO1:PSQL1` format supports the following Morphe specification features:

✅ **Models** - Represented as PostgreSQL tables  
✅ **Entities** - Represented as PostgreSQL views  
✅ **Enums** - Represented as PostgreSQL lookup tables  
❌ **Structures** - Not persisted (conceptually non-persisted)  
✅ **EnumFields** - Foreign key references to enum lookup tables  
✅ **ModelRelationPolymorphism** - Junction tables with polymorphic keys  
✅ **EntityRelationPolymorphism** - Views with polymorphic joins  
✅ **ModelRelationAliasing** - Custom relationship naming with backward-compatible SQL  
✅ **EntityRelationAliasing** - Views with aliased relationship traversal

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

PostgreSQL representation:

```sql
CREATE TABLE persons (
    id SERIAL PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    CONSTRAINT uk_persons_name_first_name_last_name UNIQUE (first_name, last_name)
);
```

### Model with Enum Field

A model using an enum as a field type:

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
  Nationality:
    type: Nationality
identifiers:
  primary: ID
```

PostgreSQL representation:

```sql
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    "name" TEXT,
    nationality_id INTEGER NOT NULL,
    CONSTRAINT fk_people_nationality_id FOREIGN KEY (nationality_id)
      REFERENCES nationalities(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_people_nationality_id ON people(nationality_id);
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

PostgreSQL representation:

```sql
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    "name" TEXT,
    contact_info_id INTEGER,
    CONSTRAINT fk_people_contact_info_id FOREIGN KEY (contact_info_id)
      REFERENCES contact_infos(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_people_contact_info_id ON people(contact_info_id);
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

PostgreSQL representation:

```sql
CREATE TABLE companies (
    id SERIAL PRIMARY KEY,
    "name" TEXT
);

CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    "name" TEXT,
    company_id INTEGER,
    CONSTRAINT fk_people_company_id FOREIGN KEY (company_id)
      REFERENCES companies(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_people_company_id ON people(company_id);
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

PostgreSQL representation:

```sql
CREATE TABLE contact_infos (
    id SERIAL PRIMARY KEY,
    email TEXT,
    person_id INTEGER,
    CONSTRAINT fk_contact_infos_person_id FOREIGN KEY (person_id)
      REFERENCES people(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_contact_infos_person_id ON contact_infos(person_id);
```

### ForMany Relationship

A model with a ForMany relationship:

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

PostgreSQL representation:

```sql
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    "name" TEXT
);

-- The ForMany relationship requires a junction table
CREATE TABLE article_tags (
    article_id INTEGER NOT NULL,
    tag_id INTEGER NOT NULL,
    PRIMARY KEY (article_id, tag_id),
    CONSTRAINT fk_article_tag_article_id FOREIGN KEY (article_id)
      REFERENCES articles(id)
      ON DELETE CASCADE,
    CONSTRAINT fk_article_tag_tag_id FOREIGN KEY (tag_id)
      REFERENCES tags(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_article_tags_article_id ON article_tags(article_id);
CREATE INDEX idx_article_tags_tag_id ON article_tags(tag_id);
```

### Special Case: ForMany <-> HasOne

We enforce the `HasOne` cardinality in this special case by adding a unique constraint to the junction table keys.

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

PostgreSQL representation:

```sql
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    "name" TEXT
);

-- The ForMany relationship requires a junction table with a Unique constraint to enforce `HasOne` cardinality
CREATE TABLE article_tags (
    article_id INTEGER NOT NULL,
    tag_id INTEGER NOT NULL,
    -- Other constraints ...
    CONSTRAINT uk_article_tags_article_id UNIQUE (article_id)
);

-- Other indices ...
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

PostgreSQL representation:

```sql
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    first_name TEXT,
    last_name TEXT
);

-- Polymorphic junction table for HasOnePoly relationship
CREATE TABLE commentables (
    id SERIAL PRIMARY KEY,
    person_id INTEGER,
    commentable_type TEXT NOT NULL,
    commentable_id INTEGER NOT NULL,
    CONSTRAINT fk_commentables_person_id FOREIGN KEY (person_id)
      REFERENCES people(id)
      ON DELETE CASCADE,
    CONSTRAINT uk_commentables_person_id UNIQUE (person_id)
);

-- Polymorphic junction table for HasManyPoly relationship  
CREATE TABLE taggables (
    id SERIAL PRIMARY KEY,
    person_id INTEGER,
    taggable_type TEXT NOT NULL,
    taggable_id INTEGER NOT NULL,
    CONSTRAINT fk_taggables_person_id FOREIGN KEY (person_id)
      REFERENCES people(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_commentables_person_id ON commentables(person_id);
CREATE INDEX idx_commentables_type_id ON commentables(commentable_type, commentable_id);
CREATE INDEX idx_taggables_person_id ON taggables(person_id);
CREATE INDEX idx_taggables_type_id ON taggables(taggable_type, taggable_id);
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

PostgreSQL representation:

```sql
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    content TEXT,
    created_at TEXT,
    commentable_type TEXT NOT NULL,
    commentable_id INTEGER NOT NULL
);

CREATE INDEX idx_comments_commentable ON comments(commentable_type, commentable_id);
```

### Relationship Aliasing

Aliasing allows multiple relationships to the same target model. In PostgreSQL, column and table names use the relationship name (not the aliased target) to maintain backward compatibility and clear semantics.

#### ForOne with Aliasing

A model with multiple ForOne relationships to the same target:

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
    aliased: Contact
  PersonalContact:
    type: ForOne
    aliased: Contact
```

PostgreSQL representation:

```sql
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    "name" TEXT,
    work_contact_id INTEGER,      -- Uses relationship name, not "contact_id"
    personal_contact_id INTEGER,   -- Distinct column for each relationship
    CONSTRAINT fk_people_work_contact_id FOREIGN KEY (work_contact_id)
      REFERENCES contacts(id)
      ON DELETE CASCADE,
    CONSTRAINT fk_people_personal_contact_id FOREIGN KEY (personal_contact_id)
      REFERENCES contacts(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_people_work_contact_id ON people(work_contact_id);
CREATE INDEX idx_people_personal_contact_id ON people(personal_contact_id);
```

#### ForMany with Aliasing

A model with multiple ForMany relationships to the same target:

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
  WorkProject:
    type: ForMany
    aliased: Project
  PersonalProject:
    type: ForMany
    aliased: Project
```

PostgreSQL representation:

```sql
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    "name" TEXT
);

-- Junction table uses relationship name, not aliased target
CREATE TABLE people_work_projects (
    id SERIAL PRIMARY KEY,
    person_id INTEGER NOT NULL,
    work_project_id INTEGER NOT NULL,
    CONSTRAINT fk_people_work_projects_person_id FOREIGN KEY (person_id)
      REFERENCES people(id)
      ON DELETE CASCADE,
    CONSTRAINT fk_people_work_projects_work_project_id FOREIGN KEY (work_project_id)
      REFERENCES projects(id)
      ON DELETE CASCADE,
    CONSTRAINT uk_people_work_projects UNIQUE (person_id, work_project_id)
);

CREATE TABLE people_personal_projects (
    id SERIAL PRIMARY KEY,
    person_id INTEGER NOT NULL,
    personal_project_id INTEGER NOT NULL,
    CONSTRAINT fk_people_personal_projects_person_id FOREIGN KEY (person_id)
      REFERENCES people(id)
      ON DELETE CASCADE,
    CONSTRAINT fk_people_personal_projects_personal_project_id FOREIGN KEY (personal_project_id)
      REFERENCES projects(id)
      ON DELETE CASCADE,
    CONSTRAINT uk_people_personal_projects UNIQUE (person_id, personal_project_id)
);
```

#### Polymorphic Aliasing

Polymorphic relationships can also use aliasing:

```yaml
name: Comment
fields:
  ID:
    type: AutoIncrement
  Text:
    type: String
identifiers:
  primary: ID
related:
  CommentableResource:
    type: ForOnePoly
    for:
      - Document
      - Video
      - Image
    aliased: Resource  # Semantic alias that doesn't map to any model
```

PostgreSQL representation:

```sql
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    text TEXT,
    commentable_resource_type TEXT NOT NULL,  -- Uses relationship name
    commentable_resource_id TEXT NOT NULL      -- Not "resource_type/id"
);

CREATE INDEX idx_comments_commentable_resource ON comments(commentable_resource_type, commentable_resource_id);
```

## Enumerations

In the `KA:MO1:PSQL1` format, enumerations are represented as lookup tables for maximum flexibility.

An enumeration definition:

```yaml
name: Nationality
type: String
entries:
  US: 'American'
  DE: 'German'
  FR: 'French'
```

PostgreSQL representation:

```sql
CREATE TABLE nationalities (
    id SERIAL PRIMARY KEY,
    key TEXT NOT NULL,
    value TEXT NOT NULL,
    CONSTRAINT uk_nationalities_key UNIQUE (key)
);

INSERT INTO nationalities (key, value) 
VALUES 
    ('US', 'American'),
    ('DE', 'German'),
    ('FR', 'French');
```

### EnumFields - Using Enums as Field Types

When enums are used as field types in models, they are transpiled as foreign key references to the enum lookup tables.

Input (.mod file):

```yaml
name: Person
fields:
  ID:
    type: AutoIncrement
  Name:
    type: String
  Nationality:
    type: Nationality
identifiers:
  primary: ID
```

Output (PostgreSQL):

```sql
CREATE TABLE persons (
    id SERIAL PRIMARY KEY,
    "name" TEXT,
    nationality_id INTEGER NOT NULL,
    CONSTRAINT fk_persons_nationality_id FOREIGN KEY (nationality_id)
      REFERENCES nationalities(id)
      ON DELETE CASCADE
);

CREATE INDEX idx_persons_nationality_id ON persons(nationality_id);
```

Benefits of lookup tables:
- Better extensibility (new values can be added without schema changes)
- Supports additional metadata (descriptions, display names, etc.)
- Enables internationalization
- Maintains referential integrity
- Simplified querying with JOIN operations

### Alternative Approaches

While PostgreSQL offers native enum types, the `KA:MO1:YAML1->PSQL1` standard deliberately avoids them:

```sql
-- NOT SUPPORTED in `KA:MO1:YAML1->PSQL1`
CREATE TYPE user_role AS ENUM ('ADMIN', 'EDITOR', 'VIEWER');
```

Reasons for not supporting native PostgreSQL enums:
- Limited extensibility (adding values requires ALTER TYPE statements)
- No support for additional metadata
- More complex to modify in production environments
- Less flexible for evolving applications
- Difficult to version control changes

Project forks may implement native enum support if their use case prioritizes:
- Performance optimization
- Simplified schema with fewer tables
- Constraints that truly never change

## Constraint Naming

The standard uses abbreviated table and column names in constraint identifiers to stay within PostgreSQL's 63-character limit.

This is only applied when the structured names exceed the 63-character limit, ie. names such as `fk_persons_nationality_id` are still supported.

### Naming Pattern
- `fk_[abbreviated_table]_[abbreviated_column]` for foreign keys
- `uk_[abbreviated_table]_[abbreviated_column]` for unique constraints
- `idx_[abbreviated_table]_[abbreviated_column]` for indices

### Abbreviation Rules
1. Use first letter of each word in table name
2. Use first letter of each word in column name
3. For collisions, append a hash of the full name

Examples:
- `persons` -> `p`
- `contact_infos` -> `c_i`
- `article_tags` -> `a_t`
- `first_name` -> `f_n`
- `last_name` -> `l_n`

### Example Constraint Names
```sql
CONSTRAINT fk_p_c_i_id

CONSTRAINT uk_p_n_f_n_l_n

CREATE INDEX idx_p_n_id
```

## Type Mappings

Morphe Type | PostgreSQL Type | Notes
------------|----------------|------
UUID | UUID | Requires `uuid-ossp` extension
AutoIncrement | SERIAL or BIGSERIAL | BIGSERIAL for larger datasets
String | TEXT | Unbounded length
Integer | INTEGER | 4 bytes, range -2147483648 to +2147483647
Float | DOUBLE PRECISION | 8 bytes, 15 decimal digits precision
Boolean | BOOLEAN | true/false/null
Time | TIMESTAMP WITH TIME ZONE | ISO 8601 format with timezone
Date | DATE | Calendar date (no time of day)
Protected | TEXT | Client-side encryption recommended
Sealed | TEXT | Store as hash, use crypt() function

## Structures

According to the Morphe specification, structures are explicitly defined as "standalone, reusable, non-persisted groupings of fields" that serve as DTOs (data-transfer-objects) in application code. The specification does not provide a mechanism for models or entities to reference structures, meaning structures have no direct representation in a persistence layer like PostgreSQL.

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

PostgreSQL representation:

```sql
-- No direct database representation
-- Structures exist only in application code
```

### Standard Optional Persistence

While structures remain conceptually non-persisted, `KA:MO1:YAML1->PSQL1` provides a standardized optional pattern for structure persistence using a single table with JSONB storage. This balances the conceptual separation with practical needs for caching and persistence.

```sql
-- Optional standard table for structure persistence
CREATE TABLE morphe_structures (
    id SERIAL PRIMARY KEY,
    "type" TEXT NOT NULL,
    "data" JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indices for efficient querying
CREATE INDEX idx_morphe_structures_type ON morphe_structures("type");
CREATE INDEX idx_morphe_structures_data ON morphe_structures USING GIN ("data");
```

Example of using the standard structures table:

```sql
-- Storing an Address structure
INSERT INTO morphe_structures ("type", "data")
VALUES (
    'Address',
    '{"street": "123 Main St", "houseNr": "4B", "zipCode": "10001", "city": "New York"}'
);

-- Querying Address structures
SELECT * FROM morphe_structures 
WHERE "type" = 'Address' 
AND "data"->>'city' = 'New York';
```

This approach provides:
- A standardized mechanism for structure persistence
- Flexibility for caching and retrieval
- Minimal impact on database schema complexity
- Clear separation from domain models
- Support for JSONB querying capabilities

### Implementation Notes

Since the core Morphe specification defines structures as non-persisted constructs, `KA:MO1:YAML1->PSQL1` maintains this as the default approach. The standardized structures table is provided as an optional pattern when persistence is required.

Application developers working with Morphe may choose to:

1. Use structures purely as in-memory DTOs in application code (default)
2. Utilize the standardized morphe_structures table when persistence is needed
3. Manually persist structure data as JSONB in model tables for specific use cases
4. Map structures to tables manually outside the Morphe transpilation process

Example of manually persisting structure data in a model table:

```sql
-- This is NOT part of the standard transpilation, but an application-level pattern
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    "type" TEXT,
    -- Manual persistence of Address structure as JSONB
    "address" JSONB
);
```

### Extension Considerations

This approach maintains alignment with the core specification by keeping structures conceptually non-persisted, while providing a pragmatic solution for real-world persistence needs. Future extensions to the Morphe specification might consider:

1. Adding a mechanism for models to contain fields of structure types
2. Defining formal persistence strategies for embedded structures
3. Establishing composition relationships between models and structures

Until such extensions are formalized, the standardized morphe_structures table offers a consistent approach that balances specification purity with practical implementation needs.

## Entities

Entities in Morphe are high-level business objects that represent aggregates or views of data. In PostgreSQL, they are represented as views:

An entity definition:

```yaml
name: PersonProfile
fields:
  ID:
    type: Person.ID
  FullName:
    type: Person.Name
  Email:
    type: Person.ContactInfo.Email
  CompanyName:
    type: Person.Company.Name
identifiers:
  primary: ID
```

PostgreSQL representation:

```sql
CREATE VIEW person_profiles AS
SELECT 
    p.id,
    p.name AS full_name,
    ci.email,
    c.name AS company_name
FROM 
    persons p
LEFT JOIN 
    contact_infos ci ON p.id = ci.person_id
LEFT JOIN 
    companies c ON p.company_id = c.id;
```

## Type Mappings

Morphe Type | PostgreSQL Type | Notes
------------|----------------|------
UUID | UUID | Requires `uuid-ossp` extension
AutoIncrement | SERIAL or BIGSERIAL | BIGSERIAL for larger datasets
String | TEXT | Unbounded length
Integer | INTEGER | 4 bytes, range -2147483648 to +2147483647
Float | DOUBLE PRECISION | 8 bytes, 15 decimal digits precision
Boolean | BOOLEAN | true/false/null
Time | TIMESTAMP WITH TIME ZONE | ISO 8601 format with timezone
Date | DATE | Calendar date (no time of day)
Protected | TEXT | Client-side encryption recommended
Sealed | TEXT | Store as hash, use crypt() function

## Indices and Performance

For optimal PostgreSQL performance, the transpiler should automatically create indices for:

1. All primary keys (implicit)
2. All foreign keys
3. All fields used in unique constraints (implicit)
4. Fields commonly used in filtering (based on annotations)

Example:

```sql
-- Automatically created for performance
CREATE INDEX idx_persons_company_id ON persons(company_id);
CREATE INDEX idx_contact_infos_email ON contact_infos(email);
```

## Migration Support

The `KA:MO1:YAML1->PSQL1` standard includes support for generating schema migrations using tools like:

1. Native SQL migration files
2. Schema versioning tables
3. Integration with common migration tools (Flyway, Liquibase, etc.)

Example migration file:

```sql
-- V1__create_initial_schema.sql
CREATE TABLE persons (
    id SERIAL PRIMARY KEY,
    "name" TEXT
);

-- V2__add_email_to_persons.sql
ALTER TABLE persons ADD COLUMN email TEXT;
```

## Contributing

See the main [Morphe `KA:MO1` specification](https://github.com/kalo-build/morphe-spec) for contribution guidelines.

## License

This project is licensed under the MIT License. 
