# Active Record Associations — Polymorphic Associations

> Source: https://guides.rubyonrails.org/association_basics.html
> Collected: 2026-09-28
> Published: Unknown
> Scope: Relevant section of the published guide.

### Polymorphic Associations

A slightly more advanced twist on associations is the _polymorphic association_.
Polymorphic associations in Rails allow a model to belong to multiple other
models through a single association. This can be particularly useful when you
have a model that needs to be linked to different types of models.

For instance, imagine you have a `Picture` model that can belong to **either**
an `Employee` or a `Product`, because each of these can have a profile picture.
Here's how this could be declared:

```ruby
class Picture < ApplicationRecord
  belongs_to :imageable, polymorphic: true
end

class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end

class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end
```

![Polymorphic Association Diagram](images/association_basics/polymorphic.png)

In the context above, `imageable` is a name chosen for the association. It's a
symbolic name that represents the polymorphic association between the `Picture`
model and other models such as `Employee` and `Product`. The important thing is
to use the same name (`imageable`) consistently across all associated models to
establish the polymorphic association correctly.

When you declare `belongs_to :imageable, polymorphic: true` in the `Picture`
model, you're saying that a `Picture` can belong to any model (like `Employee`
or `Product`) through this association.

You can think of a polymorphic `belongs_to` declaration as setting up an
interface that any other model can use. This allows you to retrieve a collection
of pictures from an instance of the `Employee` model using `@employee.pictures`.
Similarly, you can retrieve a collection of pictures from an instance of the
`Product` model using `@product.pictures`.

Additionally, if you have an instance of the `Picture` model, you can get its
parent via `@picture.imageable`, which could be an `Employee` or a `Product`.

To set up a polymorphic association manually you would need to declare both a
foreign key column (`imageable_id`) and a type column (`imageable_type`) in the
model:

```ruby
class CreatePictures < ActiveRecord::Migration[8.1]
  def change
    create_table :pictures do |t|
      t.string  :name
      t.bigint  :imageable_id
      t.string  :imageable_type
      t.timestamps
    end

    add_index :pictures, [:imageable_type, :imageable_id]
  end
end
```

In our example, `imageable_id` could be the ID of either an `Employee` or a
`Product`, and `imageable_type` is the name of the associated model's class, so
either `Employee` or `Product`.

While creating the polymorphic association manually is acceptable, it is instead
recommended to use `t.references` or its alias `t.belongs_to` and specify
`polymorphic: true` so that Rails knows that the association is polymorphic, and
it automatically adds both the foreign key and type columns to the table.

```ruby
class CreatePictures < ActiveRecord::Migration[8.1]
  def change
    create_table :pictures do |t|
      t.string :name
      t.belongs_to :imageable, polymorphic: true
      t.timestamps
    end
  end
end
```

WARNING: Since polymorphic associations rely on storing class names in the
database, that data must remain synchronized with the class name used by the
Ruby code. When renaming a class, make sure to update the data in the
polymorphic type column.

For example, if you change the class name from `Product` to `Item` then you'd
need to run a migration script to update the `imageable_type` column in the
`pictures` table (or whichever table is affected) with the new class name.
Additionally, you'll need to update any other references to the class name
throughout your application code to reflect the change.
