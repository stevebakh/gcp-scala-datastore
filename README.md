`gcp-scala-datastore` is a simple wrapper library for [Google Cloud Datastore](http://googlecloudplatform.github.io/google-cloud-java/). 


The library provides asynchronous API and supports the following types of fields in model classes:
* `Byte`
* `Int`
* `Long`
* `Float`
* `Double`
* `String`
* `Boolean`
* `java.util.Date`
* `com.google.cloud.datastore.DateTime`
* `com.google.cloud.datastore.LatLng`
* `com.google.cloud.datastore.Blob`

### Usage of the `gcp-scala-datastore`
To be stored in Cloud Datastore a model class must inherit `io.applicative.datastore.BaseEntity` and must have `id` field of type `Long` or `String`.
```
import io.applicative.datastore.{BaseEntity, DatastoreService}
import io.applicative.datastore.query._

import scala.concurrent.Future

// Sample model class
case class Item(id: Long, name: String, price: Double, size: Int, brand: Option[String]) extends BaseEntity

val item = Item(1, "foo", 20.0, 2, None)

// Save
DatastoreService.add[Item](item)

// Save with autogenerated id
for {
  key <- DatastoreService.newKey[Item]()
  user <- DatastoreService.add[Item](Item(key.id.get, "foo", 20.0, 2, None))
} yield user

// Update
DatastoreService.update[Item](item.copy(brand = Some("bar")))

// Delete
DatastoreService.delete[Item](List(1L))

// Get one by id
DatastoreService.get[Item](1)

// or
select[Item] asSingle

// Select
val items: Future[List[Item]] = select[Item] where "size" |>| 23 and "price" |<=| 200.2 ascOrderBy "size" descOrderBy "price" asList
val items2: Future[List[Item]] = select[Item]
                                  .where("size" |>| 23)
                                  .and("price" |<| 200.2)
                                  .ascOrderBy("size")
                                  .descOrderBy("price")
                                  .asList
val singleItem: Future[Option[Item]] = select[Item] where "name" |==| "foo" asSingle
```

### Indexes
By default, Cloud Datastore automatically predefines an index for each property of each entity kind(see https://cloud.google.com/datastore/docs/concepts/indexes for more details). <br>
If you want to exclude any of your properties from the indexes, just add the annotation `@excludeFromIndexes`.

```
import io.applicative.datastore.BaseEntity
import io.applicative.datastore.util.reflection.excludeFromIndexes

case class Item(id: Long, name: String, price: Double, size: Int, brand: Option[String], @excludeFromIndexes description: String) extends BaseEntity
```

### Installation using sbt

In order to install this package you will need set an extra resolver in `build.sbt`:

```
resolvers ++= Seq(
  "applctv-bintray" at "https://dl.bintray.com/applctv/gcp-scala-datastore/"
)
```

And then you can add it as a normal sbt dependency:

```
libraryDependencies ++= Seq(
  "io.applicative" %% "datastore-scala-wrapper" % "1.0-rc8"
)
```


