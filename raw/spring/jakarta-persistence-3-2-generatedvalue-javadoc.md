# Jakarta Persistence 3.2 — GeneratedValue Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/generatedvalue
> Collected: 2026-09-29
> Published: Unknown

Module jakarta.persistence
Package jakarta.persistence
Annotation Interface GeneratedValue
@Target({METHOD,FIELD})
@Retention(RUNTIME)
public @interface GeneratedValue
Specifies a generation strategy for generated primary keys.
The GeneratedValue annotation may be applied to a
primary key property or field of an entity or mapped superclass
in conjunction with the Id annotation. The persistence
provider is only required to support the GeneratedValue
for simple primary keys. Use of the GeneratedValue
annotation for derived primary keys is not supported.
Example 1:
Copy
@Id
@GeneratedValue(strategy = SEQUENCE, generator = "CUST_SEQ")
@Column(name = "CUST_ID")
public Long getId() { return id; }
Example 2:
Copy
@Id
@GeneratedValue(strategy = TABLE, generator = "CUST_GEN")
@Column(name = "CUST_ID")
Long id;
Since:
1.0
See Also:
GenerationType
Id
TableGenerator
SequenceGenerator
Optional Element Summary
Optional Elements
Modifier and Type
Optional Element
Description
String
generator
(Optional) The name of the primary key generator to
use, as specified by the SequenceGenerator
or TableGenerator annotation which declares
the generator.
GenerationType
strategy
(Optional) The primary key generation strategy that
the persistence provider must use to generate the
annotated entity primary key.
Element Details
strategy
GenerationType strategy
(Optional) The primary key generation strategy that
the persistence provider must use to generate the
annotated entity primary key.
Default:
AUTO
generator
String generator
(Optional) The name of the primary key generator to
use, as specified by the SequenceGenerator
or TableGenerator annotation which declares
the generator.
The name defaults to the entity name of the
entity in which the annotation occurs.
If there is no generator with the defaulted name,
then the persistence provider supplies a default id
generator, of a type compatible with the value of
the strategy() member.
Default:
""
Comments to: [email protected].
Copyright © 2019, 2024 Eclipse Foundation. All rights reserved.
Use is subject to license terms.
