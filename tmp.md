# Proposal: Generate Typescript Type automatically according to Odata Metadata
Author: Song Gao

Prototype: Complete
Implementation: waiting for proposal accepted

## Summary
I suggest to create one big file or some small files to represent model of Odata in type of typescript form. All files shoudl be generated automatically according to $MetaData of Odata.

## Motivation
Although front-end and back-end could use different data model, it's pretty common to see that we are just using the data from back-end. And due to the current project structure, lots of small packages will flood developper's mind and make type hard to share. For developpers who are using TS as their developping language, a general type definition would be easy to find and use and they do not need to define their own type everywhere. And maintaining types by handwriting are tedious and hard, for it's not easy to know the changes.

## Detailed design

### Spec and MetaData

Specs:
OData 4.0 XML CSDL: http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part3-csdl/odata-v4.0-errata03-os-part3-csdl-complete.html
Odata 4.01 JSON CSDL: https://docs.oasis-open.org/odata/odata-csdl-json/v4.01/odata-csdl-json-v4.01.html#sec_DerivedEntityType
Most of the content is similar, and in this proposal we will ignore most of difference.
However, we will use XML collection format(.i.e`Collection(SomeType)`) to representation a collection of type.

Odata service Example:
Campaign Odata Metadata: https://ui.ads-int.microsoft.com/ODataApi/Campaign/V2/$metadata

### Special Term
In this section, we will explain some term that could not find in Odata spec.

Indexed Access Type:
`a.b` is property access expression
`a["b"]` is element access expression
And this works for types in Typescript. Indexed Access Type is element access for type. We choose this name for it is what is being used in Typescript.

### Type Mapping between EDM and Typescript
Entity Data Model (EDM) is a model designed for general purpose, for any concentrate language, it might not have an accurate mapping of types.

All Typescript type is defined by `interface` or `type`, user might use Indexed Access Type to access internal type, but it's not intential.

##### Primitive Type Mapping
Edm.Binary                        -->  number
Edm.Boolean                       -->  boolean
Edm.Byte                          -->  number
Edm.Date                          -->  string
Edm.DateTimeOffset                -->  string
Edm.Decimal                       -->  number
Edm.Double                        -->  number
Edm.Duration                      -->  string
Edm.Guid                          -->  string
Edm.Int16                         -->  number
Edm.Int32                         -->  number
Edm.Int64                         -->  number
Edm.SByte                         -->  number
Edm.Single                        -->  number
Edm.Stream                        -->  string
Edm.String                        -->  string
Edm.TimeOfDay                     -->  string
Edm.Geography                     -->  string
Edm.GeographyPoint                -->  string
Edm.GeographyLineString           -->  string
Edm.GeographyPolygon              -->  string
Edm.GeographyMultiPoint           -->  string
Edm.GeographyMultiLineString      -->  string
Edm.GeographyMultiPolygon         -->  string
Edm.GeographyCollection           -->  string
Edm.Geometry                      -->  string
Edm.GeometryPoint                 -->  string
Edm.GeometryLineString            -->  string
Edm.GeometryPolygon               -->  string
Edm.GeometryMultiPoint            -->  string
Edm.GeometryMultiLineString       -->  string
Edm.GeometryMultiPolygon          -->  string
Edm.GeometryCollection            -->  string

#### Nominal Type Mapping
Typescirpt is strutual rather than nomial. Although it's possiable to use some trick to imitate, it's not that helpful for front-end data.
Current design is just to use `interface` or `type` keyword to define the name.

#### Structured Type Mapping
use `interface` to define the body.

#### Structural Properties Mapping
Due to the name mapping section `Name Mapping`, the mapping is easy:
`property: {AnotherType}`

#### Collection Mapping
Collection type will be transformed to `array` of type. Example:
`Collection(SomeType)` will be transformed to `SomeType[]`

#### Enumeration Type Mapping
use `const enum` to define the Enumeration Type
Enumeration type is kind of tricky. Serlization might transform the enum value to number or string, left an option for this.

### Name Mapping
EDM uses dot(.) to split namespace name and simple name in qualified name, and namesapce name might still have dots. But dot is not a valid char in typescript identifier.
Replace All dots with dollar($) of the qualified name of type in EDM format as the name of Typescript format. Example:
`Model.Type` will be transformed to `Model$Type`
`org.Model.Type` will be transformed to `org$Model$Type`

### Fixed property Mapping
This Section choose to use fixed property in JSON spec(.i.e starts with dollar rather than attribute in XML) to describe.
`$BaseType` will use `extends` to express heritage relationship.
`$OpenType` will add an indexed property signature to current type, `[openMember: string]: any;`

### Generator design
In theory, any language could be used to implement the generator, however, due to the target language is TS, TS will be the first choice to use.

This is a brief of steps:

Parse XML --> convert to JSON model --> transform Odata Model to Typescript Type node --> emit type nodes to string

## Drawbacks
In the offline discussion, `Yu Zhong` was worried about the bundle size. In current design, all things should be compiled away, however, the build tool has suprised me many times, I am not sure about this.
For current design, enumeration type is transformed to `const enum`, which will be compiled to literal value. I am not sure whether js file could benefit from new enum type.

## Alternatives
1. use a "top interface", like this:
``` ts
interface OdataNamespaces {
    'name.space1': {
        property1: string,
        property2: Namespaces['name.space1']['property1'];
        property3: {
            foo1: string;
            foo2: Namespaces['name.space1']['property1'];
        },
    },
}
```
In this way, the dot in namespace does not need be transformed to dollar, and the dot between namespace and simple name could be transform as TypeElementAccess.
And for users, there is only one entry ---- OdataNamespaces, which will be pretty easy to use.
However, this looks kind of wierd, people usually only access subtype by property access.

Example:
``` ts
import {OdataNamespaces} from 'ODataTypes';
const foo: OdataNamespaces['name.space1']['property1'];
```

2. use simple name as the name of type. However, due to current project design this might not works in conor case:
  - duplicate simple names. Although we could generate one file for one type, finally we must export all types.

Example:
``` ts
import {SomeProperty1, SomeProperty2} from 'ODataTypes';
const foo: SomeProperty1;
```

3. use a more complicated file structure like
Note: this is compiled file rather than source file.
Note: This is what material-ui is doing.
--- packageName
   --- name.space.foo1
      --- package.json
      --- entityType1.d.ts
      --- entityType2.d.ts
      --- index.d.ts
   --- name.space.foo2
   ...
   --- package.json // this file only export the whole package, nothing more

``` package.json in namespace
{
  "sideEffects": false,
//  "module": "./index.js",
//  "main": "../some/path/index.js",
  "types": "./index.d.ts"
}
```

Seems good, but will tree-shaking works so that this whole package will be gone(in development time) because we do not have any root? Or should we need to add a package name for each namespace?
what about IDE support?
What will the build tool do?

Example:
``` ts
import {SomeProperty1} from 'ODataTypes/name.space.1';
const foo: SomeProperty1;
```

For enumeration type,
Another choice is just `enum` rather than `const enum`.
`const enum` might be a problem if we do not use tsc as typescript transpiler.
An advantage is that js user could reuse the code defined here, but we must pay for bundle size for some type never used. (Does tree-shaking work for our project? I do not know.)

## open questions
`Action` and `Function` is not handled
`EntitySet` is not handled
`Annotation` is not handled
`Key` is not handled correctly
