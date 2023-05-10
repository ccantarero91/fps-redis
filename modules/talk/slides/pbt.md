# My Personal Experience Using Scala and Functional Programming




## Introduction

- I'm Cristian
- I'm based on Madrid, Spain
- Last two years on Depop as tech contractor
- Goal: share my personal experience using Scala and functional programming (FP) on Depop.


Note:
- Hello everyone, my name is Cristian Cantarero
- Based on Madrid, Spain
- I have been working on Depop for these last two years, in Sellers, Ads and now Fulfilment team.
- Tech contractor from 47 Degrees (now Xebia Functional) 
- I'm going to talk to you about some of my experience using Scala and functional programming here.
- My idea is to share my path of approaches, challenges, and may some lessons I've learned along the way.




## What is Scala & FP?

- Scala combines elements of object-oriented programming and functional programming.
- Functional programming relies on the use of pure, immutable, and side-effect-free functions.
- I wanted to write code that was more concise, and feel more confident with my codebase.


Note:
- As you may know Scala is a multi-paradigm programming language 
- It combines elements of object-oriented programming and functional programming.
- Functional programming is a paradigm that relies on 
- the use of pure, immutable, and side-effect-free functions. 
- Let's say that everything is easier with things you can predict a sustitute as we do in maths
- Why?
- I became interested in functional programming because I wanted to write code that was more concise, expressive and secure.




## Future & Asynchronous Calls

-  `Future` represents a value that may become available in the future.
-  `Future` allows asynchronous calls without blocking the main thread.

```scala
import scala.concurrent.Future

case class DepopProduct(id: Long)
def getPrice(product: DepopProduct): Future[Double] = ???

val priceOfTheProduct: Future[Double] = getPrice(DepopProduct(1L))
```

Note:
- One of the first things I started using here was `Future`, which is an abstraction that represents a value that may become available in the future.
- `Future` allows me to make asynchronous calls without blocking the main thread, and handle possible errors or results with callbacks or combinators.
- This is really useful to make or code run faster on the server. 
- `getPrice` is an async function (may calls to a db get the price, but we don't block the thread!!
- `priceOfTheProduct` is a value that may become in the future



### Typelevel Data Types EitherT & OptionT on Future
- Dealing with `Future[Either[A, B]]` or `Future[Option[A]]` can be tedious
- OptionT and EitherT can help on that example

Note:
- OptionT: `OptionT[F[_], A]` wrapper of `F[Option[A]]`
- EitherT: `EitherT[F[_], A, B]` wrapper of `F[Either[A, B]]`




Example
```scala
import scala.concurrent.Future
import cats.data.EitherT
import cats.data.OptionT
import cats.syntax.all._

case class DepopProduct(id: Long)

case class ShippingInfo(carrier: String, price: Double)

case class CheckoutProduct(product: DepopProduct, price: Double, shipping: ShippingInfo)

def getPrice(product: DepopProduct): Future[Either[String, Double]] = ???
def getShippingInfo(product: DepopProduct): Future[Option[ShippingInfo]] = ???

val product: DepopProduct = DepopProduct(1L)
val checkoutProduct: EitherT[Future, String, CheckoutProduct] = for {
  price: Double <- EitherT(getPrice(product))
  shippingInfo: ShippingInfo <- EitherT.fromOptionF(getShippingInfo(product), "Not Shipping Info available")
} yield CheckoutProduct(product, price, shippingInfo)

val res: Future[Either[String, CheckoutProduct]] = checkoutProduct.value

```
Note:
- These data types from typelevel libraries can be really useful when dealing 
- Also they help with error handling situations where something can fails or there is no data 
- In the example we can see how they allow us to work with the proper data in the for-compression 




## Opaque Types

- Opaque Types
  - have the same internal representation as another type,
  - but they are distinct at compile-time.
- Allowing strong typing
- So you don't get confused passing parameters


Note:

- Another thing I liked about Scala was the ability to use opaque types, which are types that have the same internal representation as another type, but are distinct at compile-time.
- Opaque types allow me to have strong typing with objects, which improves code safety, readability, and modularity.
- Here's an example of how I use opaque types to define a `SellerId` or `BuyerId` type that only accepts valid text strings.



Example
```scala
opaque type SellerId = Int

opaque type BuyerId = Int

def pay(sellerId: SellerId)

def charge(buyerId: BuyerId)

```

Note:
- Here's an example of how I use opaque types to define a `SellerId` or `BuyerId` to model those cases.
- Is really easy to mix Seller or buyer in our codebase and have an unexpected results.




## Using F[_], Async, and IO for Functional Abstractions

- After using `Future` and opaque types, I started to using and finding other abstractions such as `F[_]`, `Async`, `IO`, and other typelevel libraries.
- These abstractions allow me to work with side-effects in a more declarative, composable, and controlled way.
- However, I also encountered some challenges with the syntax (what that hole means) and concepts of functional programming.
- Here's an example of using IO to make some request...



## Using Scala 3 with Typelevel Libraries

- Finally, we had the change to migrate to Scala 3 a project and even create a one from the scratch
- Scala 3 has a simpler and more consistent syntax, new features such as algebraic enums, union and intersection types, and much mire things
- It was harder at the beginning because there was much more problems with the tooling
- An example of how I use Scala 3 with `cats-effect`, `http4s`, `fs2-kafka`and `skunk` to create a functional web application that consumes data from a kafka and database can be found on (https://github.com/depop/fulfilment-notification-ingest)

Note:
- Scala 3 is the latest version of the language that introduces many improvements and novelties.

## Conclusion

- I've seen the benefits of writing more concise, expressive, secure and confident codebase
- I've also encountered challenges with the syntax and concepts of functional programming. 
  - Red Book is still helping
  - Or even Practical Functional Programming with scala can be a nice read.
- But I encourage to add the libraries to your projects or find some time to try thing with them
- Overall, I've found functional programming to be a powerful tool for building robust and scalable applications.

Note:
- Thank you for listening, and I encourage you to try or learn more about Scala and functional programming!



### Questions?

![Questions](imgs/questions.webp)



### Fuentes

| Título                            | Autor   |
|---------------------------------------------------------------------------------------|---------------|
| [Practical FP in Scala](https://leanpub.com/pfp-scala)            | Gabriel Volpe |